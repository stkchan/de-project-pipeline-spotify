# de-project-pipeline-spotify

---

## Table of Contents

- [Project Overview](#-project-overview)
- [Project Objectives](#-project-objectives)
- [Create Azure Resources](#-create-azure-resources)
  - [1️⃣ Create a Resource Group](#1️⃣-create-a-resource-group)
  - [2️⃣ Create a Storage account](#2️⃣-create-a-storage-account)
  - [3️⃣ Create a Azure Data Factory](#3️⃣-create-a-azure-data-factory)

---

## Project Overview
will write later

---

## Project Objectives
Nothing. Just want to learn more about Azure Service such as Data Factory, Azure SQL, Databricks ans so on.

---

## Create Azure Resources
A Resource Group (RG) is a logical container in Azure that holds related resources (VMs, storage accounts, databases, etc.). It helps us deploy, manage, secure, tag, monitor, and delete resources as a unit.
-  Scope for RBAC (permissions), policies, locks, tags, and cost tracking
-  Resources in a group can be in different regions, but the RG’s location stores its metadata
-  Deleting an RG deletes all resources inside
---
### Create a Resource Group
1.  Sign in to Azure Portal [`Azure Portal`](https://portal.azure.com/)
2.  Search **Resource groups** → Create.
3.  Subscription: pick yours.
4.  Resource group: e.g., `de-project-pipeline-spotify`.
5.  Region: e.g., `south-east-asia` (region where RG metadata resides).
6.  (Optional) Tags: e.g., `env=dev`
7.  Review + create → Create.
---
### Create a Storage Account
A Storage Account in Azure is the foundation for storing data in the cloud — it’s like a container for your data services.

It provides access to multiple storage types under one account, such as:
-  Blob Storage – for unstructured data like files, images, videos, logs
-  File Storage – for shared file systems (SMB/NFS)
-  Queue Storage – for message queues between apps
-  Table Storage – for NoSQL key-value data

1. Go to Azure Portal → search **Storage accounts** → Create.
2. Subscription: Your subscription
3. Resource group: rg-ecom-dev-sea
4. Storage account name: stdevecomsea001
5. Region: Southeast Asia
6. Performance: Standard
7. Redundancy: Locally-redundant storage (LRS)
8. Hierarchical namespace: Enable ✅ (this switches on ADLS Gen2)
9. (Optional) Blob access tier (default): Hot


---






















