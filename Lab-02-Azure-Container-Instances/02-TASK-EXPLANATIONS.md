# Lab 02: Task-by-Task Deep Explanations

This document provides instructor-level technical context explaining **what** we are doing in each step of Lab 02, **why** each parameter is configured, and **what happens behind the scenes** inside Microsoft Azure.

---

## 📌 Task 1: Deploying an Azure Container Instance (ACI)

### 1. What is Azure Container Instances (ACI)?
**Azure Container Instances (ACI)** is Microsoft’s serverless container compute offering.
* **No VM Management:** You do not manage any underlying virtual machines, host operating systems, or patching.
* **Instant Startup:** Because the underlying hypervisors are pre-warmed by Azure, containers launch within seconds.
* **Per-Second Billing:** You only pay for the exact CPU cores and Gigabytes of RAM consumed while the container is in the `Running` state. When stopped, billing halts.

### 2. Container Compute Comparison (When to use what?)
| Service | Provisioning Time | Orchestration / Scaling | Best For |
| :--- | :--- | :--- | :--- |
| **Azure Virtual Machines** | Minutes | Manual / Scale Sets | Legacy apps, full OS access, custom kernel drivers |
| **Azure App Service** | 1–2 Minutes | Built-in autoscale | Standard web apps, APIs, staging slots |
| **Azure Container Instances (ACI)** | **Seconds** | **Single container / basic tasks** | **Quick batch jobs, simple web servers, CI/CD runners** |
| **Azure Container Apps (ACA)** | Seconds | KEDA event-driven autoscale | Microservices, API ingress, Dapr integration |
| **Azure Kubernetes Service (AKS)** | Minutes | Enterprise K8s orchestration | Complex multi-tier microservices, large clusters |

---

### 3. Understanding Container Configuration Parameters

#### A. Image Sources
* **Quickstart Images:** Curated demo images hosted by Microsoft in the **Microsoft Container Registry (`mcr.microsoft.com`)**.
* **Docker Hub:** Public or private images from Docker Hub.
* **Azure Container Registry (ACR):** Private, enterprise-grade Docker image registries managed inside your own Azure tenant with Role-Based Access Control (RBAC).

#### B. Restart Policy (AZ-104 Exam Trap!)
When creating a container, you choose one of three restart policies:
1. **Always (Default):** The container will always restart if it crashes or stops. Use this for 24/7 web servers or background daemon listeners.
2. **On failure:** The container restarts *only* if the process returns a non-zero exit code (error). If the job finishes successfully (exit code `0`), it stays stopped. Use this for scheduled batch jobs or data processing.
3. **Never:** The container runs once and never restarts, regardless of whether it succeeded or failed. Use this for run-once diagnostics or initialization scripts.

---

## 📌 Task 2: Testing Functionality & The HTTP vs. HTTPS Trap

### 1. Why did the browser fail with HTTPS?
When you first navigated to your FQDN, your browser likely tried to access:
`https://<your-dns-label>.<region>.azurecontainer.io`

This failed because:
1. The demo container (`aci-helloworld`) runs a lightweight Node.js/Alpine web server listening strictly on **HTTP Port 80**.
2. There is no TLS/SSL certificate installed inside the container, nor is port 443 exposed.
3. Modern web browsers (Edge, Chrome, Brave) enforce automatic HTTPS upgrades. When the browser sends an SSL handshake on port 443, Azure drops or refuses the connection.
4. Explicitly prefixing **`http://`** forces the browser to send an unencrypted HTTP request over Port 80, which the container successfully accepts.

```
[ Browser: http://... ]  ----(Port 80 TCP)----> [ Azure Public IP ] ----> [ ACI Container: aci-helloworld ] ✅
[ Browser: https://... ] ----(Port 443 TCP)---> [ Azure Public IP ] ----> [ Port 443 closed / No SSL ]      ❌
```

> 💡 **Production Solution:** In real-world enterprise deployments, containers in ACI are typically placed behind an **Azure Application Gateway**, **Azure Front Door**, or a reverse proxy (like NGINX with Let's Encrypt) to terminate SSL certificates on port 443.

---

### 2. How is the Public FQDN Built?
Azure constructs the FQDN using this deterministic formula:
`[DNS-Name-Label].[Azure-Region].azurecontainer.io`
* If your label is `abdullah-aci-222` and region is `eastus`, your hostname is `abdullah-aci-222.eastus.azurecontainer.io`.
* Because all containers in that region share the `.azurecontainer.io` zone, your label must be globally unique within that Azure region.

---

## 📌 Task 3: Inspecting Logs, Events, and Interactive Console

### 1. How Azure Logs Work
* Under the hood, Azure Container Instances captures everything the container application outputs to standard output (`stdout`) and standard error (`stderr`).
* When you refresh the webpage in your browser, the Node.js server prints:
  `GET / - 200 OK`
* ACI automatically pipes this stream to the **Logs** tab in the Azure Portal.

### 2. Events vs. Logs
* **Events:** Relate to the **Azure Container Infrastructure** lifecycle (e.g., allocating compute, downloading the container image layer by layer, starting the Docker daemon).
* **Logs:** Relate to the **application code running inside the container** (e.g., HTTP request traces, database connections, application exceptions).

### 3. What happens in the "Connect" Tab?
When you click **Connect** with `/bin/sh`:
* Azure opens a secure WebSocket session directly to the container runtime host.
* It executes `docker exec -it <container_id> /bin/sh` behind the scenes.
* This gives you a live terminal prompt directly inside the isolated container file system, allowing you to run diagnostic commands like:
  * `ps aux`: Shows all isolated processes running inside the Linux PID namespace.
  * `cat /etc/os-release`: Shows the stripped-down Linux distribution (Alpine Linux v3.9).
  * `ls -la`: Displays the application root directory (`/usr/src/app`).
