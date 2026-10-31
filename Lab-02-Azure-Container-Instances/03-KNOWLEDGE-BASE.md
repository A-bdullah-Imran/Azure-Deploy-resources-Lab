# Lab 02 Knowledge Base: Azure Container Instances (ACI)

This Knowledge Base serves as the comprehensive theory and reference manual for students and instructors. If a student is unsure about the underlying concepts, architecture, or exam implications of Azure Container Instances, they can refer directly to this document.

---

## 📖 1. Core Architectural Concepts

### What is a Container? (Containers vs. Virtual Machines)
* **Virtual Machines (IaaS):** Each VM virtualizes the physical hardware and runs a complete, heavy guest operating system with its own kernel, device drivers, and system services. Startup takes minutes.
* **Containers (PaaS):** Containers share the host machine’s operating system kernel. They package only the application code, runtime, and dependencies into an isolated user-space sandbox. Startup takes seconds.

```
+------------------------------------+   +------------------------------------+
|          Virtual Machine           |   |             Container              |
|  +------------------------------+  |   |  +------------------------------+  |
|  |       Application Code       |  |   |  |       Application Code       |  |
|  +------------------------------+  |   |  +------------------------------+  |
|  |     Binaries & Libraries     |  |   |  |     Binaries & Libraries     |  |
|  +------------------------------+  |   |  +------------------------------+  |
|  |   Full Guest OS (GBs size)   |  |   |  |  Container Engine (Docker)   |  |
|  +------------------------------+  |   +------------------------------------+
|  |          Hypervisor          |  |   |       Host Operating System        |
+------------------------------------+   +------------------------------------+
```

---

### What is a Container Group in ACI? (AZ-104 Exam Essential!)
The top-level resource in Azure Container Instances is a **Container Group**.
* A container group is a collection of one or more containers that are scheduled on the same physical host machine.
* The containers in a container group share:
  1. **Lifecycle:** They are created, started, stopped, and destroyed together.
  2. **Local Network:** They share an IP address and port namespace. They can talk to each other over `localhost` on different ports.
  3. **Storage Volumes:** They can mount and share the same Azure Files storage volumes.
* 💡 **Concept Bridge:** A Container Group in ACI is identical in concept to a **Pod** in Kubernetes!

> ⚠️ **Exam Trap:** You **cannot** create a multi-container group via the Azure Portal GUI. The portal only supports deploying a single container. Multi-container groups **must** be deployed using an **ARM Template**, **Bicep file**, or a **YAML specification**.

---

## 💾 2. Persistent Storage in ACI (Azure Files Mount)

By default, container storage is **ephemeral (stateless)**. When an ACI instance is restarted or deleted, any data written to the local disk is permanently erased.

To retain stateful data (e.g., database files, user uploads), Azure Container Instances allows you to mount an **Azure Files Share** directly into the container filesystem:

```bash
# Mounting an Azure File Share into ACI
az container create \
  --resource-group <your-resource-group> \
  --name <container-name> \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --azure-file-volume-account-name <storage-account-name> \
  --azure-file-volume-account-key <storage-account-key> \
  --azure-file-volume-share-name <fileshare-name> \
  --azure-file-volume-mount-path /data
```

Supported Volume Types in ACI:
1. **Azure File Share:** Persistent read/write remote SMB share.
2. **`emptyDir`:** Temporary scratchpad directory shared between containers in the same group.
3. **`secret`:** Mounts base64-encoded sensitive keys or passwords into memory.
4. **`gitRepo`:** Clones a Git repository into a local directory upon container startup.

---

## 🔒 3. Networking & Security in ACI

### Public IP vs. Virtual Network (VNet) Integration
* **Public Deployment (Default):** Azure assigns a public IP address and optional DNS label (`*.azurecontainer.io`). Ports are exposed to the public Internet.
* **VNet Deployment (Private):** You can inject an ACI container group directly into a private subnet within an Azure Virtual Network.
  * The subnet must be **delegated** exclusively to `Microsoft.ContainerInstance/containerGroups`.
  * The container receives a private IP address (e.g., `10.0.1.4`) and is completely isolated from the public internet.

---

## 🛠️ 4. Quick Reference: Azure CLI Cheatsheet for ACI

Keep this cheatsheet handy for rapid terminal operations:

```bash
# 1. Create a public ACI container with DNS label
az container create \
  --resource-group <your-resource-group> \
  --name <container-name> \
  --image mcr.microsoft.com/azuredocs/aci-helloworld:latest \
  --dns-name-label <your-dns-label> \
  --ports 80 \
  --restart-policy Always

# 2. View live streaming container logs
az container logs \
  --resource-group <your-resource-group> \
  --name <container-name> \
  --follow

# 3. Execute an interactive bash shell inside the container
az container exec \
  --resource-group <your-resource-group> \
  --name <container-name> \
  --exec-command "/bin/sh"

# 4. View container status, FQDN, and IP address
az container show \
  --resource-group <your-resource-group> \
  --name <container-name> \
  --query "{Status:containers[0].instanceView.currentState.state, FQDN:ipAddress.fqdn, IP:ipAddress.ip}" \
  --output table

# 5. Stop and Delete the container instance
az container stop --resource-group <your-resource-group> --name <container-name>
az container delete --resource-group <your-resource-group> --name <container-name> --yes
```

---

## ❓ 5. Frequently Asked Questions (FAQ) & Exam Traps

#### Q1: Why did navigating to my container via HTTPS fail?
**Answer:** The container image (`aci-helloworld`) only listens on HTTP Port 80. Modern web browsers automatically attempt to connect via `https://` (Port 443). Because no SSL certificate is bound to port 443, the connection is refused. You must explicitly enter `http://` in the browser address bar.

#### Q2: Can I resize or change the CPU/RAM of an existing running container?
**Answer:** **No.** Container instance hardware allocations (CPU and RAM) are immutable once deployed. To change resources, you must delete and redeploy the container group with new specifications.

#### Q3: What happens to an ACI container configured with `RestartPolicy: OnFailure` when it completes successfully?
**Answer:** When the process inside the container exits with exit code `0` (Success), Azure transitions the container into the `Terminated` state and **stops billing you**. It will not restart. If the process crashes with a non-zero exit code, ACI will automatically reboot it.

#### Q4: When should I choose Azure Container Apps (ACA) over Azure Container Instances (ACI)?
**Answer:** Choose **ACI** for simple, isolated, single-run workloads (batch processing, data ETL jobs, quick tests). Choose **Azure Container Apps (ACA)** when you need microservices, automatic scale-to-zero, ingress traffic splitting, and built-in background event scaling (KEDA).
