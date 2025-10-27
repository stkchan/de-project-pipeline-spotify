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
- [Create Tables & Ingest Data (Initial Load)](#create-tables--ingest-data-initial-load-in-azure-sql-database)
- [Link Azure Data Factory (ADF) with Azure SQL Database](#link-azure-data-factory-adf-with-azure-sql-database)
  - [What is a Linked Service?](#what-is-a-linked-service)
  - [Why Use SQL Authentication?](#why-use-sql-authentication)
  - [Steps to Create Linked Service in ADF (SQL Authentication)](#steps-to-create-linked-service-in-adf-sql-authentication)
    - [⚠️ Best Practices](#️-best-practices) 
  
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
  
4. Configure Connection Details
    | Field | Example Value | Description |
    |--------|----------------|-------------|
    | **Name** | `ls_azure_sql_pipeline_spotify` | Logical name for the linked service |
    | **Connect via Integration Runtime** | `AutoResolveIntegrationRuntime` | Default integration runtime used by ADF |
    | **Server name** | `project-pipeline-spotify.database.windows.net` | The fully qualified name of your Azure SQL Server |
    | **Database name** | `project-pipeline-spotify` | The name of your Azure SQL Database |
    | **Authentication type** | `SQL Authentication` | Use SQL credentials for authentication |
    | **User name** | `sqladmin` | SQL admin username created during setup |
    | **Password** | `********` | The password for the SQL admin account |
    | **Encrypt connection** | ✅ Enabled | Ensures encrypted data transmission between ADF and SQL |
    | **Trust server certificate** | ❌ Disabled | Recommended for secure connections (prevents untrusted certs) |

5. Test Connection
   -  Click Test connection.
   -  If successful ✅, click Create.
   -  If failed ❌, check:
       -  SQL Server firewall allows access from ADF.
       -  Server name and credentials are correct.
       -  Your SQL Server allows Azure services to connect.
    
6. Create the Linked Service
   -  Click Create once the test connection succeeds.
   -  ADF will now store your connection securely, allowing it to:
       -  Read from or write to your Azure SQL Database
       -  Authenticate using SQL username and password
    
7. Verify and Use in Pipelines
   -  Go to Author → Datasets → + New Dataset.
   -  Choose Azure SQL Database as your data store.
   -  Select the Linked Service that just created (`ls_azure_sql_pipeline_spotify`).
   -  Choose the desired table (`dbo.user`) for data import/export.
   -  Use this dataset as a `source` or `sink` in pipeline.
  

##### ⚠️ Best Practices
-  Use `Microsoft Entra (Managed Identity)` for production environments.
-  Restrict firewall access on your SQL Server — only allow Azure services or specific IPs.
-  Store credentials in Azure Key Vault instead of plain text.
-  Always test connection after deployment or password rotation.

---

## Incremental Ingestion Pipeline

### Link service with Azure Data Lake Storage Gen2
1. **Open ADF Studio**
   - Azure Portal → Data Factory (`de-pipeline-spotify`) → **Open Azure Data Factory Studio**

2. **Go to Manage → Linked services**
   - Left sidebar → **Manage (⚙️)** → **Linked services** → **+ New**

3. **Choose Connector**
   - Search **“Azure Data Lake Storage Gen2”** → **Continue**

4. **Configure Connection**
   - **Name:** `<name of link service>`
   - **Connect via Integration Runtime:** `AutoResolveIntegrationRuntime`
   - **Authentication type:** `Account key`
   - **Account selection method:** `From Azure subscription`
   - **Azure subscription:** `<subscription>`
   - **Storage account name:** `< name of storage account>`
   - **Test connection:** `To linked service`
   - **Create**
  
#### Quick Verify
After creating the linked service, create a quick dataset to validate:

1. **Author → Datasets → + New dataset**
2. **Azure Data Lake Storage Gen2** → **Format**: `DelimitedText` (or `Parquet/JSON`)
3. **Linked service:** `<name of link service>`
4. **File path:** `bronze/` *(or any existing container/folder)*  
5. **OK** → **Preview data** (if path/file exists) → ✅

#### Authentication Methods for ADLS Gen2 — When to Use What
| Method | What it uses | Pros | Cons | Typical Use Case |
|--------|--------------|------|------|------------------|
| **Account key** | Storage account access key | Simple to set up; works everywhere | Key secrecy/rotation burden; coarse-grained access | Dev/test, same team controls storage; quick POCs |
| **Managed Identity (Microsoft Entra)** | ADF’s Managed Identity + RBAC | No secrets; least-privilege via RBAC; rotation-free | Needs RBAC setup on Storage | **Recommended for prod**; enterprise posture |
| **Service Principal (Client Secret/Cert)** | App registration + secret/cert | Fine-grained app identity; multi-env CI/CD friendly | Secret/cert rotation; extra setup | CI/CD pipelines, cross-subscription access |
| **SAS Token** | Time-scoped, permission-scoped token | Scoped & time-limited access | Token lifecycle/rotation | Temporary or delegated access (partners/jobs) |

> **Recommendation:** Use **Managed Identity** for production (assign *Storage Blob Data Contributor* or tighter roles to ADF’s identity at the storage scope).

---









---















