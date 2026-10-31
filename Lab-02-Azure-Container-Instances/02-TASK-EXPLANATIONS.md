# Lab 02: Task-by-Task Deep Explanations

This document provides instructor-level technical context explaining **what** we are doing in each step of Lab 02, **why** each parameter is configured, and **what happens behind the scenes** inside Microsoft Azure.

---

## 📌 Task 1: Deploying an Azure Container Instance (ACI)

### 1. What is Azure Container Instances (ACI)?
**Azure Container Instances (ACI)** is Microsoft’s serverless container compute offering.
* **No VM Management:** You do not manage underlying virtual machines, host operating systems, or patching.
* **Instant Startup:** Pre-provisioned hypervisor infrastructure allows Linux and Windows containers to launch within 30–60 seconds.
* **Per-Second Billing:** You pay strictly for the CPU cores and memory allocated while the container is in the `Running` state.

---

### 2. Configuration Parameters Explained

#### A. Image Source: Quickstart Images
* In Task 1, we select the official Microsoft demo image: `mcr.microsoft.com/azuredocs/aci-helloworld:latest`.
* It is hosted on the **Microsoft Container Registry (MCR)**, optimized for fast geographic distribution across Azure regions.

#### B. Networking Tab & DNS Name Label
* By selecting **Public** networking, Azure assigns a public IPv4 address to the container.
* The **DNS name label** creates a human-readable Fully Qualified Domain Name (FQDN) using this formula:
  `[DNS-Name-Label].[Azure-Region].azurecontainer.io`
* Because all containers in that region share the `.azurecontainer.io` root zone, the label must be globally unique within that Azure region.

#### C. Advanced Tab: Reviewing Settings Without Changes
* In Step 5, the lab instructs you to review the **Advanced** tab without making changes.
* This accepts the default **Restart Policy: Always**:
  * For a continuous web application like `aci-helloworld`, `Always` ensures that if the internal web process crashes or runs out of memory, Azure automatically restarts the container to maintain availability.

---

## 📌 Task 2: Reviewing Functionality & Inspecting Logs

### 1. Why does browsing via HTTPS fail? (The Browser Trap)
When navigating to the container's FQDN, modern browsers (Edge, Chrome, Brave) automatically force `https://` in the address bar.

This causes a connection failure because:
1. The sample container (`aci-helloworld`) runs a lightweight web server listening strictly on **HTTP Port 80**.
2. There is no TLS/SSL certificate installed inside the container, nor is port 443 opened in the networking configuration.
3. When the browser sends an SSL handshake on port 443, Azure drops or refuses the connection.
4. Explicitly prefixing **`http://`** forces the browser to send an unencrypted HTTP request over Port 80, which the container successfully accepts.

```
[ Browser: http://... ]  ----(Port 80 TCP)----> [ Azure Public IP ] ----> [ ACI Container: Port 80 ] ✅
[ Browser: https://... ] ----(Port 443 TCP)---> [ Azure Public IP ] ----> [ Port 443 closed / No SSL ] ❌
```

> 💡 **Production Note:** In production, container instances are typically placed behind an **Azure Application Gateway** or **Azure Front Door** to terminate SSL certificates on port 443 and forward traffic to ACI on port 80.

---

### 2. Why do we refresh the page before checking logs?
* Under the hood, Azure Container Instances captures everything output to standard output (`stdout`) and standard error (`stderr`).
* When you open the website, the application prints a log line:
  `GET / - 200 OK`
* Refreshing the browser 3–4 times sends multiple HTTP `GET` requests.
* When you navigate to **Containers $\rightarrow$ Logs**, Azure displays these live access log entries, proving the container is actively processing real user traffic.

---

## 📌 Task 3: Clean Up Resources

* Because Azure Container Instances is billed on a per-second basis while running, deleting the resource group immediately terminates the container and stops billing.
