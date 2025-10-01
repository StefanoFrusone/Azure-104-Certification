# Module 3: Compute

## Table of Contents

- [Introduction](#introduction)
- [Virtual Machines](#virtual-machines)
- [VM Availability and Scaling](#vm-availability-and-scaling)
- [Azure App Service](#azure-app-service)
- [Azure Container Instances](#azure-container-instances)
- [Azure Kubernetes Service (AKS)](#azure-kubernetes-service-aks)
- [VM Backup and Disaster Recovery](#vm-backup-and-disaster-recovery)
- [PowerShell Commands Reference](#powershell-commands-reference)
- [Azure CLI Commands Reference](#azure-cli-commands-reference)
- [Common Exam Scenarios](#common-exam-scenarios)

---

## Introduction

**What is this module about?**

This module covers Azure Compute services - the resources that run your applications and workloads in Azure.

**Real-world analogy:**
Think of compute resources like different types of vehicles:

- **Virtual Machines** = Owning a car (full control, maintenance responsibility)
- **App Service** = Taking a taxi (just tell where to go, someone else drives)
- **Container Instances** = Renting a car for short trips (lightweight, quick)
- **Kubernetes** = Managing a fleet of vehicles (orchestration at scale)

**Key questions this module answers:**

1. **How** do I run applications in Azure? (VMs, App Service, Containers)
2. **How** do I ensure high availability? (Availability Sets, Zones, Scale Sets)
3. **How** do I scale applications? (Manual, automatic scaling)
4. **How** do I protect compute resources? (Backup, disaster recovery)
5. **When** should I use which compute service? (Decision criteria)

**Exam Weight:** 20-25% of the exam

---

## Virtual Machines

### What is a Virtual Machine?

**Simple explanation:**
A virtual machine is a computer running inside another computer. It acts like a real physical computer but exists only as software.

**Technical definition:**
An Azure Virtual Machine is an on-demand, scalable computing resource that provides the flexibility of virtualization without having to buy and maintain physical hardware.

**When to use VMs:**

- Lift-and-shift migrations from on-premises
- Full control over operating system needed
- Custom software requiring specific configurations
- Development and testing environments
- Running legacy applications

**When NOT to use VMs:**

- Simple web applications (use App Service instead)
- Stateless workloads (use containers instead)
- Don't need OS-level control (use PaaS instead)

### VM Components

When you create a VM, Azure creates several resources:

**1. Virtual Machine (the compute)**

- CPU, memory, storage
- The actual VM resource

**2. OS Disk**

- Operating system installation
- Typically 127 GB (Windows) or 30 GB (Linux)
- Automatically created

**3. Network Interface Card (NIC)**

- Connects VM to virtual network
- Has private IP address
- Can have public IP attached

**4. Network Security Group (NSG)**

- Firewall rules
- Controls inbound/outbound traffic
- Can be attached to NIC or subnet

**5. Public IP Address (optional)**

- Allows internet access to VM
- Static or dynamic
- Costs extra

**6. Virtual Network (VNet)**

- Private network for your resources
- Provides isolation and segmentation

**7. Storage Account (for diagnostics)**

- Stores boot diagnostics
- Logs and screenshots

**Relationship:**

```
Resource Group
  ├─ Virtual Machine
  ├─ OS Disk (Managed Disk)
  ├─ Data Disk (optional)
  ├─ Network Interface
  │   ├─ Private IP
  │   └─ Public IP (optional)
  ├─ Network Security Group
  ├─ Virtual Network
  └─ Storage Account (diagnostics)
```

### VM Sizes and Families

Azure offers different VM sizes optimized for different workloads.

**Format:** `Family_Version_Size`

Example: `Standard_D4s_v3`

- `Standard` = Pricing tier
- `D` = Family (general purpose)
- `4` = Number of vCPUs
- `s` = Premium storage support
- `v3` = Generation

#### VM Families

**General Purpose (B, D, DS, Dv2, DSv2, Dav4, Dv4)**

**Use case:** Balanced CPU-to-memory ratio

**Good for:**

- Development/test servers
- Small to medium databases
- Low to medium traffic web servers
- Application servers

**Example sizes:**

- `B1s`: 1 vCPU, 1 GB RAM (smallest, cheapest)
- `B2ms`: 2 vCPU, 8 GB RAM
- `D4s_v3`: 4 vCPU, 16 GB RAM
- `D16s_v3`: 16 vCPU, 64 GB RAM

**B-Series (Burstable):**

- Accumulate credits when idle
- Burst to full CPU when needed
- Perfect for workloads with low baseline CPU usage
- Development, test, low-traffic websites

**Compute Optimized (F, FS, Fsv2)**

**Use case:** High CPU-to-memory ratio

**Good for:**

- Batch processing
- Web servers with high traffic
- Application servers
- Network appliances
- Gaming servers

**Example sizes:**

- `F2s_v2`: 2 vCPU, 4 GB RAM (high CPU)
- `F8s_v2`: 8 vCPU, 16 GB RAM
- `F32s_v2`: 32 vCPU, 64 GB RAM

**Memory Optimized (E, ES, Ev3, Esv3, M)**

**Use case:** High memory-to-CPU ratio

**Good for:**

- Relational databases (SQL Server, MySQL)
- Large caches (Redis)
- In-memory analytics
- SAP HANA

**Example sizes:**

- `E4s_v3`: 4 vCPU, 32 GB RAM (1:8 ratio)
- `E8s_v3`: 8 vCPU, 64 GB RAM
- `E64s_v3`: 64 vCPU, 432 GB RAM

**M-Series (largest memory):**

- Up to 4 TB of RAM
- SAP HANA certified
- Mission-critical workloads

**Storage Optimized (L, LS)**

**Use case:** High disk throughput and IOPS

**Good for:**

- Big data applications
- SQL/NoSQL databases
- Data warehousing
- Large transactional databases

**Example sizes:**

- `L8s_v2`: 8 vCPU, 64 GB RAM, high IOPS
- `L16s_v2`: 16 vCPU, 128 GB RAM

**GPU (N-Series: NC, NCv3, ND, NV)**

**Use case:** GPU-accelerated computing

**Good for:**

- Machine learning training
- 3D rendering
- Video encoding
- Remote visualization
- Gaming

**Example sizes:**

- `NC6`: 6 vCPU, 56 GB RAM, 1 GPU
- `NV6`: 6 vCPU, 56 GB RAM, 1 GPU (graphics)

**High Performance Compute (H-Series)**

**Use case:** Massive compute power

**Good for:**

- Computational fluid dynamics
- Weather simulations
- Financial risk modeling
- Molecular modeling

### VM Size Selection Guide

**Decision flowchart:**

```
What's your workload?
  ├─ Web server (medium traffic) → D-series (General Purpose)
  ├─ Database (SQL Server) → E-series (Memory Optimized)
  ├─ Batch processing → F-series (Compute Optimized)
  ├─ Big data → L-series (Storage Optimized)
  ├─ ML training → N-series (GPU)
  └─ Dev/test (low usage) → B-series (Burstable)
```

**Important for exam:**

- You can resize VMs (requires restart)
- Not all sizes available in all regions
- Premium storage requires "s" sizes (e.g., D4s_v3)

### VM Disks

**Disk Types:**

**1. OS Disk**

- Contains operating system
- Registered as SATA drive
- Default size: 127 GB (Windows), 30 GB (Linux)
- Can be expanded
- Drive letter: C: (Windows), /dev/sda (Linux)

**2. Temporary Disk**

- Local SSD storage (not persistent)
- Lost during maintenance or reboot
- Very fast
- Drive letter: D: (Windows), /dev/sdb (Linux)
- Use for: Page files, temp data, caching
- DO NOT use for: Important data

**3. Data Disks**

- Additional persistent storage
- Attached as SCSI drives
- Up to 32 TB per disk
- Up to 64 disks per VM (depends on VM size)
- Use for: Databases, application data, user files

**Disk Performance Tiers:**

**Standard HDD**

- Magnetic drives
- Lowest cost
- Use for: Backups, non-critical, infrequent access
- IOPS: Up to 500
- Throughput: Up to 60 MB/s

**Standard SSD**

- Solid-state drives
- Medium performance and cost
- Use for: Web servers, light enterprise apps, dev/test
- IOPS: Up to 6,000
- Throughput: Up to 750 MB/s

**Premium SSD**

- High-performance SSD
- Production workloads
- Use for: Databases, high-traffic applications
- IOPS: Up to 20,000
- Throughput: Up to 900 MB/s
- Requires "s" VM sizes

**Ultra Disk**

- Highest performance
- Mission-critical workloads
- Use for: SAP HANA, top-tier databases, transaction-heavy workloads
- IOPS: Up to 160,000
- Throughput: Up to 4,000 MB/s
- Most expensive

**Disk Comparison:**

| Disk Type    | IOPS    | Throughput | Cost | Use Case                |
| ------------ | ------- | ---------- | ---- | ----------------------- |
| Standard HDD | 500     | 60 MB/s    | $    | Backups, dev/test       |
| Standard SSD | 6,000   | 750 MB/s   | $$   | Web servers, light apps |
| Premium SSD  | 20,000  | 900 MB/s   | $$$  | Production databases    |
| Ultra Disk   | 160,000 | 4,000 MB/s | $$$$ | Mission-critical        |

**Managed vs Unmanaged Disks:**

**Managed Disks (recommended):**

- Azure manages storage account
- Simpler management
- Better availability (automatically spread across storage units)
- No storage account limits
- Easier backup/restore
- Role-based access control

**Unmanaged Disks (legacy):**

- You manage storage account
- VHD files in blob storage
- Manual distribution for availability
- Storage account limits (40 VHDs per account)
- More complex

**Always use Managed Disks for new deployments.**

### VM Networking

**Private IP Address:**

- Assigned from VNet subnet
- Used for communication within VNet
- Static or dynamic allocation
- Required for every VM

**Public IP Address:**

- Optional
- Allows internet access to VM
- Static or dynamic
- Costs money
- Basic or Standard SKU

**Public IP SKU comparison:**

**Basic:**

- Dynamic or static
- Open by default (use NSG to secure)
- No availability zone support
- Free tier available

**Standard:**

- Always static
- Secure by default (NSG required)
- Availability zone support
- Recommended for production
- Always charged

**Network Security Group (NSG):**

- Virtual firewall
- Controls inbound and outbound traffic
- Rules based on:
  - Source/destination IP
  - Port
  - Protocol (TCP, UDP, Any)
- Can be applied to:
  - Network interface
  - Subnet

**NSG Rule Components:**

- **Priority:** 100-4096 (lower number = higher priority)
- **Name:** Descriptive name
- **Source:** IP, service tag, application security group
- **Destination:** IP, service tag, application security group
- **Port:** Single port or range
- **Protocol:** TCP, UDP, ICMP, Any
- **Action:** Allow or Deny

**Example NSG rules:**

```
Rule 1: Priority 100
Allow RDP (port 3389) from 203.0.113.5 (your office IP)

Rule 2: Priority 200
Allow HTTP (port 80) from Internet

Rule 3: Priority 300
Allow HTTPS (port 443) from Internet

Rule 4: Priority 65000 (default)
Deny all other traffic
```

**Important:**

- Rules evaluated in priority order (lowest first)
- Once match found, no more rules evaluated
- Default rules exist (can't be deleted, lowest priority)

### VM Creation Options

**Portal:**

- GUI interface
- Step-by-step wizard
- Good for learning
- Not repeatable

**PowerShell:**

- Scriptable
- Repeatable
- Automation
- Version control

**Azure CLI:**

- Cross-platform
- Scriptable
- Similar to PowerShell

**ARM Templates:**

- Declarative JSON
- Infrastructure as Code
- Repeatable deployments
- Version control

**Terraform:**

- Third-party IaC tool
- Multi-cloud
- Popular in enterprises

### VM States and Billing

**VM States:**

**Starting:**

- VM is starting up
- Not billed for compute
- Billed for storage

**Running:**

- VM is running
- Fully billed (compute + storage)

**Stopping:**

- VM is shutting down
- Still billed for compute (until stopped)

**Stopped (Allocated):**

- VM stopped from inside OS
- Still allocated in Azure
- Still billed for compute + storage
- Keeps IP address

**Stopped (Deallocated):**

- VM stopped from Azure
- Resources released
- Not billed for compute
- Billed only for storage
- May lose public IP (if dynamic)

**Important for exam:**

```
Stop from inside OS → Still billed
Stop from Azure Portal/PowerShell → Not billed (deallocated)

Use: Stop-AzVM -ResourceGroupName "RG" -Name "VM" (PowerShell)
Or: az vm deallocate --resource-group "RG" --name "VM" (CLI)
```

### VM Extensions

**Simple:** Add-ons that provide post-deployment configuration and automation.

**Common extensions:**

**Custom Script Extension:**

- Run PowerShell/Bash scripts
- Configure VM after deployment
- Install software
- Join domain

**Desired State Configuration (DSC):**

- Declarative configuration management
- Ensure VM stays in desired state
- Windows only (PowerShell DSC)

**Azure Monitor Agent:**

- Collect metrics and logs
- Send to Log Analytics
- Required for monitoring

**Antimalware:**

- Microsoft Antimalware
- Real-time protection
- Windows only

**Network Watcher Agent:**

- Network diagnostics
- Connection troubleshooting

**Example use cases:**

```
After VM creation:
1. Install IIS web server
2. Configure firewall rules
3. Join Active Directory domain
4. Install monitoring agent
5. Configure backup

All automated via extensions
```

---

## VM Availability and Scaling

### Availability Concepts

**Availability:** Percentage of time a service is operational

**SLA (Service Level Agreement):** Azure's uptime guarantee

**Single VM SLAs:**

- Premium SSD: 99.9% (8.76 hours downtime/year)
- Standard SSD: 99.5%
- Standard HDD: 95%

**To get higher SLA, use:**

- Availability Sets
- Availability Zones
- VM Scale Sets

### Availability Sets

**Simple:** Logical grouping of VMs that protects against hardware failures in a datacenter.

**How it works:**

**Fault Domains (FD):**

- Physical server racks
- Each rack has own power and network
- Azure spreads VMs across fault domains
- Default: 2 FDs (can be 3)

**Update Domains (UD):**

- Logical groups for maintenance
- During planned maintenance, only one UD rebooted at a time
- Default: 5 UDs (can be up to 20)

**Visual representation:**

```
Availability Set with 3 VMs:

Fault Domain 1        Fault Domain 2
┌─────────────┐       ┌─────────────┐
│   VM1       │       │   VM2       │
│ (UD 0)      │       │ (UD 1)      │
└─────────────┘       └─────────────┘
│   VM3       │
│ (UD 2)      │
└─────────────┘

If FD1 fails → VM2 still available
If UD0 maintenance → VM2 and VM3 still available
```

**SLA with Availability Set:** 99.95% (4.38 hours downtime/year)

**Important rules:**

- Must create availability set before VMs
- VMs must be in same availability set from creation
- Cannot add existing VM to availability set
- All VMs in set must be same size
- Free (no additional cost)

**When to use:**

- Multi-tier applications (web, app, database tiers)
- Active-active configurations
- Need higher SLA than single VM

**Example:**

```
3-tier application:
- Availability Set 1: Web tier (2 VMs)
- Availability Set 2: App tier (2 VMs)
- Availability Set 3: Database tier (2 VMs)

If one rack fails, one VM from each tier still available
```

### Availability Zones

**Simple:** Physically separate datacenters within same Azure region.

**How it works:**

- Each region has multiple zones (minimum 3)
- Each zone = separate building(s)
- Own power, cooling, networking
- Connected via high-speed private network
- 1-2 ms latency between zones

**Zone redundancy options:**

**1. Zonal deployment:**

- Pin VM to specific zone
- Example: VM in Zone 1
- Protection: Datacenter failure
- No protection if entire zone fails

**2. Zone-redundant deployment:**

- Spread resources across zones
- Example: Load balancer with backend VMs in Zone 1, 2, 3
- Protection: Entire zone can fail
- Highest availability

**Visual representation:**

```
Region: East US

Zone 1          Zone 2          Zone 3
┌─────┐         ┌─────┐         ┌─────┐
│ VM1 │         │ VM2 │         │ VM3 │
└─────┘         └─────┘         └─────┘
   ↓               ↓               ↓
      Load Balancer (zone-redundant)
               ↓
            Users
```

**SLA with Availability Zones:** 99.99% (52.6 minutes downtime/year)

**Important:**

- Not all regions support zones
- Not all VM sizes available in all zones
- Higher cost (cross-zone bandwidth charges)
- Best practice for production workloads

**Availability Zones vs Availability Sets:**

| Feature      | Availability Set | Availability Zone  |
| ------------ | ---------------- | ------------------ |
| Protection   | Rack failure     | Datacenter failure |
| SLA          | 99.95%           | 99.99%             |
| Latency      | <1 ms            | 1-2 ms             |
| Availability | All regions      | Select regions     |
| Cost         | Free             | Bandwidth charges  |

**Decision guide:**

- Maximum availability needed? → Availability Zones
- Region doesn't support zones? → Availability Sets
- Cost-sensitive? → Availability Sets
- Can tolerate 1-2 ms latency? → Availability Zones

### VM Scale Sets

**Simple:** Group of identical, load-balanced VMs that automatically scale based on demand.

**The problem:**

```
Fixed number of VMs:
- Peak traffic: Overloaded (slow/crashes)
- Low traffic: Wasted money (idle VMs)
```

**The solution:**

```
VM Scale Set:
- High traffic: Automatically add VMs
- Low traffic: Automatically remove VMs
- Always right capacity
```

**How it works:**

**Components:**

1. **Scale Set:** Group of identical VMs
2. **Load Balancer:** Distributes traffic
3. **Scaling Rules:** When to scale in/out
4. **VM Image:** Template for new VMs

**Scaling modes:**

**Manual Scaling:**

- You set instance count
- Fixed number of VMs
- Use for: Predictable workloads

**Automatic Scaling (Autoscale):**

- Azure adds/removes VMs based on metrics
- Metric-based (CPU, memory, HTTP queue)
- Schedule-based (time of day, day of week)

**Autoscale rules example:**

```
Scale out (add VMs):
IF average CPU > 70% for 10 minutes
THEN add 1 instance

Scale in (remove VMs):
IF average CPU < 30% for 10 minutes
THEN remove 1 instance

Limits:
Minimum instances: 2
Maximum instances: 10
```

**Scaling policies:**

**Scale out faster, scale in slower:**

```
Scale out: Add 2 instances when CPU > 80%
Scale in: Remove 1 instance when CPU < 20%

Reason: Better to have extra capacity than crash
```

**Instance limits:**

- Minimum: Ensure always some VMs running
- Maximum: Cost control
- Default: Initially deployed count

**Scale Set benefits:**

1. **High availability:**

   - Multiple VMs behind load balancer
   - If one fails, traffic goes to others

2. **Cost optimization:**

   - Pay only for what you need
   - Scale down during off-peak

3. **Automatic management:**

   - New VMs configured identically
   - Updates rolled out gradually
   - No manual intervention

4. **Integration:**
   - Works with Azure Load Balancer
   - Works with Application Gateway
   - Supports availability zones

**Important features:**

**Upgrade policy:**

- **Automatic:** Updates roll out immediately
- **Rolling:** Updates in batches (safer)
- **Manual:** You control when to update

**Health monitoring:**

- Application Health Extension
- Load Balancer health probes
- Unhealthy instances automatically replaced

**Overprovisioning:**

- Azure creates extra VMs during scale-out
- Keeps fastest responding VMs
- Deletes slower ones
- No charge for deleted VMs
- Faster scale-out

**Large-scale deployment:**

- Single-placement group: Up to 100 VMs
- Multi-placement group: Up to 1,000 VMs

**Use cases:**

- Web applications with variable traffic
- APIs with fluctuating demand
- Batch processing
- Microservices
- Stateless applications

**Not suitable for:**

- Stateful applications (use databases instead)
- Applications requiring specific server names/IPs
- Legacy applications not designed for scale

### Load Balancing

**Azure Load Balancer:**

- Layer 4 (TCP/UDP)
- Distributes traffic to VMs
- Public or internal
- Health probes
- Used with VM Scale Sets

**Basic features:**

```
Backend pool: VMs receiving traffic
Health probe: Check if VM is healthy
Load balancing rules: How to distribute traffic
Inbound NAT rules: Port forwarding to specific VM
```

**Health probes:**

- HTTP/HTTPS or TCP
- Checks endpoint regularly
- Removes unhealthy VMs from rotation
- Example: Check /health endpoint every 15 seconds

**Load balancing algorithms:**

- **5-tuple hash:** Source IP, source port, dest IP, dest port, protocol (default)
- **Source IP affinity:** Same client always goes to same VM

### Proximity Placement Groups

**Simple:** Keep VMs close together for lowest latency.

**Use case:**

- High-performance computing
- Low-latency applications
- Gaming servers
- Real-time analytics

**How it works:**

- VMs placed in same datacenter
- Or even same rack (when possible)
- Latency: Sub-millisecond

**Important:**

- Reduces availability (all VMs close together)
- Use only when latency is critical
- Consider availability zones for most workloads

---

## Azure App Service

### What is App Service?

**Simple:** Fully managed platform for hosting web applications. Like a hotel for your code - you bring the code, Azure handles everything else.

**Technical definition:**
Azure App Service is an HTTP-based PaaS offering for hosting web applications, REST APIs, and mobile backends.

**What Azure manages:**

- Infrastructure (servers, VMs)
- Operating system
- Patching and updates
- Load balancing
- Autoscaling
- High availability

**What you manage:**

- Application code
- Configuration
- Deployment

**When to use App Service:**

- Web applications
- REST APIs
- Mobile app backends
- Don't need OS-level control
- Want faster deployment
- Focus on code, not infrastructure

**When NOT to use:**

- Need custom OS configuration (use VMs)
- Need root/admin access (use VMs)
- Legacy application with specific dependencies (use VMs)
- Stateful applications requiring local disk (consider alternatives)

### App Service Plans

**Simple:** The pricing tier that determines resources and features for your app.

**Components of App Service Plan:**

- VM size and power
- Number of instances
- Features available
- Pricing

**Think of it as:** Hotel room categories (economy, business, suite)

**Pricing Tiers:**

**Free (F1):**

- Shared infrastructure
- 60 CPU minutes/day
- 1 GB disk space
- No custom domains
- No SSL
- Use for: Quick tests, POCs

**Shared (D1):**

- Shared infrastructure
- 240 CPU minutes/day
- 1 GB disk
- Custom domains
- Use for: Development, simple websites

**Basic (B1, B2, B3):**

- Dedicated VMs
- Manual scaling (up to 3 instances)
- Custom domains + SSL
- 10 GB disk
- Use for: Dev/test, low-traffic production

**Standard (S1, S2, S3):**

- Dedicated VMs
- Autoscaling (up to 10 instances)
- Staging slots (5 slots)
- Daily backups
- Traffic Manager integration
- 50 GB disk
- Use for: Production applications

**Premium (P1v2, P2v2, P3v2, P1v3, P2v3, P3v3):**

- More powerful VMs
- Autoscaling (up to 30 instances)
- Staging slots (20 slots)
- 250 GB disk
- VNet integration
- Use for: High-traffic production, enterprise

**Isolated (I1, I2, I3, I1v2, I2v2, I3v2):**

- Dedicated environment (App Service Environment)
- Complete isolation
- Private VNet
- 100 instances
- 1 TB disk
- Use for: Mission-critical, compliance, extreme scale

**Comparison:**

| Feature          | Free/Shared | Basic | Standard | Premium | Isolated |
| ---------------- | ----------- | ----- | -------- | ------- | -------- |
| Custom domain    | Shared only | ✅    | ✅       | ✅      | ✅       |
| SSL              | ❌          | ✅    | ✅       | ✅      | ✅       |
| Autoscale        | ❌          | ❌    | ✅       | ✅      | ✅       |
| Staging slots    | ❌          | ❌    | 5        | 20      | 20       |
| Daily backup     | ❌          | ❌    | ✅       | ✅      | ✅       |
| VNet integration | ❌          | ❌    | ❌       | ✅      | ✅       |

### App Service Features

**Deployment Slots:**

- Separate environments for your app
- Example: production, staging, dev
- Test changes before swapping to production
- Zero-downtime deployments
- Rollback easily if issues

**Deployment slot workflow:**

```
1. Deploy new version to staging slot
2. Test in staging
3. Swap staging → production (instant)
4. If issues, swap back

Traffic: production.azurewebsites.net (current version)
Testing: production-staging.azurewebsites.net (new version)
```

**Auto-scaling:**

- Scale based on metrics (CPU, memory, HTTP queue)
- Scale based on schedule (time-based)
- Min/max instance limits

**Example auto-scale rules:**

```
Rule 1: Scale out
IF CPU > 70% for 10 min → Add 1 instance

Rule 2: Scale in
IF CPU < 30% for 10 min → Remove 1 instance

Limits: Min 2, Max 10 instances
```

**Custom Domains and SSL:**

- Add your own domain (www.contoso.com)
- Free SSL certificates (managed by Azure)
- Bring your own SSL certificate
- SNI SSL and IP-based SSL

**Backup and Restore:**

- Automatic or manual backups
- Includes app configuration and databases
- Restore to any point in time
- Store in Azure Storage

**Monitoring:**

- Application Insights integration
- Request/response metrics
- Performance monitoring
- Error tracking
- Real user monitoring

**Authentication:**

- Built-in authentication (no code needed)
- Azure AD
- Google, Facebook, Twitter, Microsoft Account
- OpenID Connect
- Easy to enable

### Supported Languages and Frameworks

**Built-in support:**

- .NET / .NET Core
- Java
- Node.js
- Python
- PHP
- Ruby (Linux only)

**Containers:**

- Docker containers
- Custom runtimes
- Multi-container apps (Docker Compose)

### App Service on Linux vs Windows

**Windows:**

- ASP.NET, .NET Core
- Classic ASP, PHP
- More deployment options
- More features

**Linux:**

- Node.js, Python, Ruby, Java
- PHP on Linux
- Docker containers
- Lower cost
- Open-source friendly

**Important:**

- Cannot mix Windows and Linux apps in same App Service Plan
- Choose at creation time

### App Service Environment (ASE)

**Simple:** Fully isolated and dedicated App Service infrastructure.

**What you get:**

- Dedicated VMs (no sharing with other customers)
- Private VNet deployment
- High scale (100 instances)
- Compliance and security features

**Use for:**

- Compliance requirements
- Complete network isolation
- Very high scale
- Worth the higher cost

**Types:**

- ASEv2: Previous version
- ASEv3: Latest, recommended

---

## Azure Container Instances

### What are Containers?

**Simple:** Lightweight packages containing application and dependencies. Like a shipping container for code.

**Container vs VM:**

**Virtual Machine:**

```
┌─────────────────────┐
│    Application A    │
│    Guest OS         │
│    Hypervisor       │
│    Host OS          │
│    Physical Server  │
└─────────────────────┘
Heavy, slow to start
```

**Container:**

```
┌─────────────────────┐
│    Application A    │
│    Container Runtime│
│    Host OS          │
│    Physical Server  │
└─────────────────────┘
Lightweight, fast start
```

**Benefits of containers:**

- Fast startup (seconds)
- Lightweight (MBs instead of GBs)
- Portable (run anywhere)
- Consistent (same environment everywhere)
- Efficient (more containers per server)

### Azure Container Instances (ACI)

**Simple:** Run containers without managing servers or orchestrators.

**What Azure manages:**

- Infrastructure
- Container runtime
- Scaling
- Networking

**What you provide:**

- Container image
- CPU/memory requirements
- Configuration

**Use cases:**

- Simple containerized applications
- Burst workloads
- Build agents (CI/CD)
- Task automation
- Event-driven processing
- Development/testing

**Not suitable for:**

- Complex multi-container applications (use AKS)
- Need orchestration (use AKS)
- Long-running clustered applications (use AKS)

### ACI Features

**Fast deployment:**

- Start containers in seconds
- No VM provisioning needed

**Per-second billing:**

- Pay only while container runs
- Stop when not needed
- Very cost-effective for short tasks

**Networking:**

- Public IP address
- FQDN (fully qualified domain name)
- VNet integration
- Port mapping

**Persistent storage:**

- Azure Files mount
- Empty directory
- Git repo mount
- Secrets volume

**Container groups:**

- Deploy multiple containers together
- Share resources (network, storage)
- Like a pod in Kubernetes

**Example container group:**

```
Container Group: web-app
├─ Container 1: Web frontend (nginx)
├─ Container 2: Application (Node.js)
└─ Container 3: Logging sidecar

Shared:
- Network (same IP)
- Storage volume
```

**Restart policies:**

- Always: Container always restarted if stops
- OnFailure: Restart only if exits with error
- Never: Run once and stop

**Environment variables:**

- Pass configuration to containers
- Can use secure values (secrets)

**Pricing:**

- Per vCPU per second
- Per GB memory per second
- Very granular billing

**Example:**

```
Container with:
- 1 vCPU
- 1.5 GB memory
- Runs for 1 hour

Cost: ~$0.04
```

### When to Use ACI vs Other Services

| Need                    | Use            |
| ----------------------- | -------------- |
| Simple container        | ACI            |
| Orchestration needed    | AKS            |
| Web app (not container) | App Service    |
| Full control, custom OS | VM             |
| Serverless containers   | Container Apps |
| Batch processing        | Azure Batch    |

---

## Azure Kubernetes Service (AKS)

### What is Kubernetes?

**Simple:** Container orchestration platform. Like a conductor for an orchestra of containers.

**What Kubernetes does:**

- Deploys containers across multiple servers
- Manages container lifecycle
- Scales containers up/down
- Handles failures (restarts containers)
- Load balancing
- Service discovery
- Updates and rollbacks

**The problem without Kubernetes:**

```
10 containers across 5 servers:
- Which container on which server?
- What if container crashes?
- How to scale to 20 containers?
- How to update without downtime?
- Manual management = complex
```

**With Kubernetes:**

```
Declare desired state:
"I want 10 web containers, load balanced"

Kubernetes handles:
- Placement
- Scaling
- Healing
- Updates
Automatically
```

### Azure Kubernetes Service (AKS)

**Simple:** Managed Kubernetes cluster in Azure. Azure handles the control plane, you manage workloads.

**What Azure manages:**

- Kubernetes control plane (masters)
- Patching and updates
- Monitoring and logging
- High availability of control plane

**What you manage:**

- Worker nodes (VMs)
- Deployed applications
- Scaling policies
- Network configuration

**Why use AKS:**

- Run microservices
- Container orchestration
- Modern cloud-native apps
- DevOps workflows
- Multi-container applications

### AKS Architecture

**Components:**

**Control Plane (managed by Azure):**

- API Server: Entry point for all commands
- Scheduler: Decides where to run containers
- Controller Manager: Maintains desired state
- etcd: Configuration database

**Worker Nodes (managed by you):**

- Virtual machines running containers
- kubelet: Agent on each node
- Container runtime: Runs containers
- kube-proxy: Network routing

**Architecture diagram:**

```
Control Plane (Azure-managed, free)
├─ API Server
├─ Scheduler
├─ Controller Manager
└─ etcd

↓ (manages)

Worker Nodes (your VMs, you pay)
├─ Node 1 (VM)
│   ├─ Pod 1 (container)
│   └─ Pod 2 (container)
├─ Node 2 (VM)
│   ├─ Pod 3 (container)
│   └─ Pod 4 (container)
└─ Node 3 (VM)
    └─ Pod 5 (container)
```

### Key Kubernetes Concepts

**Pod:**

- Smallest deployable unit
- One or more containers
- Shared network and storage
- Usually 1 container per pod

**Deployment:**

- Defines desired state for pods
- Example: "Run 5 replicas of web app"
- Handles rolling updates
- Manages pod creation/deletion

**Service:**

- Stable endpoint for pods
- Load balancing
- Service discovery
- Types: ClusterIP, LoadBalancer, NodePort

**Namespace:**

- Virtual cluster within cluster
- Organize resources
- Example: dev, test, prod namespaces

**ConfigMap and Secret:**

- ConfigMap: Configuration data
- Secret: Sensitive data (passwords, keys)
- Mounted into pods

### AKS Features

**Node Pools:**

- Groups of nodes with same configuration
- Different VM sizes per pool
- System pool: Kubernetes components
- User pool: Your applications

**Example:**

```
AKS Cluster
├─ System Node Pool
│   └─ 3 × Standard_D2s_v3 (Kubernetes system pods)
├─ User Node Pool 1
│   └─ 5 × Standard_D4s_v3 (web applications)
└─ User Node Pool 2
    └─ 2 × Standard_E4s_v3 (memory-intensive apps)
```

**Autoscaling:**

**Cluster Autoscaler:**

- Adds/removes nodes based on demand
- Scales worker nodes

**Horizontal Pod Autoscaler:**

- Adds/removes pods based on CPU/memory
- Scales applications

**Vertical Pod Autoscaler:**

- Adjusts CPU/memory requests
- Right-sizes containers

**Networking:**

- Azure CNI: Advanced networking
- kubenet: Basic networking
- Network policies
- Private clusters (no public endpoint)

**Monitoring:**

- Container Insights
- Prometheus integration
- Log Analytics
- Azure Monitor

**Security:**

- Azure AD integration
- RBAC within cluster
- Pod security policies
- Network policies
- Azure Policy for AKS

**Upgrades:**

- Control plane upgrades (automatic or manual)
- Node pool upgrades
- Blue/green upgrades
- Zero-downtime

### AKS vs Other Container Services

| Feature            | ACI                     | App Service (Containers) | AKS                          |
| ------------------ | ----------------------- | ------------------------ | ---------------------------- |
| **Complexity**     | Simple                  | Medium                   | Complex                      |
| **Use case**       | Single container, tasks | Web apps                 | Microservices, orchestration |
| **Management**     | Minimal                 | Low                      | High                         |
| **Scaling**        | Manual                  | Automatic                | Very flexible                |
| **Cost**           | Per second              | Per hour                 | Per VM                       |
| **Learning curve** | Easy                    | Easy                     | Steep                        |

**Decision guide:**

```
Do you need orchestration?
  NO → Single container?
    YES → ACI
    NO → Web app? → App Service
  YES → Production microservices? → AKS
```

### AKS Best Practices

**For AZ-104 exam:**

- Understand basic concepts (pods, deployments, services)
- Know when to use AKS vs alternatives
- Understand node pools
- Know autoscaling options
- Understand basic networking

**Not required for AZ-104:**

- Deep Kubernetes knowledge
- Writing YAML manifests
- kubectl commands (beyond basics)
- Advanced networking

---

## VM Backup and Disaster Recovery

### Azure Backup for VMs

**Simple:** Automated backup solution for protecting VM data.

**What gets backed up:**

- Entire VM (all disks)
- OS disk
- Data disks
- VM configuration

**How it works:**

**1. Recovery Services Vault:**

- Container for backups
- Stores backup data
- Manages backup policies

**2. Backup Policy:**

- When to backup (schedule)
- How long to keep (retention)
- Backup type (full, incremental)

**3. Backup Process:**

```
1. Azure takes VM snapshot
2. Snapshot transferred to vault
3. VM remains running (no downtime)
4. Incremental backups save space
```

**Backup types:**

**Standard backup:**

- Backs up entire VM
- Application-consistent (Windows)
- File-system consistent (Linux)
- No VM downtime

**Enhanced backup:**

- Multiple backups per day
- Shorter RPO (Recovery Point Objective)
- More flexible scheduling

**Backup policies:**

**Default policy:**

- Daily backup at 12:00 AM
- Retention: 30 days

**Custom policy example:**

```
Schedule:
- Daily at 10:00 PM
- Weekly on Sunday
- Monthly on 1st day

Retention:
- Daily backups: 7 days
- Weekly backups: 4 weeks
- Monthly backups: 12 months
- Yearly backups: 5 years
```

**Important features:**

**Application-consistent backup (Windows):**

- Uses VSS (Volume Shadow Copy Service)
- Ensures app data consistency
- SQL Server, Exchange aware

**File-level restore:**

- Mount backup as disk
- Restore individual files
- No need to restore entire VM

**Cross-region restore:**

- Restore VM in different region
- Disaster recovery
- Requires GRS vault

**Soft delete:**

- Deleted backups retained 14 days
- Protection against accidental deletion
- Ransomware protection

**Pricing:**

- Per protected instance
- Storage costs
- Data transfer (restore)

**Cost example:**

```
VM with 100 GB data:
- Protected instance: ~$5/month
- Storage (assuming 100 GB backup): ~$2/month
Total: ~$7/month
```

### VM Restore Options

**1. Create new VM:**

- Restore to new VM
- Original VM remains
- Good for testing

**2. Replace existing VM:**

- Overwrites existing VM
- Faster than creating new
- Use with caution

**3. Restore disks only:**

- Just get the disk files
- Attach to different VM
- Most flexible option

**4. File-level restore:**

- Mount backup as iSCSI
- Copy specific files
- No full VM restore needed

### Azure Site Recovery (ASR)

**Simple:** Disaster recovery solution. Replicate VMs to another region for business continuity.

**Use cases:**

- Disaster recovery (DR)
- Migration to Azure
- Datacenter migration
- Protection against regional outages

**How it works:**

**Replication:**

```
Primary Region (East US)
VM writes data
  ↓ (continuous replication)
Secondary Region (West US)
Replica maintained (powered off)
```

**When disaster strikes:**

```
Primary region fails
  ↓
Failover to secondary region
  ↓
Secondary VM powered on
  ↓
Applications running in secondary
```

**Key concepts:**

**RPO (Recovery Point Objective):**

- How much data can you afford to lose?
- ASR: As low as 30 seconds
- Example: RPO = 5 minutes means max 5 min data loss

**RTO (Recovery Time Objective):**

- How quickly must you recover?
- ASR: Typically 2-4 hours
- Example: RTO = 2 hours means must be up in 2 hours

**Replication process:**

1. Enable replication on VM
2. Initial replication (full copy)
3. Continuous delta replication
4. Recovery points created regularly

**Failover types:**

**Test failover:**

- Test DR plan without affecting production
- Creates test VM in isolated network
- No disruption to replication

**Planned failover:**

- Scheduled maintenance
- Graceful shutdown of primary
- Zero data loss

**Unplanned failover:**

- Disaster scenario
- Primary region unavailable
- Possible data loss (depends on last recovery point)

**Failback:**

- Return to primary region after recovery
- Reverse replication direction
- Can be planned or unplanned

**ASR supported scenarios:**

**Azure to Azure:**

- VM in one region → another region
- Most common for DR

**VMware to Azure:**

- On-premises VMware → Azure
- Migration or DR

**Hyper-V to Azure:**

- On-premises Hyper-V → Azure
- Migration or DR

**Physical servers to Azure:**

- Physical Windows/Linux → Azure

**Recovery Plans:**

- Orchestrate failover of multiple VMs
- Define order of recovery
- Automate with scripts
- Example:
  ```
  Recovery Plan: 3-tier app
  1. Start SQL Server VMs (wait for startup)
  2. Start App Server VMs (wait for startup)
  3. Start Web Server VMs
  4. Update load balancer config
  ```

**ASR pricing:**

- Per protected instance: ~$25/month per VM
- Storage costs (replica disks)
- Network egress during failover

### Backup vs Site Recovery

| Feature      | Azure Backup                    | Site Recovery         |
| ------------ | ------------------------------- | --------------------- |
| **Purpose**  | Data protection                 | Disaster recovery     |
| **RPO**      | Hours (24h default)             | Seconds/minutes (30s) |
| **RTO**      | Hours                           | Hours                 |
| **Storage**  | Recovery vault                  | Replica VMs           |
| **Cost**     | Lower                           | Higher                |
| **Use case** | Accidental deletion, corruption | Regional disaster     |
| **Restore**  | Point in time                   | Latest state          |

**Best practice:** Use both

- Backup: Daily protection
- ASR: Continuous DR

---

## PowerShell Commands Reference

### Virtual Machines

```powershell
# Connect to Azure
Connect-AzAccount
Set-AzContext -SubscriptionName "Production"

# Create resource group
New-AzResourceGroup -Name "RG-VM" -Location "eastus"

# Create VM (simple method)
New-AzVM -ResourceGroupName "RG-VM" `
    -Name "MyVM" `
    -Location "eastus" `
    -Image "Win2022Datacenter" `
    -Size "Standard_D2s_v3" `
    -Credential (Get-Credential)

# Create VM (detailed method with networking)
# 1. Create virtual network
$vnet = New-AzVirtualNetwork -ResourceGroupName "RG-VM" `
    -Name "VNet-Prod" `
    -Location "eastus" `
    -AddressPrefix "10.0.0.0/16"

# 2. Create subnet
$subnetConfig = Add-AzVirtualNetworkSubnetConfig -Name "Subnet1" `
    -AddressPrefix "10.0.1.0/24" `
    -VirtualNetwork $vnet
$vnet | Set-AzVirtualNetwork

# 3. Create public IP
$pip = New-AzPublicIpAddress -ResourceGroupName "RG-VM" `
    -Name "MyVM-IP" `
    -Location "eastus" `
    -AllocationMethod "Static" `
    -Sku "Standard"

# 4. Create NSG with rules
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name "Allow-RDP" `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 1000 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389 `
    -Access Allow

$nsg = New-AzNetworkSecurityGroup -ResourceGroupName "RG-VM" `
    -Name "MyVM-NSG" `
    -Location "eastus" `
    -SecurityRules $nsgRuleRDP

# 5. Create NIC
$nic = New-AzNetworkInterface -ResourceGroupName "RG-VM" `
    -Name "MyVM-NIC" `
    -Location "eastus" `
    -SubnetId $vnet.Subnets[0].Id `
    -PublicIpAddressId $pip.Id `
    -NetworkSecurityGroupId $nsg.Id

# 6. Create VM configuration
$vmConfig = New-AzVMConfig -VMName "MyVM" -VMSize "Standard_D2s_v3"

# 7. Set OS
$cred = Get-Credential
$vmConfig = Set-AzVMOperatingSystem -VM $vmConfig `
    -Windows `
    -ComputerName "MyVM" `
    -Credential $cred `
    -ProvisionVMAgent `
    -EnableAutoUpdate

# 8. Set source image
$vmConfig = Set-AzVMSourceImage -VM $vmConfig `
    -PublisherName "MicrosoftWindowsServer" `
    -Offer "WindowsServer" `
    -Skus "2022-Datacenter" `
    -Version "latest"

# 9. Add NIC
$vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id

# 10. Set OS disk
$vmConfig = Set-AzVMOSDisk -VM $vmConfig `
    -Name "MyVM-OSDisk" `
    -CreateOption FromImage `
    -StorageAccountType "Premium_LRS"

# 11. Create VM
New-AzVM -ResourceGroupName "RG-VM" `
    -Location "eastus" `
    -VM $vmConfig

# Get VM
Get-AzVM -ResourceGroupName "RG-VM" -Name "MyVM"

# List all VMs
Get-AzVM

# List VMs in resource group
Get-AzVM -ResourceGroupName "RG-VM"

# Get VM status
Get-AzVM -ResourceGroupName "RG-VM" -Name "MyVM" -Status

# Start VM
Start-AzVM -ResourceGroupName "RG-VM" -Name "MyVM"

# Stop VM (deallocate - no charges)
Stop-AzVM -ResourceGroupName "RG-VM" -Name "MyVM" -Force

# Restart VM
Restart-AzVM -ResourceGroupName "RG-VM" -Name "MyVM"

# Resize VM
$vm = Get-AzVM -ResourceGroupName "RG-VM" -Name "MyVM"
$vm.HardwareProfile.VmSize = "Standard_D4s_v3"
Update-AzVM -ResourceGroupName "RG-VM" -VM $vm

# Add data disk
$vm = Get-AzVM -ResourceGroupName "RG-VM" -Name "MyVM"
$diskConfig = New-AzDiskConfig -Location "eastus" `
    -CreateOption Empty `
    -DiskSizeGB 128 `
    -SkuName "Premium_LRS"

$dataDisk = New-AzDisk -ResourceGroupName "RG-VM" `
    -DiskName "MyVM-DataDisk1" `
    -Disk $diskConfig

$vm = Add-AzVMDataDisk -VM $vm `
    -Name "MyVM-DataDisk1" `
    -CreateOption Attach `
    -ManagedDiskId $dataDisk.Id `
    -Lun 0

Update-AzVM -ResourceGroupName "RG-VM" -VM $vm

# Remove data disk
$vm = Get-AzVM -ResourceGroupName "RG-VM" -Name "MyVM"
Remove-AzVMDataDisk -VM $vm -Name "MyVM-DataDisk1"
Update-AzVM -ResourceGroupName "RG-VM" -VM $vm

# Delete VM (keeps disks)
Remove-AzVM -ResourceGroupName "RG-VM" -Name "MyVM" -Force

# Delete VM and all resources
$vm = Get-AzVM -ResourceGroupName "RG-VM" -Name "MyVM"
$vm | Remove-AzVM -Force

# Delete NIC
$nic = Get-AzNetworkInterface -ResourceGroupName "RG-VM" -Name "MyVM-NIC"
$nic | Remove-AzNetworkInterface -Force

# Delete disks
$disks = Get-AzDisk -ResourceGroupName "RG-VM"
$disks | Remove-AzDisk -Force

# Delete public IP
$pip = Get-AzPublicIpAddress -ResourceGroupName "RG-VM" -Name "MyVM-IP"
$pip | Remove-AzPublicIpAddress -Force
```

### Availability Sets

```powershell
# Create availability set
New-AzAvailabilitySet -ResourceGroupName "RG-VM" `
    -Name "WebTier-AS" `
    -Location "eastus" `
    -PlatformFaultDomainCount 2 `
    -PlatformUpdateDomainCount 5 `
    -Sku "Aligned"

# Create VM in availability set
$availSet = Get-AzAvailabilitySet -ResourceGroupName "RG-VM" -Name "WebTier-AS"

$vmConfig = New-AzVMConfig -VMName "WebVM1" `
    -VMSize "Standard_D2s_v3" `
    -AvailabilitySetId $availSet.Id

# (Continue with VM creation steps...)

# Get availability set
Get-AzAvailabilitySet -ResourceGroupName "RG-VM" -Name "WebTier-AS"

# List VMs in availability set
$availSet = Get-AzAvailabilitySet -ResourceGroupName "RG-VM" -Name "WebTier-AS"
$availSet.VirtualMachinesReferences
```

### VM Scale Sets

```powershell
# Create VM Scale Set
New-AzVmss -ResourceGroupName "RG-VMSS" `
    -VMScaleSetName "WebScaleSet" `
    -Location "eastus" `
    -VirtualNetworkName "VNet-Prod" `
    -SubnetName "Subnet1" `
    -PublicIpAddressName "WebScaleSet-IP" `
    -LoadBalancerName "WebScaleSet-LB" `
    -UpgradePolicyMode "Automatic" `
    -Credential (Get-Credential)

# Get scale set
Get-AzVmss -ResourceGroupName "RG-VMSS" -VMScaleSetName "WebScaleSet"

# List instances
Get-AzVmssVM -ResourceGroupName "RG-VMSS" -VMScaleSetName "WebScaleSet"

# Update scale set capacity (manual scaling)
$vmss = Get-AzVmss -ResourceGroupName "RG-VMSS" -VMScaleSetName "WebScaleSet"
$vmss.Sku.Capacity = 5
Update-AzVmss -ResourceGroupName "RG-VMSS" `
    -VMScaleSetName "WebScaleSet" `
    -VirtualMachineScaleSet $vmss

# Configure autoscale
$rule1 = New-AzAutoscaleRule -MetricName "Percentage CPU" `
    -MetricResourceId "/subscriptions/{sub-id}/resourceGroups/RG-VMSS/providers/Microsoft.Compute/virtualMachineScaleSets/WebScaleSet" `
    -TimeGrain 00:01:00 `
    -MetricStatistic "Average" `
    -TimeWindow 00:05:00 `
    -Operator "GreaterThan" `
    -Threshold 70 `
    -ScaleActionDirection "Increase" `
    -ScaleActionScaleType "ChangeCount" `
    -ScaleActionValue 1 `
    -ScaleActionCooldown 00:05:00

$rule2 = New-AzAutoscaleRule -MetricName "Percentage CPU" `
    -MetricResourceId "/subscriptions/{sub-id}/resourceGroups/RG-VMSS/providers/Microsoft.Compute/virtualMachineScaleSets/WebScaleSet" `
    -TimeGrain 00:01:00 `
    -MetricStatistic "Average" `
    -TimeWindow 00:05:00 `
    -Operator "LessThan" `
    -Threshold 30 `
    -ScaleActionDirection "Decrease" `
    -ScaleActionScaleType "ChangeCount" `
    -ScaleActionValue 1 `
    -ScaleActionCooldown 00:05:00

$profile = New-AzAutoscaleProfile -Name "Auto scale profile" `
    -DefaultCapacity 2 `
    -MaximumCapacity 10 `
    -MinimumCapacity 2 `
    -Rule $rule1, $rule2

Add-AzAutoscaleSetting -ResourceGroupName "RG-VMSS" `
    -Name "WebScaleSet-Autoscale" `
    -Location "eastus" `
    -TargetResourceId "/subscriptions/{sub-id}/resourceGroups/RG-VMSS/providers/Microsoft.Compute/virtualMachineScaleSets/WebScaleSet" `
    -AutoscaleProfile $profile

# Delete scale set
Remove-AzVmss -ResourceGroupName "RG-VMSS" `
    -VMScaleSetName "WebScaleSet" `
    -Force
```

### App Service

```powershell
# Create App Service Plan
New-AzAppServicePlan -ResourceGroupName "RG-AppService" `
    -Name "MyAppServicePlan" `
    -Location "eastus" `
    -Tier "Standard" `
    -NumberofWorkers 2 `
    -WorkerSize "Small"

# Create Web App
New-AzWebApp -ResourceGroupName "RG-AppService" `
    -Name "myuniquewebapp123" `
    -Location "eastus" `
    -AppServicePlan "MyAppServicePlan"

# Get Web App
Get-AzWebApp -ResourceGroupName "RG-AppService" -Name "myuniquewebapp123"

# List all Web Apps
Get-AzWebApp

# Start Web App
Start-AzWebApp -ResourceGroupName "RG-AppService" -Name "myuniquewebapp123"

# Stop Web App
Stop-AzWebApp -ResourceGroupName "RG-AppService" -Name "myuniquewebapp123"

# Restart Web App
Restart-AzWebApp -ResourceGroupName "RG-AppService" -Name "myuniquewebapp123"

# Scale App Service Plan
Set-AzAppServicePlan -ResourceGroupName "RG-AppService" `
    -Name "MyAppServicePlan" `
    -Tier "Premium" `
    -NumberofWorkers 3 `
    -WorkerSize "Medium"

# Create deployment slot
New-AzWebAppSlot -ResourceGroupName "RG-AppService" `
    -Name "myuniquewebapp123" `
    -Slot "staging"

# Swap deployment slots
Switch-AzWebAppSlot -ResourceGroupName "RG-AppService" `
    -Name "myuniquewebapp123" `
    -SourceSlotName "staging" `
    -DestinationSlotName "production"

# Delete Web App
Remove-AzWebApp -ResourceGroupName "RG-AppService" `
    -Name "myuniquewebapp123" `
    -Force

# Delete App Service Plan
Remove-AzAppServicePlan -ResourceGroupName "RG-AppService" `
    -Name "MyAppServicePlan" `
    -Force
```

### Container Instances

```powershell
# Create container group
New-AzContainerGroup -ResourceGroupName "RG-Containers" `
    -Name "mycontainer" `
    -Image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" `
    -OsType Linux `
    -DnsNameLabel "mycontainer-dns" `
    -Port 80 `
    -Cpu 1 `
    -MemoryInGB 1.5 `
    -Location "eastus"

# Get container group
Get-AzContainerGroup -ResourceGroupName "RG-Containers" -Name "mycontainer"

# List container groups
Get-AzContainerGroup -ResourceGroupName "RG-Containers"

# Get container logs
Get-AzContainerInstanceLog -ResourceGroupName "RG-Containers" `
    -ContainerGroupName "mycontainer" `
    -Name "mycontainer"

# Remove container group
Remove-AzContainerGroup -ResourceGroupName "RG-Containers" `
    -Name "mycontainer"
```

### Azure Kubernetes Service

```powershell
# Create AKS cluster
New-AzAksCluster -ResourceGroupName "RG-AKS" `
    -Name "MyAKSCluster" `
    -Location "eastus" `
    -NodeCount 3 `
    -NodeVmSize "Standard_D2s_v3" `
    -KubernetesVersion "1.27.3" `
    -EnableManagedIdentity

# Get AKS cluster
Get-AzAksCluster -ResourceGroupName "RG-AKS" -Name "MyAKSCluster"

# List AKS clusters
Get-AzAksCluster

# Get credentials (for kubectl)
Import-AzAksCredential -ResourceGroupName "RG-AKS" `
    -Name "MyAKSCluster" `
    -Force

# After importing credentials, use kubectl
kubectl get nodes
kubectl get pods --all-namespaces

# Scale AKS cluster
Set-AzAksCluster -ResourceGroupName "RG-AKS" `
    -Name "MyAKSCluster" `
    -NodeCount 5

# Add node pool
New-AzAksNodePool -ResourceGroupName "RG-AKS" `
    -ClusterName "MyAKSCluster" `
    -Name "userpool" `
    -Count 3 `
    -VmSize "Standard_D4s_v3"

# Start AKS cluster
Start-AzAksCluster -ResourceGroupName "RG-AKS" -Name "MyAKSCluster"

# Stop AKS cluster
Stop-AzAksCluster -ResourceGroupName "RG-AKS" -Name "MyAKSCluster"

# Delete AKS cluster
Remove-AzAksCluster -ResourceGroupName "RG-AKS" `
    -Name "MyAKSCluster" `
    -Force
```

### VM Backup

```bash
# Create Recovery Services Vault
az backup vault create \
    --resource-group "RG-Backup" \
    --name "MyBackupVault" \
    --location "eastus"

# Set vault backup properties
az backup vault backup-properties set \
    --resource-group "RG-Backup" \
    --name "MyBackupVault" \
    --backup-storage-redundancy "GeoRedundant"

# Enable backup for VM (with default policy)
az backup protection enable-for-vm \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --vm "MyVM" \
    --policy-name "DefaultPolicy"

# List backup policies
az backup policy list \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --output table

# Trigger backup immediately
az backup protection backup-now \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "MyVM" \
    --item-name "MyVM" \
    --retain-until "01-01-2025"

# List backup items
az backup item list \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --output table

# List recovery points
az backup recoverypoint list \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "MyVM" \
    --item-name "MyVM" \
    --output table

# Restore VM
az backup restore restore-disks \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "MyVM" \
    --item-name "MyVM" \
    --rp-name "<recovery-point-name>" \
    --storage-account "restorestorage"

# Disable backup
az backup protection disable \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "MyVM" \
    --item-name "MyVM" \
    --delete-backup-data true \
    --yes
```

---

## Azure CLI Commands Reference

---

## Virtual Machines

```bash
# Login to Azure
az login
az account set --subscription "Production"

# Create resource group
az group create --name "RG-VM" --location "eastus"

# Create VM (simple method)
az vm create \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --image "Win2022Datacenter" \
    --size "Standard_D2s_v3" \
    --admin-username "azureuser" \
    --admin-password "P@ssw0rd1234!"

# Create Linux VM with SSH key
az vm create \
    --resource-group "RG-VM" \
    --name "LinuxVM" \
    --image "Ubuntu2204" \
    --size "Standard_B2s" \
    --admin-username "azureuser" \
    --generate-ssh-keys

# Create VM with detailed networking
# 1. Create VNet
az network vnet create \
    --resource-group "RG-VM" \
    --name "VNet-Prod" \
    --address-prefix "10.0.0.0/16" \
    --subnet-name "Subnet1" \
    --subnet-prefix "10.0.1.0/24"

# 2. Create public IP
az network public-ip create \
    --resource-group "RG-VM" \
    --name "MyVM-IP" \
    --sku "Standard" \
    --allocation-method "Static"

# 3. Create NSG
az network nsg create \
    --resource-group "RG-VM" \
    --name "MyVM-NSG"

# 4. Add NSG rule
az network nsg rule create \
    --resource-group "RG-VM" \
    --nsg-name "MyVM-NSG" \
    --name "Allow-RDP" \
    --priority 1000 \
    --source-address-prefixes '*' \
    --source-port-ranges '*' \
    --destination-address-prefixes '*' \
    --destination-port-ranges 3389 \
    --access "Allow" \
    --protocol "Tcp"

# 5. Create NIC
az network nic create \
    --resource-group "RG-VM" \
    --name "MyVM-NIC" \
    --vnet-name "VNet-Prod" \
    --subnet "Subnet1" \
    --public-ip-address "MyVM-IP" \
    --network-security-group "MyVM-NSG"

# 6. Create VM with existing NIC
az vm create \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --nics "MyVM-NIC" \
    --image "Win2022Datacenter" \
    --size "Standard_D2s_v3" \
    --admin-username "azureuser" \
    --admin-password "P@ssw0rd1234!"

# Get VM details
az vm show --resource-group "RG-VM" --name "MyVM"

# List all VMs
az vm list --output table

# List VMs in resource group
az vm list --resource-group "RG-VM" --output table

# Get VM sizes available in location
az vm list-sizes --location "eastus" --output table

# Get available images
az vm image list --output table
az vm image list --publisher "MicrosoftWindowsServer" --all --output table

# Start VM
az vm start --resource-group "RG-VM" --name "MyVM"

# Stop VM (deallocate - no compute charges)
az vm deallocate --resource-group "RG-VM" --name "MyVM"

# Stop VM (without deallocation - still billed)
az vm stop --resource-group "RG-VM" --name "MyVM"

# Restart VM
az vm restart --resource-group "RG-VM" --name "MyVM"

# Get VM power state
az vm get-instance-view \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" \
    --output table

# Get all VM details including status
az vm list --resource-group "RG-VM" --show-details --output table

# Resize VM
az vm resize \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --size "Standard_D4s_v3"

# Add data disk (new)
az vm disk attach \
    --resource-group "RG-VM" \
    --vm-name "MyVM" \
    --name "MyVM-DataDisk1" \
    --new \
    --size-gb 128 \
    --sku "Premium_LRS"

# Add existing data disk
az vm disk attach \
    --resource-group "RG-VM" \
    --vm-name "MyVM" \
    --name "MyVM-DataDisk1"

# List VM disks
az vm show \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --query "storageProfile.dataDisks"

# Detach data disk
az vm disk detach \
    --resource-group "RG-VM" \
    --vm-name "MyVM" \
    --name "MyVM-DataDisk1"

# Update OS disk size
az vm deallocate --resource-group "RG-VM" --name "MyVM"
az disk update \
    --resource-group "RG-VM" \
    --name "MyVM-OSDisk" \
    --size-gb 256
az vm start --resource-group "RG-VM" --name "MyVM"

# Run command on VM (without logging in)
az vm run-command invoke \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --command-id "RunPowerShellScript" \
    --scripts "Get-Service"

# Run command on Linux VM
az vm run-command invoke \
    --resource-group "RG-VM" \
    --name "LinuxVM" \
    --command-id "RunShellScript" \
    --scripts "ifconfig"

# Open port on VM (creates NSG rule)
az vm open-port \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --port 443 \
    --priority 1100

# Enable auto-shutdown
az vm auto-shutdown \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --time "1800"

# Get VM connection info
az vm list-ip-addresses \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --output table

# Delete VM (keeps disks and network resources)
az vm delete \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --yes

# Delete VM and all associated resources
az vm delete \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --yes

az network nic delete \
    --resource-group "RG-VM" \
    --name "MyVM-NIC"

az network public-ip delete \
    --resource-group "RG-VM" \
    --name "MyVM-IP"

az network nsg delete \
    --resource-group "RG-VM" \
    --name "MyVM-NSG"

az disk delete \
    --resource-group "RG-VM" \
    --name "MyVM-OSDisk" \
    --yes
```

---

## Availability Sets

```bash
# Create availability set
az vm availability-set create \
    --resource-group "RG-VM" \
    --name "WebTier-AS" \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 5 \
    --location "eastus"

# Create availability set with managed disk alignment
az vm availability-set create \
    --resource-group "RG-VM" \
    --name "WebTier-AS" \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 5 \
    --location "eastus" \
    --sku "Aligned"

# Create VM in availability set
az vm create \
    --resource-group "RG-VM" \
    --name "WebVM1" \
    --availability-set "WebTier-AS" \
    --image "Ubuntu2204" \
    --size "Standard_D2s_v3" \
    --admin-username "azureuser" \
    --generate-ssh-keys

# Get availability set details
az vm availability-set show \
    --resource-group "RG-VM" \
    --name "WebTier-AS"

# List all availability sets
az vm availability-set list \
    --resource-group "RG-VM" \
    --output table

# List VMs in availability set
az vm availability-set list-sizes \
    --resource-group "RG-VM" \
    --name "WebTier-AS" \
    --output table

# Update availability set
az vm availability-set update \
    --resource-group "RG-VM" \
    --name "WebTier-AS" \
    --set tags.Environment=Production

# Delete availability set
az vm availability-set delete \
    --resource-group "RG-VM" \
    --name "WebTier-AS"
```

---

## VM Scale Sets

```bash
# Create VM Scale Set (simple)
az vmss create \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --image "Ubuntu2204" \
    --upgrade-policy-mode "Automatic" \
    --instance-count 2 \
    --vm-sku "Standard_D2s_v3" \
    --admin-username "azureuser" \
    --generate-ssh-keys

# Create VMSS with custom image
az vmss create \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --image "/subscriptions/{sub-id}/resourceGroups/RG-Images/providers/Microsoft.Compute/images/MyCustomImage" \
    --instance-count 3 \
    --vm-sku "Standard_D2s_v3" \
    --admin-username "azureuser" \
    --generate-ssh-keys

# Create VMSS with load balancer
az vmss create \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --image "Ubuntu2204" \
    --instance-count 2 \
    --vm-sku "Standard_D2s_v3" \
    --vnet-name "VNet-Prod" \
    --subnet "Subnet1" \
    --lb "WebScaleSet-LB" \
    --backend-pool-name "WebBackendPool" \
    --admin-username "azureuser" \
    --generate-ssh-keys

# Create VMSS across availability zones
az vmss create \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --image "Ubuntu2204" \
    --instance-count 6 \
    --vm-sku "Standard_D2s_v3" \
    --zones 1 2 3 \
    --admin-username "azureuser" \
    --generate-ssh-keys

# Get scale set details
az vmss show \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet"

# List all scale sets
az vmss list --output table

# List scale set instances
az vmss list-instances \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --output table

# Get instance details
az vmss get-instance-view \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --instance-id 0

# Manual scaling (set instance count)
az vmss scale \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --new-capacity 5

# Update VMSS (change SKU)
az vmss update \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --set sku.name=Standard_D4s_v3

# Configure autoscale
az monitor autoscale create \
    --resource-group "RG-VMSS" \
    --resource "WebScaleSet" \
    --resource-type "Microsoft.Compute/virtualMachineScaleSets" \
    --name "WebScaleSet-Autoscale" \
    --min-count 2 \
    --max-count 10 \
    --count 2

# Add scale-out rule (CPU > 70%)
az monitor autoscale rule create \
    --resource-group "RG-VMSS" \
    --autoscale-name "WebScaleSet-Autoscale" \
    --condition "Percentage CPU > 70 avg 5m" \
    --scale out 2

# Add scale-in rule (CPU < 30%)
az monitor autoscale rule create \
    --resource-group "RG-VMSS" \
    --autoscale-name "WebScaleSet-Autoscale" \
    --condition "Percentage CPU < 30 avg 10m" \
    --scale in 1

# Add schedule-based autoscale
az monitor autoscale rule create \
    --resource-group "RG-VMSS" \
    --autoscale-name "WebScaleSet-Autoscale" \
    --scale to 10 \
    --condition "time > '2024-12-25T08:00' and time < '2024-12-25T18:00'"

# Show autoscale settings
az monitor autoscale show \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet-Autoscale"

# Delete autoscale settings
az monitor autoscale delete \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet-Autoscale"

# Upgrade VMSS instances
az vmss update-instances \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --instance-ids "*"

# Update VMSS image
az vmss update \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --set virtualMachineProfile.storageProfile.imageReference.version=latest

# Reimage VMSS instance
az vmss reimage \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --instance-id 0

# Restart VMSS instances
az vmss restart \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --instance-ids "*"

# Deallocate VMSS
az vmss deallocate \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet"

# Start VMSS
az vmss start \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet"

# Delete specific instances
az vmss delete-instances \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet" \
    --instance-ids 0 1

# Delete scale set
az vmss delete \
    --resource-group "RG-VMSS" \
    --name "WebScaleSet"
```

---

## App Service

```bash
# Create App Service Plan
az appservice plan create \
    --resource-group "RG-AppService" \
    --name "MyAppServicePlan" \
    --location "eastus" \
    --sku "S1" \
    --is-linux

# Create App Service Plan (Windows)
az appservice plan create \
    --resource-group "RG-AppService" \
    --name "MyAppServicePlan" \
    --location "eastus" \
    --sku "S1"

# Create App Service Plan with autoscale
az appservice plan create \
    --resource-group "RG-AppService" \
    --name "MyAppServicePlan" \
    --location "eastus" \
    --sku "P1V2" \
    --number-of-workers 2

# Get App Service Plan
az appservice plan show \
    --resource-group "RG-AppService" \
    --name "MyAppServicePlan"

# List all App Service Plans
az appservice plan list --output table

# Update App Service Plan (scale up)
az appservice plan update \
    --resource-group "RG-AppService" \
    --name "MyAppServicePlan" \
    --sku "P1V2"

# Update App Service Plan (scale out)
az appservice plan update \
    --resource-group "RG-AppService" \
    --name "MyAppServicePlan" \
    --number-of-workers 3

# Create Web App
az webapp create \
    --resource-group "RG-AppService" \
    --plan "MyAppServicePlan" \
    --name "myuniquewebapp123" \
    --runtime "NODE|18-lts"

# Create Web App with deployment from GitHub
az webapp create \
    --resource-group "RG-AppService" \
    --plan "MyAppServicePlan" \
    --name "myuniquewebapp123" \
    --runtime "NODE|18-lts" \
    --deployment-source-url "https://github.com/user/repo" \
    --deployment-source-branch "main"

# Create Web App from container
az webapp create \
    --resource-group "RG-AppService" \
    --plan "MyAppServicePlan" \
    --name "myuniquewebapp123" \
    --deployment-container-image-name "nginx:latest"

# Get Web App details
az webapp show \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123"

# List all Web Apps
az webapp list --output table

# List Web Apps in resource group
az webapp list \
    --resource-group "RG-AppService" \
    --output table

# Get Web App URL
az webapp show \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --query "defaultHostName" \
    --output tsv

# Start Web App
az webapp start \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123"

# Stop Web App
az webapp stop \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123"

# Restart Web App
az webapp restart \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123"

# Configure app settings
az webapp config appsettings set \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --settings KEY1=VALUE1 KEY2=VALUE2

# List app settings
az webapp config appsettings list \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123"

# Configure connection strings
az webapp config connection-string set \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --connection-string-type "SQLAzure" \
    --settings DB="Server=tcp:myserver.database.windows.net..."

# Enable HTTPS only
az webapp update \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --https-only true

# Configure autoscale for App Service Plan
az monitor autoscale create \
    --resource-group "RG-AppService" \
    --resource "MyAppServicePlan" \
    --resource-type "Microsoft.Web/serverfarms" \
    --name "AppService-Autoscale" \
    --min-count 2 \
    --max-count 5 \
    --count 2

# Add autoscale rule
az monitor autoscale rule create \
    --resource-group "RG-AppService" \
    --autoscale-name "AppService-Autoscale" \
    --condition "CpuPercentage > 75 avg 5m" \
    --scale out 1

# Create deployment slot
az webapp deployment slot create \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --slot "staging"

# List deployment slots
az webapp deployment slot list \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --output table

# Swap deployment slots
az webapp deployment slot swap \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --slot "staging" \
    --target-slot "production"

# Configure custom domain
az webapp config hostname add \
    --resource-group "RG-AppService" \
    --webapp-name "myuniquewebapp123" \
    --hostname "www.contoso.com"

# Bind SSL certificate
az webapp config ssl bind \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --certificate-thumbprint "<thumbprint>" \
    --ssl-type "SNI"

# Enable backup
az webapp config backup create \
    --resource-group "RG-AppService" \
    --webapp-name "myuniquewebapp123" \
    --backup-name "backup1" \
    --storage-account-url "<sas-url>"

# View logs
az webapp log tail \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123"

# Download logs
az webapp log download \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --log-file "logs.zip"

# Delete deployment slot
az webapp deployment slot delete \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123" \
    --slot "staging"

# Delete Web App
az webapp delete \
    --resource-group "RG-AppService" \
    --name "myuniquewebapp123"

# Delete App Service Plan
az appservice plan delete \
    --resource-group "RG-AppService" \
    --name "MyAppServicePlan" \
    --yes
```

---

## Container Instances

```bash
# Create container group (simple)
az container create \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" \
    --dns-name-label "mycontainer-dns" \
    --ports 80 \
    --cpu 1 \
    --memory 1.5 \
    --os-type Linux

# Create container with environment variables
az container create \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" \
    --dns-name-label "mycontainer-dns" \
    --ports 80 \
    --environment-variables 'KEY1'='VALUE1' 'KEY2'='VALUE2' \
    --cpu 1 \
    --memory 1.5

# Create container with Azure Files mount
az container create \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" \
    --azure-file-volume-account-name "mystorageaccount" \
    --azure-file-volume-account-key "<storage-key>" \
    --azure-file-volume-share-name "myshare" \
    --azure-file-volume-mount-path "/mnt/myshare"

# Create container with restart policy
az container create \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" \
    --restart-policy "OnFailure" \
    --cpu 1 \
    --memory 1.5

# Create container from private registry
az container create \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --image "myregistry.azurecr.io/myimage:v1" \
    --registry-login-server "myregistry.azurecr.io" \
    --registry-username "<username>" \
    --registry-password "<password>" \
    --dns-name-label "mycontainer-dns" \
    --ports 80

# Create container group with multiple containers
az container create \
    --resource-group "RG-Containers" \
    --name "mycontainergroup" \
    --image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" \
    --ports 80 \
    --cpu 1 \
    --memory 1.5 \
    --location "eastus" \
    --yaml-file container-group.yaml

# Get container details
az container show \
    --resource-group "RG-Containers" \
    --name "mycontainer"

# List all containers
az container list --output table

# List containers in resource group
az container list \
    --resource-group "RG-Containers" \
    --output table

# Get container logs
az container logs \
    --resource-group "RG-Containers" \
    --name "mycontainer"

# Get container logs (with container name in group)
az container logs \
    --resource-group "RG-Containers" \
    --name "mycontainergroup" \
    --container-name "container1"

# Stream container logs
az container logs \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --follow

# Execute command in running container
az container exec \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --exec-command "/bin/bash"

# Execute specific command
az container exec \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --exec-command "ls -la /app"

# Attach to container (view output)
az container attach \
    --resource-group "RG-Containers" \
    --name "mycontainer"

# Restart container
az container restart \
    --resource-group "RG-Containers" \
    --name "mycontainer"

# Stop container
az container stop \
    --resource-group "RG-Containers" \
    --name "mycontainer"

# Start container
az container start \
    --resource-group "RG-Containers" \
    --name "mycontainer"

# Delete container
az container delete \
    --resource-group "RG-Containers" \
    --name "mycontainer" \
    --yes
```

---

## Azure Kubernetes Service

```bash
# Create AKS cluster (simple)
az aks create \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --location "eastus" \
    --node-count 3 \
    --node-vm-size "Standard_D2s_v3" \
    --kubernetes-version "1.27.3" \
    --enable-managed-identity \
    --generate-ssh-keys

# Create AKS with specific network configuration
az aks create \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --location "eastus" \
    --node-count 3 \
    --node-vm-size "Standard_D2s_v3" \
    --network-plugin "azure" \
    --vnet-subnet-id "/subscriptions/{sub-id}/resourceGroups/RG-Network/providers/Microsoft.Network/virtualNetworks/VNet-Prod/subnets/AKS-Subnet" \
    --enable-managed-identity \
    --generate-ssh-keys

# Create AKS with availability zones
az aks create \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --location "eastus" \
    --node-count 9 \
    --zones 1 2 3 \
    --node-vm-size "Standard_D2s_v3" \
    --enable-managed-identity \
    --generate-ssh-keys

# Create AKS with autoscaling
az aks create \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --location "eastus" \
    --node-count 3 \
    --min-count 2 \
    --max-count 10 \
    --enable-cluster-autoscaler \
    --node-vm-size "Standard_D2s_v3" \
    --enable-managed-identity \
    --generate-ssh-keys

# Get AKS cluster details
az aks show \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster"

# List all AKS clusters
az aks list --output table

# Get available Kubernetes versions
az aks get-versions --location "eastus" --output table

# Get credentials for kubectl
az aks get-credentials \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster"

# Get credentials with admin rights
az aks get-credentials \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --admin

# Get credentials and overwrite existing
az aks get-credentials \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --overwrite-existing

# After getting credentials, verify with kubectl
kubectl get nodes
kubectl get pods --all-namespaces
kubectl cluster-info

# Scale AKS cluster manually
az aks scale \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --node-count 5

# Update autoscaler settings
az aks update \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --enable-cluster-autoscaler \
    --min-count 2 \
    --max-count 10

# Disable autoscaler
az aks update \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --disable-cluster-autoscaler

# Add node pool
az aks nodepool add \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --name "userpool" \
    --node-count 3 \
    --node-vm-size "Standard_D4s_v3"

# Add node pool with autoscaling
az aks nodepool add \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --name "userpool" \
    --node-count 3 \
    --min-count 2 \
    --max-count 10 \
    --enable-cluster-autoscaler \
    --node-vm-size "Standard_D4s_v3"

# Add node pool with availability zones
az aks nodepool add \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --name "userpool" \
    --node-count 9 \
    --zones 1 2 3 \
    --node-vm-size "Standard_D4s_v3"

# List node pools
az aks nodepool list \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --output table

# Show node pool details
az aks nodepool show \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --name "userpool"

# Scale node pool
az aks nodepool scale \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --name "userpool" \
    --node-count 5

# Update node pool
az aks nodepool update \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --name "userpool" \
    --enable-cluster-autoscaler \
    --min-count 2 \
    --max-count 10

# Delete node pool
az aks nodepool delete \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --name "userpool"

# Upgrade AKS cluster
az aks upgrade \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --kubernetes-version "1.28.0"

# Upgrade only control plane
az aks upgrade \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --kubernetes-version "1.28.0" \
    --control-plane-only

# Upgrade specific node pool
az aks nodepool upgrade \
    --resource-group "RG-AKS" \
    --cluster-name "MyAKSCluster" \
    --name "nodepool1" \
    --kubernetes-version "1.28.0"

# Get upgrade versions available
az aks get-upgrades \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --output table

# Enable monitoring addon
az aks enable-addons \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --addons "monitoring" \
    --workspace-resource-id "/subscriptions/{sub-id}/resourcegroups/RG-Monitoring/providers/microsoft.operationalinsights/workspaces/MyWorkspace"

# Disable addon
az aks disable-addons \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --addons "monitoring"

# Update AKS cluster tags
az aks update \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --tags Environment=Production Team=DevOps

# Start AKS cluster (if stopped)
az aks start \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster"

# Stop AKS cluster (save costs)
az aks stop \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster"

# Get AKS dashboard credentials
az aks browse \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster"

# Run kubectl command without installing kubectl
az aks command invoke \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --command "kubectl get pods -n default"

# Delete AKS cluster
az aks delete \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --yes

# Delete AKS cluster and wait for completion
az aks delete \
    --resource-group "RG-AKS" \
    --name "MyAKSCluster" \
    --yes \
    --no-wait
```

---

## VM Backup

```bash
# Create Recovery Services Vault
az backup vault create \
    --resource-group "RG-Backup" \
    --name "MyBackupVault" \
    --location "eastus"

# Get vault details
az backup vault show \
    --resource-group "RG-Backup" \
    --name "MyBackupVault"

# List all vaults
az backup vault list --output table

# Set vault backup properties (storage redundancy)
az backup vault backup-properties set \
    --resource-group "RG-Backup" \
    --name "MyBackupVault" \
    --backup-storage-redundancy "GeoRedundant"

# Set vault backup properties to Locally Redundant
az backup vault backup-properties set \
    --resource-group "RG-Backup" \
    --name "MyBackupVault" \
    --backup-storage-redundancy "LocallyRedundant"

# Show vault backup properties
az backup vault backup-properties show \
    --resource-group "RG-Backup" \
    --name "MyBackupVault"

# List backup policies
az backup policy list \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --output table

# Show specific policy
az backup policy show \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --name "DefaultPolicy"

# Create custom backup policy from JSON
az backup policy create \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --name "CustomVMPolicy" \
    --policy @policy.json

# Set policy (update existing)
az backup policy set \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --policy @policy.json

# Delete policy
az backup policy delete \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --name "CustomVMPolicy"

# Enable backup for VM with default policy
az backup protection enable-for-vm \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --vm "MyVM" \
    --policy-name "DefaultPolicy"

# Enable backup for VM in different resource group
az backup protection enable-for-vm \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --vm "/subscriptions/{sub-id}/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM" \
    --policy-name "DefaultPolicy"

# List protected VMs
az backup item list \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --backup-management-type "AzureIaasVM" \
    --output table

# Show backup item details
az backup item show \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --backup-management-type "AzureIaasVM"

# Trigger backup immediately (on-demand)
az backup protection backup-now \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --backup-management-type "AzureIaasVM" \
    --retain-until "31-12-2024"

# List backup jobs
az backup job list \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --output table

# Show specific job
az backup job show \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --name "<job-id>"

# Stop running backup job
az backup job stop \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --name "<job-id>"

# Wait for job to complete
az backup job wait \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --name "<job-id>"

# List recovery points
az backup recoverypoint list \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --backup-management-type "AzureIaasVM" \
    --output table

# Show recovery point details
az backup recoverypoint show \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --backup-management-type "AzureIaasVM" \
    --name "<recovery-point-name>"

# Restore VM (create new VM)
az backup restore restore-disks \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --rp-name "<recovery-point-name>" \
    --storage-account "restorestorage" \
    --restore-mode "AlternateLocation" \
    --target-resource-group "RG-Restore"

# Restore disks only
az backup restore restore-disks \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --rp-name "<recovery-point-name>" \
    --storage-account "restorestorage"

# Restore to original location (replace)
az backup restore restore-disks \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --rp-name "<recovery-point-name>" \
    --storage-account "restorestorage" \
    --restore-mode "OriginalLocation"

# Restore files (file-level recovery)
az backup restore files mount-rp \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --rp-name "<recovery-point-name>"

# Unmount recovery point (after file recovery)
az backup restore files unmount-rp \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --rp-name "<recovery-point-name>"

# Update backup policy for protected item
az backup item set-policy \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --policy-name "NewPolicy" \
    --backup-management-type "AzureIaasVM"

# Disable backup (keep recovery points)
az backup protection disable \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --backup-management-type "AzureIaasVM" \
    --yes

# Disable backup and delete backup data
az backup protection disable \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --backup-management-type "AzureIaasVM" \
    --delete-backup-data true \
    --yes

# Resume backup protection
az backup protection resume \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VM;MyVM" \
    --item-name "vm;iaasvmcontainerv2;RG-VM;MyVM" \
    --backup-management-type "AzureIaasVM"

# Check backup status
az backup protection check-vm \
    --resource-group "RG-Backup" \
    --vault-name "MyBackupVault" \
    --vm-id "/subscriptions/{sub-id}/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM"

# Delete Recovery Services Vault
az backup vault delete \
    --resource-group "RG-Backup" \
    --name "MyBackupVault" \
    --yes
```

---

## Additional Useful Commands

### Resource Management

```bash
# List all resources in resource group
az resource list \
    --resource-group "RG-VM" \
    --output table

# Show resource details
az resource show \
    --ids "/subscriptions/{sub-id}/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM"

# Delete resource
az resource delete \
    --ids "/subscriptions/{sub-id}/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM"

# Move resources to another resource group
az resource move \
    --destination-group "RG-New" \
    --ids "/subscriptions/{sub-id}/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM"
```

### Monitoring and Diagnostics

```bash
# Enable boot diagnostics on VM
az vm boot-diagnostics enable \
    --resource-group "RG-VM" \
    --name "MyVM" \
    --storage "https://mystorageaccount.blob.core.windows.net/"

# Get boot diagnostics logs
az vm boot-diagnostics get-boot-log \
    --resource-group "RG-VM" \
    --name "MyVM"

# Get instance view (detailed status)
az vm get-instance-view \
    --resource-group "RG-VM" \
    --name "MyVM"

# List available VM extensions
az vm extension image list \
    --location "eastus" \
    --output table

# Install VM extension
az vm extension set \
    --resource-group "RG-VM" \
    --vm-name "MyVM" \
    --name "CustomScriptExtension" \
    --publisher "Microsoft.Compute" \
    --version "1.10" \
    --settings '{"fileUris":["https://mystorageaccount.blob.core.windows.net/scripts/install.ps1"],"commandToExecute":"powershell -ExecutionPolicy Unrestricted -File install.ps1"}'

# List installed extensions
az vm extension list \
    --resource-group "RG-VM" \
    --vm-name "MyVM" \
    --output table

# Delete VM extension
az vm extension delete \
    --resource-group "RG-VM" \
    --vm-name "MyVM" \
    --name "CustomScriptExtension"
```

### Tags Management

```bash
# Add tags to resource
az resource tag \
    --tags Environment=Production CostCenter=IT \
    --ids "/subscriptions/{sub-id}/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM"

# Update tags (incremental)
az resource update \
    --set tags.Owner=JohnDoe \
    --ids "/subscriptions/{sub-id}/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM"

# Remove all tags
az resource tag \
    --tags "" \
    --ids "/subscriptions/{sub-id}/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM"

# List resources by tag
az resource list \
    --tag Environment=Production \
    --output table
```

### Cost Management

```bash
# Show costs for resource group
az consumption usage list \
    --start-date 2024-01-01 \
    --end-date 2024-01-31 \
    --resource-group "RG-VM"

# Show current month costs
az consumption usage list \
    --start-date $(date -u -d "$(date +%Y-%m-01)" +%Y-%m-%d) \
    --end-date $(date -u +%Y-%m-%d)
```

---

## Quick Reference - Most Used Commands

### VM Management (Top 10)

```bash
# 1. Create VM
az vm create --resource-group "RG" --name "VM" --image "Ubuntu2204" --size "Standard_D2s_v3"

# 2. Start VM
az vm start --resource-group "RG" --name "VM"

# 3. Stop VM (deallocate)
az vm deallocate --resource-group "RG" --name "VM"

# 4. Get VM status
az vm get-instance-view --resource-group "RG" --name "VM" --query "instanceView.statuses[?starts_with(code, 'PowerState/')]"

# 5. Resize VM
az vm resize --resource-group "RG" --name "VM" --size "Standard_D4s_v3"

# 6. List VMs
az vm list --output table

# 7. Delete VM
az vm delete --resource-group "RG" --name "VM" --yes

# 8. Add data disk
az vm disk attach --resource-group "RG" --vm-name "VM" --name "disk1" --new --size-gb 128

# 9. Run command on VM
az vm run-command invoke --resource-group "RG" --name "VM" --command-id "RunShellScript" --scripts "ifconfig"

# 10. Get VM IP
az vm list-ip-addresses --resource-group "RG" --name "VM" --output table
```

### Scale Sets (Top 5)

```bash
# 1. Create VMSS
az vmss create --resource-group "RG" --name "VMSS" --image "Ubuntu2204" --instance-count 3

# 2. Scale VMSS
az vmss scale --resource-group "RG" --name "VMSS" --new-capacity 5

# 3. Update instances
az vmss update-instances --resource-group "RG" --name "VMSS" --instance-ids "*"

# 4. Configure autoscale
az monitor autoscale create --resource "VMSS" --resource-type "Microsoft.Compute/virtualMachineScaleSets" --min-count 2 --max-count 10

# 5. List instances
az vmss list-instances --resource-group "RG" --name "VMSS" --output table
```

### App Service (Top 5)

```bash
# 1. Create App Service Plan
az appservice plan create --resource-group "RG" --name "Plan" --sku "S1"

# 2. Create Web App
az webapp create --resource-group "RG" --plan "Plan" --name "webapp"

# 3. Stop Web App
az webapp stop --resource-group "RG" --name "webapp"

# 4. Create deployment slot
az webapp deployment slot create --resource-group "RG" --name "webapp" --slot "staging"

# 5. Swap slots
az webapp deployment slot swap --resource-group "RG" --name "webapp" --slot "staging"
```

### Containers (Top 3)

```bash
# 1. Create container
az container create --resource-group "RG" --name "container" --image "nginx" --ports 80

# 2. Get logs
az container logs --resource-group "RG" --name "container"

# 3. Delete container
az container delete --resource-group "RG" --name "container" --yes
```

### AKS (Top 5)

```bash
# 1. Create AKS
az aks create --resource-group "RG" --name "AKS" --node-count 3

# 2. Get credentials
az aks get-credentials --resource-group "RG" --name "AKS"

# 3. Scale AKS
az aks scale --resource-group "RG" --name "AKS" --node-count 5

# 4. Upgrade AKS
az aks upgrade --resource-group "RG" --name "AKS" --kubernetes-version "1.28.0"

# 5. Stop AKS
az aks stop --resource-group "RG" --name "AKS"
```

---

## Common Exam Scenarios

### Scenario 1: VM Performance Issues

**Problem:**
VM running slow, users complaining about poor performance.

**Troubleshooting steps:**

1. **Check VM size:**

   - Is VM properly sized for workload?
   - Burstable B-series out of credits?
   - Solution: Resize to appropriate family

2. **Check disk performance:**

   - Standard HDD too slow?
   - IOPS throttling?
   - Solution: Upgrade to Premium SSD

3. **Check CPU/memory metrics:**

   - Consistently high (>80%)?
   - Solution: Scale up (larger size) or scale out (more instances)

4. **Check network:**
   - Network throttling?
   - NSG blocking traffic?
   - Solution: Review network configuration

**PowerShell check:**

```powershell
# Get VM performance metrics
$vm = Get-AzVM -ResourceGroupName "RG-VM" -Name "MyVM"
Get-AzMetric -ResourceId $vm.Id -MetricName "Percentage CPU"
```

### Scenario 2: High Availability Requirement

**Problem:**
Application must have 99.99% uptime SLA.

**Solution options:**

**Option 1: Availability Zones (recommended)**

```
Deploy VMs across 3 availability zones
Use zone-redundant load balancer
SLA: 99.99%
```

**Option 2: Availability Set**

```
Deploy VMs in availability set
Use standard load balancer
SLA: 99.95% (doesn't meet requirement)
```

**Option 3: VM Scale Set with zones**

```
VMSS with zone redundancy
Automatic scaling
SLA: 99.99%
Best for scalable workloads
```

**Answer:** Use Availability Zones or VMSS with zones for 99.99% SLA.

### Scenario 3: Cost Optimization for Dev/Test

**Problem:**
Development VMs running 24/7, high costs.

**Solutions:**

1. **Use B-series burstable VMs:**

   - Accumulate credits when idle
   - Cost-effective for dev/test
   - 40-60% cheaper than regular VMs

2. **Auto-shutdown schedule:**

   - Stop VMs at 6 PM daily
   - Start at 8 AM daily
   - Save 60% on compute costs

3. **Deallocate when not in use:**

   - Stop VMs over weekends
   - No compute charges when deallocated
   - Only pay for storage

4. **Use Dev/Test pricing:**
   - Special rates for Visual Studio subscribers
   - Up to 55% discount on Windows VMs

**Implementation:**

```powershell
# Configure auto-shutdown
$vm = Get-AzVM -ResourceGroupName "RG-VM" -Name "DevVM"
Set-AzVMAutoShutdownConfig -ResourceId $vm.Id `
    -Time "18:00" `
    -TimeZone "Eastern Standard Time" `
    -NotificationEmail "admin@contoso.com"
```

### Scenario 4: Application Needs to Scale Automatically

**Problem:**
Web application experiences traffic spikes. Need automatic scaling.

**Best solution: VM Scale Set with autoscale**

**Configuration:**

```
Min instances: 2 (always running)
Max instances: 10 (cost limit)

Scale out: CPU > 70% for 5 min → Add 2 instances
Scale in: CPU < 30% for 10 min → Remove 1 instance

Benefits:
- Automatic response to load
- Cost optimization (scale down when idle)
- High availability (multiple instances)
```

**Alternative: App Service with autoscale**

- If application is web app without special requirements
- Simpler management (PaaS)
- Easier deployment

### Scenario 5: Cannot Connect to VM via RDP/SSH

**Problem:**
Cannot connect to Windows VM via RDP (port 3389).

**Troubleshooting:**

1. **Check NSG rules:**

   - Is port 3389 allowed?
   - Check NSG on NIC and subnet
   - Solution: Add allow rule with priority <65000

2. **Check public IP:**

   - Does VM have public IP?
   - Is IP static or dynamic?
   - Solution: Assign public IP

3. **Check VM is running:**

   - Is VM in "Running" state?
   - Solution: Start VM

4. **Check Windows Firewall (inside VM):**

   - Is RDP enabled?
   - Solution: Use Serial Console or Run Command

5. **Check source IP:**
   - Is your office IP allowed?
   - Solution: Add your IP to NSG

**PowerShell check:**

```powershell
# Check NSG rules
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "RG-VM" -Name "MyVM-NSG"
$nsg.SecurityRules | Where-Object {$_.DestinationPortRange -eq 3389}

# Check VM status
Get-AzVM -ResourceGroupName "RG-VM" -Name "MyVM" -Status
```

### Scenario 6: Need to Run Containers Without Kubernetes

**Problem:**
Need to run containers but Kubernetes is too complex.

**Solutions:**

**Option 1: Azure Container Instances (ACI)**

- Simple, single containers or small groups
- Fast deployment
- Per-second billing
- Use for: Simple apps, batch jobs, CI/CD agents

**Option 2: App Service for Containers**

- Web apps in containers
- Managed platform
- Built-in features (SSL, autoscale, slots)
- Use for: Web applications

**Option 3: Container Apps**

- Serverless containers
- Automatic scaling (to zero)
- Event-driven
- Use for: Microservices, event processing

**Decision:**

```
Single simple container? → ACI
Web application? → App Service
Complex microservices? → AKS
Serverless needs? → Container Apps
```

### Scenario 7: Disaster Recovery for Critical VM

**Problem:**
SQL Server VM in East US must survive regional disaster.

**Solution: Azure Site Recovery**

**Implementation:**

1. Create Recovery Services Vault in West US
2. Enable replication for SQL VM
3. Set RPO to 30 seconds
4. Create recovery plan with scripts
5. Test failover regularly

**Configuration:**

```
Primary: East US (SQL Server VM)
Secondary: West US (replica, powered off)
Replication: Continuous (30-second RPO)
Recovery Plan:
  1. Failover VM
  2. Run script to update DNS
  3. Notify administrators
RTO: 2-4 hours
```

**Cost:**

- ~$25/month per VM
- Storage for replica disks
- Worth it for critical workloads

### Scenario 8: App Service vs VM Decision

**Problem:**
New web application. Should you use App Service or VM?

**Use App Service if:**

- Standard web application (.NET, Java, Node.js, Python, PHP)
- Don't need OS-level access
- Want automatic patching
- Want built-in scaling and SSL
- Focus on code, not infrastructure
- Team familiar with PaaS

**Use VM if:**

- Custom OS configuration needed
- Legacy application with specific dependencies
- Need root/admin access
- Special software requirements
- Application doesn't fit PaaS model
- Lift-and-shift from on-premises

**Example scenarios:**

**App Service:**

- REST API in Node.js
- ASP.NET web application
- PHP WordPress site (with modifications)
- Python Flask application

**VM:**

- SAP application
- Oracle database
- Custom Windows service
- Application with specific kernel requirements

---

## Key Takeaways for Exam

### Must-Know Concepts

1. **VM Sizes and Families**

   - General Purpose (D-series): Balanced workloads
   - Compute Optimized (F-series): High CPU
   - Memory Optimized (E-series): Databases
   - Storage Optimized (L-series): Big data
   - GPU (N-series): ML, rendering
   - Burstable (B-series): Dev/test, low baseline CPU

2. **Disk Types**

   - Standard HDD: Cheapest, backups
   - Standard SSD: Web servers, light apps
   - Premium SSD: Production databases
   - Ultra Disk: Mission-critical
   - Managed Disks: Always recommended

3. **Availability Options**

   - Single VM: 99.9% (Premium SSD)
   - Availability Set: 99.95%
   - Availability Zones: 99.99% (best)
   - VM Scale Sets: Scalability + availability

4. **VM States and Billing**

   - Running: Fully billed
   - Stopped (Allocated): Still billed
   - Stopped (Deallocated): Only storage billed
   - Use deallocate to save money

5. **App Service**

   - PaaS for web applications
   - Tiers: Free, Shared, Basic, Standard, Premium, Isolated
   - Deployment slots for zero-downtime updates
   - Autoscaling in Standard tier and above

6. **Containers**

   - ACI: Simple containers, fast deployment
   - AKS: Kubernetes orchestration
   - When to use each

7. **Backup and DR**
   - Azure Backup: Point-in-time recovery
   - Site Recovery: Regional DR, low RPO
   - Use both for comprehensive protection

### Common Exam Tricks

**Trick 1: "VM still billed after stopping"**

- Must deallocate (stop from Azure), not from inside OS

**Trick 2: "Cannot add VM to existing availability set"**

- Must be in availability set from creation
- Cannot change afterward

**Trick 3: "Availability Zones not available"**

- Not all regions support zones
- Check region capabilities first

**Trick 4: "B-series VM performance issues"**

- Out of CPU credits
- Not suitable for sustained high CPU

**Trick 5: "Cannot change from Standard to Premium storage"**

- Must create new VM
- Or upgrade disk SKU (possible in some cases)

**Trick 6: "App Service deployment slot swap"**

- Swaps app content and settings
- Does NOT swap: IP address, SSL certificates, scale settings

**Trick 7: "VMSS not scaling"**

- Check autoscale rules enabled
- Check metrics (may not meet threshold)
- Check cooldown period

**Trick 8: "AKS control plane charges"**

- Control plane is free
- Pay only for worker nodes (VMs)

### Quick Reference Tables

**VM Size Selection:**

| Workload         | VM Family | Example Size |
| ---------------- | --------- | ------------ |
| Web server       | D-series  | D4s_v3       |
| SQL Database     | E-series  | E8s_v3       |
| Batch processing | F-series  | F8s_v3       |
| Dev/test         | B-series  | B2ms         |
| ML training      | N-series  | NC6          |

**Availability Options:**

| Option              | SLA    | Protection   | Cost      | Use Case        |
| ------------------- | ------ | ------------ | --------- | --------------- |
| Single VM (Premium) | 99.9%  | Hardware     | Standard  | Non-critical    |
| Availability Set    | 99.95% | Rack failure | Free      | Multi-tier apps |
| Availability Zones  | 99.99% | Datacenter   | Bandwidth | Production      |
| VMSS                | 99.99% | + Scaling    | Bandwidth | Scalable apps   |

**Compute Service Selection:**

| Need                      | Service             |
| ------------------------- | ------------------- |
| Full OS control           | Virtual Machine     |
| Web application (managed) | App Service         |
| Simple container          | Container Instances |
| Container orchestration   | AKS                 |
| Serverless code           | Azure Functions     |
| Batch jobs                | Azure Batch         |

### Top 10 PowerShell Commands

```powershell
# 1. Create VM
New-AzVM -ResourceGroupName "RG" -Name "VM" -Location "eastus" -Image "Win2022Datacenter" -Size "Standard_D2s_v3"

# 2. Start VM
Start-AzVM -ResourceGroupName "RG" -Name "VM"

# 3. Stop VM (deallocate)
Stop-AzVM -ResourceGroupName "RG" -Name "VM" -Force

# 4. Get VM status
Get-AzVM -ResourceGroupName "RG" -Name "VM" -Status

# 5. Resize VM
$vm = Get-AzVM -ResourceGroupName "RG" -Name "VM"
$vm.HardwareProfile.VmSize = "Standard_D4s_v3"
Update-AzVM -ResourceGroupName "RG" -VM $vm

# 6. Create availability set
New-AzAvailabilitySet -ResourceGroupName "RG" -Name "AS" -Location "eastus" -Sku "Aligned"

# 7. Create VM Scale Set
New-AzVmss -ResourceGroupName "RG" -VMScaleSetName "VMSS" -Location "eastus"

# 8. Create App Service Plan
New-AzAppServicePlan -ResourceGroupName "RG" -Name "Plan" -Location "eastus" -Tier "Standard"

# 9. Create Web App
New-AzWebApp -ResourceGroupName "RG" -Name "webapp" -Location "eastus" -AppServicePlan "Plan"

# 10. Enable VM backup
Enable-AzRecoveryServicesBackupProtection -ResourceGroupName "RG" -Name "VM" -Policy $policy
```

### Top 10 Azure CLI Commands

```bash
# 1. Create VM
az vm create --resource-group "RG" --name "VM" --image "Ubuntu2204" --size "Standard_D2s_v3"

# 2. Start VM
az vm start --resource-group "RG" --name "VM"

# 3. Stop VM (deallocate)
az vm deallocate --resource-group "RG" --name "VM"

# 4. Get VM status
az vm get-instance-view --resource-group "RG" --name "VM"

# 5. Resize VM
az vm resize --resource-group "RG" --name "VM" --size "Standard_D4s_v3"

# 6. Create availability set
az vm availability-set create --resource-group "RG" --name "AS"

# 7. Create VM Scale Set
az vmss create --resource-group "RG" --name "VMSS" --image "Ubuntu2204"

# 8. Create App Service Plan
az appservice plan create --resource-group "RG" --name "Plan" --sku "S1"

# 9. Create Web App
az webapp create --resource-group "RG" --plan "Plan" --name "webapp"

# 10. Enable VM backup
az backup protection enable-for-vm --resource-group "RG" --vault-name "Vault" --vm "VM"
```

---

## Practice Questions

### Question 1

You have a VM running in Azure with a Standard HDD disk. Application performance is slow. What is the quickest way to improve performance?

**A)** Resize VM to larger size  
**B)** Change disk to Premium SSD  
**C)** Add more data disks  
**D)** Move VM to availability zone

**Answer:** B

**Explanation:**

- Changing disk type from Standard HDD to Premium SSD significantly improves IOPS and throughput
- Can be done without recreating VM (in most cases)
- More cost-effective than larger VM size if disk is bottleneck
- A incorrect: May help CPU/memory but not disk I/O
- C incorrect: Doesn't improve single disk performance
- D incorrect: Zones don't improve performance, only availability

### Question 2

Your organization requires 99.99% uptime SLA for a web application. The application runs on VMs. What should you implement?

**A)** Single VM with Premium SSD  
**B)** Two VMs in availability set  
**C)** Three VMs across availability zones  
**D)** VM Scale Set with manual scaling

**Answer:** C

**Explanation:**

- Availability Zones provide 99.99% SLA
- Protects against datacenter-level failures
- A incorrect: Single VM only provides 99.9%
- B incorrect: Availability Set provides 99.95%
- D incorrect: Manual scaling doesn't affect SLA, needs zone redundancy

### Question 3

You need to automatically scale VMs based on CPU usage. Min 2 instances, max 10 instances. What should you create?

**A)** Availability Set with 2 VMs  
**B)** VM Scale Set with autoscale rules  
**C)** Two separate VMs with manual scaling  
**D)** App Service with Basic tier

**Answer:** B

**Explanation:**

- VM Scale Set supports automatic scaling based on metrics
- Can configure min/max instances and scale rules
- A incorrect: Availability Set doesn't provide autoscaling
- C incorrect: Manual scaling requires human intervention
- D incorrect: Basic tier doesn't support autoscaling (needs Standard or higher)

### Question 4

You stopped a VM from inside the Windows operating system. What happens to billing?

**A)** Compute charges stop immediately  
**B)** Compute charges continue, storage charges stop  
**C)** Both compute and storage charges continue  
**D)** Both charges stop

**Answer:** C

**Explanation:**

- Stopping from inside OS = Stopped (Allocated) state
- VM still allocated in Azure, still billed for compute + storage
- Must stop/deallocate from Azure Portal/PowerShell/CLI to stop compute charges
- Only stopping from Azure deallocates resources

### Question 5

Your web application needs zero-downtime deployments. You don't need OS-level control. What should you use?

**A)** Virtual Machine with Load Balancer  
**B)** App Service with deployment slots  
**C)** VM Scale Set  
**D)** Container Instances

**Answer:** B

**Explanation:**

- App Service deployment slots allow testing in staging, then instant swap to production
- Zero downtime during swap
- PaaS service, no OS management needed
- A incorrect: More complex, requires manual orchestration
- C incorrect: More complex than needed for simple web app
- D incorrect: No built-in deployment slot feature

### Question 6

You need to run a batch processing job that takes 30 minutes, once per day. The job is containerized. What's the most cost-effective solution?

**A)** AKS cluster  
**B)** VM running 24/7  
**C)** Azure Container Instances with restart policy Never  
**D)** App Service

**Answer:** C

**Explanation:**

- ACI bills per second, perfect for short-running tasks
- Restart policy "Never" means runs once and stops
- Pay only for 30 minutes per day
- A incorrect: AKS overkill, pay for nodes 24/7
- B incorrect: Pay for VM 24/7 even when idle
- D incorrect: App Service bills per hour, not optimized for batch jobs

### Question 7

A VM needs to survive zone failure and rack failure within a zone. What should you configure?

**A)** Availability Set  
**B)** Single VM with Premium SSD  
**C)** VMs across Availability Zones  
**D)** Proximity Placement Group

**Answer:** C

**Explanation:**

- Availability Zones protect against entire datacenter failure
- Each zone has multiple racks (implicit rack protection)
- A incorrect: Only protects against rack failure, not zone failure
- B incorrect: No high availability, single point of failure
- D incorrect: Reduces availability (keeps VMs close together)

### Question 8

You need to restore a single file from a VM backup taken yesterday. What's the fastest method?

**A)** Restore entire VM to new VM, then copy file  
**B)** Restore disks, attach to existing VM, copy file  
**C)** Use file-level restore feature  
**D)** Create new VM from backup

**Answer:** C

**Explanation:**

- File-level restore mounts backup as iSCSI drive
- Browse and copy specific files
- Much faster than full VM restore
- A incorrect: Slow, creates unnecessary VM
- B incorrect: Slower than file-level restore
- D incorrect: Same as A, unnecessary full restore

---

## Summary Checklist

Before moving to Module 4, ensure you can:

**Virtual Machines:**

- [ ] Explain VM families and when to use each
- [ ] Understand disk types and performance tiers
- [ ] Create and manage VMs via Portal, PowerShell, CLI
- [ ] Configure VM networking (NSG, public IP, VNet)
- [ ] Understand VM states and billing implications
- [ ] Manage VM disks (OS, data, temporary)

**Availability and Scaling:**

- [ ] Explain Availability Sets (fault/update domains)
- [ ] Explain Availability Zones and when to use them
- [ ] Understand SLA differences
- [ ] Create and configure VM Scale Sets
- [ ] Configure autoscaling rules
- [ ] Understand load balancing basics

**App Service:**

- [ ] Explain App Service Plans and tiers
- [ ] Understand when to use App Service vs VMs
- [ ] Configure deployment slots
- [ ] Implement autoscaling
- [ ] Understand App Service Environment

**Containers:**

- [ ] Understand containers vs VMs
- [ ] Create and manage Azure Container Instances
- [ ] Understand basic Kubernetes concepts
- [ ] Know when to use ACI vs AKS
- [ ] Create and manage AKS clusters (basic operations)

**Backup and DR:**

- [ ] Configure Azure Backup for VMs
- [ ] Create backup policies
- [ ] Restore VMs and files
- [ ] Understand Azure Site Recovery
- [ ] Configure VM replication
- [ ] Understand RPO and RTO

**PowerShell & CLI:**

- [ ] Execute common VM management commands
- [ ] Create and manage scale sets
- [ ] Configure App Service
- [ ] Manage backups

**Exam Readiness:**

- [ ] Complete practice questions with 80%+ accuracy
- [ ] Explain scenarios without looking at notes
- [ ] Perform hands-on labs for all topics
- [ ] Can troubleshoot common issues

---

## Additional Resources

**Microsoft Learn Modules (Specific to Module 3):**

- [Configure virtual machines](https://learn.microsoft.com/training/modules/configure-virtual-machines/)
- [Configure virtual machine availability](https://learn.microsoft.com/training/modules/configure-virtual-machine-availability/)
- [Configure VM Scale Sets](https://learn.microsoft.com/training/modules/configure-virtual-machine-scale-sets/)
- [Configure Azure App Service](https://learn.microsoft.com/training/modules/configure-azure-app-services/)
- [Configure Azure Container Instances](https://learn.microsoft.com/training/modules/run-docker-with-azure-container-instances/)
- [Introduction to Azure Kubernetes Service](https://learn.microsoft.com/training/modules/intro-to-azure-kubernetes-service/)
- [Configure VM backups](https://learn.microsoft.com/training/modules/configure-virtual-machine-backups/)

**Documentation:**

- [Virtual Machines documentation](https://learn.microsoft.com/azure/virtual-machines/)
- [VM Scale Sets documentation](https://learn.microsoft.com/azure/virtual-machine-scale-sets/)
- [App Service documentation](https://learn.microsoft.com/azure/app-service/)
- [Container Instances documentation](https://learn.microsoft.com/azure/container-instances/)
- [AKS documentation](https://learn.microsoft.com/azure/aks/)
- [Azure Backup documentation](https://learn.microsoft.com/azure/backup/)
- [Site Recovery documentation](https://learn.microsoft.com/azure/site-recovery/)

**Video Resources:**

- [John Savill's AZ-104 Compute Deep Dive](https://www.youtube.com/watch?v=9YWBHr5-RNg)
- [Azure Virtual Machines Tutorial](https://www.youtube.com/watch?v=inaXkN2UrFE)
- [VM Scale Sets Explained](https://www.youtube.com/watch?v=Xv3vaRBfG3c)

**Hands-on Labs:**

- [Microsoft Learn Sandbox](https://learn.microsoft.com/training/browse/?products=azure)
- Create VMs in different availability configurations
- Configure VM Scale Sets with autoscale
- Deploy web app to App Service with deployment slots
- Practice VM backup and restore
- Set up Site Recovery replication

---

## Next Steps

Congratulations on completing Module 3!

**You've mastered:**

- Virtual Machines and sizing
- Availability and scaling options
- App Service and containers
- Backup and disaster recovery
- Compute service selection

**Next Module:** Module 4: Networking

In Module 4, you'll learn:

- Virtual Networks (VNets) and subnets
- Network Security Groups (NSGs) and Application Security Groups
- Azure Load Balancer and Application Gateway
- VNet Peering and VPN Gateway
- Azure DNS and Traffic Manager
- Network Watcher and monitoring

**Before moving on:**

1. Complete practice labs for Module 3
2. Take the Module 3 practice quiz
3. Review any weak areas
4. Practice PowerShell/CLI commands
5. Understand decision criteria for compute services

**Estimated time for Module 4:** 10-12 hours

---

## Study Tips for Module 3

**Week 1 Focus:**

- Virtual Machines (creation, sizing, disks)
- VM networking basics
- VM states and billing
- Practice creating VMs via different methods

**Week 2 Focus:**

- Availability Sets and Zones
- VM Scale Sets
- Autoscaling concepts
- Practice hands-on labs

**Week 3 Focus:**

- App Service and containers
- AKS basics
- Backup and Site Recovery
- Decision trees for service selection

**Week 4 Focus:**

- Practice scenarios
- Command-line proficiency
- Time yourself on practice questions
- Review all weak areas

**Common Mistakes to Avoid:**

1. Confusing VM states (stopped vs deallocated)
2. Not understanding SLA differences (Availability Set vs Zones)
3. Forgetting that tags don't inherit
4. Not knowing when to use each compute service
5. Mixing up disk types and performance tiers
6. Forgetting autoscale rule cooldown periods

**Red Flags in Questions:**

- "VM stopped but still billed" → Check if deallocated
- "Need 99.99% SLA" → Must use Availability Zones
- "B-series VM slow" → Out of CPU credits
- "Automatic scaling needed" → VM Scale Sets or App Service Standard+
- "Zero downtime deployment" → App Service slots
- "Simple container task" → ACI, not AKS
- "Regional disaster recovery" → Site Recovery

---

**Good luck with your studies!**

Module 3 covers critical infrastructure components. Understanding VMs, availability, and scaling is essential not just for the exam but for real-world Azure administration.

**Key exam tip:** Always consider the cost-to-benefit ratio when selecting compute options. Azure provides multiple solutions; choose the simplest one that meets requirements.

---

**[← Back to Module 2: Storage](../Module-02-Storage/README.md) | [Next: Module 4: Networking →](../Module-04-Networking/README.md)**
