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
- [Incremental Ingestion Pipeline](#incremental-ingestion-pipeline)
  - [Link service with Azure Data Lake Storage Gen2](#link-service-with-azure-data-lake-storage-gen2)
  - [Create Pipeline & Parameters](#create-pipeline--parameters)
  - [Datasets (Reusable)](#datasets-reusable)
  - [Activities (in order)](#activities-in-order)
  - [Debug / Test](#debug--test)
  - [Pipeline Summary — `incremental_ingestion`](#pipeline-summary--incremental_ingestion)
    - [Pipeline Logic Overview](#pipeline-logic-overview)
    - [Flow Summary](#flow-summary)
    - [Source and Sink Overview](#source-and-sink-overview)
        - [What is Source?](#what-is-source)
        - [What is Sink?](#what-is-sink)
        - [How Watermarking Works](#how-watermarking-works)

  
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

### Create Pipeline & Parameters
**Pipeline name:** `incremental_ingestion`

---

#### Flow Pipeline Overview:
![Flow Pipeline Overview](https://github.com/stkchan/de-project-pipeline-spotify/blob/c30abbbaf350eb82669b83cde9802b372c7636aa/images/flow_incremental.png)

---


#### 1) Pipeline Parameters
Create **pipeline parameters** (Author → Pipeline → Parameters):

| Name | Type | Example | Purpose |
|---|---|---|---|
| `schema` | String | `dbo` | Schema of source table |
| `table` | String | `DimUser` | Source table |
| `cdc_col` | String | `updated_at` | Watermark column (datetime/datetimestamp) |

#### 2) Pipeline Variables
Create a **variable** (Pipeline → Variables):

| Name | Type | Initial Value | Purpose |
|---|---|---|---|
| `current` | String | *(blank)* | Holds `utcNow()` for dynamic output filename |

---

### Datasets (Reusable)
Create two reusable datasets with **parameterized paths**:

#### 1) `json_dynamic` (ADLS Gen2 → JSON)
- **Type:** Azure Data Lake Storage Gen2, **Format:** JSON
- **Linked service:** your ADLS LS (`ls_adlsgen2_storagepipelinespotify`)
- **Parameters**:  
  - `container` (String)  
  - `folder` (String)  
  - `file` (String)
- **File path** (use parameters):
```bash
Container: @dataset().container
Directory: @dataset().folder
File: @dataset().file
```

#### 2) `parquet_dynamic` (ADLS Gen2 → Parquet)
- **Type:** Azure Data Lake Storage Gen2, **Format:** Parquet
- **Linked service:** your ADLS LS
- **Parameters**:  
- `container` (String)  
- `folder` (String)  
- `file` (String)
- **File path** (use parameters):
```bash
Container: @dataset().container
Directory: @dataset().folder
File: @dataset().file
```
---
### Activities (in order)
#### 1) **Lookup** — `last_cdc`
Purpose: read current watermark from `bronze/cdc/cdc.json`.

- **Source dataset:** `json_dynamic`
- **Dataset properties:**
```bash
container = bronze
folder = cdc
file = cdc.json
```
- **First row only:** ✅ enabled

**Output shape (example):**
```json
{ "firstRow": { "cdc": "1900-01-01" } }
```
---

#### 2) **Copy data** — `AzureSQLToLake`
Purpose: query Azure SQL only rows newer than watermark and write to ADLS (Parquet).
-  Source:
    -  Linked service: your Azure SQL LS
    -  Use query: ✅
    -  Query (dynamic with parameters & lookup):
        ```sql
        SELECT
            *
        FROM
            @{pipeline().parameters.schema}.@{pipeline().parameters.table}
        WHERE
            @{pipeline().parameters.cdc_col} > '@{activity('last_cdc').output.firstRow.cdc}'
          ```
      > **Notes:
      > -  Ensure cdc_col is a datetime type in SQL.
      > -  If using datetimeoffset, adjust casting accordingly.

-  Sink:
    -  Sink dataset: `parquet_dynamic`
    -  Dataset properties (dynamic):
        ```ini
        container = bronze
        folder    = @pipeline().parameters.table
        file      = @concat(pipeline().parameters.table,'_',variables('current'),'.parquet')
        ```
    -  Compression: Snappy (default for Parquet) or as needed
    -  Copy behavior: default (append)
-  Mappings: Auto (optional explicit mapping for schema evolution)
Dependency: set `AzureSQLToLake` to run after → `last_cdc`.
   
---       
    
#### 3) **Set variable** — `current_timestamp`
Purpose: capture a consistent timestamp for output file naming.
-  Variable: `current`
-  Value (dynamic):
    ```java
    @utcNow()
    ```
Dependency: run after → AzureSQLToLake (or before if you want the timestamp fixed prior to copy).

---  

#### 4) **If Condition** — `If_Incremental_Data`
Purpose: run CDC update only when rows were ingested.
-  Expression:
    ```kotlin
    @greater(activity('AzureSQLToLake').output.dataRead, 0)
    ```
Put the following two activities inside the True branch:

---

##### 4.1) **Script** — `max_cdc`
Purpose: compute MAX(cdc_col) for the just-ingested slice.
-  Linked service: Azure SQL LS
-  Script (query):
    ```sql
    SELECT
        MAX(@{pipeline().parameters.cdc_col}) AS cdc
    FROM
        @{pipeline().parameters.schema}.@{pipeline().parameters.table}
    ```

    > This returns a single row with the latest CDC value.
    > You’ll reference it in the next activity’s Additional columns.
    
---

##### 4.2) **Copy data** — `update_last_cdc`
Purpose: write the new watermark back to `bronze/cdc/cdc.json`.
-  Source:
    -  Dataset: `json_dynamic`
    -  Dataset properties:
        ```ini
        container = bronze
        folder    = cdc
        file      = empty.json
        ```
    -  Additional columns:
        -  Column name: `cdc`
        -  Value (dynamic):
            ```kotlin
            @activity('max_cdc').output.resultSets[0].rows[0].cdc
            ```
    -  Sink:
        -  Dataset: `json_dynamic`
        -  Dataset properties:
            ```ini
            container = bronze
            folder    = cdc
            file      = cdc.json
            ```
        -  Copy behavior: Overwrite (ensure the sink allows overwriting)
          
Dependency: `update_last_cdc` after → `max_cdc`.

---

#### 5) **Delete** — `DeleteEmptyFile`
Purpose: clean up an unwanted file if the previous run produced an empty artifact.
-  Dataset: parquet_dynamic
-  Dataset properties:
    ```ini
    container = bronze
    folder    = @pipeline().parameters.table
    file      = @concat(pipeline().parameters.table,'_',variables('current'))
    ```
-  Logging settings: enable if required for audit
-  Dependency: Typically after → `If_Incremental_Data` (or after `AzureSQLToLake` if you only want to delete when zero rows).

    > 💡 Alternatively, you can Conditionally delete the file when dataRead == 0:
    > Wrap `DeleteEmptyFile` in a separate If Condition with:
      ```kotlin
      @equals(activity('AzureSQLToLake').output.dataRead, 0)
      ```
---
### Debug / Test
Use Debug with:
-  schema = dbo
-  table = DimUser
-  cdc_col = updated_at

**Expected**:
-  First run ingests all rows with `updated_at > '1900-01-01'`, writes Parquet:
    ```makefile
    bronze/DimUser/DimUser_2025-10-27T01:23:45Z.parquet
    ```
-  `cdc.json` is updated to **MAX(updated_at)** from `DimUser` only if rows were read.
-  Next run ingests only rows with `updated_at` **greater** than the stored `cdc`.

---

### Pipeline Summary — `incremental_ingestion`
This Incremental Ingestion Pipeline in Azure Data Factory (ADF) automates extracting only new or changed rows from `Azure SQL Database` table and landing them into `Azure Data Lake Storage Gen2 (ADLS)` as `Parquet files`.

It uses a watermarking mechanism — tracking the latest `updated_at timestamp` so that each pipeline run only loads rows newer than the last ingestion.

#### Pipeline Logic Overview
| Step       | Activity                    | Description                                           |
|------------|-----------------------------|-------------------------------------------------------|
| **1️⃣** | **Lookup – `last_cdc`**                  | Reads the current watermark value (`cdc`) from `bronze/cdc/cdc.json` in ADLS Gen2. This value indicates the last processed timestamp from the previous run. |
| **2️⃣** | **Copy data – `AzureSQLToLake`**         | Extracts only **new or updated rows** from Azure SQL Database where `cdc_col` (e.g., `updated_at`) is greater than the last watermark. The extracted data is written as Parquet files to ADLS Gen2. |
| **3️⃣** | **Set variable – `current_timestamp`**   | Captures the current UTC timestamp (`@utcNow()`) and stores it in a variable called `current`. This value is used to name output Parquet files dynamically. |
| **4️⃣** | **If Condition – `If_Incremental_Data`** | Checks whether any data was read in the previous Copy activity using the expression `@greater(activity('AzureSQLToLake').output.dataRead, 0)`. If true, runs watermark update logic. |
| **4️⃣.1️⃣** | **Script – `max_cdc`**                | Executes a SQL query in Azure SQL to retrieve the **maximum value of the CDC column** (e.g., `MAX(updated_at)`). This identifies the new watermark value. |
| **4️⃣.2️⃣** | **Copy data – `update_last_cdc`**    | Writes the new watermark (from `max_cdc`) back to the JSON file `bronze/cdc/cdc.json`, overwriting the previous value. This ensures the next run only loads newer data. |
| **5️⃣** | **Delete – `DeleteEmptyFile`**           | Deletes temporary or empty Parquet files if no data was extracted, preventing unnecessary zero-byte files in the data lake. |

---

#### Flow Summary
1. Read last watermark (`cdc.json`)  
2. Query Azure SQL for rows newer than that timestamp  
3. Write data to ADLS (Parquet)  
4. If new data exists → update watermark  
5. Optionally clean up empty files  

---

#### Source and Sink Overview
| Component | Definition | This Pipeline’s Configuration |
|------------|-------------|-------------------------------|
| **Source** | The data origin — where ADF **reads** from | **Azure SQL Database** → Table in schema `dbo` (e.g., `DimUser`), filtered using watermark (`updated_at > last_cdc`) |
| **Sink** | The data destination — where ADF **writes** to | **Azure Data Lake Storage Gen2** → Container `bronze`, folder named after the table, file named dynamically as `table_timestamp.parquet` |


##### What is Source?
**Source** refers to the **data origin** from which ADF extracts data.

In this pipeline:
-  Source type: `Azure SQL Database`
-  Source Linked Service: Connection to your Azure SQL Server (`project-pipeline-spotify.database.windows.net`)
-  Data pulled from: Table in the dbo schema (`dbo.DimUser`)
-  Filter condition: Only rows where
    ```sql
    updated_at > last watermark (from cdc.json)
    ```

    > 💡 This ensures the pipeline fetches only incremental data since the last run — not the entire table.

---

##### What is Sink?
**Sink** is the **destination** where ADF writes the extracted or transformed data.

In this pipeline:
-  Sink type: `Azure Data Lake Storage Gen2`
-  Sink Linked Service: Connection to `storagepipelinespotify`
-  Destination format: Parquet file
-  Path pattern:
    ```makefile
    bronze/<table_name>/<table_name>_<timestamp>.parquet
    ```

-  example:
    ```makefile
    bronze/DimUser/DimUser_2025-10-27T01:23:45Z.parquet
    ```

    > 💡 The Sink holds the output files in an optimized format (Parquet) for analytics or further transformation.

---

##### How Watermarking Works
1.  The pipeline first reads the last processed timestamp (`cdc`) `from bronze/cdc/cdc.json`.
2.  The SQL query filters out all rows with `updated_at` less than or equal to that timestamp.
3.  After successful ingestion, the pipeline updates the watermark in `cdc.json` to the latest value.
4.  Next run → starts again from that new watermark.
      > ✅ This prevents duplicate loading and ensures each run only ingests new or **changed data**.


| Run | Last CDC (`cdc.json`) | Data Extracted | New CDC Written |
|------|------------------------|----------------|------------------|
| **1st** | `1900-01-01` | All rows | `2025-10-27 08:15:00` |
| **2nd** | `2025-10-27 08:15:00` | Only rows newer than that | `2025-10-27 10:30:00` |

---
## Adding Backfilling feature in Azure Data Factory (ADF)
This feature lets you **override the watermark** and ingest any historical range by supplying a **`from_date`** parameter at run-time.  
If `from_date` is **blank**, the pipeline uses the **stored watermark** from each table’s `*_cdc/cdc.json`.  
If `from_date` is **provided**, the pipeline starts from that date (backfill) and—after a successful run—**updates** the watermark.

---

### 1) Add a pipeline parameter `from_date`

**Pipeline → Parameters → + New**

| Name        | Type   | Default | Purpose                                      |
|-------------|--------|---------|----------------------------------------------|
| `from_date` | String | *(blank)* | Optional backfill start (ISO-8601, UTC) e.g. `2025-09-01T00:00:00` |

> ✅ Keep it **empty** for normal incremental runs, or set a value to backfill.

---

### 2) Update the **Source** query in `AzureSQLToLake`

**Activity:** `Copy data` → *Source* → **Use query** ✅

```sql
SELECT 
    *
FROM
    @{pipeline().parameters.schema}.@{pipeline().parameters.table}
WHERE
    @{pipeline().parameters.cdc_col} >
    '@{if(empty(pipeline().parameters.from_date), activity('last_cdc').output.firstRow.cdc, pipeline().parameters.from_date)}'
```
Explanation
-  If `from_date` is empty → use `last_cdc` from the JSON watermark.
-  If `from_date` has a value → use that value to backfill.
-  Keep `cdc_col` a datetime/datetime2 in SQL. If it’s `datetimeoffset`, cast appropriately.
---

### 3) Create per-table CDC folders in ADLS Gen2
In container `bronze/`, create these folders (adjust for your tables):

```pgsql
bronze/
  DimArtist_cdc/
    cdc.json      ← {"cdc":"1900-01-01"}
    empty.json    ← {}
  DimDate_cdc/
    cdc.json      ← {"cdc":"1900-01-01"}
    empty.json    ← {}
  DimTrack_cdc/
    cdc.json      ← {"cdc":"1900-01-01"}
    empty.json    ← {}
  DimUser_cdc/
    cdc.json      ← {"cdc":"1900-01-01"}
    empty.json    ← {}
  FactStream_cdc/
    cdc.json      ← {"cdc":"1900-01-01"}
    empty.json    ← {}
```
---
### 4) Make CDC folder dynamic in Lookup last_cdc
Activity: `Lookup` → Source → Dataset: `json_dynamic`
Dataset properties (parameters):

```ini
container = bronze
folder    = @concat(pipeline().parameters.table, '_cdc')
file      = cdc.json
```
-  First row only: ✅
-  Expected JSON shape: `{"cdc":"1900-01-01"}`

**Resulting path examples**
-  Table = `DimUser` → `bronze/DimUser_cdc/cdc.json`
-  Table = `FactStream` → `bronze/FactStream_cdc/cdc.json`

---

### How to Use the Backfill
#### A) Normal incremental (no backfill)

Run with:
-  `schema = dbo`
-  `table = DimUser`
-  `cdc_col = updated_at`
-  `from_date = "" (leave blank)`

Pipeline will use the stored watermark from:

```bash
bronze/DimUser_cdc/cdc.json
```

#### B) Backfill from a chosen date
Run with:
-  `from_date = 2025-09-01T00:00:00`

Pipeline loads rows with:

```nginx
updated_at > '2025-09-01T00:00:00'
```
Then updates `bronze/DimUser_cdc/cdc.json` to new `MAX(updated_at)`.

#### C) Reset to full load behavior

Set the file back to the minimum date:

```pgsql
bronze/<Table>_cdc/cdc.json = {"cdc":"1900-01-01"}
```
or run with a very old `from_date`.

---
### Example Run Matrix

| Scenario | Params |Expected |
|------|------------------------|----------------|
| **First-ever run** | `from_date = ""`, `cdc.json = 1900-01-01` | Full data load; writes Parquet; updates `cdc.json` to latest timestamp |
| **Daily incremental** | `from_date = ""` | Loads only rows newer than last `cdc` |
| **Backfill Sept** | `from_date = 2025-09-01T00:00:00` | Loads rows since Sept 1; updates `cdc.json` after run |
| **Reset & reload** | Manually set `cdc.json` to `1900-01-01` | Next run behaves like first load |

---

## Looping the Pipelines with **ForEach** (multi-table incremental loads)
This section turns single-table pipeline into a **multi-table** runner using **ForEach**.  

---
### 1) Create a new pipeline from the previous one
- Duplicate your working pipeline (the one from **“Incremental Ingestion Pipeline”**).
- Name the new pipeline: **`incremental_loop`**.

> We’ll keep all activities (`last_cdc`, `AzureSQLToLake`, `current_timestamp`, `If_Incremental_Data` → `max_cdc` → `update_last_cdc`, `DeleteEmptyFile`), but drive them with **ForEach**.

---

### 2) Remove the old pipeline parameters
Delete these pipeline-level **Parameters** if they exist:
- `schema`
- `table`
- `cdc_col`
- `from_date`

> In this loop version, each iteration gets those values from the **ForEach item** (not from pipeline parameters).

---

### 3) Add a new pipeline parameter: `loop_input` (Array)
Create a **pipeline parameter**:

| Name         | Type  | Default Value |
|--------------|-------|----------------|
| `loop_input` | Array | *(paste JSON below)* |

Paste this JSON as the **default value** (you can edit it later to add/remove tables):

```json
[
  { "schema": "dbo", "table": "DimUser",   "cdc_col": "updated_at",      "from_date": "" },
  { "schema": "dbo", "table": "DimTrack",  "cdc_col": "updated_at",      "from_date": "" },
  { "schema": "dbo", "table": "DimDate",   "cdc_col": "date",            "from_date": "" },
  { "schema": "dbo", "table": "DimArtist", "cdc_col": "updated_at",      "from_date": "" },
  { "schema": "dbo", "table": "FactStream","cdc_col": "stream_timestamp","from_date": "" }
]
```
> You can add more objects later; each must include `schema`, `table`, and `cdc_col`. `from_date` is optional (blank = use watermark).

---

### 4) Add a ForEach activity
-  Drag ForEach onto the canvas (name it ForEach_Tables).
-  Settings:
    -  Items =
        ```text
        @pipeline().parameters.loop_input
        ```
    -  `Sequential` = Enabled ✅ (process one table at a time)
      
> Enabling Sequential guarantees that shared things (like the `current` variable and watermark writes) do not conflict across parallel iterations.

---

### 5) Move all your table activities inside ForEach
Inside **ForEach_Tables** → Activities panel, add (or move) your existing activities in this order:
1.  `last_cdc` (Lookup)
2.  `AzureSQLToLake` (Copy Data)
3.  `current_timestamp` (Set Variable)
4.  `If_Incremental_Data` (If Condition)
      -  True branch:
          -  `max_cdc` (Script)
          -  `update_last_cdc` (Copy Data)
5.  `DeleteEmptyFile` (Delete) (optional)
   
>  Keep the same dependencies you already had between these activities.

---

### 6) Update dynamic content to use `item()` (the ForEach item)
Anywhere you previously referenced `pipeline().parameters.schema` / `table` / `cdc_col` / `from_date`, switch to `item()`:

#### 7.1 Lookup `last_cdc`
-  Dataset: `json_dynamic`
-  Dataset properties:
    ```ini
    container = bronze
    folder    = @concat(item().table, '_cdc')
    file      = cdc.json
    ```
-  First row only = Enabled

#### 7.2 Copy Data `AzureSQLToLake` (Source)
-  Use query
-  Query:
    ```sql
    SELECT
      *
    FROM
        @{item().schema}.@{item().table}
    WHERE
        @{item().cdc_col} >
        '@{if(empty(item().from_date), activity('last_cdc').output.firstRow.cdc, item().from_date)}'
    ```
    Notes:
    -  `item().from_date` blank → use `last_cdc` from JSON.
    -  Otherwise → backfill from the provided date.
    -  Ensure `cdc_col` has a datetime/datetime2 type (cast if needed).


#### 7.3 Copy Data `AzureSQLToLake` (Sink → `parquet_dynamic`)
-  Dataset properties:
    ```ini
    container = bronze
    folder    = @item().table
    file      = @concat(item().table, '_', variables('current'), '.parquet')
    ```
    
#### 7.4 Script max_cdc
- Script:
  ```sql
  SELECT
    MAX(@{item().cdc_col}) AS cdc
  FROM
    @{item().schema}.@{item().table}
  ```
-  Returns a single row with the new watermark.

#### 7.5 Copy Data `update_last_cdc`
-  **Source**: `json_dynamic`
    ```ini
    container = bronze
    folder    = @concat(item().table, '_cdc')
    file      = empty.json
    ```
-  **Sink**: `json_dynamic`
     ```ini
    container = bronze
    folder    = @concat(item().table, '_cdc')
    file      = cdc.json
    ```
    -  Copy behavior: Overwrite
 
#### 7.6 **Delete DeleteEmptyFile**
-  Dataset: `parquet_dynamic`
   ```ini
   container = bronze
   folder    = @item().table
   file      = @concat(item().table, '_', variables('current'), '.parquet')
   ```


---















