# Lab 01: Task-by-Task Deep Explanations

This document provides instructor-level technical context explaining **what** we are doing in each step of Lab 01, **why** each parameter is configured, and **what happens behind the scenes** inside Microsoft Azure.

---

## 📌 Task 1: Creating an Azure Web App & App Service Plan

### 1. What is an App Service Plan (ASP)?
When students create an App Service, they are not simply creating a website; they are provisioning an **App Service Plan (ASP)**.
* Think of the **App Service Plan** as the dedicated VM infrastructure (Compute, RAM, Storage, OS, and Region).
* The **Web App** is the software process running inside that compute container.
* Multiple Web Apps can live inside a single App Service Plan and share its computing resources.

### 2. Why does the lab specify the Premium V3 (P1V3) Tier?
| Feature | Free (F1) / Shared | Basic (B1) | Standard (S1) | Premium V3 (P1V3) |
| :--- | :---: | :---: | :---: | :---: |
| **Compute Architecture** | Shared CPU | Dedicated (v1) | Dedicated (v2) | Hyper-V Isolated (v3 - Fastest SSDs) |
| **Deployment Slots** | ❌ None | ❌ None | Up to 5 slots | **Up to 20 slots** |
| **Autoscaling** | ❌ None | ❌ Manual only | Up to 10 instances | **Up to 30 instances + Automatic burst** |
| **Diagnostics & Integrated Load Testing** | Basic | Basic | Supported | **Full Support** |
| SLA | None | 99.95% | 99.95% | 99.95% |

> **Key Takeaway for Students:** In AZ-104 questions, if a scenario demands **Deployment Slots** or **Autoscaling**, any answer proposing *Basic* or *Free* tiers is immediately incorrect. The official lab uses **Premium V3 P1V3** because it is Azure's modern flagship PaaS tier supporting both rapid slot swaps and Automatic Burst scaling.

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
Every Azure Web App has a companion management site called **Kudu** or the **Service Control Manager (SCM)**:
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
az webapp deploy --resource-group abdullah-rg9 --name abdullahapp222 --slot staging --src-path app.zip --type zip
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

## 📌 Task 5: Autoscaling Rationale & Integrated Load Testing

### 1. Automatic Scaling vs Rule-Based Scaling
* **Rule-Based Scaling (Legacy/Custom):** Requires manually defining metric conditions (e.g., "if CPU > 70% add 1 instance, if CPU < 30% remove 1 instance").
* **Automatic Scaling (Modern PaaS feature in Premium V3):** Azure intelligently monitors incoming HTTP traffic and server health. You simply set:
  * **Always ready instances (Minimum):** The baseline capacity you want running 24/7 (e.g., `1`).
  * **Maximum burst:** The maximum scale ceiling Azure is allowed to burst out to during traffic spikes (e.g., `2`).
  * This prevents administrative overhead and protects the organization against runaway scaling costs.

### 2. Testing Autoscaling via App Service Diagnostics (Azure Load Testing)
* In the official lab, testing is initiated from **Diagnose and solve problems** $\rightarrow$ **Load Test your App**.
* This triggers an **Azure Load Testing** cloud engine that deploys headless test runners to send concurrent HTTP requests to your web app's Default domain URL.

### 3. Why students must click into the specific test run:
* Azure Load Testing manages tests in a two-tier hierarchy:
  1. **Test Definition:** The reusable test plan/template.
  2. **Test Run:** An immutable execution instance (e.g., Run #1 on 10/31).
* The parent test blade only displays configuration metadata. Clicking the **individual test run** opens the real-time telemetry dashboard where the student can observe **Virtual Users**, **Response Time**, and **Throughput**.
