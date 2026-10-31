# Lab 01: Implement Azure App Services

## Lab Overview & Scenario
In this lab, you will evaluate Azure App Service as a Platform-as-a-Service (PaaS) solution to host and scale web applications without managing underlying virtual machines or operating systems. 

You will create an App Service Plan and a Web App, configure a staging deployment slot for zero-downtime updates, deploy application code, perform a slot swap into production, configure autoscale rules, and test application load handling.

---

## 🎯 Lab Objectives
1. **Task 1:** Create and configure an Azure Web App (PHP 8.2 on Linux).
2. **Task 2:** Create a staging deployment slot.
3. **Task 3:** Configure deployment settings & deploy sample code *(includes instructor troubleshooting for the Kudu 503 error)*.
4. **Task 4:** Test staging deployment and perform a production slot swap.
5. **Task 5:** Configure automated scale-out and scale-in rules.
6. **Task 6:** Validate autoscaling & review test metrics *(includes portal navigation UI tip)*.
7. **Task 7:** Clean up resources to prevent recurring charges.

---

## ⏱️ Estimated Time: 40 minutes

---

## Task 1: Create an Azure Web App

1. Sign in to the **Azure Portal** ([portal.azure.com](https://portal.azure.com)).
2. In the top search bar, search for and select **App Services**.
3. On the App Services blade, click **+ Create** and select **Web App**.
4. In the **Basics** tab, configure the following settings:
   * **Subscription:** Select your active Azure subscription.
   * **Resource Group:** Click **Create new** $\rightarrow$ enter `az104-rg9` (or `az104-09a-rg1`).
   * **Name:** Enter a globally unique name (e.g., `abdullahapp222` or `app-student-<unique-id>`).
   * **Publish:** Select **Code**.
   * **Runtime stack:** Select **PHP 8.2** (or current supported PHP Linux stack).
   * **Operating System:** Select **Linux**.
   * **Region:** Select a region close to you (e.g., *East US*, *West Europe*, or *Central India*).
5. In the **App Service Plan** section:
   * **Pricing Plan:** Select **Standard (S1)** or **Premium V3 (P0v3 / P1v3)**.
     > ⚠️ **Important for Students:** The *Free (F1)* and *Basic (B1)* tiers **do not support** deployment slots or autoscaling. You must select at least **Standard (S1)** or higher.
6. Click **Review + Create**, verify validation passes, and click **Create**.
7. Wait for deployment to finish, then click **Go to resource**.

---

## Task 2: Create a Staging Deployment Slot

1. On your Web App blade, scroll down the left menu to the **Deployment** section and click **Deployment slots**.
2. Click **+ Add Slot**.
3. In the **Add a slot** panel on the right:
   * **Name:** Enter `staging`.
   * **Clone settings from:** Select **Do not clone settings**.
4. Click **Add**.
5. Once created, click **Close**. Notice that both slots are now listed:
   * `<your-app-name>` (Traffic: 100% - Production)
   * `<your-app-name>-staging` (Traffic: 0% - Staging)

---

## Task 3: Configure Deployment Settings & Deploy Code

### Step 3.1: Navigate to the Staging Slot
1. On the **Deployment slots** blade, click directly on the newly created **staging** slot (e.g., `<your-app-name>-staging`).
2. Verify you are inside the staging slot by checking the top breadcrumb/title banner:
   `Home > App Services > <your-app-name> | Deployment slots > <your-app-name>-staging`

---

### Step 3.2: Enable SCM Basic Auth and Always On
> 💡 **Instructor Insight:** Azure App Service disables Basic Authentication and sets *Always On* to `Off` by default on new slots. This can cause the Kudu background worker to sleep or block source control sync, throwing a `503 Server Unavailable` error.

1. In the left navigation of the **staging slot**, under **Settings**, select **Configuration**.
2. Click the **General settings** tab:
   * Locate **Always On** and switch it to **On**.
   * Locate **SCM Basic Auth Publishing Credentials** and ensure it is **On** (or checked).
3. Click **Save** at the top and confirm with **Continue**.
4. Go to **Overview** on the staging slot, click **Restart**, and wait **30 seconds** for the container to initialize.

---

### Step 3.3: Deploy Application Code

You have two paths to deploy the code:

#### Method A: Portal UI (Standard Lab Flow)
1. In the staging slot menu, under **Deployment**, select **Deployment Center**.
2. Go to the **Settings** tab.
3. Configure:
   * **Source:** `External Git`
   * **Repository:** `https://github.com/Azure-Samples/php-docs-hello-world`
   * **Branch:** `master`
4. Click **Save**.

---

#### Method B: Fast-Track CLI Deployment (Guaranteed Instructor Fix)
If Method A throws the following error:
> `Failed to set up deployment: Repository 'UpdateSiteSourceControl' operation failed with System.Net.WebException: The remote server returned an error: (503) Server Unavailable.`

Use this proven CLI workaround directly inside **Azure Cloud Shell**:

1. Click the **Cloud Shell** icon (`>_`) in the top toolbar of the Azure Portal (choose **PowerShell**).
2. Run these commands:
   ```powershell
   Set-Content -Path index.php -Value "<?php echo '<h1>Hello World from Staging Slot!</h1>'; ?>"
   Compress-Archive -Path index.php -DestinationPath app.zip -Force
   az webapp deploy --resource-group az104-rg9 --name <your-app-name> --slot staging --src-path app.zip --type zip
   ```
   *(Replace `<your-app-name>` with your actual base app name; do **not** add `/staging`)*.

3. Verify that the output shows `Deployment successful`.

---

## Task 4: Verify Staging & Swap Slots into Production

1. On the **staging slot** blade, click **Overview**.
2. Locate the **Default domain** URL (e.g., `https://<your-app-name>-staging.azurewebsites.net`) and click it.
3. Confirm that the browser displays: **Hello World from Staging Slot!** (or the sample PHP page).
4. Go back to your main Web App (click your main app name in the top breadcrumb navigation).
5. From the main app **Overview** blade, click its **Default domain** URL (e.g., `https://<your-app-name>.azurewebsites.net`).
   * *Notice that production still shows the default Azure hosting landing page.*
6. Return to the portal and navigate to **Deployment slots** on the main Web App.
7. Click the **Swap** button at the top toolbar:
   * **Source:** `staging`
   * **Target:** `production`
   * **Swap type:** `Swap`
8. Click **Swap** at the bottom and wait 1–2 minutes for the operation to complete.
9. Refresh both browser tabs:
   * **Production URL:** Now displays **Hello World from Staging Slot!**
   * **Staging URL:** Now displays the old default landing page.
   * *Zero downtime occurred during this transaction!*

---

## Task 5: Configure Autoscaling Rules

1. On the main Web App menu, under **Settings**, select **Scale out (App Service plan)**.
2. Select **Rule based** (or **Custom autoscale**).
3. In the **Scale mode** section, choose **Scale based on a metric**.
4. In the **Default** scale condition:
   * **Instance limits:**
     * **Minimum:** `1`
     * **Maximum:** `3`
     * **Default:** `1`
5. Click **+ Add a rule** to configure scale-out:
   * **Metric source:** Current resource
   * **Metric namespace:** App Service Plan metrics
   * **Metric name:** `CPU Percentage`
   * **Operator:** `Greater than`
   * **Threshold:** `70`
   * **Duration (minutes):** `10`
   * **Operation:** `Increase count by`
   * **Instance count:** `1`
   * **Cool down (minutes):** `5`
   * Click **Add**.
6. Click **+ Add a rule** to configure scale-in (scale down):
   * **Metric name:** `CPU Percentage`
   * **Operator:** `Less than`
   * **Threshold:** `30`
   * **Duration (minutes):** `10`
   * **Operation:** `Decrease count by`
   * **Instance count:** `1`
   * **Cool down (minutes):** `5`
   * Click **Add**.
7. Click **Save** at the top of the blade.

---

## Task 6: Test & Review Performance Metrics

1. In the Web App menu, under **Development Tools**, select **Performance test** (or use **Azure Load Testing** if configured).
2. Create and run a simulated test (e.g., 250 virtual users for 60 seconds against the production URL).
3. ⚠️ **Portal UI Tip for Students:**
   > When viewing test results in the portal, the overview chart may initially appear empty or show default stats. **You must explicitly click on your specific test run from the test history list** to open its detailed reporting blade. There you will see real-time charts for:
   > - Virtual Users
   > - Requests per Second
   > - Average Response Time (ms)
   > - HTTP 200 vs 5xx errors

---

## Task 7: Clean Up Resources

To avoid incurring unwanted subscription charges after completing the lab:

1. Open **Azure Cloud Shell** (PowerShell) and execute:
   ```powershell
   Remove-AzResourceGroup -Name "az104-rg9" -Force -AsJob
   ```
2. Or in the Azure Portal, open the Resource Group (`az104-rg9`), click **Delete resource group**, enter the resource group name to confirm, and click **Delete**.
