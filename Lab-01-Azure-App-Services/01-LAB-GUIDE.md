# Lab 01: Implement Azure App Services

## Lab Overview & Scenario
In this lab, you will evaluate Azure App Service as a Platform-as-a-Service (PaaS) solution to host and scale web applications without managing underlying virtual machines or operating systems. 

You will create an App Service Plan on the **Premium V3 P1V3** tier and deploy a Linux-based PHP Web App inside your dedicated resource group. You will configure a staging deployment slot for zero-downtime updates, deploy application code, perform a slot swap into production, configure **Automatic scaling** with burst limits, and run an integrated **Azure Load Test** through App Service Diagnostics.

---

## 🎯 Lab Objectives
* **Task 1:** Create an Azure web app (PHP 8.2 on Linux with **Premium V3 P1V3**).
* **Task 2:** Create and configure a deployment slot (`staging`).
* **Task 3:** Configure Web App deployment settings *(includes instructor fix for Kudu 503 error)*.
* **Task 4:** Swap deployment slots (promoting staging into production).
* **Task 5:** Configure and test autoscaling of the Azure web app (Automatic scaling & Azure Load Testing).
* **Task 6:** Clean up lab resources.

---

## 🏷️ Suggested Naming Conventions for Students

> 💡 **Important for Students:**  
> The names below are **suggested templates**. Wherever you see placeholders like `<your-name>` or `<your-app-name>`, substitute your own initials, name, or student ID.  
> *Note:* Web App names must be **globally unique across all of Azure** worldwide.

| Resource Type | Suggested Pattern | Example (e.g., Instructor Abdullah) | Description |
| :--- | :--- | :--- | :--- |
| **Resource Group** | `<your-name>-rg9` | `abdullah-rg9` | Dedicated resource group for Lab 01 |
| **App Service Plan** | `<your-name>-asp` | `abdullah-asp` | Premium V3 P1V3 compute plan |
| **Web App Name** | `<your-name>-app<unique#>` | `abdullahapp222` | Must be globally unique across Azure |
| **Deployment Slot** | `staging` | `staging` | Secondary testing slot |
| **Load Test Resource** | `<your-name>-loadtest` | `abdullah-loadtest` | Diagnostic performance testing engine |

---

## ⏱️ Estimated Time: 35–45 minutes

---

## Task 1: Create an Azure web app

1. Sign in to the **Azure Portal** ([https://portal.azure.com](https://portal.azure.com)).
2. In the top search bar, search for and select **App Services**.
3. On the App Services blade, click **+ Create** and select **Web App**.
4. On the **Basics** tab, configure the following settings:

| Setting | Value |
| :--- | :--- |
| **Subscription** | Select your active Azure subscription |
| **Resource group** | Select **Create new** $\rightarrow$ enter **`<your-name>-rg9`** *(e.g., `abdullah-rg9`)* |
| **Name** | Enter a globally unique name: **`<your-name>-app<unique#>`** *(e.g., `abdullahapp222`)* |
| **Publish** | **Code** |
| **Runtime stack** | **PHP 8.2** |
| **Operating system** | **Linux** |
| **Region** | Select an Azure region near you (e.g., **East US** or **Central India**) |
| **Pricing plans** | **Premium V3 P1V3** |
| **Zone redundancy** | Accept defaults (Disabled) |

5. Click **Review + create**, verify that validation passes, and then click **Create**.
6. Wait for the deployment to complete (about 1 minute), then click **Go to resource**.

---

## Task 2: Create and configure a deployment slot

1. On the blade of your newly deployed Web App, under the **Overview** section, click the **Default domain** link to view the default placeholder web page, then close that tab.
2. Back in the Azure portal, navigate down the left-hand menu to the **Deployment** section and click **Deployment slots**.
   * *Notice that the web app currently has a single slot labeled `production` with 100% traffic.*
3. Click **+ Add slot** and configure:
   * **Name:** `staging`
   * **Clone settings from:** `Do not clone settings`
4. Click **Add**.
5. Once created, click **Close**. Notice that both slots are now listed:
   * `<your-app-name>` (Traffic: 100% - Production)
   * `<your-app-name>-staging` (Traffic: 0% - Staging)
6. Click directly on the newly created **staging** slot entry to open its management blade.
   * *Verify at the top breadcrumb that you are working on the staging slot (`<your-app-name>/staging`).*

---

## Task 3: Configure Web App deployment settings

### Step 3.1: Enable SCM Basic Auth and Always On
1. In the left navigation of the **staging slot**, under **Settings**, select **Configuration**.
2. Click the **General settings** tab:
   * Locate **SCM Basic Auth Publishing Credentials** and ensure it is **On** (or checked). Click **Apply** / **Save** if prompted.
   * Verify that **Always On** is set to **On** (prevents slot container from sleeping).
3. Click **Save** at the top.
4. On the staging slot **Overview** blade, click **Restart** and wait **30 seconds**.

---

### Step 3.2: Configure Deployment Source

#### Option A: Azure Portal UI (Standard Lab Instruction)
1. On the staging slot blade, under the **Deployment** section, select **Deployment Center**.
2. Select the **Settings** tab and configure:
   * **Source:** `External Git`
   * **Repository:** `https://github.com/Azure-Samples/php-docs-hello-world`
   * **Branch:** `master`
3. Click **Save**.

---

#### Option B: Azure Cloud Shell CLI Workaround (Instructor Fix for 503 Error)
> ⚠️ **Troubleshooting Callout:**  
> If clicking **Save** in the UI fails with:  
> `Failed to set up deployment: Repository 'UpdateSiteSourceControl' operation failed ... (503) Server Unavailable`  
> Open **Azure Cloud Shell** (PowerShell) and run these 3 lines:

```powershell
Set-Content -Path index.php -Value "<?php echo '<h1>Hello World from Staging Slot!</h1>'; ?>"
Compress-Archive -Path index.php -DestinationPath app.zip -Force
az webapp deploy --resource-group <your-resource-group> --name <your-app-name> --slot staging --src-path app.zip --type zip
```
*(Replace `<your-resource-group>` and `<your-app-name>` with your actual names; do not add `/staging`).*

---

### Step 3.3: Verify Staging Deployment
1. On the **staging slot** blade, click **Overview**.
2. Click the **Default domain** link (e.g., `https://<your-app-name>-staging.azurewebsites.net`).
3. Verify that the browser displays: **Hello World!**

---

## Task 4: Swap deployment slots

1. Navigate back to your main Web App blade (click your main app name in the top breadcrumb navigation).
2. Under **Overview**, click the **Default domain** URL (`https://<your-app-name>.azurewebsites.net`).
   * *Notice that production still displays the default Azure placeholder page.*
3. On the left menu under **Deployment**, select **Deployment slots**.
4. Click the **Swap** button at the top toolbar:
   * **Source:** `staging`
   * **Target:** `production`
   * **Swap type:** `Swap`
5. Click **Swap** at the bottom and wait 1–2 minutes for the operation to complete.
6. Once the notification confirms success, refresh your browser tabs:
   * **Production URL:** Now serves the **Hello World!** application.
   * **Staging URL:** Now displays the old placeholder page.
   * *Zero downtime occurred during the swap!*

---

## Task 5: Configure and test autoscaling of the Azure web app

### Step 5.1: Configure Automatic Scaling
1. On the main Web App menu, under the **App Service plan** section, select **Scale out**.
2. Under the **Scaling** section, select **Automatic** *(Rule-based)*.
3. Configure the burst parameters:
   * **Maximum burst:** `2`
   * **Always ready instances (Minimum):** `1`
4. Click **Save** at the top of the blade.

---

### Step 5.2: Test Autoscaling with Azure Load Testing
1. In the left-hand pane of your main Web App, select **Diagnose and solve problems**.
2. Scroll to the **Load Test your App** card and select **Create Load Test**.
3. In the creation wizard:
   * Enter a test name: **`<your-name>-loadtest`** *(e.g., `abdullah-loadtest`)*.
   * Click **Review + create** $\rightarrow$ **Create**.
4. Once deployment succeeds, click **Go to resource**.
5. Under *Overview*, choose **Create by adding HTTP requests** and select **Create**.
6. On the **Test plan** tab:
   * Click **Add request**.
   * Paste your web app's **Default domain URL** (`https://<your-app-name>.azurewebsites.net`).
   * Click **Add**.
7. Click **Review + create** and then click **Create**.
8. Wait for the test execution to complete (about 2–3 minutes).

---

### ⚠️ Critical UI Tip for Viewing Metrics
> **Instructor Note:** When the test completes, the parent page may not immediately show live graphs.  
> **You must click into your specific test run from the test history list.**  
> Inside that specific test run, you will view the live performance telemetry:
> * **Virtual Users**
> * **Requests per Second (Throughput)**
> * **Average Response Time (ms)**
> * **Error rate (HTTP 200 vs. non-200)**

---

## Task 6: Clean up resources

To prevent ongoing charges on the Premium V3 tier, delete the resource group when testing is complete:

1. Open **Azure Cloud Shell** (PowerShell) and run:
   ```powershell
   Remove-AzResourceGroup -Name "<your-resource-group>" -Force -AsJob
   ```
   *(e.g., `Remove-AzResourceGroup -Name "abdullah-rg9" -Force -AsJob`)*
2. Or in the Azure Portal, open your resource group, click **Delete resource group**, enter its name, and confirm.
