# Lab 01: Task-by-Task Deep Explanations

This document provides instructor-level technical context explaining **what** we are doing in each step of Lab 01, **why** each parameter is configured, and **what happens behind the scenes** inside Microsoft Azure.

---

## 📌 Task 1: Creating an Azure Web App & App Service Plan

### 1. What is an App Service Plan (ASP)?
When students create an App Service, they are not just creating a website; they are provisioning an **App Service Plan (ASP)**.
* Think of the **App Service Plan** as the dedicated VM infrastructure (Compute, RAM, Storage, OS, and Region).
* The **Web App** is the software process running inside that container/VM.
* Multiple Web Apps can live inside a single App Service Plan and share its computing resources, just like multiple websites on an IIS or Apache web server.

### 2. Why did we choose Standard (S1) or Premium V3 instead of Free (F1)?
| Feature | Free / Shared | Basic (B1) | Standard (S1) | Premium (P1v3) |
| :--- | :---: | :---: | :---: | :---: |
| Custom Domains & SSL | ❌ (Free) | ✅ | ✅ | ✅ |
| **Deployment Slots** | ❌ | ❌ | ✅ (Up to 5) | ✅ (Up to 20) |
| **Autoscaling** | ❌ | ❌ (Manual only) | ✅ (Up to 10 instances) | ✅ (Up to 30 instances) |
| Dedicated Compute | ❌ (Shared) | ✅ | ✅ | ✅ |
| SLA | None | 99.95% | 99.95% | 99.95% |

> **Key Takeaway for Students:** In AZ-104 questions, if a scenario demands **Deployment Slots** or **Autoscaling based on metrics**, any answer proposing *Basic* or *Free* tiers is immediately incorrect.

---

## 📌 Task 2: Creating a Staging Deployment Slot

### 1. What actually is a Deployment Slot?
A deployment slot is **not** a folder inside your web server; it is a **live, fully functional Azure Web App** with its own distinct public URL (`<app-name>-<slot-name>.azurewebsites.net`).
* It shares the same compute instances, RAM, and CPUs of the parent App Service Plan.
* Because it shares the same plan, **you do not pay extra compute charges for adding slots** (up to the tier limit).

### 2. Why use Deployment Slots?
1. **Zero-Downtime Deployments:** Instead of stopping the production site, copying files, and restarting it (which causes downtime), you deploy to staging first.
2. **Pre-flight Testing:** You can validate real database connections, SSL certificates, and environment settings in a production-identical environment.
3. **Warm-up:** Avoid the "cold start" delay for users. The code is compiled and loaded into memory *before* traffic hits it.

---

## 📌 Task 3: Configuring Deployment & The Kudu 503 Fix

### 1. What is Kudu (SCM)?
Every Azure Web App has a hidden companion management site called **Kudu** or the **Service Control Manager (SCM)**:
* URL format: `https://<your-app>.scm.azurewebsites.net`
* Kudu handles git deployments, webhooks, deployment logs, background jobs, and console debugging.

### 2. Why did the `503 Server Unavailable` error occur?
When you configured `External Git` and clicked **Save**, Azure Resource Manager (ARM) called the Kudu deployment engine via the API:
`POST /api/deployments` $\rightarrow$ `UpdateSiteSourceControl`

This failed with a **503** for three technical reasons:
1. **Cold State / Idle Worker:** In Linux App Services, the slot container goes dormant when idle if **Always On** is `Off`. When ARM sent the git clone webhook request, Kudu wasn't ready to respond within the gateway timeout.
2. **SCM Basic Authentication Policy:** In recent updates, Microsoft disabled Basic Auth by default across all newly provisioned Azure App Services for security compliance. This blocks external unauthenticated deployment calls unless explicitly enabled.
3. **Control Plane Handshake Delay:** A freshly provisioned slot takes 60–90 seconds to establish its internal routing endpoints.

### 3. Why did the PowerShell CLI command fix it?
```powershell
az webapp deploy --resource-group az104-rg9 --name <app> --slot staging --src-path app.zip --type zip
```
* The `az webapp deploy` command uses **OneDeploy**—Azure's modern unified deployment API.
* It uploads the zip package directly via authenticated Azure CLI credentials, automatically pings and warms up Kudu before deployment (`Warming up Kudu instance successfully`), and extracts the payload straight into `/home/site/wwwroot` without waiting on an external Git clone handshake.

---

## 📌 Task 4: Slot Swapping (Under the Hood)

### What happens when you click "Swap"?
The swap operation does **not** physically move files from one server to another. It performs an intelligent **Virtual IP / Routing Pointer swap**:

```mermaid
flowchart TD
    subgraph Before Swap
        DNS_Prod["Production URL<br>(myapp.azurewebsites.net)"] --> Node_Prod["App Code v1<br>(Old Production)"]
        DNS_Stage["Staging URL<br>(myapp-staging.azurewebsites.net)"] --> Node_Stage["App Code v2<br>(New Staging)"]
    end

    subgraph Step 1: Warm-up
        WarmUp["Azure pings root path of Staging<br>to warm up workers & caches"]
    end

    subgraph After Swap
        DNS_Prod2["Production URL<br>(myapp.azurewebsites.net)"] --> Node_Stage2["App Code v2<br>(Live Production)"]
        DNS_Stage2["Staging URL<br>(myapp-staging.azurewebsites.net)"] --> Node_Prod2["App Code v1<br>(Previous Production)"]
    end

    Before Swap --> Step 1
    Step 1 --> After Swap
```

1. **Warm-up:** Azure sends HTTP requests to the staging slot to spin up workers and fill caches.
2. **If warm-up succeeds:** Azure updates the internal load balancer rules. The production hostname now routes to the staging container instances, and vice versa.
3. **If warm-up fails:** The swap is automatically aborted. The production website continues serving users without any impact.
4. **Instant Fallback:** If a bug is detected in production after swapping, clicking **Swap** again rolls back to the previous version in seconds.

---

## 📌 Task 5: Autoscaling Rationale & Rule Design

### 1. Horizontal Scaling vs Vertical Scaling
* **Scale Up (Vertical):** Changing the CPU/RAM hardware tier (e.g., upgrading from Basic B1 to Premium P1v3). Requires a brief server reboot.
* **Scale Out (Horizontal):** Adding more identical worker instances (e.g., increasing instance count from 1 to 3). Zero downtime.

### 2. Why do we need BOTH Scale-out and Scale-in rules?
* If you only configure a **Scale-Out** rule (e.g., CPU > 70%), Azure will add instances when traffic spikes. But once the traffic subsides, your app will remain at the higher instance count forever, resulting in massive unwanted hosting bills!
* A **Scale-In** rule (e.g., CPU < 30%) ensures that when traffic normalizes, instances are released back down to your minimum boundary.

### 3. Why are Cooldown Periods Critical?
* A cooldown period (e.g., 5 minutes) tells Azure: *"After adding a new VM instance, wait 5 minutes before evaluating the metrics again."*
* Without a cooldown, Azure might see high CPU on the old instance before the new instance has fully booted and started taking traffic, causing it to prematurely add another unnecessary instance (a problem called **flapping** or **thrashing**).

---

## 📌 Task 6: Performance Testing & The Portal UI Behavior

### Why do students have to re-select the test in the portal?
In Azure Portal:
1. Performance tests and Azure Load Testing run as **asynchronous backend jobs**.
2. When the test begins, the test runner container is deployed in the background.
3. The parent blade list displays the aggregate historical test jobs. Clicking the specific completed test run opens the **Telemetry Blade**, which pulls live Azure Monitor and Application Insights metrics for that specific job run identifier.
