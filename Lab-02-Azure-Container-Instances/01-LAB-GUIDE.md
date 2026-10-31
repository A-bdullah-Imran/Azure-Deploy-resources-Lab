# Lab 02: Implement Azure Container Instances (ACI)

## Lab Overview & Scenario
In this lab, you will evaluate **Azure Container Instances (ACI)** as a serverless container solution. ACI allows you to run Docker containers in the cloud within seconds without provisioning, configuring, or managing underlying virtual machines or container orchestrators (like Kubernetes).

You will deploy a containerized web application using a public Docker container image, configure public networking with a custom DNS name label, test the application through a web browser, and inspect the incoming HTTP GET request logs in the Azure Portal.

---

## 🎯 Official Lab Objectives
* **Task 1:** Deploy an Azure Container Instance using a Docker image.
* **Task 2:** Review the functionality of the Azure Container Instance *(includes HTTP vs. HTTPS browser tip)*.
* **Task 3:** Clean up resources.

---

## 🏷️ Suggested Naming Conventions for Students

> 💡 **Important for Students:**  
> The names below are **suggested templates**. Wherever you see placeholders like `<your-name>` or `<your-dns-label>`, substitute your own initials, name, or student ID.  
> *Note:* The DNS Name Label must be **globally unique across all of Azure** in that region.

| Setting | Suggested Pattern | Example (e.g., Instructor Abdullah) | Description |
| :--- | :--- | :--- | :--- |
| **Resource Group** | `<your-name>-rg9b` | `abdullah-rg9b` | Dedicated resource group for Lab 02 |
| **Container Name** | `<your-name>-c1` | `abdullah-c1` | Name of the container instance |
| **DNS Name Label** | `<your-name>-aci-<unique#>` | `abdullah-aci-222` | Prefix for the public FQDN |
| **Public FQDN** | `<your-dns-label>.<region>.azurecontainer.io` | `abdullah-aci-222.eastus.azurecontainer.io` | Fully Qualified Domain Name |

---

## ⏱️ Estimated Time: 15 minutes

---

## Task 1: Deploy an Azure Container Instance using a Docker image

1. Sign in to the **Azure Portal** ([https://portal.azure.com](https://portal.azure.com)).
2. In the Azure portal, search for and select **Container instances** and then, on the Container instances blade, click **+ Create**.
3. On the **Basics** tab of the *Create container instance* blade, specify the following settings (leave others with their default values):

| Setting | Value |
| :--- | :--- |
| **Subscription** | Select your active Azure subscription |
| **Resource group** | Select **Create new** $\rightarrow$ enter **`<your-name>-rg9b`** *(e.g., `abdullah-rg9b`)* |
| **Container name** | Enter **`<your-name>-c1`** *(e.g., `abdullah-c1`)* |
| **Region** | Select an Azure region near you (e.g., **East US** or **Central India**) |
| **Image source** | **Quickstart images** |
| **Image** | **`mcr.microsoft.com/azuredocs/aci-helloworld:latest (Linux)`** |

4. Click **Next: Networking >** and specify the following settings (leave others with their default values):

| Setting | Value |
| :--- | :--- |
| **DNS name label** | Enter a globally unique DNS hostname: **`<your-name>-aci-<unique#>`** *(e.g., `abdullah-aci-222`)* |

> 📌 **Note:** Your container will be publicly reachable at `http://<your-dns-label>.<region>.azurecontainer.io`.

5. Click **Next: Advanced >**, review the settings on the Advanced tab of the Create container instance blade without making any changes.
6. Click **Review + Create**, ensure that validation passed, and click **Create**.
7. Wait for the deployment to complete (about 1 minute), then click **Go to resource**.

---

## Task 2: Review the functionality of the Azure Container Instance

1. On the **Overview** blade of your deployed container instance, verify that the **Status** is reported as **Running**.
2. Copy the **FQDN** (e.g., `<your-dns-label>.<region>.azurecontainer.io`).

---

### ⚠️ Critical Tip for Students: The Browser HTTPS Trap!
> **Browser Warning:** Modern browsers (Chrome, Edge, Brave) automatically force `https://` in the address bar.  
> Because this sample container only listens on **HTTP Port 80** and has no SSL certificate, navigating to `https://...` will fail with:  
> `This site can't be reached` or `Connection refused`.
> 
> **Action:** In your browser address bar, make sure to explicitly use **`http://`** (without the `s`):  
> `http://<your-dns-label>.<region>.azurecontainer.io`

---

3. Open a new browser tab, enter the URL using `http://`, and verify that the **"Welcome to Azure Container Instances!"** page is displayed.
4. Refresh the browser page several times to generate HTTP `GET` request traffic, then close the browser tab.
5. Back in the Azure Portal on the container instance blade, in the left-hand navigation menu under **Settings**, select **Containers**.
6. Select the **Logs** tab:
   * Verify that you can see the log entries representing the HTTP `GET` requests generated when you refreshed the browser page.

---

## Task 3: Clean up resources

Azure Container Instances bills while in the running state. To avoid ongoing charges:

1. Open **Azure Cloud Shell** (PowerShell) and run:
   ```powershell
   Remove-AzResourceGroup -Name "<your-resource-group>" -Force -AsJob
   ```
   *(e.g., `Remove-AzResourceGroup -Name "abdullah-rg9b" -Force -AsJob`)*
2. Or in the Azure Portal, open your resource group, click **Delete resource group**, enter its name, and confirm.
