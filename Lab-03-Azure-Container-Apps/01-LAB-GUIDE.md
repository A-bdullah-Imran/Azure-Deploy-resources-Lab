# Lab 03: Implement Azure Container Apps (ACA)

## Lab Overview & Scenario
In this lab, you will evaluate **Azure Container Apps (ACA)** as a modern, serverless container platform designed for microservices and cloud-native applications. Built on top of enterprise Kubernetes and Envoy proxy, Azure Container Apps abstracts away all cluster management while providing native capabilities like dynamic scaling, automated ingress, and automatic managed HTTPS certificates.

You will provision a **Container Apps Environment**, deploy a containerized application using a quickstart image, configure public ingress, and verify the live application over HTTPS.

---

## 🎯 Official Lab Objectives
* **Task 1:** Create and configure an Azure Container App and environment.
* **Task 2:** Test and verify deployment of the Azure Container App.
* **Task 3:** Clean up resources.

---

## 🏷️ Suggested Naming Conventions for Students

> 💡 **Important for Students:**  
> The names below are **suggested templates**. Wherever you see placeholders like `<your-name>`, substitute your own initials, name, or student ID.

| Setting | Suggested Pattern | Example (e.g., Instructor Abdullah) | Description |
| :--- | :--- | :--- | :--- |
| **Resource Group** | `<your-name>-rg9c` | `abdullah-rg9c` | Dedicated resource group for Lab 03 |
| **Container App Name** | `<your-name>-app` | `abdullah-app` | Unique container app identifier |
| **Environment Name** | `<your-name>-env` | `abdullah-env` | Managed Container Apps environment |
| **Application URL** | `https://<app>.<random>.<region>.azurecontainerapps.io` | Live HTTPS web endpoint |

---

## ⏱️ Estimated Time: 15–20 minutes

---

## Task 1: Create and configure an Azure Container App and environment

1. Sign in to the **Azure Portal** ([https://portal.azure.com](https://portal.azure.com)).
2. In the top search bar, search for and select **Container Apps**.
3. Select **+ Create** and choose **Container App** from the dropdown menu.
4. On the **Basics** tab, configure the following settings:

| Setting | Value |
| :--- | :--- |
| **Subscription** | Select your active Azure subscription |
| **Resource group** | Select **Create new** $\rightarrow$ enter **`<your-name>-rg9c`** *(e.g., `abdullah-rg9c`)* |
| **Container app name** | Enter **`<your-name>-app`** *(e.g., `abdullah-app`)* |
| **Region** | Select an Azure region near you (e.g., **East US** or **Central India**) |
| **Container Apps Environment** | Accept default or click **Create new** and name it **`<your-name>-env`** |

5. Select the **Container** tab:
   * Verify that **Use quickstart image** is checked.
   * Verify the **Quickstart image** is set to **Simple hello world container** (`mcr.microsoft.com/azuredocs/aci-helloworld` or quickstart image).
   * Leave container CPU and Memory allocations at default values (0.25 vCPU, 0.5 GiB).

6. Select the **Ingress** tab (or check the *Application ingress settings*):
   * **Ingress:** Ensure it is **Enabled** (checked).
   * **Ingress traffic:** Select **Accepting traffic from anywhere**.
   * **Ingress type:** **HTTP**.
   * **Target port:** **`80`**.

7. Click **Review + create**, verify that validation passes, and then click **Create**.
8. Wait for the deployment to finish (about 2–3 minutes), then click **Go to resource**.

---

## Task 2: Test and verify deployment of the Azure Container App

1. On the **Overview** blade of your deployed Container App, locate the **Application URL**.
   * *Notice that Azure Container Apps automatically generates a secure URL beginning with **`https://`**.*
   * *Format: `https://<app-name>.<unique-string>.<region>.azurecontainerapps.io`*

> 💡 **Comparison Note vs. Lab 02 (ACI):**  
> In Lab 02, the container only supported plain `http://` on Port 80 because ACI does not provide built-in SSL termination.  
> Azure Container Apps uses a built-in **Envoy proxy** that automatically provisions and manages free public **TLS/SSL certificates**, allowing native HTTPS out of the box!

2. Click on the **Application URL** link (or copy and paste it into a new browser tab).
3. Verify that the webpage opens securely and displays:
   **"Your container app is running with a Hello World image"** *(or the welcome container page)*.

---

## Task 3: Clean up resources

To avoid incurring ongoing charges for the Container App environment and Log Analytics workspace:

1. Open **Azure Cloud Shell** (PowerShell) and execute:
   ```powershell
   Remove-AzResourceGroup -Name "<your-resource-group>" -Force -AsJob
   ```
   *(e.g., `Remove-AzResourceGroup -Name "abdullah-rg9c" -Force -AsJob`)*
2. Or in the Azure Portal, open your resource group, click **Delete resource group**, enter its name, and confirm.
