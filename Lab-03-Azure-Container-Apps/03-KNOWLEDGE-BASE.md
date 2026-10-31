# Lab 03 Knowledge Base: Azure Container Apps (ACA)

This Knowledge Base serves as the comprehensive theory and reference manual for students and instructors. If a student is unsure about the underlying concepts, architecture, or exam implications of Azure Container Apps, they can refer directly to this document.

---

## 📖 1. Master Azure Compute Comparison Matrix

This table summarizes the core differences between all four Azure compute options tested in the AZ-104 and AZ-204 exams:

| Feature | Azure Virtual Machines (IaaS) | Azure App Service (PaaS) | Azure Container Instances (ACI) | Azure Container Apps (ACA) |
| :--- | :--- | :--- | :--- | :--- |
| **Lab Reference** | *Skipped (IaaS)* | **Lab 01** | **Lab 02** | **Lab 03** |
| **Underlying Tech** | Hyper-V Virtual Machines | Hosted Application Container on VM | Lightweight Hyper-V isolated container | Managed Kubernetes (AKS) + KEDA + Envoy |
| **Startup Time** | Minutes | 1–2 minutes | Seconds | Seconds |
| **Ingress & SSL** | Manual IIS/Nginx setup | Built-in (Custom SSL bindings) | Raw IP/DNS (Port 80 only, no built-in SSL) | **Built-in Envoy with automatic managed SSL** |
| **Scale to Zero?** | ❌ No (Billed 24/7) | ❌ No (Plan billed 24/7) | ❌ No (Runs or stops) | **✅ YES (0 compute cost when idle)** |
| **Scaling Engine** | VM Scale Sets (VMSS) | Metric rules / Automatic burst | Manual container restart | **KEDA (HTTP requests, Queue depth, CPU/RAM)** |
| **Traffic Splitting** | Requires Traffic Manager / App Gateway | Deployment Slots (0–100%) | ❌ None | **Native Revision Splitting (Canary / Blue-Green)** |
| **Primary Use Case**| Full OS control, legacy apps | Web apps, monolithic APIs | Batch processing, one-time jobs | **Microservices, event-driven apps, container APIs** |

---

## 🔄 2. Revisions & Traffic Splitting in ACA

Every deployment or configuration update to a Container App creates an immutable snapshot known as a **Revision**.

```
                           [ Incoming User Traffic ]
                                      |
                           [ Envoy Ingress Proxy ]
                                   /      \
                             80%  /        \  20%
                                 v          v
                       [ Revision 1 (v1.0) ]  [ Revision 2 (v2.0) ]
```

### Revision Modes
1. **Single Revision Mode (Default):**  
   * Only one revision is active at any time.  
   * When a new revision is deployed, traffic automatically cuts over 100% to the new version with zero downtime.
2. **Multiple Revision Mode:**  
   * Allows multiple revisions to be active simultaneously.  
   * Perfect for **Canary Deployments** (e.g., routing 10% of users to a new feature release) or **A/B Testing**.

---

## ⚡ 3. Autoscaling in ACA (Powered by KEDA)

Azure Container Apps uses **KEDA (Kubernetes Event-driven Autoscaling)** behind the scenes, offering three distinct scaling triggers:

1. **HTTP Concurrency:** Scales instances based on the number of simultaneous HTTP requests (e.g., scale out when concurrent requests exceed 50 per replica).
2. **System Resources:** Scales based on CPU percentage or Memory usage thresholds.
3. **Event-Driven (KEDA Scalers):** Scales based on external event sources, such as:
   * Number of messages waiting in an **Azure Service Bus Queue**.
   * Consumer lag in an **Apache Kafka** topic.
   * Number of items in an **Azure Storage Queue**.
   * If the queue reaches 0 messages, ACA automatically **scales to 0**, saving 100% of compute cost.

---

## 🛠️ 4. Quick Reference: Azure CLI Cheatsheet for ACA

Keep this cheatsheet handy for rapid terminal operations:

```bash
# 1. Create a Resource Group
az group create --name <your-resource-group> --location eastus

# 2. Create a Container Apps Environment
az containerapp env create \
  --name <your-environment-name> \
  --resource-group <your-resource-group> \
  --location eastus

# 3. Create a Container App with Public Ingress
az containerapp create \
  --name <your-app-name> \
  --resource-group <your-resource-group> \
  --environment <your-environment-name> \
  --image mcr.microsoft.com/azuredocs/aci-helloworld:latest \
  --target-port 80 \
  --ingress external \
  --query properties.configuration.ingress.fqdn

# 4. View active Revisions and Traffic distribution
az containerapp revision list \
  --name <your-app-name> \
  --resource-group <your-resource-group> \
  --output table

# 5. Split traffic 80/20 between two Revisions
az containerapp ingress traffic set \
  --name <your-app-name> \
  --resource-group <your-resource-group> \
  --revision-weight <revision-1>=80 <revision-2>=20

# 6. Delete all lab resources
az group delete --name <your-resource-group> --yes --no-wait
```

---

## ❓ 5. Frequently Asked Questions (FAQ) & Exam Traps

#### Q1: What is the main difference between Azure Container Instances (ACI) and Azure Container Apps (ACA)?
**Answer:** ACI is designed for simple, standalone containers and batch jobs without built-in ingress, SSL management, or dynamic autoscaling. ACA is an enterprise microservice platform with built-in HTTPS ingress, service discovery, KEDA autoscaling, and traffic splitting across revisions.

#### Q2: How does Azure Container Apps handle SSL/TLS certificates?
**Answer:** Azure Container Apps automatically provisions and manages free public SSL/TLS certificates for all apps using external ingress under the `*.azurecontainerapps.io` domain. You do not need to buy or upload certificates.

#### Q3: What is "Scale to Zero" and does it cause a cold start?
**Answer:** "Scale to Zero" means when no requests or queue events are detected, ACA releases all container replicas so you pay $0 for compute. When a new request arrives, ACA boots a new replica. This introduces a slight latency penalty (cold start, typically 1–3 seconds) on the very first incoming request.

#### Q4: Can two container apps in the same environment communicate securely without public IPs?
**Answer:** **Yes!** By setting their ingress to *Limited to Container Apps Environment* (internal ingress), container apps can communicate privately over the shared environment network using their app names as hostnames (e.g., `http://inventory-service`).
