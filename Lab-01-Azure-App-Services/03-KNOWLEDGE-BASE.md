# Lab 01 Knowledge Base: Azure App Services & Web Apps

This Knowledge Base serves as the comprehensive theory and reference manual for students and instructors. If a student is unsure about the underlying concepts, architecture, or exam implications of Azure App Service, they can refer directly to this document.

---

## 📖 1. Core Architectural Concepts

### What is Azure App Service?
**Azure App Service** is a fully managed Platform-as-a-Service (PaaS) engine for building, deploying, and scaling web apps, mobile app backends, and RESTful APIs.
* **No OS Management:** Microsoft manages OS patching, hardware failures, security baselines, and networking layers.
* **Supported Frameworks:** .NET, .NET Core, Java, Ruby, Node.js, PHP, Python, or custom Docker containers.
* **Built-in DevOps:** Native integration with GitHub Actions, Azure DevOps, Bitbucket, and local Git.

---

### App Service Plan (ASP) Architecture

An **App Service Plan** represents the compute units allocated to run your applications. 

```
+-------------------------------------------------------------+
|                     App Service Plan: <your-name>-asp       |
|  Region: East US | SKU: Premium V3 P1V3 (1 vCPU, 4 GB RAM)  |
|                                                             |
|   +-----------------------+     +------------------------+  |
|   | Web App: Production   |     | Slot: Staging          |  |
|   | (Traffic: 100%)       |     | (Traffic: 0%)          |  |
|   +-----------------------+     +------------------------+  |
|               \                             /               |
|                \                           /                |
|           Shared Underlying VM / Worker Resources           |
+-------------------------------------------------------------+
```

#### Key Rules of App Service Plans:
1. **Multi-Tenancy per Plan:** Multiple web apps can share one plan. However, they compete for the same CPU and RAM.
2. **Region & Subscription Bound:** An App Service Plan belongs to one subscription and one physical Azure region. Any Web App attached to that plan must be created in that exact same region.
3. **Billing Unit:** You pay for the **App Service Plan**, not the individual Web Apps or Deployment Slots running inside it. If you have 5 web apps in one P1v3 plan, you pay for one P1v3 instance.

---

## 🔄 2. Deployment Slots Deep Dive

Deployment slots are live web apps with their own hostnames. App content and configurations elements can be swapped between two deployment slots.

### The Golden Rule: What Swaps vs. What Stays (AZ-104 Exam Essential!)

This is one of the most frequently tested concepts on the AZ-104 exam. Students must memorize which settings swap and which remain stuck (*sticky*) to the slot:

| Configuration Setting | Swapped? | Sticky (Stays with Slot)? |
| :--- | :---: | :---: |
| **Application Code & Files** | ✅ **YES** | ❌ No |
| **Framework / Runtime Version** | ✅ **YES** | ❌ No |
| **General App Settings** *(Unless marked sticky)* | ✅ **YES** | ❌ No |
| **Custom Domain Names** | ❌ No | ✅ **YES (Sticky)** |
| **TLS/SSL Certificate Bindings** | ❌ No | ✅ **YES (Sticky)** |
| **Scale Settings (Autoscale rules, Instance Count)** | ❌ No | ✅ **YES (Sticky)** |
| **Publishing Endpoints & WebHooks** | ❌ No | ✅ **YES (Sticky)** |
| **Virtual Network Integration** | ❌ No | ✅ **YES (Sticky)** |
| **IP Restrictions & Access Rules** | ❌ No | ✅ **YES (Sticky)** |

> 💡 **Instructor Tip:** Notice why this makes sense! Your production domain (`www.company.com` or `<your-app>.azurewebsites.net`) should **never** follow code into staging; it must remain pointing to production!

---

## 📈 3. Autoscaling Mechanics

Autoscaling dynamically adjusts the number of compute instances running your application based on demand.

### Scaling Modes in Premium V3
1. **Automatic Scaling:**  
   * Introduced in Premium V2/V3 tiers.
   * Microsoft manages the scaling triggers based on incoming HTTP traffic and health metrics.
   * Administrators configure:
     * **Always ready instances (Minimum):** Baseline capacity (e.g., `1`).
     * **Maximum burst:** The ceiling to prevent unexpected costs (e.g., `2`).
2. **Rule-Based Scaling (Custom Autoscale):**  
   * Explicit metric thresholds: e.g., CPU percentage $> 70\%$ adds instances, CPU $< 30\%$ removes instances.
   * Requires cooldown periods (e.g., 5–10 minutes) to prevent thrashing/flapping.

---

## 🛠️ 4. Quick Reference: Azure CLI Cheatsheet

> *Note: Replace `<your-resource-group>` (e.g., `abdullah-rg9`) and `<your-app-name>` (e.g., `abdullahapp222`) with your own values.*

```bash
# 1. Create Resource Group
az group create --name <your-resource-group> --location eastus

# 2. Create an App Service Plan (Premium V3 P1V3)
az appservice plan create --name <your-name>-asp --resource-group <your-resource-group> --sku P1V3 --is-linux

# 3. Create Web App (PHP 8.2 runtime)
az webapp create --resource-group <your-resource-group> --plan <your-name>-asp --name <your-app-name> --runtime "PHP|8.2"

# 4. Create Staging Deployment Slot
az webapp deployment slot create --resource-group <your-resource-group> --name <your-app-name> --slot staging

# 5. Enable "Always On" & Basic Auth on Staging Slot
az webapp config set --resource-group <your-resource-group> --name <your-app-name> --slot staging --always-on true

# 6. Deploy Zip Package directly to Slot (OneDeploy)
az webapp deploy --resource-group <your-resource-group> --name <your-app-name> --slot staging --src-path app.zip --type zip

# 7. Perform Zero-Downtime Slot Swap
az webapp deployment slot swap --resource-group <your-resource-group> --name <your-app-name> --slot staging --target-slot production

# 8. Clean up all lab resources
az group delete --name <your-resource-group> --yes --no-wait
```

---

## ❓ 5. Frequently Asked Questions (FAQ) & Exam Traps

#### Q1: If I create 4 deployment slots in a P1V3 plan, will my Azure bill increase?
**Answer:** **No.** Deployment slots run on the existing compute instances allocated to that App Service Plan. However, because all slots share the same CPU and RAM, high activity on a staging slot can consume resources needed by production.

#### Q2: What happens if a slot swap fails in the middle of execution?
**Answer:** Azure will **not** partially complete a swap. If the warm-up requests or health check endpoints return a non-200 code during the swap initialization phase, the entire transaction rolls back, leaving your production site untouched.

#### Q3: Can I route a percentage of live traffic to my staging slot for A/B testing?
**Answer:** **Yes!** In the **Deployment slots** blade, there is a **Traffic %** column. You can specify `90%` to production and `10%` to staging. Azure will use routing cookies (`x-ms-routing-name`) to pin users to a specific version.

#### Q4: Why did we experience a 503 error when connecting External Git?
**Answer:** A `503 Service Unavailable` on `UpdateSiteSourceControl` happens when Azure's control plane cannot reach the slot's Kudu backend. On Linux App Services, slots default to **Always On = Off**, which puts the background worker to sleep when idle. Turning **Always On = On** and verifying **SCM Basic Auth** restores communication.
