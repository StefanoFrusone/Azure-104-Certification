# Module 2: Storage

## Table of Contents

- [Introduction](#introduction)
- [Storage Accounts](#storage-accounts)
- [Blob Storage](#blob-storage)
- [Azure Files](#azure-files)
- [Storage Security](#storage-security)
- [Data Transfer and Management](#data-transfer-and-management)
- [Storage Monitoring and Optimization](#storage-monitoring-and-optimization)
- [PowerShell Commands Reference](#powershell-commands-reference)
- [Azure CLI Commands Reference](#azure-cli-commands-reference)
- [Common Exam Scenarios](#common-exam-scenarios)

---

## Introduction

**What is this module about?**

This module covers Azure Storage - the foundation for storing data in Azure. You'll learn how to store files, blobs, and configure secure, reliable storage solutions.

**Real-world analogy:**
Think of Azure Storage like a massive warehouse:

- **Storage Account** = The warehouse building itself
- **Blob Storage** = Shelves for boxes of any size (unstructured data)
- **Azure Files** = Filing cabinets with folders (shared file storage)
- **Access Tiers** = Different storage areas (some climate-controlled and expensive, some basic and cheap)
- **Replication** = Having backup warehouses in different cities

**Key questions this module answers:**

1. **Where** do I store data in Azure? (Storage Accounts)
2. **How** do I store different types of data? (Blob, Files, Queues, Tables)
3. **How** do I keep data secure? (Security features)
4. **How** do I optimize costs? (Access tiers, lifecycle management)
5. **How** do I ensure data availability? (Replication strategies)

**Exam Weight:** 15-20% of the exam

---

## Storage Accounts

### What is a Storage Account?

**Simple explanation:**
A storage account is a container that holds all your Azure Storage data objects: blobs, files, queues, tables, and disks. It's like getting a warehouse where you can store different types of items.

**Technical definition:**
A storage account provides a unique namespace in Azure for your data. Every object you store has an address that includes your unique account name. The combination of the account name and the Azure Storage service endpoint forms the endpoints for your storage account.

**Example:**

```
Storage Account Name: mystorageaccount
Blob endpoint: https://mystorageaccount.blob.core.windows.net
File endpoint: https://mystorageaccount.file.core.windows.net
Queue endpoint: https://mystorageaccount.queue.core.windows.net
Table endpoint: https://mystorageaccount.table.core.windows.net
```

### Storage Account Types

#### 1. Standard General-Purpose v2 (GPv2)

**Simple:** The most common type - supports all storage services and features.

**Use for:**

- Most scenarios
- Blobs, files, queues, tables
- Cost-effective for most workloads

**Supported services:**

- Blob Storage (all types)
- Azure Files
- Queue Storage
- Table Storage

**Performance:** Standard (HDD-backed)

**Replication options:** LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS (all available)

**Access tiers:** Hot, Cool, Cold, Archive

**Best for:**

- General-purpose applications
- Websites
- Mobile apps
- Backups
- Most production workloads

**Example scenario:**

```
Web application storing:
- User profile images (Blob)
- Application logs (Blob)
- Shared configuration files (Files)
- Message queue (Queue)
- Session data (Table)
```

#### 2. Premium Block Blobs

**Simple:** High-performance storage for blobs only. Think of it as the "sports car" of blob storage.

**Use for:**

- High transaction rates
- Low latency requirements
- Small objects
- Streaming scenarios

**Supported services:**

- Block blobs only
- Append blobs only

**Performance:** Premium (SSD-backed)

**Replication options:** LRS, ZRS only (no geo-redundancy)

**Access tiers:** Not applicable (premium is always "hot")

**Best for:**

- Interactive applications
- IoT data ingestion
- Real-time analytics
- High-frequency data access

**Example scenario:**

```
E-commerce website:
- Product images loaded thousands of times per second
- Need sub-10ms latency
- Willing to pay premium for performance
```

**Important:** Cannot convert between Standard and Premium after creation.

#### 3. Premium File Shares

**Simple:** High-performance file shares for enterprise workloads.

**Use for:**

- Enterprise file shares
- Databases on file shares
- High-performance computing

**Supported services:**

- Azure Files only

**Performance:** Premium (SSD-backed)

**Replication options:** LRS, ZRS only

**Protocols:** SMB and NFS

**Best for:**

- SQL Server databases
- SAP applications
- Dev/test environments requiring high IOPS
- Lift-and-shift scenarios

**Example scenario:**

```
SQL Server failover cluster:
- Database files on shared storage
- Requires <5ms latency
- 100,000+ IOPS needed
```

#### 4. Premium Page Blobs

**Simple:** High-performance storage for Azure VM disks (managed and unmanaged).

**Use for:**

- Virtual machine disks
- Databases that require page blob storage

**Supported services:**

- Page blobs only

**Performance:** Premium (SSD-backed)

**Replication options:** LRS only

**Best for:**

- VM OS and data disks
- High I/O applications
- Database workloads on VMs

**Example scenario:**

```
Production database server VM:
- OS disk on premium page blob
- Data disk on premium page blob
- Requires high IOPS and low latency
```

### Comparison Table

| Feature          | Standard GPv2            | Premium Block Blob | Premium File    | Premium Page Blob |
| ---------------- | ------------------------ | ------------------ | --------------- | ----------------- |
| **Services**     | Blob, File, Queue, Table | Block/Append Blob  | Files only      | Page Blob only    |
| **Performance**  | Standard (HDD)           | Premium (SSD)      | Premium (SSD)   | Premium (SSD)     |
| **Latency**      | Higher                   | Very Low           | Very Low        | Very Low          |
| **Cost**         | Low                      | High               | High            | High              |
| **Replication**  | All options              | LRS, ZRS           | LRS, ZRS        | LRS only          |
| **Access Tiers** | Yes                      | No                 | No              | No                |
| **Use Case**     | General purpose          | High-perf blobs    | High-perf files | VM disks          |

### Performance Tiers

**Standard (HDD-backed):**

- Cost-effective
- Good for infrequent access
- Throughput: Up to 60 MB/s per TB
- IOPS: Up to 500 per TB
- Use for: Backups, archives, non-critical workloads

**Premium (SSD-backed):**

- High performance
- Predictable latency (single-digit milliseconds)
- Throughput: Up to 200 MB/s and higher
- IOPS: Up to 100,000+
- Use for: Production databases, high-traffic websites, real-time analytics

**Important for exam:** You CANNOT change performance tier after creation. Must create new account and migrate data.

### Replication Options

Replication ensures your data is durable and highly available.

#### Locally Redundant Storage (LRS)

**Simple:** Three copies in one datacenter.

**How it works:**

```
Your Data → Copy 1 (same datacenter)
         → Copy 2 (same datacenter)
         → Copy 3 (same datacenter)
```

**Protection against:**

- ✅ Server rack failure
- ✅ Drive failure
- ✅ Hardware failure

**Does NOT protect against:**

- ❌ Entire datacenter failure
- ❌ Regional disaster

**Durability:** 99.999999999% (11 nines) over a year

**Cost:** Lowest

**Use case:**

- Non-critical data
- Data that can be easily recreated
- Regulatory requirements to keep data in one location
- Cost-sensitive scenarios

**Example:**

```
Dev/test environments
Temp processing data
Cache data
Build artifacts
```

#### Zone-Redundant Storage (ZRS)

**Simple:** Three copies across three availability zones in the same region.

**How it works:**

```
Your Data → Copy 1 (Availability Zone 1)
         → Copy 2 (Availability Zone 2)
         → Copy 3 (Availability Zone 3)
```

**Protection against:**

- ✅ Entire datacenter/zone failure
- ✅ Hardware failure
- ✅ Network failure in one zone

**Does NOT protect against:**

- ❌ Regional disaster (earthquake, flood affecting entire region)

**Durability:** 99.9999999999% (12 nines) over a year

**Cost:** Medium

**Availability zones:** Separate datacenters within same region (different power, cooling, networking)

**Use case:**

- Production workloads
- High availability requirements
- Data must stay in specific region (compliance)

**Example:**

```
Production web application data
Critical business data
Financial transaction logs
```

**Important:** Not all regions support ZRS. Check region availability.

#### Geo-Redundant Storage (GRS)

**Simple:** LRS in primary region + three copies in secondary region (hundreds of miles away).

**How it works:**

```
Primary Region:
Your Data → Copy 1 (same datacenter)
         → Copy 2 (same datacenter)
         → Copy 3 (same datacenter)
         ↓
    (Async replication)
         ↓
Secondary Region (paired region):
         → Copy 4 (same datacenter)
         → Copy 5 (same datacenter)
         → Copy 6 (same datacenter)
```

**Protection against:**

- ✅ Hardware failure
- ✅ Datacenter failure
- ✅ Regional disaster

**Does NOT allow:**

- ❌ Read access to secondary region (unless you fail over)

**Durability:** 99.99999999999999% (16 nines) over a year

**Cost:** Higher

**Replication time:** Data replicated to secondary region within 15 minutes (typically)

**Region pairs:** Azure automatically pairs regions (e.g., East US ↔ West US, North Europe ↔ West Europe)

**Use case:**

- Critical production data
- Disaster recovery requirements
- Compliance requiring geo-redundancy
- Can't afford data loss

**Example:**

```
Customer databases
Financial records
Healthcare data
Legal documents
```

**Important:** Secondary region data is NOT accessible for read/write unless you initiate failover.

#### Read-Access Geo-Redundant Storage (RA-GRS)

**Simple:** Same as GRS but you can READ from secondary region anytime.

**How it works:**
Same as GRS, but secondary endpoint is always available for read operations.

**Endpoints:**

```
Primary: https://mystorageaccount.blob.core.windows.net (read/write)
Secondary: https://mystorageaccount-secondary.blob.core.windows.net (read-only)
```

**Protection:** Same as GRS

**Additional benefit:**

- ✅ Read access to secondary region without failover
- ✅ Load balancing read traffic
- ✅ Disaster recovery without data loss

**Cost:** Same as GRS (or slightly higher)

**Use case:**

- Applications that can read from secondary region
- High availability for read operations
- Global applications serving users from multiple regions
- Analytics workloads that can tolerate slightly stale data

**Example:**

```
Global content delivery:
- Users in US read from primary (East US)
- Users in Europe read from secondary (North Europe)
- Reduced latency for European users
```

**Important:** Data in secondary region may be slightly behind (eventual consistency).

#### Geo-Zone-Redundant Storage (GZRS)

**Simple:** ZRS in primary region + LRS in secondary region.

**How it works:**

```
Primary Region:
Your Data → Copy 1 (Zone 1)
         → Copy 2 (Zone 2)
         → Copy 3 (Zone 3)
         ↓
    (Async replication)
         ↓
Secondary Region:
         → Copy 4 (same datacenter)
         → Copy 5 (same datacenter)
         → Copy 6 (same datacenter)
```

**Protection against:**

- ✅ Zone failure in primary region
- ✅ Regional disaster
- ✅ Highest availability

**Durability:** 99.99999999999999% (16 nines) over a year

**Cost:** Highest

**Use case:**

- Maximum durability and availability
- Mission-critical applications
- Compliance requiring both zone and geo redundancy
- Cannot tolerate any downtime

**Example:**

```
Banking systems
Healthcare records
Mission-critical SaaS applications
```

#### Read-Access Geo-Zone-Redundant Storage (RA-GZRS)

**Simple:** GZRS + read access to secondary region.

**How it works:** Same as GZRS but with read-only access to secondary region.

**Protection:** Maximum (same as GZRS)

**Additional benefit:** Read access to secondary

**Cost:** Highest

**Use case:**

- Maximum availability and durability
- Global read access
- Absolutely cannot lose data or have downtime

### Replication Comparison Table

| Replication | Copies | Locations               | Protects Against   | Read Secondary?     | Durability | Cost |
| ----------- | ------ | ----------------------- | ------------------ | ------------------- | ---------- | ---- |
| **LRS**     | 3      | 1 datacenter            | Hardware failure   | No                  | 11 nines   | $    |
| **ZRS**     | 3      | 3 zones                 | Datacenter failure | No                  | 12 nines   | $$   |
| **GRS**     | 6      | 2 regions               | Regional disaster  | No (until failover) | 16 nines   | $$$  |
| **RA-GRS**  | 6      | 2 regions               | Regional disaster  | Yes                 | 16 nines   | $$$  |
| **GZRS**    | 6      | 2 regions (ZRS primary) | Zone + Regional    | No (until failover) | 16 nines   | $$$$ |
| **RA-GZRS** | 6      | 2 regions (ZRS primary) | Zone + Regional    | Yes                 | 16 nines   | $$$$ |

### Access Tiers (for Blob Storage)

Access tiers optimize storage costs based on how frequently you access data.

#### Hot Tier

**Simple:** For data you access frequently.

**Characteristics:**

- Highest storage cost
- Lowest access cost
- Optimized for frequent read/write operations

**Cost structure:**

- Storage: $$$ per GB
- Transactions: $ per 10,000 operations

**Use for:**

- Data in active use
- Data intended for processing
- Frequently accessed files

**Examples:**

```
- Website images
- Application data
- Active database backups
- Streaming video content
- Recently uploaded documents
```

**Availability:** 99.9% (or higher with redundancy)

#### Cool Tier

**Simple:** For data you don't access often (at least 30 days).

**Characteristics:**

- Lower storage cost than Hot
- Higher access cost than Hot
- Minimum 30-day storage duration (early deletion fee)

**Cost structure:**

- Storage: $$ per GB (~50% less than Hot)
- Transactions: $$ per 10,000 operations

**Use for:**

- Short-term backup
- Older data accessed less frequently
- Data that needs to be available quickly but isn't accessed often

**Examples:**

```
- Older media content
- Compliance data (accessed occasionally)
- Short-term backups
- Disaster recovery datasets
```

**Availability:** 99% (slightly lower than Hot)

**Important:** If you delete data before 30 days, you pay early deletion charge.

#### Cold Tier

**Simple:** For data you rarely access (at least 90 days).

**Characteristics:**

- Even lower storage cost
- Higher access cost than Cool
- Minimum 90-day storage duration

**Cost structure:**

- Storage: $ per GB (~30% less than Cool)
- Transactions: $$$ per 10,000 operations

**Use for:**

- Long-term backup
- Rarely accessed data
- Data that must be retained but almost never accessed

**Examples:**

```
- Historical data
- Long-term compliance archives
- Old project files
- Infrequently accessed backups
```

**Availability:** 99% (same as Cool)

**Important:** Not available for all storage account types. Requires GPv2 or Blob Storage accounts.

#### Archive Tier

**Simple:** For data you almost never access (at least 180 days). Cheapest storage, but takes hours to retrieve.

**Characteristics:**

- Lowest storage cost (up to 90% less than Hot)
- Highest access cost
- Data is offline (must rehydrate before access)
- Minimum 180-day storage duration

**Cost structure:**

- Storage: $ per GB (lowest)
- Transactions: $$$$ per 10,000 operations
- Rehydration: Additional cost + time

**Rehydration time:**

- Standard priority: Up to 15 hours
- High priority: Under 1 hour (more expensive)

**Use for:**

- Long-term archival
- Compliance data rarely accessed
- Data that must be kept but almost never retrieved

**Examples:**

```
- 7-year financial records (compliance)
- Medical records (legal requirements)
- Old security camera footage
- Historical scientific data
- Legal discovery data
```

**Availability:** N/A (data is offline until rehydrated)

**Important:**

- Data must be rehydrated to Hot or Cool before access
- Cannot read Archive blobs directly
- Rehydration takes hours and costs extra

### Access Tier Comparison

| Tier        | Storage Cost | Access Cost | Min Duration | Retrieval Time | Use Case              |
| ----------- | ------------ | ----------- | ------------ | -------------- | --------------------- |
| **Hot**     | Highest      | Lowest      | None         | Instant        | Frequently accessed   |
| **Cool**    | Medium       | Medium      | 30 days      | Instant        | Infrequently accessed |
| **Cold**    | Low          | High        | 90 days      | Instant        | Rarely accessed       |
| **Archive** | Lowest       | Highest     | 180 days     | Up to 15 hours | Almost never accessed |

**Decision flowchart:**

```
How often do you access the data?
  ├─ Multiple times per day → Hot
  ├─ Once per week/month → Cool
  ├─ Once per quarter → Cold
  └─ Once per year or less → Archive
```

### Storage Account Naming Rules

**Important for exam:**

**Rules:**

- 3-24 characters
- Lowercase letters and numbers only
- No special characters
- Must be globally unique across ALL Azure
- Must start with a letter or number

**Valid examples:**

```
mystorage123
contosoprod2024
backupstorage001
```

**Invalid examples:**

```
MyStorage123 (uppercase not allowed)
my-storage (hyphens not allowed)
storage_account (underscores not allowed)
my (too short)
mystorageaccountwithverylongname123 (too long)
```

### Storage Account Endpoints

Each storage service in an account has its own endpoint:

```
Blob:   https://<storage-account-name>.blob.core.windows.net
File:   https://<storage-account-name>.file.core.windows.net
Queue:  https://<storage-account-name>.queue.core.windows.net
Table:  https://<storage-account-name>.table.core.windows.net
```

**Custom domain:** You can configure a custom domain (e.g., https://cdn.contoso.com) to point to your blob storage.

---

## Blob Storage

### What is Blob Storage?

**Simple explanation:**
Blob (Binary Large Object) storage is for storing unstructured data - anything that doesn't fit into a database. Think of it as a huge file system in the cloud for any type of file.

**Technical definition:**
Blob storage is optimized for storing massive amounts of unstructured data. Data that doesn't adhere to a particular data model or definition, such as text or binary data.

**Real-world analogy:**
Like a massive storage unit where you can put boxes of any size and shape - photos, videos, documents, backup files, anything.

### Types of Blobs

#### 1. Block Blobs

**Simple:** For files you upload and store (most common type).

**Characteristics:**

- Made up of blocks of data
- Optimized for upload/download operations
- Can upload blocks in parallel
- Maximum size: ~190 TB (4.75 TB per blob, with multiple versions)

**How it works:**

```
File (100 MB) is split into:
  Block 1 (4 MB)
  Block 2 (4 MB)
  Block 3 (4 MB)
  ...
  Block 25 (4 MB)

Upload blocks in parallel → Commit block list
```

**Use for:**

- Documents (PDFs, Word, Excel)
- Images and videos
- Backup files
- Application binaries
- Log files
- Any file you upload and download

**Examples:**

```
- User profile photos
- Product catalog images
- Video content for streaming
- Software installers
- Database backups
```

**Upload methods:**

- Upload entire file at once (up to 5 GB)
- Upload in blocks (parallel upload, resume failed uploads)

#### 2. Append Blobs

**Simple:** For data you continuously add to (append-only).

**Characteristics:**

- Optimized for append operations
- Cannot modify existing data (only add to end)
- Maximum size: ~195 GB
- Ideal for logging scenarios

**How it works:**

```
Log File:
  [Initial content]
  [New log entry appended]
  [Another log entry appended]
  [Another log entry appended]

Cannot modify or delete previous entries
```

**Use for:**

- Log files (application, system, audit logs)
- Continuous data streams
- Any append-only scenario

**Examples:**

```
- Application logs
- IoT sensor data streams
- Financial transaction logs
- Audit trails
```

**Important:** Once written, data cannot be modified (immutable logging).

#### 3. Page Blobs

**Simple:** For random read/write operations (like a hard drive).

**Characteristics:**

- Optimized for random access
- Made up of 512-byte pages
- Maximum size: 8 TB
- Used for virtual machine disks (VHDs)

**How it works:**

```
Page Blob (like a hard drive):
  Page 1 (512 bytes)
  Page 2 (512 bytes)
  ...
  Page n (512 bytes)

Can read/write any page randomly
```

**Use for:**

- Virtual machine disks (OS and data disks)
- Databases requiring random access
- Any scenario needing random read/write

**Examples:**

```
- Azure VM OS disks
- Azure VM data disks
- Database files stored in Azure
```

**Important:** Page blobs are the storage behind Azure Managed Disks and unmanaged disks.

### Blob Type Comparison

| Feature            | Block Blob              | Append Blob                       | Page Blob           |
| ------------------ | ----------------------- | --------------------------------- | ------------------- |
| **Max Size**       | ~190 TB                 | ~195 GB                           | 8 TB                |
| **Optimized For**  | Upload/download         | Append operations                 | Random access       |
| **Use Case**       | Files, images, videos   | Logs, streams                     | VM disks            |
| **Modification**   | Can replace entire blob | Can only append                   | Can modify any page |
| **Access Pattern** | Sequential              | Sequential (write), Random (read) | Random              |

### Containers

**Simple:** Folders that organize your blobs. Like directories in a file system.

**Technical:** Containers provide grouping of blobs within a storage account. All blobs must be in a container.

**Naming rules:**

- 3-63 characters
- Lowercase letters, numbers, hyphens
- Must start with letter or number
- No consecutive hyphens

**Examples:**

```
Storage Account: mystorageaccount
  ├─ Container: images
  │   ├─ logo.png
  │   ├─ banner.jpg
  │   └─ profile/user123.jpg
  ├─ Container: videos
  │   ├─ intro.mp4
  │   └─ tutorial.mp4
  └─ Container: backups
      ├─ db-backup-2024.bak
      └─ files-backup-2024.zip
```

**Container hierarchy:**

- Storage Account (top level)
  - Container (like a folder, but flat namespace)
    - Blob (with optional virtual directories in name)

**Virtual directories:**
Blob storage doesn't have true folders, but you can simulate them with naming:

```
Container: documents
  Blobs:
    - projects/alpha/document1.pdf (looks like projects/alpha folder)
    - projects/alpha/document2.pdf
    - projects/beta/report.xlsx
```

The blob name includes the full "path": `projects/alpha/document1.pdf`

### Container Access Levels

Controls who can read blobs in a container.

#### Private (No anonymous access)

**Simple:** Default - authentication required for all access.

**Who can access:**

- ✅ Authenticated users with proper permissions
- ❌ Anonymous users (public internet)

**Use case:**

- Internal data
- Sensitive information
- Customer data
- Default security posture

**Example:**

```
Container: customer-data
Access: Private

Result: Must authenticate with Azure AD or access key to read blobs
URL: https://mystorageaccount.blob.core.windows.net/customer-data/file.txt
Direct browser access: ❌ Denied
```

#### Blob (Anonymous read access for blobs only)

**Simple:** Anyone can read individual blobs if they know the exact URL, but cannot list container contents.

**Who can access:**

- ✅ Anyone with exact blob URL
- ❌ Cannot list blobs in container

**Use case:**

- Public images/videos
- CDN content
- Public downloads

**Example:**

```
Container: images
Access: Blob

Direct blob URL: https://mystorageaccount.blob.core.windows.net/images/logo.png
Direct browser access: ✅ Works

List container: https://mystorageaccount.blob.core.windows.net/images?restype=container&comp=list
List access: ❌ Denied
```

#### Container (Anonymous read access for container and blobs)

**Simple:** Anyone can read blobs AND list all blobs in the container.

**Who can access:**

- ✅ Anyone can read any blob
- ✅ Anyone can list all blobs

**Use case:**

- Fully public content
- Open data sets
- Public file repositories

**Example:**

```
Container: public-downloads
Access: Container

List all blobs: ✅ Works
Download any blob: ✅ Works

Result: Completely public (like a public FTP server)
```

**Security warning:** Container-level access is very permissive. Use carefully!

### Access Level Comparison

| Level         | Read Blobs           | List Blobs | Use Case            |
| ------------- | -------------------- | ---------- | ------------------- |
| **Private**   | ❌                   | ❌         | Default, secure     |
| **Blob**      | ✅ (if you know URL) | ❌         | Public content, CDN |
| **Container** | ✅                   | ✅         | Fully public data   |

### Blob Versioning

**Simple:** Automatically keeps previous versions of a blob when you modify it.

**How it works:**

```
1. Upload file.txt (Version 1 - current)
2. Modify file.txt (Version 1 becomes old, Version 2 is current)
3. Modify file.txt (Version 2 becomes old, Version 3 is current)

All versions are kept and can be restored
```

**Benefits:**

- Protect against accidental overwrites
- Protect against accidental deletes
- Recover from data corruption
- Roll back to previous version

**Use case:**

- Critical data that changes over time
- Compliance requirements
- Accidental modification protection

**Cost:** Each version consumes storage (pay for all versions)

**Important:**

- Enabled at storage account level
- Applies to block blobs only
- Different from snapshots (which are manual)

### Blob Snapshots

**Simple:** Manual point-in-time copy of a blob.

**How it works:**

```
1. Create blob: file.txt
2. Create snapshot: file.txt (snapshot 1 - timestamp: 2024-01-15 10:00)
3. Modify blob: file.txt (current version changed)
4. Create snapshot: file.txt (snapshot 2 - timestamp: 2024-01-15 11:00)

Snapshots are read-only
```

**vs Versioning:**

- Versioning: Automatic on every change
- Snapshots: Manual, when you choose

**Use case:**

- Before major updates
- Testing changes (snapshot, modify, restore if needed)
- Backup at specific points in time

**Cost:** Incremental (only pay for changed data between snapshots)

### Soft Delete

**Simple:** Recycle bin for blobs - deleted items kept for X days before permanent deletion.

**How it works:**

```
1. Delete blob
2. Blob moves to "deleted" state (not permanently gone)
3. Blob recoverable for retention period (e.g., 7 days)
4. After retention period, blob permanently deleted
```

**Protects against:**

- Accidental deletion
- Application bugs that delete data
- Malicious deletion

**Retention period:**

- Configurable: 1-365 days
- Recommended: 7-30 days

**Use case:**

- All production storage accounts
- Protection against accidental deletions

**Cost:** Deleted blobs consume storage during retention period

**Types:**

- Blob soft delete: For individual blobs
- Container soft delete: For entire containers

**Important for exam:** Soft delete is a best practice for all production storage.

### Lifecycle Management

**Simple:** Automatically move blobs to cheaper tiers or delete them based on rules.

**Why needed:**

```
Problem:
- Old logs sitting in Hot tier (expensive)
- 1-year-old backups in Cool tier (could be Archive)
- Test data never cleaned up

Solution:
Automatic lifecycle policies
```

**How it works:**
Define rules like:

```
IF blob is older than 30 days → Move to Cool
IF blob is older than 90 days → Move to Archive
IF blob is older than 365 days → Delete
```

**Rule components:**

1. **Filter (which blobs):**

   - All blobs in container
   - Blobs with specific prefix (e.g., "logs/")
   - Blobs with specific blob type

2. **Condition (when):**

   - Days since creation
   - Days since last modification
   - Days since last access (requires access tracking)

3. **Action (what to do):**
   - Move to Cool tier
   - Move to Cold tier
   - Move to Archive tier
   - Delete blob

**Example rules:**

**Rule 1: Archive old backups**

```
IF container = "backups"
AND days since creation > 90
THEN move to Archive tier
```

**Rule 2: Delete old logs**

```
IF blob name starts with "logs/"
AND days since modification > 180
THEN delete blob
```

**Rule 3: Cool down infrequently accessed data**

```
IF days since last access > 30
THEN move to Cool tier
```

**JSON example:**

```json
{
  "rules": [
    {
      "name": "archive-old-backups",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["backups/"]
        },
        "actions": {
          "baseBlob": {
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            }
          }
        }
      }
    }
  ]
}
```

**Best practices:**

- Start with non-critical data
- Test rules before applying to production
- Monitor cost savings
- Review rules quarterly

**Cost savings example:**

```
100 GB in Hot tier: $18/month
100 GB in Cool tier: $10/month
100 GB in Archive tier: $1/month

If 80% of data is older than 90 days:
Before: 100 GB × $18 = $1,800/month
After: 20 GB × $18 + 80 GB × $1 = $360 + $80 = $440/month
Savings: $1,360/month (76% reduction!)
```

**Important for exam:**

- Lifecycle policies run once per day
- Actions are not immediate
- Cannot move from Archive back to Hot/Cool automatically (must rehydrate manually)
- Policies apply at container or account level

---

## Azure Files

### What is Azure Files?

**Simple explanation:**
Azure Files is like a network drive (file share) in the cloud. It works exactly like the shared folders you use in offices - multiple users can access the same files.

**Technical definition:**
Azure Files offers fully managed file shares in the cloud that are accessible via the industry-standard Server Message Block (SMB) protocol and Network File System (NFS) protocol.

**Real-world analogy:**
Like a shared drive at work where everyone can access documents, but stored in Azure instead of on a server under someone's desk.

### Key Differences: Azure Files vs Blob Storage

| Feature       | Azure Files                  | Blob Storage                         |
| ------------- | ---------------------------- | ------------------------------------ |
| **Interface** | File system (drive letter)   | REST API, URLs                       |
| **Protocol**  | SMB, NFS                     | HTTP/HTTPS                           |
| **Use case**  | Lift-and-shift, shared files | Cloud-native apps, unstructured data |
| **Access**    | Mount as drive (Z:, etc.)    | Access via code/tools                |
| **Hierarchy** | True folders/directories     | Flat namespace (virtual folders)     |
| **Locking**   | File locking supported       | No file locking                      |

**When to use Azure Files:**

- Lift-and-shift applications expecting file system
- Shared access to files from multiple VMs
- Replace on-premises file servers
- Store configuration files
- Development tools and dependencies

**When to use Blob Storage:**

- Cloud-native applications
- REST API access
- Streaming content
- Backup and archival
- Big data analytics

### Protocols

#### SMB (Server Message Block)

**Simple:** The standard Windows file sharing protocol.

**Versions:**

- SMB 2.1: Basic functionality, older Windows/Linux
- SMB 3.0: Encryption, better performance, Windows 8+, Windows Server 2012+
- SMB 3.1.1: Enhanced security, Windows 10+, Windows Server 2016+

**Supported on:**

- ✅ Windows (native support)
- ✅ Linux (via CIFS/SMB client)
- ✅ macOS

**Authentication:**

- Storage account key
- Azure AD (for domain-joined machines)

**Use case:**

- Windows applications
- Cross-platform scenarios
- Most common scenario

**Example mounting:**

```
Windows: net use Z: \\mystorageaccount.file.core.windows.net\myshare
Linux: mount -t cifs //mystorageaccount.file.core.windows.net/myshare /mnt/myshare
```

#### NFS (Network File System)

**Simple:** Linux/Unix standard file sharing protocol.

**Version:** NFS 4.1

**Supported on:**

- ✅ Linux
- ✅ Windows (with NFS client installed)
- ❌ Not for macOS in Azure Files

**Authentication:**

- Network-based (no storage key needed)
- IP-based access control

**Requirements:**

- Premium file share only
- Virtual network required
- Private endpoint or service endpoint

**Use case:**

- Linux workloads
- POSIX compliance requirements
- Container storage

**Important difference from SMB:**

- NFS doesn't support encryption in transit (use private endpoints)
- Better for Linux workloads
- No Active Directory integration

### Performance Tiers

#### Standard File Shares (HDD-based)

**Storage tiers:**

**Transaction Optimized:**

- Default tier
- Best for: High transaction workloads
- Use case: Databases, dev/test, general purpose
- IOPS: Up to 10,000 (burst)
- Throughput: Up to 300 MiB/s

**Hot:**

- Similar to Transaction Optimized
- Optimized for team shares
- Use case: File shares, collaboration
- IOPS: Up to 10,000 (burst)
- Throughput: Up to 300 MiB/s

**Cool:**

- Lower storage cost, higher transaction cost
- Use case: Archive, backup, infrequently accessed data
- IOPS: Up to 10,000 (burst)
- Throughput: Up to 300 MiB/s

**Capacity:** Up to 100 TiB per share

**Cost:** Pay for storage used + transactions

#### Premium File Shares (SSD-based)

**Characteristics:**

- SSD-backed storage
- Provisioned capacity (pay for what you provision, not what you use)
- Consistent low latency (<10 ms)
- High IOPS and throughput

**Performance:**

- IOPS: Up to 100,000
- Throughput: Up to 10 GiB/s
- Latency: Single-digit milliseconds

**Capacity:** Up to 100 TiB per share

**Use case:**

- IO-intensive workloads
- Databases (SQL Server)
- SAP applications
- Enterprise applications

**Cost model:**

```
Standard: Pay for GB used + transactions
Premium: Pay for GB provisioned (even if not used)

Example:
Provision 1 TB Premium: Pay for 1 TB
Actually use 100 GB: Still pay for 1 TB

Benefit: Predictable costs, guaranteed performance
```

### File Share Features

#### Snapshots

**Simple:** Point-in-time backup of entire file share.

**Characteristics:**

- Read-only copy
- Incremental (only changes stored)
- Up to 200 snapshots per share
- Preserved for 10 years

**Use case:**

- Protection against accidental deletion
- Backup before major changes
- Compliance and retention
- Recovery from ransomware

**How it works:**

```
1. Create snapshot of file share
2. Continue working on files (snapshot unchanged)
3. If needed, restore individual files or entire share
```

**Cost:** Pay only for changed data (incremental)

**Example:**

```
Monday: Create snapshot (1 TB)
Tuesday: Modify 10 GB of files
Wednesday: Create another snapshot

Storage used: 1 TB + 10 GB = 1.01 TB
(Not 1 TB + 1 TB)
```

#### Soft Delete

**Simple:** Recycle bin for file shares.

**How it works:**

```
1. Delete file share
2. Share moves to soft-deleted state
3. Recoverable for retention period (1-365 days)
4. After retention, permanently deleted
```

**Protects against:**

- Accidental share deletion
- Application errors
- Malicious deletion

**Important:**

- Applies to entire share (not individual files)
- Snapshots also protected by soft delete
- Must be enabled before deletion

**Cost:** Soft-deleted shares still consume storage

### Azure File Sync

**Simple:** Sync files between on-premises Windows Servers and Azure Files. Like Dropbox for servers.

**The problem it solves:**

```
Traditional scenario:
- Files on local file server
- Need backup in Azure
- Need files accessible from multiple locations
- Local server limited capacity

Solution: Azure File Sync
```

**How it works:**

```
On-Premises File Server (Windows Server)
  ↕️ (Sync)
Azure File Share
  ↕️ (Sync)
Another On-Premises Server
  ↕️ (Sync)
Azure VMs
```

**Components:**

1. **Storage Sync Service:** Azure resource that manages sync
2. **Sync Group:** Defines which Azure File Share and which servers sync together
3. **Server Endpoint:** Path on Windows Server to sync (e.g., D:\SharedFiles)
4. **Cloud Endpoint:** Azure File Share
5. **Azure File Sync Agent:** Installed on Windows Server

**Key features:**

**Cloud Tiering:**

```
Simple: Keep frequently accessed files locally, move old files to cloud

Example:
1 TB of data total
Local server has 200 GB disk
Cloud tiering enabled:
- Recent files: Kept locally (150 GB)
- Old files: Only pointers locally, actual data in Azure
- Appears as 1 TB to users
- 850 GB saved on local disk
```

**Multi-site sync:**

```
Office A (London) ←→ Azure ←→ Office B (New York)

Files available everywhere
Changes sync automatically
Local speed access
Cloud durability
```

**Disaster recovery:**

```
Local server dies
Spin up new server
Install Azure File Sync
Sync down from Azure
Back in business
```

**Requirements:**

- Windows Server 2012 R2 or newer
- Azure File Share (SMB)
- Internet connectivity

**Use cases:**

- Branch office file servers
- Disaster recovery for file servers
- Lift-and-shift to Azure
- Archive old data to cloud (free up local space)

**Cost:**

- Storage in Azure Files
- Egress charges (data out of Azure)
- Sync service (minimal cost)

**Important for exam:**

- Only works with Windows Server
- Requires SMB file shares (not NFS)
- Cloud tiering requires NTFS volumes
- Can have multiple server endpoints in same sync group

---

## Storage Security

Security is CRITICAL for storage accounts. Multiple layers protect your data.

### Storage Account Keys (Access Keys)

**Simple:** Master passwords for your storage account. Full access to everything.

**What they are:**

- Two 512-bit keys (Key1 and Key2)
- Automatically generated when account created
- Provide complete access to storage account

**Think of them as:** Root passwords for your storage

**What you can do with keys:**

- Full read/write access
- Delete data
- Modify settings
- No restrictions

**Why two keys?**
For key rotation without downtime:

```
1. Applications use Key1
2. Regenerate Key2 (Key1 still works)
3. Update applications to use Key2
4. Regenerate Key1
5. Update remaining applications to use Key1
6. Repeat regularly
```

**Security concerns:**

- ⚠️ Anyone with key has full access
- ⚠️ Cannot limit permissions
- ⚠️ Cannot set expiration
- ⚠️ Cannot audit who used the key
- ⚠️ If leaked, must regenerate (breaks all apps using it)

**Best practices:**

- ❌ DON'T put keys in code
- ❌ DON'T share keys with users
- ❌ DON'T use keys for end-user access
- ✅ Store in Azure Key Vault
- ✅ Use managed identities instead when possible
- ✅ Rotate keys regularly (quarterly)
- ✅ Use SAS tokens for limited access

### Shared Access Signature (SAS)

**Simple:** Temporary, limited-permission URLs for accessing storage.

**Think of it as:** A temporary access card with specific permissions and expiration.

**Why use SAS?**

```
Problem: Need to give someone access to one file
Bad solution: Give them storage account key (full access)
Good solution: Create SAS token (limited access)
```

**Types of SAS:**

#### 1. User Delegation SAS

**Simple:** SAS token secured by Azure AD credentials (most secure).

**Characteristics:**

- ✅ Secured with Azure AD
- ✅ Can be revoked
- ✅ Auditable
- ✅ No storage key exposure
- ❌ Blob storage only

**When to use:**

- Production scenarios
- Maximum security needed
- Need to revoke access
- Compliance requirements

**Example scenario:**

```
User needs to upload profile photo
1. User authenticates with Azure AD
2. Application requests user delegation SAS
3. User gets temporary URL (valid 1 hour)
4. User uploads photo using URL
5. URL expires after 1 hour
```

#### 2. Service SAS

**Simple:** SAS for specific service (Blob, File, Queue, or Table).

**Characteristics:**

- Secured with storage account key
- Scoped to one service
- Cannot be revoked (until expiration)
- Very granular permissions

**Use case:**

- Access to specific blob/file/queue/table
- Application needs limited access
- Temporary access for users

**Example:**

```
Share one blob (file) with external partner:
- Grant read-only access
- Valid for 7 days
- No other permissions
```

#### 3. Account SAS

**Simple:** SAS across multiple services in storage account.

**Characteristics:**

- Secured with storage account key
- Can access multiple services
- Service-level permissions

**Use case:**

- Application needs access to multiple services
- Backup tools
- Migration tools

**SAS Components:**

Every SAS URL contains:

**1. Resource URI:** What you're accessing

```
https://mystorageaccount.blob.core.windows.net/container/file.txt
```

**2. SAS Token:** Query string with permissions

```
?sv=2021-06-08&se=2024-12-31T23:59:00Z&sr=b&sp=r&sig=signature
```

**Parameters:**

- `sv`: Storage service version
- `se`: Expiry time (when SAS stops working)
- `st`: Start time (optional, when SAS becomes valid)
- `sr`: Resource type (b=blob, c=container, etc.)
- `sp`: Permissions (r=read, w=write, d=delete, l=list)
- `sip`: IP range (optional, limit to specific IPs)
- `spr`: Protocol (https only recommended)
- `sig`: Signature (proves it's authentic)

**Permission options:**

- `r` - Read
- `w` - Write
- `d` - Delete
- `l` - List
- `a` - Add
- `c` - Create
- `u` - Update
- `p` - Process

**Example SAS URLs:**

**Read-only access to one blob:**

```
https://mystorageaccount.blob.core.windows.net/container/file.txt?sv=2021-06-08&se=2024-12-31T23:59:00Z&sr=b&sp=r&sig=abc123...
```

**Read and write to container:**

```
https://mystorageaccount.blob.core.windows.net/container?sv=2021-06-08&se=2024-12-31T23:59:00Z&sr=c&sp=rwl&sig=abc123...
```

**Best practices:**

1. **Always set expiration time**

   - Short as possible (hours/days, not months)
   - Force renewal for extended access

2. **Use HTTPS only**

   - Set `spr=https` parameter
   - Prevent man-in-the-middle attacks

3. **Limit IP addresses** (when possible)

   - `sip=192.168.1.1` or range
   - Reduce attack surface

4. **Minimum permissions**

   - Only grant what's needed
   - Read-only if possible

5. **Use user delegation SAS**

   - More secure than key-based SAS
   - Can be revoked

6. **Monitor SAS usage**
   - Enable storage analytics
   - Alert on suspicious activity

### Stored Access Policy

**Simple:** Named policies for SAS tokens that can be modified or revoked.

**The problem:**

```
Regular SAS:
- Create SAS with 7-day expiration
- Share with 100 users
- Need to revoke access early
- ❌ Cannot revoke (must wait for expiration)
```

**The solution:**

```
Stored Access Policy:
- Create policy named "temp-access" with permissions
- Create SAS tokens referencing this policy
- Share with 100 users
- Need to revoke? Delete or modify the policy
- ✅ All SAS tokens using that policy immediately revoked
```

**Benefits:**

- Revoke access anytime
- Change permissions without reissuing SAS
- Centralized management
- Better audit trail

**Example:**

```
1. Create stored access policy: "external-partners"
   - Permissions: Read-only
   - Expiry: 30 days

2. Generate 50 SAS tokens using this policy

3. Problem detected? Delete the policy
   → All 50 SAS tokens immediately invalid
```

**Limitations:**

- Maximum 5 policies per container
- Only for service SAS (not account SAS)

### Azure AD Authentication

**Simple:** Use Azure AD identities instead of keys for authentication (most secure option).

**How it works:**

```
User/Application
  ↓ (Azure AD token)
Storage Account (validates token)
  ↓ (checks RBAC permissions)
Allow or Deny access
```

**Benefits:**

- ✅ No keys to manage
- ✅ Granular RBAC permissions
- ✅ Full audit trail (who accessed what)
- ✅ Conditional Access policies can be applied
- ✅ MFA can be required
- ✅ Centralized identity management

**Supported for:**

- ✅ Blob Storage
- ✅ Queue Storage
- ✅ Azure Files (SMB with domain-joined machines)
- ❌ Table Storage (not supported)

**Required RBAC roles:**

| Role                                        | Permissions                              |
| ------------------------------------------- | ---------------------------------------- |
| **Storage Blob Data Owner**                 | Full access to blob data                 |
| **Storage Blob Data Contributor**           | Read, write, delete blobs and containers |
| **Storage Blob Data Reader**                | Read blobs and list containers           |
| **Storage Queue Data Contributor**          | Read, write, delete queue messages       |
| **Storage Queue Data Reader**               | Read queue messages                      |
| **Storage File Data SMB Share Contributor** | Read, write, delete files over SMB       |
| **Storage File Data SMB Share Reader**      | Read files over SMB                      |

**Example scenario:**

```
Web application needs to store user uploads:
1. Enable managed identity on App Service
2. Assign "Storage Blob Data Contributor" role to managed identity
3. Application authenticates with Azure AD (automatic)
4. Application uploads blobs (no keys needed)
```

**Important for exam:**

- Azure AD authentication is the MOST secure option
- Should be preferred over keys when possible
- Supports managed identities
- Full Azure RBAC integration

### Encryption

Azure Storage encrypts ALL data at rest. You cannot disable it.

#### Encryption at Rest

**Simple:** All data stored in Azure is automatically encrypted.

**How it works:**

```
Your data → [Encryption] → Stored encrypted on disk → [Decryption] → Retrieved data
                256-bit AES           (happens automatically)
```

**Encryption keys - 3 options:**

**1. Microsoft-Managed Keys (default)**

- Azure manages everything
- No configuration needed
- ✅ Easy
- ❌ Less control

**2. Customer-Managed Keys (CMK)**

- You control keys in Azure Key Vault
- Can rotate keys
- Can revoke access
- Full audit trail
- ✅ More control
- ✅ Compliance requirements
- ❌ More complex

**3. Customer-Provided Keys**

- You provide key with each request
- Key never stored in Azure
- ✅ Maximum control
- ❌ Most complex
- ❌ Must manage key infrastructure

**For AZ-104:** Focus on Microsoft-managed and Customer-managed keys.

#### Encryption in Transit

**Simple:** Data encrypted while traveling over network.

**Methods:**

**HTTPS/TLS:**

- Always use HTTPS endpoints
- Enforced by default for most access
- Can require HTTPS-only (recommended)

**SMB 3.0+ Encryption:**

- Azure Files over SMB 3.0 supports encryption
- Protects data in transit
- Windows 8+, Windows Server 2012+

**Best practice:**

```powershell
# Require HTTPS only
Set-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount" `
    -EnableHttpsTrafficOnly $true
```

### Network Security

Control WHO can access storage account from WHERE.

#### Firewalls and Virtual Networks

**Simple:** Restrict access to storage account from specific networks.

**Default:** Storage accounts accessible from any internet

**Restricted access options:**

**1. Selected networks only:**

```
Allow access from:
- Specific virtual networks (VNet service endpoints)
- Specific public IP addresses/ranges
- Azure services (optional)

Block: All other access (including other Azure services)
```

**Configuration example:**

```
Firewall rules:
- Allow: VNet "VNet-Prod" (subnet "AppSubnet")
- Allow: IP range 203.0.113.0/24 (office IP)
- Allow: Azure services (for backup, monitoring)
- Deny: Everything else
```

**Use case:**

- Production storage (only accessible from your VNets)
- Compliance requirements
- Defense in depth

**2. Private endpoints:**

- Storage gets private IP in your VNet
- Accessed directly from VNet (not public internet)
- Most secure option

**3. Service endpoints:**

- Route traffic to storage through Azure backbone
- Still uses public IP but traffic never leaves Azure network
- Faster and more secure than internet

#### Private Endpoints

**Simple:** Give storage account a private IP in your VNet (storage becomes part of your network).

**How it works:**

```
Before (public endpoint):
VM → Internet → Storage Account (public IP)

After (private endpoint):
VM → VNet (private) → Storage Account (private IP: 10.0.1.5)
```

**Benefits:**

- ✅ No public IP needed
- ✅ Traffic never leaves your network
- ✅ Works with on-premises (via VPN/ExpressRoute)
- ✅ Most secure option
- ✅ Access from peered VNets

**Use case:**

- Maximum security
- Compliance requirements (no public internet access)
- Hybrid scenarios (on-prem access)

**Cost:** Additional charge for private endpoint

**Example scenario:**

```
Requirement: Database backups must never traverse public internet

Solution:
1. Create private endpoint for storage account
2. Storage gets IP 10.0.1.100 in VNet
3. SQL Server connects to 10.0.1.100
4. Traffic stays in private network
```

#### Service Endpoints

**Simple:** Route VNet traffic to storage through Azure backbone (not public internet).

**How it works:**

```
VM in VNet → Azure backbone network → Storage Account
(Still uses public storage endpoint but traffic is private)
```

**Benefits:**

- ✅ Better performance (Azure backbone is faster)
- ✅ More secure (traffic doesn't go over internet)
- ✅ Free (no additional cost)
- ✅ Simple to configure

**Limitations:**

- ❌ Still uses public IP
- ❌ Doesn't work with on-premises
- ❌ Works only from that VNet

**When to use:**

- Good balance of security and simplicity
- VNet-to-storage communication
- Don't need on-premises access

**vs Private Endpoint:**

| Feature         | Service Endpoint | Private Endpoint |
| --------------- | ---------------- | ---------------- |
| **IP Type**     | Public           | Private          |
| **On-premises** | No               | Yes              |
| **Cost**        | Free             | Paid             |
| **Security**    | Good             | Best             |
| **Setup**       | Simple           | More complex     |

### Comparison: Authentication Methods

| Method               | Security | Complexity | Use Case                              |
| -------------------- | -------- | ---------- | ------------------------------------- |
| **Storage Key**      | Low      | Simple     | Dev/test, admin tasks                 |
| **SAS Token**        | Medium   | Medium     | Temporary access, limited permissions |
| **Azure AD**         | High     | Medium     | Production apps, users                |
| **Managed Identity** | Highest  | Simple     | Azure resources accessing storage     |

**Recommendation hierarchy (best to worst):**

1. Managed Identity + Azure AD (best)
2. Azure AD with user accounts
3. User Delegation SAS
4. Service SAS with stored access policy
5. Service SAS without policy
6. Storage account keys (worst - only for admin)

---

## Data Transfer and Management

### AzCopy

**Simple:** Command-line tool for copying data to/from Azure Storage (like robocopy for Azure).

**What it does:**

- Upload files to Azure
- Download files from Azure
- Copy between storage accounts
- Sync directories

**Supported storage:**

- ✅ Blob Storage
- ✅ Azure Files
- ❌ Queue Storage (not supported)
- ❌ Table Storage (not supported)

**Key features:**

**1. High performance:**

- Parallel transfers
- Automatic retry
- Resumable transfers

**2. Authentication options:**

- Azure AD (recommended)
- SAS tokens
- Storage keys

**3. Common commands:**

**Upload file:**

```bash
azcopy copy 'C:\local\file.txt' 'https://mystorageaccount.blob.core.windows.net/container/file.txt'
```

**Upload directory (recursive):**

```bash
azcopy copy 'C:\local\folder\*' 'https://mystorageaccount.blob.core.windows.net/container' --recursive
```

**Download file:**

```bash
azcopy copy 'https://mystorageaccount.blob.core.windows.net/container/file.txt' 'C:\local\file.txt'
```

**Copy between storage accounts:**

```bash
azcopy copy 'https://source.blob.core.windows.net/container/*' 'https://dest.blob.core.windows.net/container' --recursive
```

**Sync directory (like rsync):**

```bash
azcopy sync 'C:\local\folder' 'https://mystorageaccount.blob.core.windows.net/container'
```

**With SAS token:**

```bash
azcopy copy 'C:\local\file.txt' 'https://mystorageaccount.blob.core.windows.net/container/file.txt?<sas-token>'
```

**Authentication with Azure AD:**

```bash
azcopy login
azcopy copy 'C:\local\file.txt' 'https://mystorageaccount.blob.core.windows.net/container/file.txt'
```

**Use cases:**

- Bulk upload/download
- Migration between storage accounts
- Automated backups
- CI/CD pipelines

**Best practices:**

- Use Azure AD authentication (not keys)
- Use `--recursive` for directories
- Monitor progress with `--log-level=INFO`
- Use `azcopy jobs list` to track transfers

### Azure Storage Explorer

**Simple:** GUI application for managing Azure Storage (like Windows Explorer for Azure).

**What it does:**

- Browse storage accounts
- Upload/download files
- Manage containers and shares
- Generate SAS tokens
- Set permissions
- Monitor metrics

**Supported storage:**

- ✅ Blob Storage
- ✅ Azure Files
- ✅ Queue Storage
- ✅ Table Storage
- ✅ Azure Data Lake Storage
- ✅ Managed Disks

**Authentication:**

- Azure AD
- Storage account key
- SAS token
- Connection string

**Key features:**

- Drag-and-drop upload/download
- Search across storage
- Access multiple subscriptions
- Works with local storage emulator
- Cross-platform (Windows, Mac, Linux)

**Use cases:**

- Interactive management
- Quick file transfers
- Troubleshooting
- One-time operations
- Learning Azure Storage

### Azure Import/Export Service

**Simple:** Ship physical hard drives to/from Azure datacenter for massive data transfers.

**Why needed:**

```
Problem: Need to transfer 100 TB to Azure
Internet upload: 100 TB ÷ 100 Mbps = 100+ days
Cost: $$ in bandwidth

Solution: Ship hard drives
- Load 100 TB on drives
- Ship to Microsoft
- Microsoft uploads to your storage
- Time: ~1 week
```

**Supported scenarios:**

**Import (to Azure):**

- Ship drives with data → Azure uploads to storage account

**Export (from Azure):**

- Azure copies data to drives → Ships drives to you

**Process:**

**Import job:**

```
1. Prepare drives (format, copy data, run WAImportExport tool)
2. Create import job in Azure Portal
3. Ship drives to Azure datacenter
4. Microsoft uploads data to storage account
5. Drives shipped back to you (or securely destroyed)
```

**Export job:**

```
1. Create export job in Azure Portal
2. Ship empty drives to Azure datacenter
3. Microsoft copies data to drives
4. Drives shipped back to you
```

**Requirements:**

- SATA II/III or SSD drives
- BitLocker encryption
- 2.5" or 3.5" drives
- Maximum 10 drives per job

**Use cases:**

- Initial cloud migration (TB/PB)
- Large backups
- Disaster recovery
- Poor/expensive internet connectivity
- Compliance (data cannot traverse internet)

**Cost:**

- Import: $80 + $40 per drive
- Export: $60 + $40 per drive
- Shipping costs

**Important for exam:**

- For massive data transfers (TB/PB scale)
- When network transfer not feasible
- Data encrypted with BitLocker
- Blob Storage only (not Files)

### Azure Data Box

**Simple:** Microsoft ships you physical appliances pre-loaded with your data (easier than Import/Export).

**Family of products:**

#### Data Box Disk

**Capacity:** 8 TB per disk (up to 5 disks = 40 TB)

**Use case:**

- Small to medium data transfers
- Each site needs only few TBs

**Process:**

```
1. Order in Azure Portal
2. Microsoft ships encrypted SSDs
3. Copy data to disks
4. Ship back to Microsoft
5. Microsoft uploads to your storage
```

#### Data Box

**Capacity:** 80 TB usable

**Form factor:** Rugged shipping case (weighs ~40 lbs)

**Use case:**

- Medium to large data transfers
- Single location

**Process:**
Same as Data Box Disk but larger capacity

#### Data Box Heavy

**Capacity:** 770 TB usable

**Form factor:** Full rack-mounted device (weighs ~500 lbs)

**Use case:**

- Massive data transfers
- Datacenter-scale migrations

**Process:**

```
1. Order in Azure Portal
2. Microsoft ships device
3. Connect to network
4. Copy data (can transfer while copying)
5. Ship back (Microsoft handles logistics)
```

#### Data Box Gateway

**Virtual appliance** (no physical shipping)

**Use case:**

- Continuous data transfer
- Acts as NFS/SMB gateway
- Data flows to Azure automatically

### Comparison: Transfer Methods

| Method               | Size       | Speed                    | Use Case              |
| -------------------- | ---------- | ------------------------ | --------------------- |
| **Portal Upload**    | < 1 GB     | Slow                     | Small files           |
| **AzCopy**           | < 10 TB    | Fast (network dependent) | Bulk transfers        |
| **Storage Explorer** | < 1 TB     | Medium                   | Interactive transfers |
| **Import/Export**    | 10-100+ TB | ~1 week                  | Massive datasets      |
| **Data Box Disk**    | < 40 TB    | ~1 week                  | Small datacenter      |
| **Data Box**         | < 80 TB    | ~1 week                  | Medium migration      |
| **Data Box Heavy**   | < 770 TB   | ~2 weeks                 | Large migration       |

**Decision flowchart:**

```
Data size?
  ├─ < 100 GB → AzCopy or Portal
  ├─ 100 GB - 10 TB → AzCopy (if good internet)
  ├─ 10-40 TB → Data Box Disk
  ├─ 40-80 TB → Data Box
  └─ > 80 TB → Data Box Heavy or Import/Export
```

---

## Storage Monitoring and Optimization

### Storage Analytics

**Simple:** Built-in monitoring for storage accounts (logs and metrics).

**What you get:**

**Metrics:**

- Capacity used
- Transaction count
- Request latency
- Availability
- Success/failure rates

**Logs:**

- Every storage request
- Authentication method used
- Operation type
- Success/failure
- Error codes
- Client IP addresses

**Retention:**

- Metrics: Up to 1 year
- Logs: Up to 1 year (manual deletion required)

**Available for:**

- Blob Storage
- Queue Storage
- Table Storage
- Azure Files (metrics only)

**Configuration:**

```
Enable via:
- Azure Portal (Monitoring > Diagnostic settings)
- PowerShell
- Azure CLI
- REST API
```

**Use cases:**

- Performance troubleshooting
- Security auditing
- Capacity planning
- Billing analysis

### Azure Monitor Integration

**Simple:** Advanced monitoring with alerts, dashboards, and integration with Log Analytics.

**Metrics available:**

- Transaction count
- Ingress/egress (data in/out)
- Latency (E2E and server)
- Availability percentage
- Used capacity

**Example alerts:**

```
Alert 1: Availability < 99%
Alert 2: Transaction count > 10,000/hour
Alert 3: Ingress > 10 GB/hour
Alert 4: Storage capacity > 80%
```

**Log Analytics queries:**

```
StorageBlobLogs
| where TimeGenerated > ago(1h)
| where StatusCode >= 400
| summarize count() by StatusCode, OperationName
```

### Cost Optimization

**Strategies:**

**1. Choose appropriate access tier**

```
Hot tier: Frequently accessed (daily)
Cool tier: Infrequently accessed (monthly)
Archive tier: Rarely accessed (annually)

Cost savings: Up to 90% by moving to appropriate tier
```

**2. Use lifecycle management**

```
Automate tier transitions:
- After 30 days → Cool
- After 90 days → Archive
- After 365 days → Delete

Example savings:
Before: $1,000/month (all Hot)
After: $200/month (tiered appropriately)
```

**3. Delete unnecessary data**

```
- Old logs
- Temporary files
- Abandoned projects
- Test data
```

**4. Use appropriate replication**

```
LRS vs GRS cost difference: ~2x
Use LRS for: Non-critical, dev/test
Use GRS for: Production, critical data
```

**5. Monitor and alert**

```
Set budget alerts
Review cost reports monthly
Identify unexpected growth
```

**6. Use reserved capacity**

```
1-year or 3-year commitment
Up to 38% discount
For predictable storage needs
```

---

## PowerShell Commands Reference

### Storage Account Management

```powershell
# Connect to Azure
Connect-AzAccount
Set-AzContext -SubscriptionName "Production"

# Create resource group
New-AzResourceGroup -Name "RG-Storage" -Location "eastus"

# Create storage account (Standard GPv2)
New-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount" `
    -Location "eastus" `
    -SkuName "Standard_LRS" `
    -Kind "StorageV2" `
    -AccessTier "Hot"

# Create storage account with GRS replication
New-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccountgrs" `
    -Location "eastus" `
    -SkuName "Standard_GRS" `
    -Kind "StorageV2"

# Create Premium Block Blob storage
New-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mypremiumblob" `
    -Location "eastus" `
    -SkuName "Premium_LRS" `
    -Kind "BlockBlobStorage"

# Get storage account
Get-AzStorageAccount -ResourceGroupName "RG-Storage" -Name "mystorageaccount"

# List all storage accounts in resource group
Get-AzStorageAccount -ResourceGroupName "RG-Storage"

# Update storage account (change access tier)
Set-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount" `
    -AccessTier "Cool"

# Update replication type
Set-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount" `
    -SkuName "Standard_GRS"

# Enable HTTPS-only traffic
Set-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount" `
    -EnableHttpsTrafficOnly $true

# Get storage account keys
Get-AzStorageAccountKey -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount"

# Regenerate storage account key
New-AzStorageAccountKey -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount" `
    -KeyName "key1"

# Delete storage account
Remove-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount"
```

### Blob Storage Management

```powershell
# Get storage account context
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" `
    -StorageAccountKey "<key>"

# Or with connection string
$ctx = New-AzStorageContext -ConnectionString "<connection-string>"

# Or with SAS token
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" `
    -SasToken "<sas-token>"

# Create container
New-AzStorageContainer -Name "mycontainer" -Context $ctx

# Create container with public access
New-AzStorageContainer -Name "publiccontainer" `
    -Context $ctx `
    -Permission "Blob"

# List containers
Get-AzStorageContainer -Context $ctx

# Get specific container
Get-AzStorageContainer -Name "mycontainer" -Context $ctx

# Set container access level
Set-AzStorageContainerAcl -Name "mycontainer" `
    -Context $ctx `
    -Permission "Container"

# Upload blob
Set-AzStorageBlobContent -File "C:\local\file.txt" `
    -Container "mycontainer" `
    -Blob "file.txt" `
    -Context $ctx

# Upload blob to specific tier
Set-AzStorageBlobContent -File "C:\local\backup.zip" `
    -Container "backups" `
    -Blob "backup.zip" `
    -Context $ctx `
    -StandardBlobTier "Cool"

# Upload directory (multiple files)
Get-ChildItem -Path "C:\local\folder" -File | ForEach-Object {
    Set-AzStorageBlobContent -File $_.FullName `
        -Container "mycontainer" `
        -Blob $_.Name `
        -Context $ctx
}

# Download blob
Get-AzStorageBlobContent -Container "mycontainer" `
    -Blob "file.txt" `
    -Destination "C:\local\file.txt" `
    -Context $ctx

# List blobs in container
Get-AzStorageBlob -Container "mycontainer" -Context $ctx

# List blobs with prefix
Get-AzStorageBlob -Container "mycontainer" `
    -Prefix "logs/" `
    -Context $ctx

# Copy blob between containers
Start-AzStorageBlobCopy -SrcContainer "source-container" `
    -SrcBlob "file.txt" `
    -DestContainer "dest-container" `
    -DestBlob "file.txt" `
    -Context $ctx

# Copy blob between storage accounts
$srcCtx = New-AzStorageContext -StorageAccountName "sourceaccount" -StorageAccountKey "<key>"
$destCtx = New-AzStorageContext -StorageAccountName "destaccount" -StorageAccountKey "<key>"

Start-AzStorageBlobCopy -SrcContainer "container" `
    -SrcBlob "file.txt" `
    -Context $srcCtx `
    -DestContainer "container" `
    -DestBlob "file.txt" `
    -DestContext $destCtx

# Check copy status
Get-AzStorageBlobCopyState -Container "dest-container" `
    -Blob "file.txt" `
    -Context $ctx

# Change blob tier
Set-AzStorageBlobContent -Container "mycontainer" `
    -Blob "file.txt" `
    -StandardBlobTier "Archive" `
    -Context $ctx

# Create blob snapshot
$blob = Get-AzStorageBlob -Container "mycontainer" -Blob "file.txt" -Context $ctx
$snapshot = $blob.ICloudBlob.CreateSnapshot()

# List blob snapshots
Get-AzStorageBlob -Container "mycontainer" `
    -Prefix "file.txt" `
    -Context $ctx | Where-Object { $_.ICloudBlob.IsSnapshot }

# Delete blob
Remove-AzStorageBlob -Container "mycontainer" `
    -Blob "file.txt" `
    -Context $ctx

# Delete container
Remove-AzStorageContainer -Name "mycontainer" `
    -Context $ctx `
    -Force
```

### SAS Token Generation

```powershell
# Get storage account context
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" `
    -StorageAccountKey "<key>"

# Generate account SAS (multiple services)
$accountSas = New-AzStorageAccountSASToken -Service Blob,File `
    -ResourceType Service,Container,Object `
    -Permission "rwdlacup" `
    -ExpiryTime (Get-Date).AddDays(7) `
    -Context $ctx

# Generate container SAS
$containerSas = New-AzStorageContainerSASToken -Name "mycontainer" `
    -Permission "rwl" `
    -ExpiryTime (Get-Date).AddDays(7) `
    -Context $ctx

# Generate blob SAS (read-only)
$blobSas = New-AzStorageBlobSASToken -Container "mycontainer" `
    -Blob "file.txt" `
    -Permission "r" `
    -ExpiryTime (Get-Date).AddHours(2) `
    -Context $ctx

# Generate SAS with IP restriction
$blobSas = New-AzStorageBlobSASToken -Container "mycontainer" `
    -Blob "file.txt" `
    -Permission "r" `
    -ExpiryTime (Get-Date).AddHours(2) `
    -IPAddressOrRange "203.0.113.0/24" `
    -Protocol "HttpsOnly" `
    -Context $ctx

# Full blob URL with SAS
$blobUrl = "https://mystorageaccount.blob.core.windows.net/mycontainer/file.txt$blobSas"
```

### Azure Files Management

```powershell
# Create file share
New-AzStorageShare -Name "myshare" `
    -Context $ctx `
    -QuotaGiB 100

# Create Premium file share
New-AzRmStorageShare -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mypremiumfiles" `
    -Name "premiumshare" `
    -QuotaGiB 1024

# Get file share
Get-AzStorageShare -Name "myshare" -Context $ctx

# List file shares
Get-AzStorageShare -Context $ctx

# Update file share quota
Set-AzStorageShareQuota -ShareName "myshare" `
    -Quota 200 `
    -Context $ctx

# Create directory in share
New-AzStorageDirectory -ShareName "myshare" `
    -Path "documents" `
    -Context $ctx

# Upload file to share
Set-AzStorageFileContent -ShareName "myshare" `
    -Source "C:\local\file.txt" `
    -Path "documents/file.txt" `
    -Context $ctx

# Download file from share
Get-AzStorageFileContent -ShareName "myshare" `
    -Path "documents/file.txt" `
    -Destination "C:\local\file.txt" `
    -Context $ctx

# List files in share
Get-AzStorageFile -ShareName "myshare" -Context $ctx

# Create file share snapshot
$share = Get-AzStorageShare -Name "myshare" -Context $ctx
$snapshot = $share.CloudFileShare.Snapshot()

# List file share snapshots
Get-AzStorageShare -Prefix "myshare" -Context $ctx |
    Where-Object { $_.IsSnapshot }

# Restore from snapshot
# (Need to copy files from snapshot to share)

# Delete file
Remove-AzStorageFile -ShareName "myshare" `
    -Path "documents/file.txt" `
    -Context $ctx

# Delete file share
Remove-AzStorageShare -Name "myshare" `
    -Context $ctx `
    -Force

# Get file share connection string (for mounting)
$storageAccount = Get-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount"
$key = (Get-AzStorageAccountKey -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount")[0].Value

Write-Host "Mount command:"
Write-Host "net use Z: \\$($storageAccount.StorageAccountName).file.core.windows.net\myshare /user:Azure\$($storageAccount.StorageAccountName) $key"
```

### Lifecycle Management

```powershell
# Define lifecycle management rule
$rule1 = New-AzStorageAccountManagementPolicyRule -Name "MoveToArchive" `
    -Action "tierToArchive" `
    -DaysAfterModificationGreaterThan 90 `
    -BlobType blockBlob

$rule2 = New-AzStorageAccountManagementPolicyRule -Name "DeleteOldLogs" `
    -Action "delete" `
    -DaysAfterModificationGreaterThan 365 `
    -BlobType blockBlob `
    -PrefixMatch "logs/"

# Create policy
$policy = New-AzStorageAccountManagementPolicyPolicy -Rule $rule1, $rule2

# Set lifecycle management policy
Set-AzStorageAccountManagementPolicy -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -Policy $policy

# Get lifecycle management policy
Get-AzStorageAccountManagementPolicy -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount"

# Remove lifecycle management policy
Remove-AzStorageAccountManagementPolicy -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount"
```

### Storage Security

```powershell
# Enable blob soft delete
Enable-AzStorageBlobDeleteRetentionPolicy -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -RetentionDays 7

# Enable container soft delete
Enable-AzStorageContainerDeleteRetentionPolicy -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -RetentionDays 7

# Enable blob versioning
Update-AzStorageBlobServiceProperty -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -IsVersioningEnabled $true

# Configure network rules (firewall)
# Allow specific VNet
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetworkName "VNet-Prod" `
    -ResourceGroupName "RG-Network" `
    -Name "AppSubnet"

Add-AzStorageAccountNetworkRule -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -VirtualNetworkResourceId $subnet.Id

# Allow specific IP address
Add-AzStorageAccountNetworkRule -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -IPAddressOrRange "203.0.113.5"

# Allow IP range
Add-AzStorageAccountNetworkRule -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -IPAddressOrRange "203.0.113.0/24"

# Set default action to Deny (block all except allowed)
Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -DefaultAction Deny

# Allow Azure services
az storage account update \
    --resource-group "RG-Storage" \
    --name "mystorageaccount" \
    --bypass "AzureServices"

# Remove network rule
az storage account network-rule remove \
    --resource-group "RG-Storage" \
    --account-name "mystorageaccount" \
    --ip-address "203.0.113.5"

# Create private endpoint
az network private-endpoint create \
    --resource-group "RG-Storage" \
    --name "storage-private-endpoint" \
    --location "eastus" \
    --vnet-name "VNet-Prod" \
    --subnet "AppSubnet" \
    --private-connection-resource-id "/subscriptions/{sub-id}/resourceGroups/RG-Storage/providers/Microsoft.Storage/storageAccounts/mystorageaccount" \
    --group-id "blob" \
    --connection-name "storage-connection"
```

### Lifecycle Management

```bash
# Create lifecycle management policy from JSON file
az storage account management-policy create \
    --resource-group "RG-Storage" \
    --account-name "mystorageaccount" \
    --policy @policy.json

# Example policy.json:
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"]
        }
      }
    },
    {
      "enabled": true,
      "name": "archive-old-backups",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["backups/"]
        }
      }
    }
  ]
}

# Get lifecycle management policy
az storage account management-policy show \
    --resource-group "RG-Storage" \
    --account-name "mystorageaccount"

# Delete lifecycle management policy
az storage account management-policy delete \
    --resource-group "RG-Storage" \
    --account-name "mystorageaccount"
```

---

## Common Exam Scenarios

### Scenario 1: Cannot Access Storage Account

**Problem:**
User cannot access blobs in storage account. Getting "403 Forbidden" error.

**Possible causes:**

1. **Firewall rules blocking access**

   - Check: Storage account network rules
   - Solution: Add user's IP or VNet to allowed list

2. **No proper authentication**

   - Check: Using correct access key, SAS token, or Azure AD
   - Solution: Provide valid credentials

3. **RBAC permissions missing (Azure AD auth)**

   - Check: User has appropriate RBAC role
   - Solution: Assign "Storage Blob Data Reader" or "Storage Blob Data Contributor"

4. **Container is private**

   - Check: Container access level
   - Solution: Change to Blob or Container level (if appropriate)

5. **SAS token expired**
   - Check: Token expiration time
   - Solution: Generate new SAS token

**Troubleshooting steps:**

```powershell
# Check network rules
Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount"

# Check RBAC assignments
Get-AzRoleAssignment -SignInName "user@contoso.com" `
    -Scope "/subscriptions/{sub-id}/resourceGroups/RG-Storage/providers/Microsoft.Storage/storageAccounts/mystorageaccount"

# Check container access level
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "<key>"
Get-AzStorageContainer -Name "mycontainer" -Context $ctx
```

### Scenario 2: High Storage Costs

**Problem:**
Storage costs unexpectedly high. Need to reduce costs.

**Solutions:**

1. **Move infrequently accessed data to cheaper tiers**

   - Implement lifecycle management
   - Move to Cool/Cold/Archive tiers

2. **Delete unnecessary data**

   - Old logs
   - Test/dev data
   - Abandoned projects

3. **Use appropriate replication**

   - Change GRS to LRS for non-critical data
   - Save ~50% on replication costs

4. **Enable soft delete with shorter retention**

   - Instead of 30 days, use 7 days
   - Reduces storage of deleted items

5. **Review blob versions and snapshots**
   - Old versions consuming space
   - Delete unnecessary snapshots

**Cost analysis:**

```powershell
# Get storage metrics
$storageAccount = Get-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount"

Get-AzMetric -ResourceId $storageAccount.Id `
    -MetricName "UsedCapacity" `
    -TimeGrain 01:00:00 `
    -StartTime (Get-Date).AddDays(-30)

# Implement lifecycle policy
# (See lifecycle management commands above)
```

### Scenario 3: Data Must Stay in Specific Region

**Problem:**
Compliance requires data to stay in Europe. How to ensure?

**Solution:**

1. **Create storage account in European region**

   - West Europe, North Europe, UK South, etc.

2. **Use LRS or ZRS replication**

   - Data stays in region
   - GRS would replicate to paired region (might be outside Europe)

3. **Use Azure Policy to enforce**
   - Deny storage account creation outside Europe
   - Deny GRS replication

**Implementation:**

```powershell
# Create storage in Europe with LRS
New-AzStorageAccount -ResourceGroupName "RG-Storage-EU" `
    -Name "eustorage" `
    -Location "westeurope" `
    -SkuName "Standard_LRS" `
    -Kind "StorageV2"

# Azure Policy to enforce location
$policyDef = @{
    Name = "storage-location-restriction"
    DisplayName = "Restrict Storage Account Locations"
    Policy = '{
        "if": {
            "allOf": [
                {
                    "field": "type",
                    "equals": "Microsoft.Storage/storageAccounts"
                },
                {
                    "not": {
                        "field": "location",
                        "in": ["westeurope", "northeurope"]
                    }
                }
            ]
        },
        "then": {
            "effect": "deny"
        }
    }'
}
New-AzPolicyDefinition @policyDef
```

### Scenario 4: Need Disaster Recovery for Storage

**Problem:**
Application uses storage. Need disaster recovery plan.

**Solutions:**

**Option 1: Geo-Redundant Storage (GRS/RA-GRS)**

```
Primary region: East US
Secondary region: West US (automatic)
Data replicated automatically
```

Benefits:

- Automatic replication
- RPO: ~15 minutes
- Failover: Microsoft-initiated (GRS) or always readable (RA-GRS)

**Option 2: Storage Account Failover**

```
Enable geo-replication (GRS/RA-GRS/GZRS/RA-GZRS)
If primary region fails:
- Initiate failover (customer or Microsoft)
- Secondary becomes primary
- New secondary established in different region
```

**Option 3: Application-level replication**

```
Two storage accounts in different regions
Application writes to both
More control, more complexity
```

**Implementation:**

```powershell
# Create storage with RA-GRS
New-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "drstorage" `
    -Location "eastus" `
    -SkuName "Standard_RAGRS" `
    -Kind "StorageV2"

# Initiate failover (if needed)
Invoke-AzStorageAccountFailover -ResourceGroupName "RG-Storage" `
    -Name "drstorage" `
    -Force
```

### Scenario 5: Large File Upload Failing

**Problem:**
Uploading 10 GB file to blob storage keeps failing.

**Solutions:**

1. **Use AzCopy (recommended)**

   - Designed for large files
   - Automatic retry
   - Resume capability

   ```bash
   azcopy copy "C:\large-file.zip" "https://storage.blob.core.windows.net/container/large-file.zip"
   ```

2. **Use block blob with parallel upload**

   - Split into blocks
   - Upload in parallel
   - More resilient

3. **Check network issues**

   - Timeout settings
   - Network stability
   - Firewall rules

4. **Use Azure Data Box**
   - For very large files (TB scale)
   - Ship physical drive

### Scenario 6: Shared Access for External Users

**Problem:**
Need to give external partner temporary access to specific blobs for 2 weeks.

**Best solution: SAS Token**

```powershell
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" `
    -StorageAccountKey "<key>"

# Generate SAS for specific container (read/list only)
$sas = New-AzStorageContainerSASToken -Name "shared-documents" `
    -Permission "rl" `
    -ExpiryTime (Get-Date).AddDays(14) `
    -Context $ctx `
    -Protocol "HttpsOnly"

# Provide URL to partner
$url = "https://mystorageaccount.blob.core.windows.net/shared-documents$sas"
```

**Why not:**

- ❌ Storage key: Too much access
- ❌ Azure AD: Partner not in your tenant
- ❌ Public container: Anyone could access

### Scenario 7: Compliance Requires Immutable Storage

**Problem:**
Financial records must be stored in write-once-read-many (WORM) format for 7 years.

**Solution: Blob Immutability Policies**

**Types:**

1. **Time-based retention**

   - Blobs cannot be deleted/modified for X days
   - Use for: Compliance retention

2. **Legal hold**
   - Blobs cannot be deleted/modified until hold removed
   - Use for: Active legal cases

**Implementation:**

```powershell
# Enable blob versioning (required)
Update-AzStorageBlobServiceProperty -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount" `
    -IsVersioningEnabled $true

# Set time-based retention (container level)
$container = Get-AzStorageContainer -Name "financial-records" -Context $ctx
Set-AzStorageContainerImmutabilityPolicy -Container $container `
    -ImmutabilityPeriod 2555 `
    -AllowProtectedAppendWrites $false

# 2555 days = ~7 years
```

### Scenario 8: VM Cannot Mount Azure File Share

**Problem:**
Windows VM cannot mount Azure Files share.

**Common causes:**

1. **Port 445 blocked**

   - ISP or firewall blocking SMB port
   - Solution: Use VPN/ExpressRoute or check firewall

2. **Wrong credentials**

   - Incorrect storage account name/key
   - Solution: Verify credentials

3. **Network rules blocking VM**

   - Storage firewall blocking VM's IP
   - Solution: Add VM's VNet to allowed networks

4. **SMB version mismatch**
   - Old OS doesn't support SMB 3.0
   - Solution: Update OS or use NFS

**Troubleshooting:**

```powershell
# Test port 445 connectivity
Test-NetConnection -ComputerName "mystorageaccount.file.core.windows.net" -Port 445

# Mount command
$key = "<storage-key>"
net use Z: \\mystorageaccount.file.core.windows.net\myshare /user:Azure\mystorageaccount $key

# Check network rules
Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount"
```

---

## Practice Questions

### Question 1

You have a storage account with Standard LRS replication. Management wants disaster recovery in case primary region fails. What should you do?

**A)** Enable geo-replication by changing to GRS  
**B)** Create storage account snapshot  
**C)** Enable soft delete  
**D)** Create another storage account and replicate manually

**Answer:** A

**Explanation:**

- Changing replication from LRS to GRS provides automatic geo-redundancy
- Can be done without recreating storage account
- Data automatically replicated to paired region
- B is incorrect: Snapshots don't provide geo-redundancy
- C is incorrect: Soft delete protects against deletion, not regional failure
- D is incorrect: Possible but manual and complex; GRS is simpler

### Question 2

Users need temporary read-only access to specific blobs for 24 hours. What's the most secure method?

**A)** Give them storage account key  
**B)** Make container public  
**C)** Generate SAS token with read permission and 24-hour expiry  
**D)** Create new storage account and share credentials

**Answer:** C

**Explanation:**

- SAS token provides temporary, limited access
- Can set specific permissions (read-only)
- Expires automatically after 24 hours
- A is incorrect: Storage key gives full access and doesn't expire
- B is incorrect: Makes data public to everyone
- D is incorrect: Unnecessary complexity

### Question 3

You need to move 50 TB of data from on-premises to Azure. Internet upload would take 2 months. What service should you use?

**A)** AzCopy  
**B)** Azure Storage Explorer  
**C)** Azure Data Box  
**D)** Azure File Sync

**Answer:** C

**Explanation:**

- Azure Data Box designed for offline bulk transfer
- Ship physical device with your data
- Much faster than internet upload
- A is incorrect: AzCopy still uses internet (slow for 50 TB)
- B is incorrect: GUI tool but still uses internet
- D is incorrect: File Sync is for ongoing synchronization, not initial bulk transfer

### Question 4

An application needs to access blob storage. You want most secure authentication method with no password management. What should you use?

**A)** Storage account key  
**B)** SAS token  
**C)** Managed identity with Azure AD  
**D)** Connection string

**Answer:** C

**Explanation:**

- Managed identity provides password-less authentication
- Automatically managed by Azure
- Most secure option
- A is incorrect: Requires password (key) management
- B is incorrect: Requires token management and renewal
- D is incorrect: Contains credentials that must be managed

### Question 5

You need to ensure old log files in storage account are automatically deleted after 90 days. What should you configure?

**A)** Soft delete with 90-day retention  
**B)** Lifecycle management policy  
**C)** Azure Automation runbook  
**D)** Azure Logic App

**Answer:** B

**Explanation:**

- Lifecycle management policies automatically delete/tier blobs based on age
- Native storage feature, no code needed
- A is incorrect: Soft delete is for recovery, not automatic deletion
- C and D are incorrect: Possible but unnecessary complexity; lifecycle management is built-in

### Question 6

A VM in your VNet cannot access storage account. Storage firewall is enabled. What should you do?

**A)** Disable storage firewall  
**B)** Add VM's public IP to allowed list  
**C)** Add VM's VNet/subnet to allowed networks using service endpoint  
**D)** Regenerate storage keys

**Answer:** C

**Explanation:**

- Service endpoint allows VNet access through Azure backbone
- More secure than public IP
- Best practice for VNet-to-storage communication
- A is incorrect: Reduces security
- B is incorrect: VMs in VNet don't have predictable public IPs; less secure
- D is incorrect: Keys aren't the issue; it's network access

### Question 7

You need to keep deleted blobs recoverable for 14 days. What should you enable?

**A)** Blob versioning  
**B)** Blob snapshots  
**C)** Soft delete with 14-day retention  
**D)** Lifecycle management

**Answer:** C

**Explanation:**

- Soft delete provides "recycle bin" functionality
- Deleted blobs recoverable for retention period
- A is incorrect: Versioning tracks changes, doesn't focus on deleted items
- B is incorrect: Snapshots are manual, not automatic for deletions
- D is incorrect: Lifecycle management for automatic tiering/deletion, not recovery

### Question 8

A blob contains sensitive data and must not traverse public internet. What should you implement?

**A)** HTTPS-only access  
**B)** Private endpoint  
**C)** SAS token  
**D)** Service endpoint

**Answer:** B

**Explanation:**

- Private endpoint gives storage private IP in VNet
- Traffic never leaves private network
- Most secure option
- A is incorrect: HTTPS still uses public internet (encrypted but public)
- C is incorrect: SAS controls authorization, not network path
- D is incorrect: Service endpoint uses Azure backbone but still uses public endpoint

---

## Key Takeaways for Exam

### Must-Know Concepts

1. **Storage Account Types**

   - Standard GPv2: Most common, all services, all features
   - Premium: SSD-backed, higher cost, better performance
   - Cannot change performance tier after creation

2. **Replication Options**

   - LRS: 3 copies in one datacenter (cheapest)
   - ZRS: 3 copies across availability zones
   - GRS: LRS + 3 copies in paired region
   - RA-GRS: GRS + read access to secondary
   - GZRS/RA-GZRS: ZRS primary + geo-replication

3. **Access Tiers**

   - Hot: Frequent access, highest storage cost, lowest access cost
   - Cool: Infrequent access (30+ days), medium cost
   - Cold: Rare access (90+ days), lower cost
   - Archive: Almost never (180+ days), lowest storage cost, hours to retrieve

4. **Blob Types**

   - Block blob: Files, documents, images (most common)
   - Append blob: Logs, append-only data
   - Page blob: VM disks, random access

5. **Authentication Methods**

   - Storage keys: Full access, manage carefully
   - SAS tokens: Temporary, limited permissions (preferred for delegation)
   - Azure AD: Most secure, use with managed identities
   - Managed identity: Best for Azure resources accessing storage

6. **Security Best Practices**

   - Always use HTTPS
   - Use Azure AD when possible
   - Implement network restrictions (firewall, private endpoints)
   - Enable soft delete
   - Rotate keys regularly
   - Never put keys in code

7. **Azure Files vs Blob Storage**

   - Files: File system interface, SMB/NFS, lift-and-shift
   - Blob: REST API, cloud-native apps, unstructured data

8. **Data Transfer**
   - Small (< 10 GB): Portal, Storage Explorer
   - Medium (10 GB - 10 TB): AzCopy
   - Large (> 10 TB): Import/Export, Data Box

### Common Exam Tricks

**Trick 1: "Change Standard to Premium"**

- Cannot change performance tier after creation
- Must create new account and migrate data

**Trick 2: "GRS replicates instantly"**

- GRS has ~15 minute replication lag
- Not synchronous replication

**Trick 3: "Archive tier provides instant access"**

- Archive requires rehydration (hours)
- Cannot read directly

**Trick 4: "Container public access is secure"**

- Container-level access makes ALL blobs public
- Use Blob-level or Private for security

**Trick 5: "Soft delete prevents all data loss"**

- Soft delete has retention period
- After retention, data permanently deleted
- Must recover within retention period

**Trick 6: "Storage keys can have specific permissions"**

- Storage keys provide FULL access
- Cannot limit what keys can do
- Use SAS for limited permissions

**Trick 7: "Lifecycle policies apply immediately"**

- Policies run once per day
- Not instant

**Trick 8: "Service endpoint works from on-premises"**

- Service endpoints only work from VNet
- Use private endpoint for on-premises access

### Quick Reference Tables

**Replication Comparison:**

| Type   | Copies | Locations               | DR        | Cost |
| ------ | ------ | ----------------------- | --------- | ---- |
| LRS    | 3      | 1 datacenter            | ❌        | $    |
| ZRS    | 3      | 3 zones                 | ❌        | $    |
| GRS    | 6      | 2 regions               | ✅        | $$   |
| RA-GRS | 6      | 2 regions               | ✅ + Read | $$   |
| GZRS   | 6      | 2 regions (ZRS primary) | ✅        | $$   |

**Access Tier Decision:**

| Access Frequency     | Recommended Tier |
| -------------------- | ---------------- |
| Multiple times daily | Hot              |
| Weekly/Monthly       | Cool             |
| Quarterly            | Cold             |
| Annually or less     | Archive          |

**Authentication Method Selection:**

| Scenario                         | Best Method                       |
| -------------------------------- | --------------------------------- |
| Azure resource accessing storage | Managed Identity                  |
| User access via application      | Azure AD                          |
| Temporary external access        | SAS Token                         |
| Admin/management tasks           | Storage Key (stored in Key Vault) |

### Top 10 PowerShell Commands

```powershell
# 1. Create storage account
New-AzStorageAccount -ResourceGroupName "RG" -Name "storage" -Location "eastus" -SkuName "Standard_LRS"

# 2. Get storage context
$ctx = New-AzStorageContext -StorageAccountName "storage" -StorageAccountKey "<key>"

# 3. Create container
New-AzStorageContainer -Name "container" -Context $ctx

# 4. Upload blob
Set-AzStorageBlobContent -File "file.txt" -Container "container" -Blob "file.txt" -Context $ctx

# 5. Download blob
Get-AzStorageBlobContent -Container "container" -Blob "file.txt" -Destination "file.txt" -Context $ctx

# 6. Generate SAS token
New-AzStorageContainerSASToken -Name "container" -Permission "rl" -ExpiryTime (Get-Date).AddDays(7) -Context $ctx

# 7. Create file share
New-AzStorageShare -Name "share" -Context $ctx

# 8. Enable soft delete
Enable-AzStorageBlobDeleteRetentionPolicy -ResourceGroupName "RG" -StorageAccountName "storage" -RetentionDays 7

# 9. Configure firewall
Add-AzStorageAccountNetworkRule -ResourceGroupName "RG" -StorageAccountName "storage" -IPAddressOrRange "203.0.113.5"

# 10. Get storage keys
Get-AzStorageAccountKey -ResourceGroupName "RG" -Name "storage"
```

### Top 10 Azure CLI Commands

```bash
# 1. Create storage account
az storage account create --resource-group "RG" --name "storage" --location "eastus" --sku "Standard_LRS"

# 2. Get storage key
az storage account keys list --resource-group "RG" --account-name "storage" --query "[0].value" -o tsv

# 3. Create container
az storage container create --name "container" --account-name "storage"

# 4. Upload blob
az storage blob upload --container-name "container" --file "file.txt" --name "file.txt" --account-name "storage"

# 5. Download blob
az storage blob download --container-name "container" --name "file.txt" --file "file.txt" --account-name "storage"

# 6. Generate SAS token
az storage container generate-sas --name "container" --permissions "rl" --expiry "2024-12-31" --account-name "storage"

# 7. Create file share
az storage share create --name "share" --account-name "storage"

# 8. Enable soft delete
az storage account blob-service-properties update --resource-group "RG" --account-name "storage" --enable-delete-retention true --delete-retention-days 7

# 9. Configure firewall
az storage account network-rule add --resource-group "RG" --account-name "storage" --ip-address "203.0.113.5"

# 10. List blobs
az storage blob list --container-name "container" --account-name "storage" --output table
```

---

## Summary Checklist

Before moving to Module 3, ensure you can:

**Storage Accounts:**

- [ ] Explain storage account types and when to use each
- [ ] Understand performance tiers (Standard vs Premium)
- [ ] Describe all replication options (LRS, ZRS, GRS, etc.)
- [ ] Create and configure storage accounts
- [ ] Manage storage account keys

**Blob Storage:**

- [ ] Explain blob types (Block, Append, Page)
- [ ] Understand access tiers and when to use each
- [ ] Create containers with appropriate access levels
- [ ] Upload and download blobs
- [ ] Configure lifecycle management policies
- [ ] Implement blob versioning and snapshots

**Azure Files:**

- [ ] Understand SMB vs NFS protocols
- [ ] Create and manage file shares
- [ ] Mount file shares from Windows/Linux
- [ ] Configure file share quotas
- [ ] Explain Azure File Sync

**Security:**

- [ ] Generate and use SAS tokens
- [ ] Implement Azure AD authentication
- [ ] Configure storage firewalls
- [ ] Create private endpoints
- [ ] Enable soft delete and versioning
- [ ] Manage encryption (at rest and in transit)

**Data Transfer:**

- [ ] Use AzCopy for bulk transfers
- [ ] Use Azure Storage Explorer
- [ ] Understand when to use Import/Export or Data Box
- [ ] Choose appropriate transfer method based on data size

**Monitoring:**

- [ ] Enable storage analytics
- [ ] View metrics and logs
- [ ] Set up alerts
- [ ] Optimize costs

**PowerShell & CLI:**

- [ ] Execute common storage management commands
- [ ] Create and manage blobs and containers
- [ ] Generate SAS tokens
- [ ] Configure security settings

**Exam Readiness:**

- [ ] Complete practice questions with 80%+ accuracy
- [ ] Explain scenarios without looking at notes
- [ ] Perform hands-on labs for all topics
- [ ] Can troubleshoot common issues

---

## Next Steps

Congratulations on completing Module 2!

**You've mastered:**

- Storage account types and configuration
- Blob storage and lifecycle management
- Azure Files and file sharing
- Storage security and access control
- Data transfer methods

**Next Module:** Module 3: Compute

In Module 3, you'll learn:

- Virtual Machines (creation, sizing, availability)
- VM Scale Sets
- Azure App Service
- Azure Container Instances
- Azure Kubernetes Service (AKS) basics
- VM backup and disaster recovery

**Before moving on:**

1. Complete practice labs for Module 2
2. Take the Module 2 practice quiz
3. Review any weak areas
4. Practice PowerShell/CLI commands

---

**Good luck with your studies!**

Remember: Storage is one of the most tested topics on AZ-104. Make sure you understand authentication methods, replication options, and access tiers thoroughly.

**[← Back to Module 1](../Module-01-Identity-Governance/README.md) | [Next: Module 3: Compute →](../Module-03-Compute/README.md)**
Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "RG-Storage" `    -StorageAccountName "mystorageaccount"`
-Bypass AzureServices

# Remove network rule

Remove-AzStorageAccountNetworkRule -ResourceGroupName "RG-Storage" `    -StorageAccountName "mystorageaccount"`
-IPAddressOrRange "203.0.113.5"

# Create private endpoint

$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetworkName "VNet-Prod" `    -ResourceGroupName "RG-Network"`
-Name "AppSubnet"

$storageAccount = Get-AzStorageAccount -ResourceGroupName "RG-Storage" `
-Name "mystorageaccount"

$privateEndpoint = New-AzPrivateEndpoint -Name "storage-private-endpoint" `    -ResourceGroupName "RG-Storage"`
-Location "eastus" `    -Subnet $subnet`
-PrivateLinkServiceConnection (New-AzPrivateLinkServiceConnection `        -Name "storage-connection"`
-PrivateLinkServiceId $storageAccount.Id `
-GroupId "blob")

````

### Monitoring and Diagnostics

```powershell
# Enable diagnostic settings (send to Log Analytics)
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "RG-Monitoring" `
    -Name "LogAnalyticsWorkspace"

$storageAccount = Get-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "mystorageaccount"

$metric = New-AzDiagnosticSettingMetricSettingsObject -Enabled $true `
    -Category "Transaction"

$log = New-AzDiagnosticSettingLogSettingsObject -Enabled $true `
    -Category "StorageRead"

New-AzDiagnosticSetting -Name "storage-diagnostics" `
    -ResourceId "$($storageAccount.Id)/blobServices/default" `
    -WorkspaceId $workspace.ResourceId `
    -Metric $metric `
    -Log $log

# Get storage account metrics
Get-AzMetric -ResourceId $storageAccount.Id `
    -MetricName "UsedCapacity" `
    -TimeGrain 01:00:00 `
    -StartTime (Get-Date).AddDays(-7) `
    -EndTime (Get-Date)

# Get blob service properties
Get-AzStorageBlobServiceProperty -ResourceGroupName "RG-Storage" `
    -StorageAccountName "mystorageaccount"
````

---

## Azure CLI Commands Reference

### Storage Account Management

```bash
# Login to Azure
az login
az account set --subscription "Production"

# Create resource group
az group create --name "RG-Storage" --location "eastus"

# Create storage account (Standard GPv2)
az storage account create \
    --resource-group "RG-Storage" \
    --name "mystorageaccount" \
    --location "eastus" \
    --sku "Standard_LRS" \
    --kind "StorageV2" \
    --access-tier "Hot"

# Create storage account with GRS
az storage account create \
    --resource-group "RG-Storage" \
    --name "mystorageaccountgrs" \
    --location "eastus" \
    --sku "Standard_GRS" \
    --kind "StorageV2"

# Create Premium Block Blob storage
az storage account create \
    --resource-group "RG-Storage" \
    --name "mypremiumblob" \
    --location "eastus" \
    --sku "Premium_LRS" \
    --kind "BlockBlobStorage"

# Get storage account
az storage account show \
    --resource-group "RG-Storage" \
    --name "mystorageaccount"

# List all storage accounts
az storage account list --resource-group "RG-Storage" --output table

# Update storage account (change access tier)
az storage account update \
    --resource-group "RG-Storage" \
    --name "mystorageaccount" \
    --access-tier "Cool"

# Update replication type
az storage account update \
    --resource-group "RG-Storage" \
    --name "mystorageaccount" \
    --sku "Standard_GRS"

# Enable HTTPS-only traffic
az storage account update \
    --resource-group "RG-Storage" \
    --name "mystorageaccount" \
    --https-only true

# Get storage account keys
az storage account keys list \
    --resource-group "RG-Storage" \
    --account-name "mystorageaccount"

# Regenerate storage account key
az storage account keys renew \
    --resource-group "RG-Storage" \
    --account-name "mystorageaccount" \
    --key "key1"

# Delete storage account
az storage account delete \
    --resource-group "RG-Storage" \
    --name "mystorageaccount" \
    --yes
```

### Blob Storage Management

```bash
# Get storage account key (for authentication)
KEY=$(az storage account keys list \
    --resource-group "RG-Storage" \
    --account-name "mystorageaccount" \
    --query "[0].value" -o tsv)

# Create container
az storage container create \
    --name "mycontainer" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Create container with public access
az storage container create \
    --name "publiccontainer" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --public-access "blob"

# List containers
az storage container list \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --output table

# Set container access level
az storage container set-permission \
    --name "mycontainer" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --public-access "container"

# Upload blob
az storage blob upload \
    --container-name "mycontainer" \
    --file "C:\local\file.txt" \
    --name "file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Upload blob to specific tier
az storage blob upload \
    --container-name "backups" \
    --file "backup.zip" \
    --name "backup.zip" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --tier "Cool"

# Upload directory
az storage blob upload-batch \
    --destination "mycontainer" \
    --source "C:\local\folder" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Download blob
az storage blob download \
    --container-name "mycontainer" \
    --name "file.txt" \
    --file "C:\local\file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Download directory
az storage blob download-batch \
    --source "mycontainer" \
    --destination "C:\local\folder" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# List blobs
az storage blob list \
    --container-name "mycontainer" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --output table

# List blobs with prefix
az storage blob list \
    --container-name "mycontainer" \
    --prefix "logs/" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Copy blob
az storage blob copy start \
    --source-container "source-container" \
    --source-blob "file.txt" \
    --destination-container "dest-container" \
    --destination-blob "file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Check copy status
az storage blob show \
    --container-name "dest-container" \
    --name "file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --query "properties.copy"

# Change blob tier
az storage blob set-tier \
    --container-name "mycontainer" \
    --name "file.txt" \
    --tier "Archive" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Create blob snapshot
az storage blob snapshot \
    --container-name "mycontainer" \
    --name "file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Delete blob
az storage blob delete \
    --container-name "mycontainer" \
    --name "file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Delete container
az storage container delete \
    --name "mycontainer" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"
```

### SAS Token Generation

```bash
# Generate account SAS
az storage account generate-sas \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --services "bf" \
    --resource-types "sco" \
    --permissions "rwdlacup" \
    --expiry "2024-12-31T23:59:00Z" \
    --https-only

# Generate container SAS
az storage container generate-sas \
    --name "mycontainer" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --permissions "rwl" \
    --expiry "2024-12-31T23:59:00Z" \
    --https-only

# Generate blob SAS (read-only)
az storage blob generate-sas \
    --container-name "mycontainer" \
    --name "file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --permissions "r" \
    --expiry "2024-06-15T12:00:00Z" \
    --https-only

# Generate SAS with IP restriction
az storage blob generate-sas \
    --container-name "mycontainer" \
    --name "file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --permissions "r" \
    --expiry "2024-06-15T12:00:00Z" \
    --ip "203.0.113.0/24" \
    --https-only
```

### Azure Files Management

```bash
# Create file share
az storage share create \
    --name "myshare" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --quota 100

# Get file share
az storage share show \
    --name "myshare" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# List file shares
az storage share list \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --output table

# Update file share quota
az storage share update \
    --name "myshare" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --quota 200

# Create directory in share
az storage directory create \
    --share-name "myshare" \
    --name "documents" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Upload file to share
az storage file upload \
    --share-name "myshare" \
    --source "C:\local\file.txt" \
    --path "documents/file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Download file from share
az storage file download \
    --share-name "myshare" \
    --path "documents/file.txt" \
    --dest "C:\local\file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# List files in share
az storage file list \
    --share-name "myshare" \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --output table

# Create file share snapshot
az storage share snapshot \
    --name "myshare" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# List file share snapshots
az storage share list \
    --account-name "mystorageaccount" \
    --account-key "$KEY" \
    --include-snapshot \
    --query "[?snapshot!=null]" \
    --output table

# Delete file
az storage file delete \
    --share-name "myshare" \
    --path "documents/file.txt" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"

# Delete file share
az storage share delete \
    --name "myshare" \
    --account-name "mystorageaccount" \
    --account-key "$KEY"
```

---

**Good luck with your studies! You're doing great! 💪**

Remember: Consistent practice is more important than cramming. Take breaks, do hands-on labs, and don't move forward until you're comfortable with the material.

**[← Back to Module 1: Identity Governance](../Module-01-Identity-Governance/README.md) | [Next: Module 3: Compute →](../Module-03-Compute/README.md)**
