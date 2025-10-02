# AZ-104 Cheat Sheets - Quick Reference

## Overview

This section contains quick reference materials for the AZ-104 exam. Print these out or keep them handy for last-minute review.

**Contents:**
1. PowerShell Commands Quick Reference
2. Azure CLI Commands Quick Reference
3. Decision Trees
4. CIDR Calculator and Subnetting
5. SKU Comparison Tables
6. Common Ports Reference
7. SLA Reference
8. Pricing Quick Guide
9. Keyboard Shortcuts (Portal)
10. Exam Day Checklist

---

## 1. PowerShell Commands Quick Reference

### Essential Commands (Top 20)

```powershell
# Authentication
Connect-AzAccount
Set-AzContext -SubscriptionName "Production"
Get-AzSubscription

# Resource Groups
New-AzResourceGroup -Name "RG" -Location "eastus"
Get-AzResourceGroup
Remove-AzResourceGroup -Name "RG" -Force

# Virtual Machines
New-AzVM -ResourceGroupName "RG" -Name "VM" -Image "Win2022Datacenter"
Get-AzVM -ResourceGroupName "RG" -Name "VM"
Start-AzVM -ResourceGroupName "RG" -Name "VM"
Stop-AzVM -ResourceGroupName "RG" -Name "VM" -Force
Remove-AzVM -ResourceGroupName "RG" -Name "VM" -Force

# Storage
New-AzStorageAccount -ResourceGroupName "RG" -Name "storage" -Location "eastus" -SkuName "Standard_LRS"
Get-AzStorageAccount -ResourceGroupName "RG"
New-AzStorageContainer -Name "container" -Context $ctx

# Networking
New-AzVirtualNetwork -ResourceGroupName "RG" -Name "VNet" -AddressPrefix "10.0.0.0/16"
New-AzNetworkSecurityGroup -ResourceGroupName "RG" -Name "NSG"
Add-AzVirtualNetworkPeering -Name "Peering" -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.Id

# RBAC
New-AzRoleAssignment -SignInName user@contoso.com -RoleDefinitionName "Contributor" -ResourceGroupName "RG"
Get-AzRoleAssignment -ResourceGroupName "RG"

# Monitoring
Set-AzDiagnosticSetting -ResourceId $id -WorkspaceId $workspace -Enabled $true
New-AzMetricAlertRuleV2 -Name "Alert" -ResourceGroupName "RG"
```

### Command Patterns

**Get information:**
```powershell
Get-Az[Resource]             # List all
Get-Az[Resource] -Name "x"   # Get specific
Get-Az[Resource] | Where-Object {condition}  # Filter
```

**Create resources:**
```powershell
New-Az[Resource] -ResourceGroupName "RG" -Name "name" -Location "region"
```

**Update resources:**
```powershell
$resource = Get-Az[Resource]
$resource.Property = "value"
Set-Az[Resource] -Resource $resource
# Or: Update-Az[Resource]
```

**Delete resources:**
```powershell
Remove-Az[Resource] -ResourceGroupName "RG" -Name "name" -Force
```

---

## 2. Azure CLI Commands Quick Reference

### Essential Commands (Top 20)

```bash
# Authentication
az login
az account set --subscription "Production"
az account list --output table

# Resource Groups
az group create --name "RG" --location "eastus"
az group list --output table
az group delete --name "RG" --yes

# Virtual Machines
az vm create --resource-group "RG" --name "VM" --image "Ubuntu2204"
az vm show --resource-group "RG" --name "VM"
az vm start --resource-group "RG" --name "VM"
az vm deallocate --resource-group "RG" --name "VM"
az vm delete --resource-group "RG" --name "VM" --yes

# Storage
az storage account create --name "storage" --resource-group "RG" --sku "Standard_LRS"
az storage account list --output table
az storage container create --name "container" --account-name "storage"

# Networking
az network vnet create --resource-group "RG" --name "VNet" --address-prefix "10.0.0.0/16"
az network nsg create --resource-group "RG" --name "NSG"
az network vnet peering create --resource-group "RG" --name "Peering" --vnet-name "VNet1" --remote-vnet $vnet2Id

# RBAC
az role assignment create --assignee user@contoso.com --role "Contributor" --resource-group "RG"
az role assignment list --resource-group "RG" --output table

# Monitoring
az monitor diagnostic-settings create --name "Diag" --resource $id --workspace $workspace
az monitor metrics alert create --name "Alert" --resource-group "RG"
```

### Command Patterns

**Get information:**
```bash
az [service] [resource] list               # List all
az [service] [resource] show --name "x"    # Get specific
az [service] [resource] list --query "[?condition]"  # Filter
```

**Create resources:**
```bash
az [service] [resource] create --resource-group "RG" --name "name" --location "region"
```

**Update resources:**
```bash
az [service] [resource] update --resource-group "RG" --name "name" --set property=value
```

**Delete resources:**
```bash
az [service] [resource] delete --resource-group "RG" --name "name" --yes
```

---

## 3. Decision Trees

### Which Compute Service?

```
Need OS control?
├─ YES → Virtual Machine
└─ NO → Continue

Web application?
├─ YES → App Service
└─ NO → Continue

Simple container?
├─ YES → Container Instances
└─ NO → Continue

Multiple containers with orchestration?
├─ YES → Azure Kubernetes Service
└─ NO → Continue

Event-driven code?
└─ YES → Azure Functions
```

### Which Load Balancer?

```
Layer 4 (TCP/UDP) or Layer 7 (HTTP)?
├─ Layer 4 → Continue
│   └─ Regional or Global?
│       ├─ Regional → Azure Load Balancer
│       └─ Global → Traffic Manager (DNS-based)
└─ Layer 7 → Continue
    └─ Regional or Global?
        ├─ Regional → Application Gateway
        └─ Global → Azure Front Door
```

### Which Storage Type?

```
What type of data?
├─ Files (SMB/NFS) → Azure Files
├─ Unstructured (objects) → Blob Storage
├─ Tables (NoSQL) → Azure Table Storage
├─ Messages → Azure Queue Storage
└─ Managed disks → Azure Managed Disks
```

### Which Backup Solution?

```
What to protect?
├─ Azure VMs → Azure Backup (VM backup)
├─ Azure Files → Azure Backup (Files backup)
├─ SQL in Azure VM → Azure Backup (SQL backup)
├─ Entire site DR → Azure Site Recovery
└─ On-premises → Azure Backup (with agent)
```

### Which Monitoring Tool?

```
What to monitor?
├─ Infrastructure metrics → Azure Monitor
├─ Application performance → Application Insights
├─ Logs and queries → Log Analytics
├─ Costs → Cost Management
└─ Network → Network Watcher
```

### Which Identity Solution?

```
Need cloud identity?
├─ YES → Azure AD
└─ NO → Continue

Need hybrid (cloud + on-prem)?
├─ YES → Azure AD + AD Connect
└─ NO → Continue

Need authentication for apps?
├─ YES → Azure AD B2C (customers)
│        Azure AD B2B (partners)
└─ NO → Azure AD (employees)
```

---

## 4. CIDR Calculator and Subnetting

### CIDR Notation Reference

| CIDR | Subnet Mask | Total IPs | Usable IPs (Azure) | Use Case |
|------|-------------|-----------|-------------------|----------|
| /8 | 255.0.0.0 | 16,777,216 | 16,777,211 | Entire Class A |
| /16 | 255.255.0.0 | 65,536 | 65,531 | Large VNet |
| /20 | 255.255.240.0 | 4,096 | 4,091 | Medium VNet |
| /24 | 255.255.255.0 | 256 | 251 | Standard subnet |
| /25 | 255.255.255.128 | 128 | 123 | Small subnet |
| /26 | 255.255.255.192 | 64 | 59 | Azure Bastion min |
| /27 | 255.255.255.224 | 32 | 27 | VPN Gateway min |
| /28 | 255.255.255.240 | 16 | 11 | Very small |
| /29 | 255.255.255.248 | 8 | 3 | Tiny subnet |

### Azure Reserved IPs (Per Subnet)

Azure reserves 5 IP addresses in each subnet:
- `.0` - Network address
- `.1` - Default gateway (Azure router)
- `.2` - Azure DNS
- `.3` - Azure DNS (backup)
- `.255` (or last) - Broadcast address

**Example: 10.0.1.0/24**
- Total: 256 IPs
- Reserved: .0, .1, .2, .3, .255 (5 IPs)
- Usable: .4 through .254 (251 IPs)

### Subnetting Examples

**VNet: 10.0.0.0/16 (65,536 IPs)**

Split into /24 subnets:
```
10.0.0.0/24   → Subnet 1 (251 usable)
10.0.1.0/24   → Subnet 2 (251 usable)
10.0.2.0/24   → Subnet 3 (251 usable)
...
10.0.255.0/24 → Subnet 256 (251 usable)
```

**Smaller subnets:**
```
10.0.0.0/26   → 59 usable IPs (Azure Firewall)
10.0.0.64/26  → 59 usable IPs (App Gateway)
10.0.0.128/27 → 27 usable IPs (VPN Gateway)
10.0.0.160/27 → 27 usable IPs (Bastion)
```

### Quick Calculation

**Formula:** Usable IPs = 2^(32-CIDR) - 5

Examples:
- /24: 2^(32-24) - 5 = 2^8 - 5 = 256 - 5 = 251
- /27: 2^(32-27) - 5 = 2^5 - 5 = 32 - 5 = 27
- /29: 2^(32-29) - 5 = 2^3 - 5 = 8 - 5 = 3

---

## 5. SKU Comparison Tables

### VM Sizes Quick Reference

| Family | vCPU Range | RAM Range | Use Case |
|--------|------------|-----------|----------|
| **B** (Burstable) | 1-20 | 0.5-80 GB | Dev/test, low baseline |
| **D** (General) | 2-96 | 8-384 GB | Balanced workloads |
| **E** (Memory) | 2-96 | 16-672 GB | Databases, caching |
| **F** (Compute) | 2-72 | 4-144 GB | Batch, web servers |
| **L** (Storage) | 8-80 | 64-640 GB | Big data, NoSQL |
| **M** (Memory XL) | 8-416 | 220-12,000 GB | SAP HANA, huge DBs |
| **N** (GPU) | 6-24 | 56-672 GB | ML, rendering |

### Storage Account SKUs

| SKU | Performance | Replication | Cost | Use Case |
|-----|-------------|-------------|------|----------|
| **Standard_LRS** | Standard | Local | $ | Dev/test, non-critical |
| **Standard_ZRS** | Standard | Zone | $$ | Production, same region |
| **Standard_GRS** | Standard | Geo | $$$ | DR, cross-region |
| **Standard_GZRS** | Standard | Geo+Zone | $$$$ | Max availability |
| **Premium_LRS** | Premium | Local | $$$$ | Low latency, IOPS |

### Managed Disk Types

| Type | IOPS | Throughput | Cost | Use Case |
|------|------|------------|------|----------|
| **Standard HDD** | 500 | 60 MB/s | $ | Backups, infrequent |
| **Standard SSD** | 6,000 | 750 MB/s | $$ | Web servers, light apps |
| **Premium SSD** | 20,000 | 900 MB/s | $$$ | Production databases |
| **Ultra Disk** | 160,000 | 4,000 MB/s | $$$$ | Mission-critical |

### App Service Plans

| Tier | Features | Scaling | Cost | Use Case |
|------|----------|---------|------|----------|
| **Free** | Shared, 60 min CPU | None | Free | Quick tests |
| **Shared** | Shared, custom domain | None | $ | Simple sites |
| **Basic** | Dedicated, SSL | Manual (3) | $$ | Dev/test |
| **Standard** | Staging slots (5) | Auto (10) | $$$ | Production |
| **Premium** | Staging slots (20) | Auto (30) | $$$$ | High-traffic |
| **Isolated** | Private VNet | Auto (100) | $$$$$ | Mission-critical |

### VPN Gateway SKUs

| SKU | Bandwidth | Tunnels | P2S | Cost/mo | Use Case |
|-----|-----------|---------|-----|---------|----------|
| **Basic** | 100 Mbps | 10 | No | ~$30 | Dev/test |
| **VpnGw1** | 650 Mbps | 30 | Yes | ~$150 | Small prod |
| **VpnGw2** | 1 Gbps | 30 | Yes | ~$350 | Medium prod |
| **VpnGw3** | 1.25 Gbps | 30 | Yes | ~$450 | Large prod |
| **VpnGw4** | 5 Gbps | 100 | Yes | ~$700 | High bandwidth |
| **VpnGw5** | 10 Gbps | 100 | Yes | ~$850 | Maximum |

---

## 6. Common Ports Reference

### Azure Services

| Service | Port | Protocol | Notes |
|---------|------|----------|-------|
| **HTTP** | 80 | TCP | Web traffic |
| **HTTPS** | 443 | TCP | Secure web |
| **RDP** | 3389 | TCP | Windows remote |
| **SSH** | 22 | TCP | Linux remote |
| **DNS** | 53 | UDP/TCP | Name resolution |
| **SMTP** | 25, 587 | TCP | Email sending |
| **POP3** | 110 | TCP | Email retrieval |
| **IMAP** | 143 | TCP | Email retrieval |
| **SQL Server** | 1433 | TCP | Database |
| **MySQL** | 3306 | TCP | Database |
| **PostgreSQL** | 5432 | TCP | Database |
| **Redis** | 6379 | TCP | Cache |
| **MongoDB** | 27017 | TCP | NoSQL database |
| **FTP** | 21 | TCP | File transfer |
| **FTPS** | 990 | TCP | Secure FTP |
| **SFTP** | 22 | TCP | SSH file transfer |
| **SMB/CIFS** | 445 | TCP | File sharing |
| **NFS** | 2049 | TCP/UDP | File sharing |
| **LDAP** | 389 | TCP | Directory |
| **LDAPS** | 636 | TCP | Secure directory |
| **WinRM HTTP** | 5985 | TCP | PowerShell remoting |
| **WinRM HTTPS** | 5986 | TCP | Secure PS remoting |

### Azure-Specific Ports

| Service | Port | Purpose |
|---------|------|---------|
| **Azure Bastion** | 443 | HTTPS to portal |
| **App Gateway** | 65200-65535 | Backend communication |
| **Load Balancer Probes** | Custom | Health checks |
| **VPN (IKEv2)** | 500, 4500 | UDP |
| **SSTP VPN** | 443 | TCP |

---

## 7. SLA Reference

### Compute SLA

| Configuration | SLA | Downtime/Year | Use Case |
|---------------|-----|---------------|----------|
| **Single VM (Premium SSD)** | 99.9% | 8.76 hours | Non-critical |
| **Single VM (Standard SSD)** | 99.5% | 43.8 hours | Dev/test |
| **Availability Set** | 99.95% | 4.38 hours | Multi-tier apps |
| **Availability Zone** | 99.99% | 52.6 minutes | Production |
| **Multiple Regions** | 99.99%+ | <52.6 minutes | Mission-critical |

### Storage SLA

| Type | SLA | Notes |
|------|-----|-------|
| **LRS** | 99.9% | Local redundancy |
| **ZRS** | 99.9% | Zone redundancy |
| **GRS (read secondary)** | 99.99% | Geo redundancy + read |
| **GRS (write)** | 99.9% | Geo redundancy |

### Network SLA

| Service | SLA | Notes |
|---------|-----|-------|
| **VPN Gateway** | 99.95% | Single gateway |
| **ExpressRoute** | 99.95% | Dedicated connection |
| **Load Balancer (Standard)** | 99.99% | With 2+ VMs |
| **Application Gateway** | 99.95% | 2+ instances |

### Formula

**Composite SLA** = SLA₁ × SLA₂ × SLA₃ ...

Example:
```
Web Tier: 99.95% (Availability Set)
App Tier: 99.95% (Availability Set)
DB Tier: 99.95% (Availability Set)

Total: 0.9995 × 0.9995 × 0.9995 = 99.85%
```

---

## 8. Pricing Quick Guide

### Cost Factors

**Virtual Machines:**
- Compute: Per hour (even if stopped, not deallocated)
- Disks: Per GB/month
- Networking: Egress data transfer
- Public IPs: Static IPs charged

**Storage:**
- Capacity: Per GB/month
- Operations: Per 10,000 transactions
- Replication: GRS costs more than LRS

**Networking:**
- Ingress: Free
- Egress: First 5-100 GB free, then per GB
- VNet peering: Per GB transferred

**Cost-Saving Tips:**
1. Deallocate unused VMs (not just stop)
2. Use Reserved Instances (1-3 year commitment)
3. Use Azure Hybrid Benefit (if eligible)
4. Delete unattached disks
5. Use Spot VMs for fault-tolerant workloads
6. Right-size VMs (don't over-provision)
7. Use autoscaling (scale down when idle)
8. Set budget alerts
9. Use LRS instead of GRS (if acceptable)
10. Clean up old snapshots and backups

### Pricing Calculator

**Approximate monthly costs:**

**VMs (East US, Windows):**
- B2s (2 vCPU, 4 GB): ~$60
- D2s_v3 (2 vCPU, 8 GB): ~$100
- E4s_v3 (4 vCPU, 32 GB): ~$250

**Storage:**
- Standard HDD (1 TB): ~$40
- Standard SSD (1 TB): ~$75
- Premium SSD (1 TB): ~$135

**App Service:**
- Basic B1: ~$55
- Standard S1: ~$75
- Premium P1v2: ~$100

---

## 9. Keyboard Shortcuts (Portal)

### Navigation

| Shortcut | Action |
|----------|--------|
| **G + N** | Notifications |
| **G + /** | Search resources |
| **G + C** | Create resource |
| **G + D** | Dashboard |
| **G + A** | All resources |
| **G + R** | Resource groups |
| **G + S** | Subscriptions |
| **G + H** | Home |

### General

| Shortcut | Action |
|----------|--------|
| **?** | Show keyboard shortcuts |
| **Esc** | Close panel/dialog |
| **Ctrl + /** | Focus command bar |

---

## 10. Exam Day Checklist

### One Week Before

- [ ] Review all module Key Takeaways
- [ ] Review all Exam Tricks sections
- [ ] Complete practice exams (score 700+)
- [ ] Review weak areas
- [ ] Do hands-on labs for weak topics

### One Day Before

- [ ] Light review (don't cram)
- [ ] Review cheat sheets
- [ ] Review decision trees
- [ ] Get good sleep (8 hours)
- [ ] Prepare exam environment (if online)

### Exam Day Morning

- [ ] Eat good breakfast
- [ ] Test computer (if online)
- [ ] Test webcam/microphone
- [ ] Close all applications
- [ ] Have water and snacks ready
- [ ] Arrive early/login early (30 min)

### During Exam

- [ ] Read questions carefully
- [ ] Watch for keywords (NOT, EXCEPT, LEAST)
- [ ] Eliminate wrong answers first
- [ ] Flag difficult questions
- [ ] Manage time (1.5 min per question)
- [ ] Don't overthink
- [ ] Review flagged questions

### Question Keywords

**"What should you do FIRST?"**
→ Simplest, least invasive solution

**"Minimize cost"**
→ Smallest size, Basic tier, LRS

**"Maximize availability"**
→ Zones, redundancy, multiple instances

**"Least administrative effort"**
→ Built-in features, avoid custom scripts

**"NOT/EXCEPT"**
→ Read very carefully, looking for what DOESN'T fit

---

## Quick Reference Cards (Print & Cut)

### Card 1: CIDR Subnetting

```
┌─────────────────────────────────┐
│ CIDR Quick Reference            │
├─────────────────────────────────┤
│ /24 = 256 IPs (251 usable)     │
│ /27 = 32 IPs (27 usable)       │
│ /26 = 64 IPs (59 usable)       │
│                                 │
│ Azure reserves 5 IPs:           │
│ .0, .1, .2, .3, .255           │
└─────────────────────────────────┘
```

### Card 2: SLA

```
┌─────────────────────────────────┐
│ SLA Quick Reference             │
├─────────────────────────────────┤
│ Single VM: 99.9%                │
│ Availability Set: 99.95%        │
│ Availability Zone: 99.99%       │
│                                 │
│ Composite = SLA1 × SLA2 × SLA3  │
└─────────────────────────────────┘
```

### Card 3: Decision - Compute

```
┌─────────────────────────────────┐
│ Compute Service Selection       │
├─────────────────────────────────┤
│ Need OS control? → VM           │
│ Web app? → App Service          │
│ Simple container? → ACI          │
│ Orchestration? → AKS            │
│ Event-driven? → Functions       │
└─────────────────────────────────┘
```

### Card 4: Decision - Load Balancer

```
┌─────────────────────────────────┐
│ Load Balancer Selection         │
├─────────────────────────────────┤
│ L4 Regional → Load Balancer     │
│ L4 Global → Traffic Manager     │
│ L7 Regional → App Gateway       │
│ L7 Global → Front Door          │
└─────────────────────────────────┘
```

---

## Final Tips

### Study Strategy

**Last 24 hours:**
- Light review only (no cramming)
- Review these cheat sheets
- Get good sleep
- Stay calm and confident

**During exam:**
- Read questions twice
- Trust your preparation
- Don't overthink
- Manage time wisely

**Remember:**
- You've prepared thoroughly
- You know the material
- Stay calm and focused
- You've got this!

---

**[← Back to Practice Questions](../Practice-Questions/README.md) | [Back to Main README→](../README.md)**