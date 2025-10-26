# de-project-pipeline-spotify

---

## Table of Contents

- [Project Overview](#-project-overview)
- [Project Objectives](#-project-objectives)
- [Create Azure Resources](#-create-azure-resources)
  - [1️⃣ Create a Resource Group](#1️⃣-create-a-resource-group)
  - [2️⃣ Create a Storage account](#2️⃣-create-a-storage-account)
  - [3️⃣ Create a Azure Data Factory](#3️⃣-create-a-azure-data-factory)
    -  [Link Azure Data Factory (ADF) to GitHub](#link-azure-data-factory-adf-to-github)
  - [4️⃣ Create a Azure SQL](#4️⃣-create-a-azure-sql)
    -  [Azure SQL Database](#azure-sql-database)

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
### 1️⃣ Create a Resource Group
1.  Sign in to Azure Portal [`Azure Portal`](https://portal.azure.com/)
2.  Search **Resource groups** → Create.
3.  Subscription: pick yours.
4.  Resource group: e.g., `de-project-pipeline-spotify`
5.  Region: e.g., `south-east-asia` (region where RG metadata resides).
6.  (Optional) Tags: e.g., `env=dev`
7.  Review + create → Create.
---
### 2️⃣ Create a Storage Account
A Storage Account in Azure is the foundation for storing data in the cloud — it’s like a container for our data services.

It provides access to multiple storage types under one account, such as:
-  Blob Storage – for unstructured data like files, images, videos, logs
-  File Storage – for shared file systems (SMB/NFS)
-  Queue Storage – for message queues between apps
-  Table Storage – for NoSQL key-value data

---

1. Go to Azure Portal → search **Storage accounts** → Create.
2. Subscription: Your subscription
3. Resource group: `de-project-pipeline-spotify`
4. Storage account name: `storagepipelinespotify`
5. Region: `south-east-asia`
6. Performance: `Standard`
7. Redundancy: `Locally-redundant storage (LRS)`
8. Hierarchical namespace: `Enabled` (this switches on ADLS Gen2)
9. (Optional) Blob access tier (default): `Hot`
10. Review + create → Create.
11. Create containers (data zones) in Storage Account
    -  Open the storage account → Containers → + Container
    -  Add: `bronze`, `silver`, `gold` → Create.

---

**Preferred Storage Type**
 | Feature | Azure Blob Storage | Azure Data Lake Storage Gen2 |
 |---------|--------------------|------------------------------|
 | Purpose | General-purpose storage for files, images, backups, and archives | Designed for big data analytics and data lake workloads |
 | Structure | Flat namespace (files live in a single level) | Hierarchical namespace (supports folders and subdirectories) |
 | Performance | Optimized for simple file operations (upload/download) | Optimized for analytical operations (rename, move, directory queries) |
 | Security & Access Control | Uses Azure RBAC (role-based access) at container level | Supports both Azure RBAC and POSIX-style ACLs (fine-grained folder/file access) |
 | Integration | Works well with general apps, web services, and storage SDKs | Fully integrated with big data tools (Databricks, Spark, Synapse, HDInsight) |
 | Cost | Slightly cheaper | Slightly higher due to hierarchical namespace |
 | Use Cases | File backups, static website hosting, archival storage | Data lakes, ETL pipelines, analytics workloads |
 | Enable Hierarchical Namespace? | ❌ Not supported | ✅ Required (enables folder structure and ACLs) |

---

**Redundancy (Replication Options)**
 | Type   | Full Name                      | Scope                        | Copies  | Description                                                                                                      |
|--------|--------------------------------|-------------------------------|----------|------------------------------------------------------------------------------------------------------------------|
| **LRS** | Locally Redundant Storage      | One region                    | 3 copies | Data is stored in a single datacenter within one region. Lowest cost, best for non-critical dev/test workloads.  |
| **ZRS** | Zone Redundant Storage         | One region (across zones)     | 3 copies | Data replicated across three availability zones in the same region for higher availability.                      |
| **GRS** | Geo-Redundant Storage          | Two regions                   | 6 copies | Data copied to a secondary region (hundreds of km away). Used for disaster recovery.                             |
| **RA-GRS** | Read-Access Geo-Redundant   | Two regions                   | 6 copies + read access | Same as GRS but allows read access to the secondary region. Useful for high availability and disaster recovery.   |

For most `dev/test` environments, `LRS (Locally-redundant storage)` is cost-effective.
For `production` and critical workloads, consider `ZRS` or `GRS`.

---


### 3️⃣ Create a Azure Data Factory
**Azure Data Factory (ADF)** is a cloud-based data integration and orchestration service.
It helps us build, schedule, and manage ETL/ELT pipelines that move and transform data between on-premises and cloud systems.

💡 Think of ADF as a pipeline engine that connects our data sources (Storage, SQL, APIs) to our data destinations (Data Lake, Synapse, BigQuery, etc.) — all with monitoring, triggers, and automation.

---

1. Go to Azure Portal → Search for **Data factories** → Click Create.
2. Subscription: Your subscription
3. Resource group: `de-project-pipeline-spotify`
4. name: `df-pipeline-spotify`
5. Region: `south-east-asia`
6. Review + create → Create.

---

#### Link Azure Data Factory (ADF) to GitHub
##### 1) Connect ADF to GitHub (ADF Studio)
1.  Open ADF Studio → Manage (gear icon).
2.  Select Git configuration → Configure.
3.  Choose repository type: `GitHub`
4.  Authorize ADF to access GitHub (OAuth prompt) → choose our GitHub account/org.
5.  Fill in the fields:
    -  **GitHub account**: Your GitHub user/org that owns the repo
    -  **Repository name**: e.g., `ecom-data-platform`
    -  **Collaboration branch**: `main`
    -  **Root folder**: `/adf`
    -  **Publish branch**: `adf_publish` (default, recommended)

##### 2) Day-to-Day Workflow (Branches)
1.  In ADF Studio (Git mode), create a feature branch:
     -  Top bar → Branch dropdown → New branch  (`feature/update_20251023_base`)
  
2.  Build/edit artifacts: **Pipelines**, **Dataset**s, **Linked services**, **Triggers**.
3.  **Save** (saves to feature branch in Git).
4.  Create `Pull Request` from feature branch → collaboration branch (`main`).
5.  Review & merge Pull Request in GitHub.
---

### 4️⃣ Create a Azure SQL
**What is Azure SQL?**

-  **Azure SQL** is a family of fully managed SQL Server–based services in Azure:
      -  **Azure SQL Database** → PaaS, a single database or elastic pool (most common for apps & analytics).
      -  **Azure SQL Managed** Instance → PaaS with near full SQL Server compatibility (instance-level features).
      -  **SQL Server on Azure VM** → IaaS VM with SQL Server (you manage OS/SQL).

          In this guide weare creating **Azure SQL Database** (a single DB) on a logical server.
         
---
#### Azure SQL Database
1.  Portal → search **Azure SQL** → + Create → **SQL databases** → Create.
2.  Subscription / Resource group: select (`project-pipeline-spotify`)
3.  Database name: `project-pipeline-spotify`
4.  Server: Create new
     -  Server name: `project-pipeline-spotify`
     -  Location: `Southeast Asia`
     -  Authentication: `Use both SQL and Microsoft Entra`
     -  Set Microsoft Entra admin: `pick user/group`
     -  Create SQL admin login/password: e.g., sqladmin
5.  Workload environment: `Development` (affects recommendations & defaults)
6.  Compute + Storage
     -  Click Configure database:
         -  Choose vCore → General Purpose (cost-effective)
         -  Provisioned (fixed vCores) or `Serverless` (auto-scale, auto-pause)
         -  Pick vCores, data max size (e.g., 5–32 GB for dev), and storage type
         -  Apply
      
7.  Networking
     -  Connectivity method: `Public endpoint`
  
8.  Backup storage redundancy
     -  Select `Locally-redundant backup storage (LRS)`
  
9.  Review + create → Create.

---

## Create Tables & Ingest Data (Initial Load) in Azure SQL Database
We use the SQL script below to create tables and insert seed data into **Azure SQL Database**.

📄 **Script Location:** [`scripts/initial_load.sql`](scripts/initial_load.sql)

### Steps
1. Open your Azure SQL Database → **Query editor (preview)**.
2. Sign in with Microsoft Entra or SQL login.
3. Copy the SQL code from the script above.
4. Paste and **Run** it.
5. Verify table creation with:
   ```sql
   SELECT
     COUNT(*)
   FROM 
     dbt.DimUser;
   ```

   
---
## Link Azure Data Factory (ADF) with Azure SQL Database

### What is a Linked Service?
In **Azure Data Factory (ADF)**, a Linked Service defines the connection information required for ADF to access external data sources such as Azure Storage, Azure SQL Database, REST APIs, etc.

-  Read data from sources
-  Write data to destinations
-  Authenticate securely using credentials or managed identities
    💡 In this step, we WILL connect ADF → Azure SQL Database using SQL Authentication.
---
### Why Use SQL Authentication?
During your Azure SQL Database setup, you enabled both SQL and  **Microsoft Entra (Azure AD) authentication**

| Authentication Type | Description | Use Case |
|----------------------|-------------|-----------|
| **Microsoft Entra (Azure AD)** | Authenticates using Azure Active Directory (no passwords, more secure). | Production or enterprise setup |
| **SQL Authentication** | Authenticates using a username and password (`sqladmin`). | Development or quick testing |

✅ In this project, we use SQL Authentication for simplicity during development.

---
#### Steps to Create Linked Service in ADF (SQL Authentication)
1. Open Azure Data Factory Studio
   -  Go to Azure Portal [`Azure Portal`](https://portal.azure.com/)
   -  Open Data Factory
   -  Click Open **Azure Data Factory Studio**
  
2. Navigate to Manage
   -  In the left sidebar, click the ⚙️ Manage icon.
   -  Under **Connections**, select **Linked services**.
   -  Click + New to create a **new linked service**.
  
3. Choose Data Store
   -  In the New Linked Service panel, search for `Azure SQL Database`.
   -  Select it and click Continue.



---















