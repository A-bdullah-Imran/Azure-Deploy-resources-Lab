# Lab 03: Task-by-Task Deep Explanations

This document provides instructor-level technical context explaining **what** we are doing in each step of Lab 03, **why** each parameter is configured, and **what happens behind the scenes** inside Microsoft Azure.

---

## 📌 Task 1: Creating an Azure Container App and Environment

### 1. What is Azure Container Apps (ACA)?
**Azure Container Apps** is a fully managed, serverless container platform designed specifically for **microservices**, **web apps**, and **background event-driven jobs**.
* **Under the Hood:** ACA is built on top of **Azure Kubernetes Service (AKS)**, **KEDA** (Kubernetes Event-driven Autoscaling), **Envoy Proxy**, and **Dapr** (Distributed Application Runtime).
* **The Value Proposition:** You gain all the power of enterprise Kubernetes (traffic splitting, zero-downtime rolling upgrades, microservice discovery, autoscaling) without ever touching Kubernetes manifest YAML files, nodes, or control plane updates.

---

### 2. What is a Container Apps Environment?
When creating a Container App in Step 4, Azure provisions a **Container Apps Environment**:
* The **Environment** acts as a secure boundary around a collection of related container apps (microservices).
* **Shared Virtual Network:** Container apps deployed within the same environment are placed on the same virtual network.
* **Internal Service Discovery:** Microservices in the same environment can call each other directly by name (e.g., `http://order-service` calling `http://inventory-service`) without traversing the public internet.
* **Shared Log Analytics:** All logs and telemetry across all containers in the environment automatically stream to a single centralized Azure Log Analytics workspace.

```
+-------------------------------------------------------------------------+
|                  Container Apps Environment: abdullah-env                |
|                                                                         |
|   +--------------------------+          +--------------------------+    |
|   | Container App: Frontend  |          | Container App: Backend   |    |
|   | (External Ingress: 443)  | -------> | (Internal Ingress only)  |    |
|   +--------------------------+          +--------------------------+    |
|                \                                     /                  |
|                 \                                   /                   |
|                  +---------------------------------+                    |
|                  | Centralized Log Analytics Worksp|                    |
+-------------------------------------------------------------------------+
```

---

### 3. Understanding Ingress Settings

#### A. Ingress: Enabled vs. Disabled
* **Enabled (External):** The app is assigned a publicly reachable FQDN accessible over the Internet. Ideal for frontends and public APIs.
* **Limited to Container Apps Environment (Internal):** The app is only reachable by other container apps inside the same environment. Ideal for private backend services, databases, and message processors.

#### B. Ingress Type & Target Port (Target Port 80)
* **Target Port:** The port inside the container that your code or web server listens on.
* For the sample Hello World image, the internal Node.js/Alpine server listens on **Port 80**.
* **Envoy Reverse Proxy:** Azure’s edge proxy receives incoming external HTTPS traffic on port 443 and automatically forwards it to the container’s internal **Target Port 80**.

---

## 📌 Task 2: Testing & Verification (The HTTPS Advantage)

### 1. Why does Container Apps support HTTPS out of the box?
In **Lab 02 (ACI)**, navigating to `https://` failed because ACI provides raw compute without an ingress proxy.

In **Lab 03 (Container Apps)**:
1. Every Container App receives a managed hostname under `*.azurecontainerapps.io`.
2. Azure automatically provisions, binds, and renews a free, trusted public **TLS/SSL certificate** for your domain.
3. The built-in **Envoy Proxy** handles TLS termination at the edge, decrypts the request, and forwards it to your container over the private network.
4. Users get an enterprise-grade secure connection (`https://`) with zero certificate management!

---

### 2. Revisions and Scale-to-Zero Capability
While not explicitly drilled in this introductory lab, students should know the two superpowers of Azure Container Apps:
1. **Revisions (Blue/Green & Canary Deployments):** Each time you update container code or configuration, Azure creates an immutable snapshot called a **Revision**. You can split traffic between Revisions (e.g., 80% to v1, 20% to v2).
2. **Scale to Zero:** Unlike Azure App Service Plans which bill you continuously 24/7, Container Apps can scale down to **0 active replicas** when there is no incoming traffic, resulting in **$0 compute charges** during idle periods!

---

## 📌 Task 3: Clean Up Resources

* Deleting the resource group removes the Container App, the Container Apps Environment, and the linked Log Analytics workspace in one automated operation, completely preventing any ongoing charges.
