# Lab 02: Implement Azure Container Instances (ACI)

## Lab Overview & Scenario
In this lab, you will evaluate **Azure Container Instances (ACI)** as a serverless container solution. ACI allows you to run Docker containers in the cloud within seconds without provisioning, configuring, or managing underlying virtual machines or container orchestrators (like Kubernetes).

You will deploy a containerized web application using a public Docker container image, configure public networking with a custom DNS name label, test the application through a web browser, and inspect live container logs and console sessions.

---

## 🎯 Lab Objectives
* **Task 1:** Deploy an Azure Container Instance using a Docker image.
* **Task 2:** Review the functionality and lifecycle of the Azure Container Instance *(includes the HTTP vs. HTTPS browser trap)*.
* **Task 3:** Inspect container logs and open an interactive bash console.
* **Task 4:** Clean up lab resources.

---

## 🏷️ Suggested Naming Conventions for Students

> 💡 **Important for Students:**  
> The names below are **suggested templates**. Wherever you see placeholders like `<your-name>` or `<your-dns-label>`, substitute your own initials, name, or student ID.  
> *Note:* The DNS Name Label must be **globally unique across all of Azure** in that region.

| Resource Type | Suggested Pattern | Example (e.g., Instructor Abdullah) | Description |
| :--- | :--- | :--- | :--- |
| **Resource Group** | `<your-name>-rg9b` | `abdullah-rg9b` | Dedicated resource group for Lab 02 |
| **Container Name** | `<your-name>-c1` | `abdullah-c1` | Name of the container instance |
| **DNS Name Label** | `<your-name>-aci-<unique#>` | `abdullah-aci-222` | Prefix for the public FQDN |
| **Public FQDN** | `<your-dns-label>.<region>.azurecontainer.io` | `abdullah-aci-222.eastus.azurecontainer.io` | Fully Qualified Domain Name |

---

## ⏱️ Estimated Time: 15–20 minutes

---

## Task 1: Deploy an Azure Container Instance

1. Sign in to the **Azure Portal** ([https://portal.azure.com](https://portal.azure.com)).
2. In the top search bar, search for and select **Container instances**.
3. On the Container instances blade, click **+ Create**.
4. On the **Basics** tab, configure the following settings:

| Setting | Value |
| :--- | :--- |
| **Subscription** | Select your active Azure subscription |
| **Resource group** | Select **Create new** $\rightarrow$ enter **`<your-name>-rg9b`** *(e.g., `abdullah-rg9b`)* |
| **Container name** | Enter **`<your-name>-c1`** *(e.g., `abdullah-c1`)* |
| **Region** | Select an Azure region near you (e.g., **East US** or **Central India**) |
| **Image source** | **Quickstart images** |
| **Image** | **`mcr.microsoft.com/azuredocs/aci-helloworld:latest (Linux)`** |

5. Select the **Networking** tab and configure:

| Setting | Value |
| :--- | :--- |
| **Networking type** | **Public** |
| **DNS name label** | Enter a globally unique label: **`<your-name>-aci-<unique#>`** *(e.g., `abdullah-aci-222`)* |
| **Ports** | Keep default: **Port `80`**, Protocol **`TCP`** |

6. Select the **Advanced** tab:
   * **Restart policy:** Select **On failure** *(or keep default Always)*.
   * Leave environment variables empty.
7. Click **Review + create**, verify that validation passes, and then click **Create**.
8. Wait for the deployment to finish (typically 30–60 seconds), then click **Go to resource**.

---

## Task 2: Review Container Functionality & The HTTP Trap

1. On the container instance **Overview** blade, verify that:
   * **Status:** Displays **Running**.
   * **Provisioning state:** Displays **Succeeded**.
   * Locate the **FQDN** (e.g., `<your-dns-label>.<region>.azurecontainer.io`).
   * Locate the **IP address** (Public IP).

---

### ⚠️ Critical Instructor Warning: The Browser HTTPS Trap!
> **Student Alert:** Modern browsers (Chrome, Edge, Brave, Safari) automatically force `https://` in the address bar.  
> Because this sample container only listens on **HTTP Port 80** and has no SSL certificate, navigating to `https://...` will immediately fail with:  
> `This site can't be reached` or `Connection refused`.

2. Open a new web browser tab.
3. In the address bar, explicitly type **`http://`** (without the `s`) followed by your FQDN:
   ```text
   http://<your-dns-label>.<region>.azurecontainer.io
   ```
   *(Example: `http://abdullah-aci-222.eastus.azurecontainer.io`)*.
4. Verify that the webpage displays:
   **"Welcome to Azure Container Instances!"**
5. *(Optional verification)* You can also visit `http://<Public-IP-Address>` to verify access directly via IP.
6. Refresh the browser page 3–4 times to generate incoming HTTP request traffic for the next task.

---

## Task 3: Inspect Container Logs & Interactive Console

### Step 3.1: Inspect Container Logs
1. Return to the Azure Portal on your container instance blade.
2. In the left navigation menu under **Settings**, select **Containers**.
3. Click the **Logs** tab:
   * Notice the Node.js/Alpine Linux startup logs.
   * Notice the live incoming **`HTTP GET /`** request logs corresponding to your browser refreshes from Task 2.

---

### Step 3.2: Inspect Lifecycle Events
1. In the same **Containers** blade, select the **Events** tab.
2. Review the chronological lifecycle events that Azure executed:
   * `Pulling image "mcr.microsoft.com/azuredocs/aci-helloworld:latest"`
   * `Successfully pulled image ...`
   * `Created container ...`
   * `Started container ...`

---

### Step 3.3: Connect to the Running Container via Interactive Console
1. In the same **Containers** blade, select the **Connect** tab.
2. Under **Command**, keep `/bin/sh` selected.
3. Click **Connect**.
4. A live terminal prompt (`/usr/src/app #`) will open inside the container.
5. Run the following Linux inspection commands:
   ```sh
   # View running processes inside the container
   ps aux

   # View the application source files
   ls -la

   # View the container OS distribution details
   cat /etc/os-release
   ```
6. Once satisfied, you can close the terminal session.

---

## Task 4: Clean Up Resources

Azure Container Instances bills per second of execution while the container is running. To avoid incurring subscription charges:

1. Open **Azure Cloud Shell** (PowerShell) and execute:
   ```powershell
   Remove-AzResourceGroup -Name "<your-resource-group>" -Force -AsJob
   ```
   *(e.g., `Remove-AzResourceGroup -Name "abdullah-rg9b" -Force -AsJob`)*
2. Or in the Azure Portal, open the resource group, click **Delete resource group**, enter its name, and confirm.
