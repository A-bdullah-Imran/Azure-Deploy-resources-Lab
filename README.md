# Azure Administrator (AZ-104) Hands-on Labs: Deploy and Manage Compute Resources

Welcome to the hands-on lab repository for the **AZ-104: Deploy and Manage Azure Compute Resources** learning path. 

This repository is curated by instructors to provide student-tested, industry-ready lab walkthroughs. Each lab contains complete step-by-step guides, in-depth architectural explanations, and a comprehensive knowledge base with real-world scenarios and AZ-104 exam tips.

---

## 📚 Lab Curriculum (PaaS & Serverless Compute)

> *Note: In accordance with our syllabus focus, standard Virtual Machine (IaaS) infrastructure is skipped in favor of modern PaaS and Serverless Compute patterns.*

| Lab # | Lab Title | Core Technologies | Status | Directory |
| :--- | :--- | :--- | :--- | :--- |
| **Lab 01** | **Implement Azure App Services** | App Service Plans, Deployment Slots, Autoscaling | ✅ Ready | [`Lab-01-Azure-App-Services/`](./Lab-01-Azure-App-Services/) |
| **Lab 02** | **Implement Azure Container Instances (ACI)** | Serverless Containers, Docker Images, Public DNS | 🔄 In Progress | [`Lab-02-Azure-Container-Instances/`](./Lab-02-Azure-Container-Instances/) |
| **Lab 03** | **Implement Azure Container Apps (ACA)** | Microservices, Container Apps Environment, Ingress | ⏳ Upcoming | [`Lab-03-Azure-Container-Apps/`](./Lab-03-Azure-Container-Apps/) |

---

## 📁 Three-Tier Documentation Structure per Lab

To ensure students both **execute** and **understand** every concept, each lab folder contains three dedicated guides:

1. **`01-LAB-GUIDE.md` (Task Execution Guide)**  
   * Complete, numbered click-by-click instructions in the Azure Portal.
   * Includes tested instructor workarounds, edge-case troubleshooting (e.g., Kudu 503 deployment errors, CLI fallbacks), and UI tips.
2. **`02-TASK-EXPLANATIONS.md` (Architectural & Operational Rationale)**  
   * Deep dive into *why* each configuration is chosen (e.g., why Standard/Premium tiers are required for slots, how slot swap works behind the scenes, why warm-up requests prevent downtime).
3. **`03-KNOWLEDGE-BASE.md` (Theory, FAQs & Exam Preparation)**  
   * Core theory, architectural diagrams, AZ-104 exam traps, common student questions, and interview flashcards.

---

## 🛠️ Lab Prerequisites

1. An active **Microsoft Azure Subscription** (Free Trial, Student, or Pay-As-You-Go).
2. Role Assignment: **Contributor** or **Owner** at the Resource Group or Subscription level.
3. Access to **Azure Cloud Shell** (PowerShell or Bash) directly in the Azure Portal.
