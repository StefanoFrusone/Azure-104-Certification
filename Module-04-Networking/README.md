# Module 4: Networking

## Table of Contents

- [Introduction](#introduction)
- [Virtual Networks (VNets)](#virtual-networks-vnets)
- [Network Security](#network-security)
- [Load Balancing Solutions](#load-balancing-solutions)
- [VNet Connectivity](#vnet-connectivity)
- [DNS and Name Resolution](#dns-and-name-resolution)
- [Network Monitoring](#network-monitoring)
- [PowerShell Commands Reference](#powershell-commands-reference)
- [Azure CLI Commands Reference](#azure-cli-commands-reference)
- [Common Exam Scenarios](#common-exam-scenarios)
- [Practice Questions](#practice-questions)

---

## Introduction

**What is this module about?**

This module covers Azure Networking - how resources communicate with each other, the internet, and on-premises networks.

**Real-world analogy:**
Think of Azure networking like a city's infrastructure:

- **Virtual Networks** = Neighborhoods (isolated areas)
- **Subnets** = Streets within neighborhoods
- **NSGs** = Security gates at street entrances
- **Load Balancers** = Traffic lights distributing cars
- **VPN Gateway** = Highway connecting to another city
- **DNS** = Street signs helping you find addresses

**Key questions this module answers:**

1. **How** do I create isolated networks in Azure? (VNets)
2. **How** do I control traffic between resources? (NSGs, Firewalls)
3. **How** do I distribute traffic across multiple servers? (Load Balancers)
4. **How** do I connect Azure to on-premises? (VPN, ExpressRoute)
5. **How** do I resolve names to IP addresses? (Azure DNS)
6. **How** do I troubleshoot connectivity issues? (Network Watcher)

**Exam Weight:** 25-30% of the exam (largest section!)

---

## Virtual Networks (VNets)

### What is a Virtual Network?

**Simple explanation:**
A Virtual Network (VNet) is your private network in Azure. It's like having your own isolated network space where your resources can communicate securely.

**Technical definition:**
An Azure Virtual Network is a logical isolation of the Azure cloud dedicated to your subscription. It provides isolation, segmentation, and communication capabilities for Azure resources.

**Why do you need VNets?**

- Isolation from other customers
- Segmentation of your resources
- Communication between Azure resources
- Filtering network traffic
- Routing traffic
- Integration with on-premises networks

### VNet Components

**1. Address Space**

The range of IP addresses available in your VNet.

**Format:** CIDR notation (Classless Inter-Domain Routing)

**Example:** `10.0.0.0/16`

- `10.0.0.0` = Network address
- `/16` = Subnet mask (first 16 bits are network, last 16 are hosts)
- Available IPs: 65,536 addresses (2^16)

**Common private IP ranges (RFC 1918):**

- `10.0.0.0/8` (10.0.0.0 - 10.255.255.255) - 16 million IPs
- `172.16.0.0/12` (172.16.0.0 - 172.31.255.255) - 1 million IPs
- `192.168.0.0/16` (192.168.0.0 - 192.168.255.255) - 65,536 IPs

**CIDR notation quick reference:**

| CIDR | Subnet Mask     | Available IPs |
| ---- | --------------- | ------------- |
| /8   | 255.0.0.0       | 16,777,216    |
| /16  | 255.255.0.0     | 65,536        |
| /24  | 255.255.255.0   | 256           |
| /27  | 255.255.255.224 | 32            |
| /28  | 255.255.255.240 | 16            |
| /29  | 255.255.255.248 | 8             |

**Important for exam:**
Azure reserves 5 IP addresses in each subnet:

- First address: Network address
- Second address: Default gateway
- Third and fourth: DNS servers (Azure DNS)
- Last address: Broadcast address

Example in `10.0.1.0/24`:

- `10.0.1.0` - Network address (reserved)
- `10.0.1.1` - Default gateway (reserved)
- `10.0.1.2` - DNS mapping (reserved)
- `10.0.1.3` - DNS mapping (reserved)
- `10.0.1.255` - Broadcast (reserved)
- **Available for use:** `10.0.1.4` to `10.0.1.254` (251 addresses)

**2. Subnets**

Subdivisions of your VNet address space.

**Why use subnets?**

- Organize resources by function (web tier, app tier, database tier)
- Apply different security rules per subnet
- Better resource management
- Required for some Azure services (VPN Gateway, Azure Firewall, etc.)

**Example VNet design:**

```
VNet: 10.0.0.0/16 (65,536 IPs)
├─ Subnet 1: 10.0.1.0/24 (256 IPs) - Web tier
├─ Subnet 2: 10.0.2.0/24 (256 IPs) - App tier
├─ Subnet 3: 10.0.3.0/24 (256 IPs) - Database tier
└─ Subnet 4: 10.0.4.0/24 (256 IPs) - Management
```

**Best practices for subnets:**

- Plan for growth (don't make subnets too small)
- Use /24 or larger for production workloads
- Reserve address space for future use
- Use descriptive names (WebSubnet, DatabaseSubnet)

**3. Network Interface Card (NIC)**

Connects VMs and other resources to the VNet.

**Properties:**

- Private IP address (static or dynamic)
- Public IP address (optional)
- NSG association (optional)
- One or more per VM

**4. Public IP Addresses**

Allow resources to communicate with the internet.

**SKUs:**

**Basic:**

- Dynamic or static allocation
- Open by default (use NSG to secure)
- No availability zone support
- Free tier available
- Idle timeout: 4-30 minutes

**Standard:**

- Always static
- Secure by default (NSG required)
- Availability zone support
- No free tier
- Idle timeout: 4-30 minutes

**Comparison:**

| Feature            | Basic             | Standard          |
| ------------------ | ----------------- | ----------------- |
| Allocation         | Dynamic or Static | Static only       |
| Security           | Open by default   | Closed by default |
| Availability Zones | No                | Yes               |
| Cost               | Free option       | Always charged    |
| Load Balancer      | Basic LB          | Standard LB       |
| Use case           | Dev/test          | Production        |

### VNet Communication Patterns

**1. Resource to Resource (within VNet)**

Resources in the same VNet can communicate by default.

```
VM1 (10.0.1.4) ←→ VM2 (10.0.1.5)
Same subnet: Direct communication
Different subnets: Routed through Azure infrastructure
```

**2. Resource to Internet**

Outbound internet access is allowed by default.
Inbound requires public IP or load balancer.

**3. Resource to On-Premises**

Requires VPN Gateway or ExpressRoute.

**4. Resource to Azure Services**

Two options:

- **Service Endpoints:** Keep traffic on Azure backbone
- **Private Endpoints:** Assign private IP to Azure service

### System Routes vs Custom Routes

**System Routes (Default):**

Azure automatically creates routes for:

- Traffic within VNet
- Traffic to internet
- Traffic between VNets (if peered)
- Traffic to on-premises (if VPN configured)

**You cannot:**

- Delete system routes
- Modify system routes

**Custom Routes (User-Defined Routes - UDR):**

Override system routes for:

- Force traffic through firewall/NVA
- Block internet access
- Custom routing scenarios

**Example use case:**

```
Default: VM → Internet (direct)

With UDR: VM → Azure Firewall → Internet
(All outbound traffic inspected)
```

**Route table components:**

- **Address prefix:** Destination (e.g., 0.0.0.0/0 for internet)
- **Next hop type:** Where to send traffic
  - Virtual appliance
  - VNet gateway
  - Internet
  - VNet
  - None (drop traffic)
- **Next hop IP:** IP of next hop device

### Service Endpoints

**Simple:** Extend your VNet to Azure services over Azure backbone network.

**How it works:**

```
Without Service Endpoint:
VM → Public Internet → Azure Storage

With Service Endpoint:
VM → Azure Backbone → Azure Storage
(Faster, more secure, no public internet)
```

**Benefits:**

- Improved security (traffic never leaves Azure network)
- Better performance (optimized routing)
- Simple configuration (just enable on subnet)

**Supported services:**

- Azure Storage
- Azure SQL Database
- Azure Cosmos DB
- Azure Key Vault
- Azure Service Bus
- And more...

**Important:**

- Service still has public IP
- Access control via service firewall
- No additional cost

### Private Endpoints

**Simple:** Assign a private IP address from your VNet to an Azure service.

**How it works:**

```
Azure Storage gets private IP: 10.0.3.50
VM connects to 10.0.3.50 (private IP)
No public IP needed
```

**vs Service Endpoints:**

| Feature        | Service Endpoint        | Private Endpoint          |
| -------------- | ----------------------- | ------------------------- |
| Traffic path   | Azure backbone          | Private IP in VNet        |
| Service IP     | Public IP               | Private IP                |
| Access control | Service firewall        | NSG                       |
| DNS            | Public DNS              | Private DNS               |
| Cost           | Free                    | ~$7.30/month per endpoint |
| Use case       | Cost-effective security | Maximum security          |

**When to use Private Endpoints:**

- Compliance requirements (no public IP)
- Need NSG control
- On-premises access to Azure services
- Multi-VNet access to same service

### VNet Design Considerations

**1. Address space planning:**

```
Small organization:
VNet: 10.0.0.0/16 (65,536 IPs)

Large organization:
VNet 1: 10.1.0.0/16
VNet 2: 10.2.0.0/16
VNet 3: 10.3.0.0/16
...
```

**2. Subnet sizing:**

```
Gateway subnet: /27 or larger (required for VPN Gateway)
Azure Firewall subnet: /26 or larger
Application Gateway subnet: /24 or larger
Regular subnets: /24 (recommended)
```

**3. Network topology patterns:**

**Single VNet (Simple):**

```
All resources in one VNet
Multiple subnets for segmentation
Good for: Small deployments, single application
```

**Hub-Spoke (Recommended for Enterprise):**

```
Hub VNet:
  ├─ Shared services (AD, DNS, firewall)
  └─ Gateway to on-premises

Spoke VNet 1: Production workloads
Spoke VNet 2: Development workloads
Spoke VNet 3: Test workloads

Spokes peer to Hub (not to each other)
All traffic routes through Hub
```

**Multi-Region:**

```
East US VNet ←→ (Peering) ←→ West US VNet
For: High availability, disaster recovery
```

---

## Network Security

### Network Security Groups (NSGs)

**Simple:** Virtual firewall rules that control inbound and outbound traffic.

**Real-world analogy:**
NSG = Security checkpoint with a list of who's allowed in/out

**How NSGs work:**

```
Request arrives → NSG evaluates rules by priority → Allow or Deny

Example:
Priority 100: Allow HTTP from Internet → MATCH → Allow
Priority 200: Deny all from Internet → NOT EVALUATED
```

### NSG Components

**1. Security Rules**

Each rule has:

- **Name:** Descriptive identifier
- **Priority:** 100-4096 (lower = higher priority)
- **Source:** Where traffic originates
- **Source port:** Usually \* (any)
- **Destination:** Where traffic is going
- **Destination port:** Service port (80, 443, 3389, etc.)
- **Protocol:** TCP, UDP, ICMP, Any
- **Action:** Allow or Deny
- **Direction:** Inbound or Outbound

**2. Source/Destination Options**

**IP Address:**

```
Single: 203.0.113.5
Range: 203.0.113.0/24
Multiple: 203.0.113.5, 203.0.113.10
```

**Service Tags:**
Pre-defined groups of IP addresses for Azure services.

Common service tags:

- **Internet:** All public internet addresses
- **VirtualNetwork:** All addresses in your VNet
- **AzureLoadBalancer:** Azure load balancer IPs
- **AzureCloud:** All Azure datacenter IPs
- **AzureCloud.Region:** Specific region (e.g., AzureCloud.EastUS)
- **Storage:** Azure Storage service IPs
- **Sql:** Azure SQL service IPs

**Benefits of service tags:**

- No need to track changing Azure IP ranges
- Microsoft manages the IP lists
- Simplified rule management

**Application Security Groups (ASG):**
Logical grouping of VMs for easier rule management.

### NSG Rule Priority

**How priority works:**

```
Priority 100: Allow RDP from 203.0.113.5
Priority 200: Allow RDP from 203.0.113.0/24
Priority 300: Deny all inbound

Traffic from 203.0.113.5 → Matches rule 100 → Allowed (stops evaluation)
Traffic from 203.0.113.10 → Skips 100, matches 200 → Allowed
Traffic from 192.0.2.5 → Skips 100, 200, matches 300 → Denied
```

**Important rules:**

- Lower priority number = evaluated first
- Once match found, no further evaluation
- If no match, default rules apply

### Default Security Rules

Every NSG has default rules (cannot be deleted, lowest priority: 65000+).

**Inbound default rules:**

1. **AllowVNetInBound (65000):** Allow traffic from VNet
2. **AllowAzureLoadBalancerInBound (65001):** Allow Azure load balancer
3. **DenyAllInBound (65500):** Deny everything else

**Outbound default rules:**

1. **AllowVNetOutBound (65000):** Allow traffic to VNet
2. **AllowInternetOutBound (65001):** Allow traffic to internet
3. **DenyAllOutBound (65500):** Deny everything else

**Key exam point:**
By default:

- VNet-to-VNet: Allowed
- Outbound to Internet: Allowed
- Inbound from Internet: Denied (except load balancer)

### NSG Association

NSGs can be associated with:

**1. Subnet (recommended):**

```
All resources in subnet protected by NSG
Easier management
Fewer NSGs to maintain
```

**2. Network Interface:**

```
Per-VM security rules
More granular control
More complex to manage
```

**3. Both (combined):**

```
Subnet NSG: Common rules for all VMs
NIC NSG: VM-specific rules

Evaluation:
Inbound: Subnet NSG first → then NIC NSG
Outbound: NIC NSG first → then Subnet NSG
```

### NSG Example Scenarios

**Scenario 1: Web Server (Allow HTTP/HTTPS only)**

```
Priority 100: Allow HTTP (80) from Internet → Allow
Priority 110: Allow HTTPS (443) from Internet → Allow
Priority 120: Allow RDP (3389) from 203.0.113.5 (admin IP) → Allow
Priority 65500: Deny all (default rule)
```

**Scenario 2: Database Server (No Internet Access)**

```
Priority 100: Allow SQL (1433) from 10.0.1.0/24 (app subnet) → Allow
Priority 110: Allow RDP (3389) from 10.0.4.0/24 (management subnet) → Allow
Priority 120: Deny Internet (outbound) → Deny
```

**Scenario 3: Multi-Tier Application**

```
Web Tier (Public Subnet):
- Allow 80/443 from Internet
- Allow outbound to App Tier (10.0.2.0/24)

App Tier (Private Subnet):
- Allow inbound from Web Tier only
- Allow outbound to Database Tier (10.0.3.0/24)

Database Tier (Private Subnet):
- Allow inbound from App Tier only
- Deny all outbound internet
```

### Application Security Groups (ASG)

**Simple:** Logical grouping of VMs for simpler NSG rules.

**Problem without ASG:**

```
NSG Rule: Allow 443 from 10.0.1.4, 10.0.1.5, 10.0.1.6, 10.0.1.7...
(Must update rule when adding new web servers)
```

**Solution with ASG:**

```
1. Create ASG named "WebServers"
2. Add VMs to "WebServers" ASG
3. NSG Rule: Allow 443 from ASG "WebServers"
(Adding new VM to ASG automatically applies rule)
```

**Benefits:**

- Easier management
- No IP address tracking
- Logical grouping by application role
- Reduces number of rules

**Example:**

```
ASGs:
- WebServers
- AppServers
- DatabaseServers

NSG Rules:
Priority 100: Allow 443 from Internet to WebServers
Priority 200: Allow 8080 from WebServers to AppServers
Priority 300: Allow 1433 from AppServers to DatabaseServers
```

### Azure Firewall

**Simple:** Centralized, cloud-based firewall service with threat intelligence.

**NSG vs Azure Firewall:**

| Feature          | NSG                 | Azure Firewall           |
| ---------------- | ------------------- | ------------------------ |
| **Type**         | Stateful firewall   | Full firewall service    |
| **Layer**        | Layer 3-4 (network) | Layer 3-7 (application)  |
| **Rules**        | IP-based            | IP, FQDN, tags           |
| **Threat Intel** | No                  | Yes (Microsoft)          |
| **Logging**      | Basic               | Advanced (Log Analytics) |
| **HA**           | Built-in            | Built-in                 |
| **Cost**         | Free                | ~$1.25/hour + data       |
| **Use case**     | Basic filtering     | Advanced security        |

**Azure Firewall features:**

**1. Application rules:**

- Filter based on FQDN (Fully Qualified Domain Name)
- Example: Allow \*.microsoft.com
- HTTP/HTTPS inspection

**2. Network rules:**

- IP-based filtering
- Protocols: TCP, UDP, ICMP
- Layer 3-4 protection

**3. NAT rules:**

- Translate inbound traffic
- Port forwarding
- Publish internal services

**4. Threat intelligence:**

- Microsoft security intelligence
- Blocks known malicious IPs and domains
- Auto-updated

**5. High availability:**

- Built-in redundancy
- No single point of failure
- SLA: 99.95%

**Architecture:**

```
Internet
   ↓
Azure Firewall (10.0.0.4/26 subnet)
   ↓
Route Table (UDR)
   ↓
VNet Subnets (Web, App, Database)
```

**Typical use cases:**

- Hub-spoke topology (centralized firewall in hub)
- Outbound filtering (block malicious sites)
- Inbound protection (publish web apps securely)
- Threat prevention
- Compliance requirements

**Azure Firewall SKUs:**

**Standard:**

- All base features
- Threat intelligence
- L3-L7 filtering
- Good for most scenarios

**Premium:**

- TLS inspection
- IDPS (Intrusion Detection and Prevention)
- URL filtering
- Web categories
- For high-security requirements

**Basic:**

- Small deployments (< 250 Mbps)
- Limited features
- Lower cost
- For SMB environments

### Azure Bastion

**Simple:** Secure RDP/SSH access without public IPs or VPN.

**Problem:**

```
Traditional: VM with public IP → Exposed to internet → Security risk
```

**Solution with Bastion:**

```
VM has no public IP → Azure Bastion → Browser-based RDP/SSH → Secure
```

**How it works:**

1. Deploy Azure Bastion in VNet
2. Connect via Azure Portal
3. RDP/SSH session in browser
4. No public IP on VMs needed

**Benefits:**

- No public IPs on VMs (reduced attack surface)
- No need to manage jump boxes
- Integrated with Azure RBAC
- Protected from port scanning
- Protection against zero-day exploits
- NSG still applies

**Cost:**

- ~$140/month for Basic SKU
- ~$280/month for Standard SKU
- Worth it for security compliance

---

## Load Balancing Solutions

### Overview

Azure offers 4 load balancing services. Each serves different purposes.

**Quick decision tree:**

```
HTTP/HTTPS traffic?
  YES → Layer 7
    Global? → Azure Front Door or Traffic Manager
    Regional? → Application Gateway
  NO → Layer 4
    Global? → Traffic Manager
    Regional? → Azure Load Balancer
```

### Azure Load Balancer

**Simple:** Distributes TCP/UDP traffic across VMs (Layer 4).

**Real-world analogy:**
Load Balancer = Traffic light distributing cars across lanes

**How it works:**

```
Internet
   ↓
Load Balancer (Public IP: 20.50.100.5)
   ↓ (Distributes traffic)
VM1 (10.0.1.4) | VM2 (10.0.1.5) | VM3 (10.0.1.6)
```

**Types:**

**Public Load Balancer:**

- Has public IP
- Receives traffic from internet
- Distributes to VMs with private IPs
- Use for: Web applications, APIs

**Internal Load Balancer:**

- Has private IP only
- Receives traffic from within VNet
- Distributes to backend VMs
- Use for: Database tier, internal APIs

### Load Balancer Components

**1. Frontend IP Configuration**

The IP address that receives traffic.

- Public Load Balancer: Public IP
- Internal Load Balancer: Private IP from VNet

**2. Backend Pool**

Group of VMs that receive traffic.

Can include:

- VMs in availability set
- VMs in VM scale set
- Individual VMs
- Virtual machine scale sets

**3. Health Probes**

Check if backend VMs are healthy.

**Types:**

- **HTTP/HTTPS:** Check specific URL (e.g., /health)
- **TCP:** Check if port is open

**Settings:**

- **Interval:** How often to check (default: 5 seconds)
- **Unhealthy threshold:** Failures before marking unhealthy (default: 2)

**Example:**

```
Probe: HTTP /health every 5 seconds
VM responds 200 OK → Healthy → Receives traffic
VM responds 500 or times out → After 2 failures → Unhealthy → No traffic
```

**4. Load Balancing Rules**

How to distribute traffic.

**Rule components:**

- Frontend IP and port (e.g., 20.50.100.5:80)
- Backend pool and port (e.g., 10.0.1.0/24:80)
- Protocol (TCP or UDP)
- Session persistence (None, Client IP, Client IP and Protocol)
- Idle timeout (4-30 minutes)
- Health probe

**5. Inbound NAT Rules**

Direct traffic to specific VM (not load balanced).

**Use case:** RDP/SSH to specific VM

**Example:**

```
Load Balancer IP:3389 → VM1:3389
Load Balancer IP:3390 → VM2:3389
Load Balancer IP:3391 → VM3:3389
```

### Load Balancer SKUs

**Basic (deprecated, being retired):**

- Free
- Up to 300 instances
- No SLA
- No availability zone support
- Don't use for new deployments

**Standard (use this):**

- ~$18/month
- Up to 1000 instances
- 99.99% SLA
- Availability zone support
- Security by default (NSG required)
- Production recommended

**Comparison:**

| Feature            | Basic     | Standard          |
| ------------------ | --------- | ----------------- |
| Backend pool size  | 300       | 1000              |
| Health probes      | HTTP, TCP | HTTP, HTTPS, TCP  |
| Availability zones | No        | Yes               |
| SLA                | None      | 99.99%            |
| Security           | Open      | Closed by default |
| Cost               | Free      | ~$18/month        |

### Load Balancer Distribution Modes

**1. 5-tuple hash (default):**

Based on:

- Source IP
- Source port
- Destination IP
- Destination port
- Protocol

**Result:** Each request may go to different VM

**2. Source IP affinity (Client IP):**

Based on:

- Source IP
- Destination IP

**Result:** Same client always goes to same VM

**Use case:** Shopping carts, session state

**3. Source IP and Protocol:**

Based on:

- Source IP
- Destination IP
- Protocol

**Result:** Similar to above but protocol-specific

### Application Gateway

**Simple:** Layer 7 (HTTP/HTTPS) load balancer with advanced features.

**Layer 4 vs Layer 7:**

```
Layer 4 (Load Balancer):
Looks at: IP addresses and ports
Decision: Distribute TCP traffic

Layer 7 (Application Gateway):
Looks at: HTTP headers, URLs, cookies
Decision: Route based on URL path, host header, etc.
```

**Application Gateway features:**

**1. URL-based routing:**

```
contoso.com/images/* → Image server pool
contoso.com/videos/* → Video server pool
contoso.com/* → General web server pool
```

**2. Multi-site hosting:**

```
contoso.com → Server pool 1
fabrikam.com → Server pool 2
(Single Application Gateway, multiple websites)
```

**3. Redirection:**

```
HTTP → HTTPS (automatic redirect)
/old-page → /new-page
```

**4. Session affinity (cookie-based):**

```
First request → Server 1 → Cookie set
Subsequent requests with cookie → Always Server 1
```

**5. SSL termination:**

```
Client → HTTPS → Application Gateway → HTTP → Backend servers
(Application Gateway handles SSL, reduces backend load)
```

**6. End-to-end SSL:**

```
Client → HTTPS → Application Gateway → HTTPS → Backend servers
(Encrypted all the way)
```

**7. Web Application Firewall (WAF):**

Protects against:

- SQL injection
- Cross-site scripting (XSS)
- Command injection
- HTTP request smuggling
- And more (OWASP Top 10)

**Modes:**

- **Detection:** Logs threats, doesn't block
- **Prevention:** Blocks malicious requests

**8. Autoscaling:**

Automatically scale based on traffic.

**9. Health probes:**

Custom health checks for backend servers.

### Application Gateway Components

**1. Frontend IP:**

- Public IP (internet-facing)
- Private IP (internal)
- Or both

**2. Listeners:**

Listens for incoming requests.

**Types:**

- **Basic:** Single site
- **Multi-site:** Multiple sites on one gateway

**Settings:**

- Protocol (HTTP/HTTPS)
- Port (80, 443, custom)
- Hostname (optional for multi-site)
- SSL certificate (for HTTPS)

**3. Backend Pools:**

Destinations for traffic.

Can include:

- VMs
- VM scale sets
- IP addresses
- FQDNs (fully qualified domain names)
- App Services

**4. HTTP Settings:**

How to communicate with backend.

**Settings:**

- Protocol (HTTP/HTTPS)
- Port
- Cookie-based affinity (enable/disable)
- Connection draining (graceful removal from pool)
- Request timeout
- Custom probes

**5. Routing Rules:**

Connect listener to backend pool via HTTP settings.

**Types:**

- **Basic:** One listener → one backend pool
- **Path-based:** Different URL paths → different backend pools

**Example path-based rule:**

```
Rule: contoso.com
Path: /images/* → Image servers
Path: /videos/* → Video servers
Default: /* → Web servers
```

### Application Gateway SKUs

**Standard_v2 (without WAF):**

- Layer 7 load balancing
- URL routing
- Multi-site hosting
- Autoscaling
- Zone redundancy

**WAF_v2 (with WAF):**

- All Standard_v2 features
- Plus: Web Application Firewall
- OWASP protection
- Custom rules

**Pricing:**

- Fixed cost: ~$250/month
- Capacity units: Based on compute, connections, throughput
- Typically: $300-500/month for production

### Azure Traffic Manager

**Simple:** DNS-based global load balancer.

**How it works:**

```
User queries: www.contoso.com
   ↓
DNS query goes to Traffic Manager
   ↓
Traffic Manager returns: IP of best endpoint
   ↓
User connects directly to that endpoint
```

**Important:** Traffic Manager doesn't handle traffic itself, only DNS resolution.

**Routing methods:**

**1. Priority (Failover):**

```
Primary: East US (priority 1)
Secondary: West US (priority 2)

All traffic to East US → If down → Route to West US
```

**Use case:** Active-passive failover

**2. Weighted:**

```
East US: 70%
West US: 30%

70% of users go to East US, 30% to West US
```

**Use case:** Gradual deployment, load distribution

**3. Performance:**

```
User in Europe → Routes to Europe endpoint (lowest latency)
User in Asia → Routes to Asia endpoint
```

**Use case:** Global applications, best user experience

**4. Geographic:**

```
Users from EU → Europe endpoint (GDPR compliance)
Users from US → US endpoint
Users from Asia → Asia endpoint
```

**Use case:** Compliance, data sovereignty

**5. Multi-value:**

Returns multiple healthy endpoints (up to 8).
Client chooses which to use.

**6. Subnet:**

Route based on source IP subnet.

**Use case:** Internal users vs external users

### Azure Front Door

**Simple:** Global, scalable entry point with CDN and WAF.

**Application Gateway vs Front Door:**

| Feature      | Application Gateway   | Front Door                   |
| ------------ | --------------------- | ---------------------------- |
| **Scope**    | Regional              | Global                       |
| **Layer**    | 7 (HTTP/HTTPS)        | 7 (HTTP/HTTPS)               |
| **Routing**  | URL-based, multi-site | Global routing, caching      |
| **CDN**      | No                    | Yes (built-in)               |
| **WAF**      | Yes                   | Yes                          |
| **Backend**  | Within region         | Global (any region, on-prem) |
| **Cost**     | ~$300/month           | ~$300-500/month              |
| **Use case** | Regional apps         | Global apps                  |

**Front Door features:**

**1. Global load balancing:**

```
Users worldwide
   ↓
Front Door (nearest POP)
   ↓
Routes to best backend (lowest latency)
   ↓
East US, West US, Europe, Asia backends
```

**2. CDN (Content Delivery Network):**

Caches static content at edge locations.

- Images, CSS, JavaScript cached near users
- Faster load times
- Reduced backend load

**3. URL-based routing:**

Same as Application Gateway, but global.

**4. Session affinity:**

Sticky sessions across global backends.

**5. SSL offloading:**

Terminates SSL at edge, reduces backend load.

**6. Web Application Firewall:**

Global protection against threats.

**7. Health probes:**

Monitors backend health globally.

**8. Automatic failover:**

If backend unhealthy, routes to alternate.

**Front Door routing:**

```
front-door-name.azurefd.net
   ↓
Routes to backend based on:
- Path pattern
- Health status
- Latency
- Priority/weight
```

**Use cases for Front Door:**

- Global web applications
- APIs with worldwide users
- Media streaming
- E-commerce platforms
- Content-heavy websites

### Load Balancing Decision Matrix

| Requirement                 | Solution                          |
| --------------------------- | --------------------------------- |
| Layer 4 (TCP/UDP), regional | Azure Load Balancer               |
| Layer 7 (HTTP), regional    | Application Gateway               |
| Layer 7 (HTTP), global      | Azure Front Door                  |
| DNS-based routing           | Traffic Manager                   |
| Need WAF, regional          | Application Gateway WAF           |
| Need WAF + CDN, global      | Front Door                        |
| Internal load balancing     | Internal Load Balancer            |
| Multi-site hosting          | Application Gateway or Front Door |

---

## VNet Connectivity

### VNet Peering

**Simple:** Connect two VNets so resources can communicate.

**Real-world analogy:**
VNet Peering = Building a bridge between two neighborhoods

**How it works:**

```
VNet A (10.0.0.0/16) ←→ Peering ←→ VNet B (10.1.0.0/16)
VM in VNet A can reach VM in VNet B directly
```

**Benefits:**

- Low latency (Azure backbone network)
- High bandwidth
- No downtime for existing resources
- Cross-region support
- Cross-subscription support

**Types:**

**1. Regional VNet Peering:**

- VNets in same region
- Example: East US ←→ East US

**2. Global VNet Peering:**

- VNets in different regions
- Example: East US ←→ West Europe
- Additional data transfer costs

**Important characteristics:**

**Non-transitive:**

```
VNet A ←→ VNet B ←→ VNet C
VNet A CANNOT reach VNet C
(Must peer A to C directly)
```

**Allow gateway transit:**

```
Hub VNet has VPN Gateway
Spoke VNets can use Hub's gateway
(Hub-spoke topology)
```

**Peering states:**

- **Initiated:** Peering request sent
- **Connected:** Peering active
- **Disconnected:** Peering removed

**Restrictions:**

- Address spaces cannot overlap
- Cannot add/remove address ranges after peering (must delete and recreate)
- Some Azure services have limitations

### Hub-Spoke Network Topology

**Simple:** Centralized network architecture with shared services in hub.

**Architecture:**

```
                    Hub VNet (10.0.0.0/16)
                    ├─ VPN Gateway
                    ├─ Azure Firewall
                    ├─ Azure Bastion
                    └─ Shared services
                         ↕ (Peering)
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
Spoke 1 (Prod)    Spoke 2 (Dev)    Spoke 3 (Test)
10.1.0.0/16       10.2.0.0/16      10.3.0.0/16
```

**Hub VNet contains:**

- VPN Gateway (connection to on-premises)
- Azure Firewall (centralized security)
- Azure Bastion (secure access)
- Domain controllers
- Monitoring tools

**Spoke VNets contain:**

- Application workloads
- Environment-specific resources
- Peered to Hub only

**Benefits:**

- Centralized management
- Cost savings (one VPN Gateway, one Firewall)
- Security isolation (spokes can't directly communicate)
- Easy to add new spokes

**Implementing spoke-to-spoke communication:**

**Option 1: User-Defined Routes (UDR)**

```
Spoke 1 → Route Table → Next hop: Azure Firewall → Spoke 2
All inter-spoke traffic goes through Firewall in Hub
```

**Option 2: Azure Virtual WAN**

```
Managed hub-spoke solution
Automatic routing
Simplified management
```

### VPN Gateway

**Simple:** Secure connection between Azure and on-premises or other networks over internet.

**Types of VPN connections:**

**1. Site-to-Site (S2S):**

```
On-premises network ←→ VPN Gateway ←→ Azure VNet

Use case: Connect entire office to Azure
```

**2. Point-to-Site (P2S):**

```
Individual computer ←→ VPN Gateway ←→ Azure VNet

Use case: Remote workers accessing Azure
```

**3. VNet-to-VNet:**

```
VNet A ←→ VPN Gateway ←→ VPN Gateway ←→ VNet B

Use case: Connect VNets across regions (alternative to peering)
```

### VPN Gateway Components

**1. Gateway Subnet**

Special subnet required for VPN Gateway.

**Requirements:**

- Must be named "GatewaySubnet"
- Minimum size: /29
- Recommended: /27 or /26
- Cannot have NSG attached
- Cannot contain other resources

**2. Public IP Address**

VPN Gateway needs public IP for external connectivity.

**3. Local Network Gateway**

Represents on-premises network.

**Contains:**

- On-premises VPN device public IP
- On-premises address spaces

**4. Connection**

Links VPN Gateway to Local Network Gateway.

**Settings:**

- Connection type (IPsec/IKE)
- Shared key (pre-shared key)
- Connection protocol

### VPN Gateway SKUs

| SKU        | Bandwidth | Tunnels | Use Case        |
| ---------- | --------- | ------- | --------------- |
| **Basic**  | 100 Mbps  | 10      | Dev/test        |
| **VpnGw1** | 650 Mbps  | 30      | Small prod      |
| **VpnGw2** | 1 Gbps    | 30      | Medium prod     |
| **VpnGw3** | 1.25 Gbps | 30      | Large prod      |
| **VpnGw4** | 5 Gbps    | 100     | High throughput |
| **VpnGw5** | 10 Gbps   | 100     | Maximum perf    |

**Generation 1 vs Generation 2:**

- Gen 2 has better performance
- Gen 2 supports more features
- Use Gen 2 for new deployments

**Important:**

- Cannot resize between Basic and VpnGw SKUs
- Can resize within VpnGw family (e.g., VpnGw1 → VpnGw2)

### VPN Gateway Routing

**Policy-Based VPN:**

- Static routing
- One tunnel
- Limited scenarios
- Legacy devices
- Basic SKU only

**Route-Based VPN (recommended):**

- Dynamic routing
- Multiple tunnels
- Point-to-Site support
- VNet-to-VNet support
- Coexistence with ExpressRoute
- All modern SKUs

### Point-to-Site VPN Protocols

**1. OpenVPN (recommended):**

- SSL/TLS based
- Works through firewalls
- Windows, Linux, macOS, iOS, Android
- Most flexible

**2. SSTP (Secure Socket Tunneling Protocol):**

- Microsoft proprietary
- SSL-based
- Windows only
- Works through firewalls

**3. IKEv2:**

- Standard protocol
- Good for mobile devices
- iOS and macOS
- May be blocked by firewalls

### VPN Gateway High Availability

**Active-Standby (default):**

```
Primary instance: Active
Secondary instance: Standby

Failover time: 10-15 seconds (planned)
               60-90 seconds (unplanned)
```

**Active-Active:**

```
Both instances active
Two public IPs
Requires two on-premises VPN devices
Better redundancy
```

**Zone-Redundant Gateway:**

```
Deployed across availability zones
Higher SLA
Automatic failover
Costs more
```

### ExpressRoute

**Simple:** Private, dedicated connection from on-premises to Azure (not over internet).

**VPN vs ExpressRoute:**

| Feature       | VPN Gateway      | ExpressRoute                |
| ------------- | ---------------- | --------------------------- |
| **Path**      | Over internet    | Private connection          |
| **Bandwidth** | Up to 10 Gbps    | Up to 100 Gbps              |
| **Latency**   | Higher           | Lower, predictable          |
| **SLA**       | 99.95%           | 99.95%                      |
| **Setup**     | Quick            | Requires ISP/carrier        |
| **Cost**      | Lower (~$150/mo) | Higher (~$500+/mo)          |
| **Use case**  | Small/medium     | Enterprise, high throughput |

**ExpressRoute connection models:**

**1. CloudExchange Co-location:**

```
Your datacenter in same facility as Azure
Direct cross-connect
Fastest setup
```

**2. Point-to-Point Ethernet:**

```
Dedicated link from your office to Azure
Through service provider
```

**3. Any-to-Any (IPVPN):**

```
Your existing MPLS network
Integrated with Azure
```

**ExpressRoute peering types:**

**1. Private Peering:**

- Connect to Azure VNets
- Access VMs, internal services
- Most common

**2. Microsoft Peering:**

- Connect to Azure PaaS services
- Office 365, Dynamics 365
- Public IPs required

**ExpressRoute benefits:**

- Predictable latency
- Higher bandwidth
- Better security (no internet exposure)
- SLA for connectivity
- Global connectivity (with Premium)

**ExpressRoute + VPN (coexistence):**

```
Primary: ExpressRoute
Backup: VPN Gateway

If ExpressRoute fails → Automatic failover to VPN
```

### Azure Virtual WAN

**Simple:** Managed networking service that brings together VPN, ExpressRoute, and routing.

**Traditional hub-spoke vs Virtual WAN:**

**Traditional (manual):**

```
You create: Hub VNet, VPN Gateway, Azure Firewall
You configure: Peering, UDRs, NSGs
You manage: Everything
```

**Virtual WAN (managed):**

```
Azure creates: Virtual Hub
Azure configures: Automatic routing
You manage: Just workloads
```

**Virtual WAN components:**

**1. Virtual Hub:**

- Managed hub VNet
- Automatic routing
- Built-in VPN/ExpressRoute gateways

**2. Virtual Hub Connections:**

- Spoke VNet connections
- Site-to-Site VPN
- Point-to-Site VPN
- ExpressRoute

**Benefits:**

- Simplified management
- Automatic routing between spokes
- Global transit network
- Integrated security (Azure Firewall)
- Built-in monitoring

**Virtual WAN SKUs:**

**Basic:**

- Site-to-Site VPN only
- Limited features

**Standard:**

- All connection types
- Spoke-to-spoke routing
- Azure Firewall integration
- Recommended for production

---

## DNS and Name Resolution

### Azure DNS

**Simple:** Host your DNS domains in Azure.

**What is DNS?**
DNS = Phone book of the internet
Translates names (www.contoso.com) to IP addresses (20.50.100.5)

**Azure DNS features:**

**1. Public DNS Zones:**

- Host public domains
- Internet-accessible
- Manage records in Azure

**2. Private DNS Zones:**

- Internal name resolution
- Only accessible from linked VNets
- No internet exposure

### DNS Record Types

**Common record types:**

**A Record:**

```
Name → IPv4 address
www.contoso.com → 20.50.100.5
```

**AAAA Record:**

```
Name → IPv6 address
www.contoso.com → 2001:db8::1
```

**CNAME Record:**

```
Alias → Another name
www.contoso.com → contoso.azurewebsites.net
```

**MX Record:**

```
Mail server priority
contoso.com → mail.contoso.com (priority 10)
```

**TXT Record:**

```
Text information
Used for: Domain verification, SPF, DKIM
```

**NS Record:**

```
Name server records
Delegates subdomain to other DNS servers
```

**SOA Record:**

```
Start of Authority
Contains zone information
Automatically created
```

### Azure DNS Zones

**Creating public DNS zone:**

1. Create DNS zone (contoso.com)
2. Azure provides name servers:
   ```
   ns1-01.azure-dns.com
   ns2-01.azure-dns.net
   ns3-01.azure-dns.org
   ns4-01.azure-dns.info
   ```
3. Update domain registrar with Azure name servers
4. Add DNS records in Azure

**Creating private DNS zone:**

1. Create private DNS zone (contoso.internal)
2. Link to VNets (virtual network links)
3. Enable auto-registration (optional)
4. VMs automatically get DNS records

**Auto-registration:**

```
VM created in VNet → Automatically added to private DNS zone
VM deleted → Automatically removed

Example:
VM name: web01
DNS record: web01.contoso.internal → 10.0.1.4
```

### Name Resolution in Azure

**Options for VM name resolution:**

**1. Azure-provided DNS (default):**

```
Automatic
No configuration needed
VMs can resolve each other by name (same VNet)
Cannot customize
```

**2. Custom DNS Server:**

```
Deploy your own DNS server (Windows/Linux)
Configure VNet to use custom DNS
Full control
More management
```

**3. Azure Private DNS:**

```
Managed service
No DNS server needed
Auto-registration
Cross-VNet resolution
Recommended
```

**Name resolution scenarios:**

**Within same VNet:**

```
Azure-provided DNS: ✅ Works
VM1 can ping VM2 by name
```

**Between peered VNets:**

```
Azure-provided DNS: ❌ Doesn't work
Solution: Use Azure Private DNS or custom DNS
```

**From on-premises:**

```
Requires: Custom DNS server or Azure Private DNS Resolver
Forward queries to Azure
```

### Azure Private DNS Resolver

**Simple:** Bridge between on-premises DNS and Azure Private DNS.

**Problem:**

```
On-premises users need to resolve Azure private DNS names
Azure VMs need to resolve on-premises DNS names
```

**Solution:**

```
On-premises ←→ Private DNS Resolver ←→ Azure Private DNS
Bidirectional resolution
```

**Components:**

**Inbound endpoint:**

- Receives queries from on-premises
- Forwards to Azure Private DNS

**Outbound endpoint:**

- Sends queries to on-premises DNS
- For Azure VMs resolving on-premises names

---

## Network Monitoring

### Azure Network Watcher

**Simple:** Network diagnostics and monitoring service.

**Key capabilities:**

**1. Topology:**

- Visual representation of VNet resources
- See all resources and connections
- Understand network layout

**2. IP Flow Verify:**

- Check if traffic is allowed or denied
- Tests NSG rules
- Troubleshoot connectivity issues

**Example:**

```
Question: Can VM1 reach VM2 on port 443?
IP Flow Verify: Yes, allowed by NSG rule "AllowHTTPS"
```

**3. Next Hop:**

- Shows where traffic goes
- Identifies routing issues
- Shows route table applied

**Example:**

```
From: 10.0.1.4
To: 10.0.2.5
Next Hop: VNet gateway (if going to on-premises)
```

**4. Connection Monitor:**

- Monitor end-to-end connectivity
- Check latency and packet loss
- Alert on connectivity issues

**Example:**

```
Monitor: VM1 → Azure SQL Database
Metrics: Latency, packet loss, success rate
Alert if: Packet loss > 5%
```

**5. Packet Capture:**

- Capture network traffic
- Like Wireshark for Azure
- Download and analyze with Wireshark

**Use case:**

```
Problem: Connection drops randomly
Solution: Capture packets, analyze to find cause
```

**6. NSG Flow Logs:**

- Log all traffic allowed/denied by NSG
- Sent to Storage Account
- Analyze with Log Analytics or Power BI

**Information logged:**

- Source/destination IP
- Source/destination port
- Protocol
- Action (allow/deny)
- Timestamp

**7. VPN Troubleshoot:**

- Diagnose VPN Gateway issues
- Check tunnel status
- Identify configuration problems

**8. Connection Troubleshoot:**

- Test connectivity between resources
- Identify where connection fails
- Provides hop-by-hop analysis

**Example:**

```
Test: VM → SQL Database
Result: Failed at NSG rule priority 200
Action: Update NSG rule
```

### Network Watcher Regions

**Important:**

- Network Watcher is region-specific
- Must enable in each region where you have resources
- Automatically enabled in new regions (by default)

### NSG Flow Logs

**Two versions:**

**Version 1:**

- Basic flow information
- Source/dest IP and port
- Protocol
- Action

**Version 2 (recommended):**

- All Version 1 info
- Plus: Flow state (begin, ongoing, end)
- Plus: Byte and packet counts
- Better analytics

**Storage and analysis:**

**Storage:**

```
Flow logs → Storage Account → JSON files
Retention: 0-365 days
```

**Analysis options:**

- Traffic Analytics (built-in)
- Log Analytics
- Power BI
- Third-party tools

### Traffic Analytics

**Simple:** Visualize network traffic patterns from NSG Flow Logs.

**Provides:**

- Top talkers (which VMs send most traffic)
- Traffic distribution by protocol
- Allowed vs denied traffic
- Geographic traffic flow
- Threat detection (malicious IPs)
- Bandwidth utilization

**Setup:**

```
1. Enable NSG Flow Logs
2. Send to Log Analytics workspace
3. Enable Traffic Analytics
4. View insights in Azure Portal
```

---

## PowerShell Commands Reference

### Virtual Networks

```powershell
# Connect to Azure
Connect-AzAccount
Set-AzContext -SubscriptionName "Production"

# Create resource group
New-AzResourceGroup -Name "RG-Network" -Location "eastus"

# Create VNet with subnet
$subnet = New-AzVirtualNetworkSubnetConfig -Name "Subnet1" `
    -AddressPrefix "10.0.1.0/24"

New-AzVirtualNetwork -ResourceGroupName "RG-Network" `
    -Name "VNet-Prod" `
    -Location "eastus" `
    -AddressPrefix "10.0.0.0/16" `
    -Subnet $subnet

# Get VNet and subnet for Application Gateway
$vnet = Get-AzVirtualNetwork -ResourceGroupName "RG-Network" -Name "VNet-Prod"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "AppGWSubnet"

# Create IP configuration
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name "AppGWIPConfig" `
    -Subnet $subnet

# Create frontend IP configuration
$fipconfig = New-AzApplicationGatewayFrontendIPConfig -Name "FrontendIP" `
    -PublicIPAddress $pip

# Create frontend port
$frontendport = New-AzApplicationGatewayFrontendPort -Name "FrontendPort" `
    -Port 80

# Create backend pool
$backendPool = New-AzApplicationGatewayBackendAddressPool -Name "BackendPool" `
    -BackendIPAddresses 10.0.1.4, 10.0.1.5

# Create backend HTTP settings
$poolSetting = New-AzApplicationGatewayBackendHttpSetting -Name "PoolSettings" `
    -Port 80 `
    -Protocol Http `
    -CookieBasedAffinity Disabled `
    -RequestTimeout 120

# Create listener
$listener = New-AzApplicationGatewayHttpListener -Name "Listener" `
    -Protocol Http `
    -FrontendIPConfiguration $fipconfig `
    -FrontendPort $frontendport

# Create routing rule
$rule = New-AzApplicationGatewayRequestRoutingRule -Name "Rule1" `
    -RuleType Basic `
    -HttpListener $listener `
    -BackendAddressPool $backendPool `
    -BackendHttpSettings $poolSetting `
    -Priority 100

# Create SKU
$sku = New-AzApplicationGatewaySku -Name "Standard_v2" `
    -Tier "Standard_v2" `
    -Capacity 2

# Create Application Gateway
New-AzApplicationGateway -ResourceGroupName "RG-Network" `
    -Name "AppGateway" `
    -Location "eastus" `
    -BackendAddressPools $backendPool `
    -BackendHttpSettingsCollection $poolSetting `
    -FrontendIpConfigurations $fipconfig `
    -GatewayIpConfigurations $gipconfig `
    -FrontendPorts $frontendport `
    -HttpListeners $listener `
    -RequestRoutingRules $rule `
    -Sku $sku

# Get Application Gateway
Get-AzApplicationGateway -ResourceGroupName "RG-Network" -Name "AppGateway"
```

### VPN Gateway

```powershell
# Create gateway subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "RG-Network" -Name "VNet-Prod"
Add-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" `
    -VirtualNetwork $vnet `
    -AddressPrefix "10.0.255.0/27"
$vnet | Set-AzVirtualNetwork

# Create public IP for VPN Gateway
$pip = New-AzPublicIpAddress -ResourceGroupName "RG-Network" `
    -Name "VPNGateway-PIP" `
    -Location "eastus" `
    -AllocationMethod "Static" `
    -Sku "Standard"

# Create VPN Gateway IP configuration
$vnet = Get-AzVirtualNetwork -ResourceGroupName "RG-Network" -Name "VNet-Prod"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "GatewaySubnet"
$gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name "GatewayIPConfig" `
    -SubnetId $subnet.Id `
    -PublicIpAddressId $pip.Id

# Create VPN Gateway (takes 30-45 minutes)
New-AzVirtualNetworkGateway -ResourceGroupName "RG-Network" `
    -Name "VPNGateway" `
    -Location "eastus" `
    -IpConfigurations $gwipconfig `
    -GatewayType "Vpn" `
    -VpnType "RouteBased" `
    -GatewaySku "VpnGw1" `
    -VpnGatewayGeneration "Generation2"

# Get VPN Gateway
Get-AzVirtualNetworkGateway -ResourceGroupName "RG-Network" -Name "VPNGateway"

# Create Local Network Gateway (represents on-premises)
New-AzLocalNetworkGateway -ResourceGroupName "RG-Network" `
    -Name "OnPremisesGateway" `
    -Location "eastus" `
    -GatewayIpAddress "203.0.113.5" `
    -AddressPrefix "192.168.0.0/16"

# Create VPN connection
$gateway1 = Get-AzVirtualNetworkGateway -ResourceGroupName "RG-Network" -Name "VPNGateway"
$local = Get-AzLocalNetworkGateway -ResourceGroupName "RG-Network" -Name "OnPremisesGateway"
New-AzVirtualNetworkGatewayConnection -ResourceGroupName "RG-Network" `
    -Name "OnPremConnection" `
    -Location "eastus" `
    -VirtualNetworkGateway1 $gateway1 `
    -LocalNetworkGateway2 $local `
    -ConnectionType IPsec `
    -SharedKey "YourSharedKey123!"

# Get connection status
Get-AzVirtualNetworkGatewayConnection -ResourceGroupName "RG-Network" `
    -Name "OnPremConnection"

# Reset VPN Gateway (if troubleshooting)
Reset-AzVirtualNetworkGateway -VirtualNetworkGateway $gateway1
```

### Azure DNS

```powershell
# Create public DNS zone
New-AzDnsZone -ResourceGroupName "RG-Network" `
    -Name "contoso.com"

# Get DNS zone
Get-AzDnsZone -ResourceGroupName "RG-Network" -Name "contoso.com"

# List all DNS zones
Get-AzDnsZone -ResourceGroupName "RG-Network"

# Create A record
New-AzDnsRecordSet -ResourceGroupName "RG-Network" `
    -ZoneName "contoso.com" `
    -Name "www" `
    -RecordType A `
    -Ttl 3600 `
    -DnsRecords (New-AzDnsRecordConfig -IPv4Address "20.50.100.5")

# Create multiple A records (record set)
$records = @()
$records += New-AzDnsRecordConfig -IPv4Address "20.50.100.5"
$records += New-AzDnsRecordConfig -IPv4Address "20.50.100.6"
New-AzDnsRecordSet -ResourceGroupName "RG-Network" `
    -ZoneName "contoso.com" `
    -Name "www" `
    -RecordType A `
    -Ttl 3600 `
    -DnsRecords $records

# Create CNAME record
New-AzDnsRecordSet -ResourceGroupName "RG-Network" `
    -ZoneName "contoso.com" `
    -Name "blog" `
    -RecordType CNAME `
    -Ttl 3600 `
    -DnsRecords (New-AzDnsRecordConfig -Cname "contoso.azurewebsites.net")

# Get record set
Get-AzDnsRecordSet -ResourceGroupName "RG-Network" `
    -ZoneName "contoso.com" `
    -Name "www" `
    -RecordType A

# Update record set
$recordSet = Get-AzDnsRecordSet -ResourceGroupName "RG-Network" `
    -ZoneName "contoso.com" `
    -Name "www" `
    -RecordType A
$recordSet.Records[0].Ipv4Address = "20.50.100.10"
Set-AzDnsRecordSet -RecordSet $recordSet

# Remove record set
Remove-AzDnsRecordSet -ResourceGroupName "RG-Network" `
    -ZoneName "contoso.com" `
    -Name "www" `
    -RecordType A `
    -Force

# Create private DNS zone
New-AzPrivateDnsZone -ResourceGroupName "RG-Network" `
    -Name "contoso.internal"

# Link private DNS zone to VNet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "RG-Network" -Name "VNet-Prod"
New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName "RG-Network" `
    -ZoneName "contoso.internal" `
    -Name "VNetLink" `
    -VirtualNetworkId $vnet.Id `
    -EnableRegistration

# Create A record in private DNS zone
New-AzPrivateDnsRecordSet -ResourceGroupName "RG-Network" `
    -ZoneName "contoso.internal" `
    -Name "server1" `
    -RecordType A `
    -Ttl 3600 `
    -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address "10.0.1.4")
```

### Network Watcher

```powershell
# Enable Network Watcher in region
Register-AzProviderFeature -FeatureName "AllowNetworkWatcher" `
    -ProviderNamespace "Microsoft.Network"

# Get Network Watcher
Get-AzNetworkWatcher -Location "eastus"

# Test IP flow verify
$vm = Get-AzVM -ResourceGroupName "RG-Network" -Name "VM1"
$nic = Get-AzNetworkInterface -ResourceGroupName "RG-Network" -Name "VM1-NIC"
Test-AzNetworkWatcherIPFlow -NetworkWatcher $nw `
    -TargetVirtualMachineId $vm.Id `
    -Direction Outbound `
    -Protocol TCP `
    -LocalIPAddress "10.0.1.4" `
    -LocalPort 80 `
    -RemoteIPAddress "20.50.100.5" `
    -RemotePort 80

# Get next hop
$nw = Get-AzNetworkWatcher -Location "eastus"
Get-AzNetworkWatcherNextHop -NetworkWatcher $nw `
    -TargetVirtualMachineId $vm.Id `
    -SourceIPAddress "10.0.1.4" `
    -DestinationIPAddress "10.0.2.5"

# Test connectivity
Test-AzNetworkWatcherConnectivity -NetworkWatcher $nw `
    -SourceId $vm.Id `
    -DestinationAddress "www.microsoft.com" `
    -DestinationPort 443

# Start packet capture
$storageAccount = Get-AzStorageAccount -ResourceGroupName "RG-Network" -Name "storage"
New-AzNetworkWatcherPacketCapture -NetworkWatcher $nw `
    -PacketCaptureName "Capture1" `
    -TargetVirtualMachineId $vm.Id `
    -StorageAccountId $storageAccount.Id `
    -TimeLimitInSeconds 60

# Stop packet capture
Stop-AzNetworkWatcherPacketCapture -NetworkWatcher $nw `
    -PacketCaptureName "Capture1"

# Enable NSG flow logs
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "RG-Network" -Name "NSG-Web"
$storageAccount = Get-AzStorageAccount -ResourceGroupName "RG-Network" -Name "storage"
Set-AzNetworkWatcherConfigFlowLog -NetworkWatcher $nw `
    -TargetResourceId $nsg.Id `
    -StorageAccountId $storageAccount.Id `
    -EnableFlowLog $true `
    -EnableRetention $true `
    -RetentionInDays 30

# Get flow log configuration
Get-AzNetworkWatcherFlowLogStatus -NetworkWatcher $nw `
    -TargetResourceId $nsg.Id
```

---

## Azure CLI Commands Reference

### Virtual Networks

```bash
# Login to Azure
az login
az account set --subscription "Production"

# Create resource group
az group create --name "RG-Network" --location "eastus"

# Create VNet with subnet
az network vnet create \
    --resource-group "RG-Network" \
    --name "VNet-Prod" \
    --location "eastus" \
    --address-prefix "10.0.0.0/16" \
    --subnet-name "Subnet1" \
    --subnet-prefix "10.0.1.0/24"

# Get VNet
az network vnet show \
    --resource-group "RG-Network" \
    --name "VNet-Prod"

# List all VNets
az network vnet list --output table

# Add address space
az network vnet update \
    --resource-group "RG-Network" \
    --name "VNet-Prod" \
    --address-prefixes "10.0.0.0/16" "10.1.0.0/16"

# Add subnet
az network vnet subnet create \
    --resource-group "RG-Network" \
    --vnet-name "VNet-Prod" \
    --name "Subnet2" \
    --address-prefix "10.0.2.0/24"

# Update subnet
az network vnet subnet update \
    --resource-group "RG-Network" \
    --vnet-name "VNet-Prod" \
    --name "Subnet1" \
    --address-prefix "10.0.1.0/25"

# List subnets
az network vnet subnet list \
    --resource-group "RG-Network" \
    --vnet-name "VNet-Prod" \
    --output table

# Delete subnet
az network vnet subnet delete \
    --resource-group "RG-Network" \
    --vnet-name "VNet-Prod" \
    --name "Subnet2"

# Delete VNet
az network vnet delete \
    --resource-group "RG-Network" \
    --name "VNet-Prod"
```

### Network Security Groups

```bash
# Create NSG
az network nsg create \
    --resource-group "RG-Network" \
    --name "NSG-Web" \
    --location "eastus"

# Create NSG rule
az network nsg rule create \
    --resource-group "RG-Network" \
    --nsg-name "NSG-Web" \
    --name "Allow-HTTP" \
    --priority 100 \
    --source-address-prefixes "Internet" \
    --source-port-ranges "*" \
    --destination-address-prefixes "*" \
    --destination-port-ranges 80 \
    --access "Allow" \
    --protocol "Tcp" \
    --direction "Inbound"

# Create multiple rules
az network nsg rule create \
    --resource-group "RG-Network" \
    --nsg-name "NSG-Web" \
    --name "Allow-HTTPS" \
    --priority 110 \
    --destination-port-ranges 443 \
    --access "Allow" \
    --protocol "Tcp"

az network nsg rule create \
    --resource-group "RG-Network" \
    --nsg-name "NSG-Web" \
    --name "Allow-RDP" \
    --priority 120 \
    --source-address-prefixes "203.0.113.5" \
    --destination-port-ranges 3389 \
    --access "Allow" \
    --protocol "Tcp"

# List NSG rules
az network nsg rule list \
    --resource-group "RG-Network" \
    --nsg-name "NSG-Web" \
    --output table

# Show specific rule
az network nsg rule show \
    --resource-group "RG-Network" \
    --nsg-name "NSG-Web" \
    --name "Allow-HTTP"

# Update rule
az network nsg rule update \
    --resource-group "RG-Network" \
    --nsg-name "NSG-Web" \
    --name "Allow-RDP" \
    --source-address-prefixes "203.0.113.0/24"

# Associate NSG with subnet
az network vnet subnet update \
    --resource-group "RG-Network" \
    --vnet-name "VNet-Prod" \
    --name "Subnet1" \
    --network-security-group "NSG-Web"

# Associate NSG with NIC
az network nic update \
    --resource-group "RG-Network" \
    --name "VM-NIC" \
    --network-security-group "NSG-Web"

# Remove NSG from subnet
az network vnet subnet update \
    --resource-group "RG-Network" \
    --vnet-name "VNet-Prod" \
    --name "Subnet1" \
    --network-security-group ""

# Delete rule
az network nsg rule delete \
    --resource-group "RG-Network" \
    --nsg-name "NSG-Web" \
    --name "Allow-SSH"

# Delete NSG
az network nsg delete \
    --resource-group "RG-Network" \
    --name "NSG-Web"
```

### Public IP Addresses

```bash
# Create basic public IP (dynamic)
az network public-ip create \
    --resource-group "RG-Network" \
    --name "PIP-VM" \
    --location "eastus" \
    --allocation-method "Dynamic"

# Create standard public IP (static)
az network public-ip create \
    --resource-group "RG-Network" \
    --name "PIP-LB" \
    --location "eastus" \
    --sku "Standard" \
    --allocation-method "Static"

# Create zone-redundant public IP
az network public-ip create \
    --resource-group "RG-Network" \
    --name "PIP-Gateway" \
    --location "eastus" \
    --sku "Standard" \
    --allocation-method "Static" \
    --zone 1 2 3

# Get public IP
az network public-ip show \
    --resource-group "RG-Network" \
    --name "PIP-VM"

# List all public IPs
az network public-ip list --output table

# Update public IP
az network public-ip update \
    --resource-group "RG-Network" \
    --name "PIP-VM" \
    --allocation-method "Static"

# Delete public IP
az network public-ip delete \
    --resource-group "RG-Network" \
    --name "PIP-VM"
```

### VNet Peering

```bash
# Get VNet IDs
vnet1Id=$(az network vnet show \
    --resource-group "RG-Network" \
    --name "VNet1" \
    --query id \
    --output tsv)

vnet2Id=$(az network vnet show \
    --resource-group "RG-Network" \
    --name "VNet2" \
    --query id \
    --output tsv)

# Create peering (VNet1 to VNet2)
az network vnet peering create \
    --resource-group "RG-Network" \
    --name "VNet1-to-VNet2" \
    --vnet-name "VNet1" \
    --remote-vnet $vnet2Id \
    --allow-vnet-access

# Create reverse peering (VNet2 to VNet1)
az network vnet peering create \
    --resource-group "RG-Network" \
    --name "VNet2-to-VNet1" \
    --vnet-name "VNet2" \
    --remote-vnet $vnet1Id \
    --allow-vnet-access

# Create peering with gateway transit
az network vnet peering create \
    --resource-group "RG-Network" \
    --name "Spoke-to-Hub" \
    --vnet-name "Spoke" \
    --remote-vnet $hubId \
    --allow-vnet-access \
    --use-remote-gateways

az network vnet peering create \
    --resource-group "RG-Network" \
    --name "Hub-to-Spoke" \
    --vnet-name "Hub" \
    --remote-vnet $spokeId \
    --allow-vnet-access \
    --allow-gateway-transit

# List peerings
az network vnet peering list \
    --resource-group "RG-Network" \
    --vnet-name "VNet1" \
    --output table

# Show peering
az network vnet peering show \
    --resource-group "RG-Network" \
    --name "VNet1-to-VNet2" \
    --vnet-name "VNet1"

# Delete peering
az network vnet peering delete \
    --resource-group "RG-Network" \
    --name "VNet1-to-VNet2" \
    --vnet-name "VNet1"
```

### Load Balancer

```bash
# Create public IP for load balancer
az network public-ip create \
    --resource-group "RG-Network" \
    --name "LB-PIP" \
    --sku "Standard" \
    --allocation-method "Static"

# Create load balancer
az network lb create \
    --resource-group "RG-Network" \
    --name "LoadBalancer" \
    --sku "Standard" \
    --public-ip-address "LB-PIP" \
    --frontend-ip-name "FrontEnd" \
    --backend-pool-name "BackendPool"

# Create health probe
az network lb probe create \
    --resource-group "RG-Network" \
    --lb-name "LoadBalancer" \
    --name "HealthProbe" \
    --protocol http \
    --port 80 \
    --path "/"

# Create load balancing rule
az network lb rule create \
    --resource-group "RG-Network" \
    --lb-name "LoadBalancer" \
    --name "HTTPRule" \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name "FrontEnd" \
    --backend-pool-name "BackendPool" \
    --probe-name "HealthProbe"

# Get load balancer
az network lb show \
    --resource-group "RG-Network" \
    --name "LoadBalancer"

# Add NIC to backend pool
az network nic ip-config address-pool add \
    --resource-group "RG-Network" \
    --nic-name "VM-NIC" \
    --ip-config-name "ipconfig1" \
    --lb-name "LoadBalancer" \
    --address-pool "BackendPool"

# Create inbound NAT rule
az network lb inbound-nat-rule create \
    --resource-group "RG-Network" \
    --lb-name "LoadBalancer" \
    --name "RDP-VM1" \
    --protocol tcp \
    --frontend-port 3389 \
    --backend-port 3389 \
    --frontend-ip-name "FrontEnd"

# List backend pool addresses
az network lb address-pool address list \
    --resource-group "RG-Network" \
    --lb-name "LoadBalancer" \
    --pool-name "BackendPool"
```

### Application Gateway

```bash
# Create public IP
az network public-ip create \
    --resource-group "RG-Network" \
    --name "AppGW-PIP" \
    --sku "Standard" \
    --allocation-method "Static"

# Create Application Gateway
az network application-gateway create \
    --resource-group "RG-Network" \
    --name "AppGateway" \
    --location "eastus" \
    --sku "Standard_v2" \
    --capacity 2 \
    --vnet-name "VNet-Prod" \
    --subnet "AppGWSubnet" \
    --public-ip-address "AppGW-PIP" \
    --servers 10.0.1.4 10.0.1.5

# Get Application Gateway
az network application-gateway show \
    --resource-group "RG-Network" \
    --name "AppGateway"

# Add backend pool
az network application-gateway address-pool create \
    --resource-group "RG-Network" \
    --gateway-name "AppGateway" \
    --name "ImagePool" \
    --servers 10.0.2.4 10.0.2.5

# Create URL path map
az network application-gateway url-path-map create \
    --resource-group "RG-Network" \
    --gateway-name "AppGateway" \
    --name "PathMap" \
    --paths "/images/*" \
    --address-pool "ImagePool" \
    --default-address-pool "BackendPool" \
    --http-settings "appGatewayBackendHttpSettings"

# Create path-based rule
az network application-gateway url-path-map rule create \
    --resource-group "RG-Network" \
    --gateway-name "AppGateway" \
    --path-map-name "PathMap" \
    --name "ImagesRule" \
    --paths "/images/*" \
    --address-pool "ImagePool" \
    --http-settings "appGatewayBackendHttpSettings"

# Enable WAF
az network application-gateway waf-config set \
    --resource-group "RG-Network" \
    --gateway-name "AppGateway" \
    --enabled true \
    --firewall-mode "Prevention" \
    --rule-set-version "3.2"
```

### VPN Gateway

```bash
# Create gateway subnet
az network vnet subnet create \
    --resource-group "RG-Network" \
    --vnet-name "VNet-Prod" \
    --name "GatewaySubnet" \
    --address-prefix "10.0.255.0/27"

# Create public IP for VPN Gateway
az network public-ip create \
    --resource-group "RG-Network" \
    --name "VPNGateway-PIP" \
    --sku "Standard" \
    --allocation-method "Static"

# Create VPN Gateway (takes 30-45 minutes)
az network vnet-gateway create \
    --resource-group "RG-Network" \
    --name "VPNGateway" \
    --location "eastus" \
    --public-ip-address "VPNGateway-PIP" \
    --vnet "VNet-Prod" \
    --gateway-type "Vpn" \
    --vpn-type "RouteBased" \
    --sku "VpnGw1" \
    --vpn-gateway-generation "Generation2"

# Get VPN Gateway
az network vnet-gateway show \
    --resource-group "RG-Network" \
    --name "VPNGateway"

# Create Local Network Gateway
az network local-gateway create \
    --resource-group "RG-Network" \
    --name "OnPremisesGateway" \
    --location "eastus" \
    --gateway-ip-address "203.0.113.5" \
    --local-address-prefixes "192.168.0.0/16"

# Create VPN connection
az network vpn-connection create \
    --resource-group "RG-Network" \
    --name "OnPremConnection" \
    --location "eastus" \
    --vnet-gateway1 "VPNGateway" \
    --local-gateway2 "OnPremisesGateway" \
    --shared-key "YourSharedKey123!"

# Show connection
az network vpn-connection show \
    --resource-group "RG-Network" \
    --name "OnPremConnection"

# Check connection status
az network vpn-connection show \
    --resource-group "RG-Network" \
    --name "OnPremConnection" \
    --query "connectionStatus"
```

### Azure DNS

```bash
# Create public DNS zone
az network dns zone create \
    --resource-group "RG-Network" \
    --name "contoso.com"

# List DNS zones
az network dns zone list --output table

# Create A record
az network dns record-set a add-record \
    --resource-group "RG-Network" \
    --zone-name "contoso.com" \
    --record-set-name "www" \
    --ipv4-address "20.50.100.5"

# Create multiple A records
az network dns record-set a add-record \
    --resource-group "RG-Network" \
    --zone-name "contoso.com" \
    --record-set-name "www" \
    --ipv4-address "20.50.100.6"

# Create CNAME record
az network dns record-set cname set-record \
    --resource-group "RG-Network" \
    --zone-name "contoso.com" \
    --record-set-name "blog" \
    --cname "contoso.azurewebsites.net"

# List record sets
az network dns record-set list \
    --resource-group "RG-Network" \
    --zone-name "contoso.com" \
    --output table

# Show specific record
az network dns record-set a show \
    --resource-group "RG-Network" \
    --zone-name "contoso.com" \
    --name "www"

# Delete record
az network dns record-set a remove-record \
    --resource-group "RG-Network" \
    --zone-name "contoso.com" \
    --record-set-name "www" \
    --ipv4-address "20.50.100.6"

# Delete record set
az network dns record-set a delete \
    --resource-group "RG-Network" \
    --zone-name "contoso.com" \
    --name "www" \
    --yes

# Create private DNS zone
az network private-dns zone create \
    --resource-group "RG-Network" \
    --name "contoso.internal"

# Link private DNS zone to VNet
az network private-dns link vnet create \
    --resource-group "RG-Network" \
    --zone-name "contoso.internal" \
    --name "VNetLink" \
    --virtual-network "VNet-Prod" \
    --registration-enabled true

# Create A record in private DNS
az network private-dns record-set a add-record \
    --resource-group "RG-Network" \
    --zone-name "contoso.internal" \
    --record-set-name "server1" \
    --ipv4-address "10.0.1.4"
```

### Network Watcher

```bash
# Enable Network Watcher
az network watcher configure \
    --resource-group "NetworkWatcherRG" \
    --locations "eastus" \
    --enabled true

# Test IP flow
az network watcher test-ip-flow \
    --resource-group "RG-Network" \
    --vm "VM1" \
    --direction "Outbound" \
    --protocol "TCP" \
    --local "10.0.1.4:80" \
    --remote "20.50.100.5:80"

# Get next hop
az network watcher show-next-hop \
    --resource-group "RG-Network" \
    --vm "VM1" \
    --source-ip "10.0.1.4" \
    --dest-ip "10.0.2.5"

# Test connectivity
az network watcher test-connectivity \
    --resource-group "RG-Network" \
    --source-resource "VM1" \
    --dest-address "www.microsoft.com" \
    --dest-port 443

# Start packet capture
az network watcher packet-capture create \
    --resource-group "RG-Network" \
    --vm "VM1" \
    --name "Capture1" \
    --storage-account "mystorageaccount" \
    --time-limit 60

# Stop packet capture
az network watcher packet-capture stop \
    --location "eastus" \
    --name "Capture1"

# Configure NSG flow logs
az network watcher flow-log create \
    --resource-group "RG-Network" \
    --nsg "NSG-Web" \
    --name "FlowLog1" \
    --storage-account "mystorageaccount" \
    --enabled true \
    --retention 30 \
    --format JSON \
    --log-version 2

# Show flow log configuration
az network watcher flow-log show \
    --location "eastus" \
    --name "FlowLog1"

# Run connectivity troubleshooting
az network watcher troubleshooting start \
    --resource "VPNGateway" \
    --resource-type "vnetGateway" \
    --storage-account "mystorageaccount" \
    --storage-path "https://mystorageaccount.blob.core.windows.net/troubleshooting"
```

---

## Common Exam Scenarios

### Scenario 1: VM Cannot Access Internet

**Problem:**
VM has no outbound internet connectivity.

**Troubleshooting steps:**

1. **Check NSG rules:**

   ```
   Default: Outbound to Internet allowed
   Check: Custom rules blocking outbound?
   Solution: Review NSG rules, ensure no deny rules
   ```

2. **Check User-Defined Routes (UDR):**

   ```
   Custom route to 0.0.0.0/0 with next hop None?
   Solution: Remove blocking route or update next hop
   ```

3. **Check Azure Firewall:**

   ```
   If using Azure Firewall, check network rules
   Solution: Add allow rule for outbound traffic
   ```

4. **Check VM has internet access enabled:**
   ```
   Verify VNet configuration allows internet access
   ```

**CLI check:**

```bash
az network watcher test-ip-flow \
    --vm "VM1" \
    --direction "Outbound" \
    --protocol "TCP" \
    --local "10.0.1.4:80" \
    --remote "8.8.8.8:80"
```

### Scenario 2: Cannot Connect VMs in Peered VNets

**Problem:**
VM in VNet A cannot reach VM in VNet B despite peering configured.

**Common causes:**

1. **Peering not bidirectional:**

   ```
   VNet A → VNet B: ✅ Configured
   VNet B → VNet A: ❌ Missing

   Solution: Create reverse peering
   ```

2. **NSG blocking traffic:**

   ```
   Check NSG rules on both VMs
   Solution: Add allow rules for peered VNet address space
   ```

3. **Overlapping address spaces:**

   ```
   VNet A: 10.0.0.0/16
   VNet B: 10.0.0.0/16

   Cannot peer (overlap)
   Solution: Redesign address spaces
   ```

4. **Using service chaining without UDR:**
   ```
   VNet A → Hub (firewall) → VNet B
   Without UDR, traffic goes direct
   Solution: Add UDR to route through Hub
   ```

### Scenario 3: Choose Load Balancing Solution

**Problem:**
Application needs load balancing. Which service to use?

**Decision criteria:**

**Scenario A: Internal API (Layer 4, regional)**

```
Requirements:
- TCP traffic
- Backend VMs in same region
- No HTTP inspection needed

Solution: Internal Load Balancer
Cost: ~$18/month
```

**Scenario B: Web application (Layer 7, regional)**

```
Requirements:
- HTTP/HTTPS traffic
- URL-based routing needed
- Multi-site hosting
- Regional deployment

Solution: Application Gateway
Cost: ~$300/month
```

**Scenario C: Global web application**

```
Requirements:
- HTTP/HTTPS traffic
- Users worldwide
- Need CDN
- WAF protection

Solution: Azure Front Door
Cost: ~$400/month
```

**Scenario D: DNS-based failover**

```
Requirements:
- Automatic failover between regions
- No additional latency
- Works with any protocol

Solution: Traffic Manager
Cost: ~$0.54 per million queries
```

### Scenario 4: Site-to-Site VPN Not Connecting

**Problem:**
VPN tunnel not establishing.

**Troubleshooting checklist:**

1. **On-premises VPN device configuration:**

   ```
   Correct Azure Gateway IP?
   Correct shared key?
   Correct protocol (IKEv2)?
   ```

2. **Local Network Gateway:**

   ```
   Correct on-premises public IP?
   Correct on-premises address spaces?
   ```

3. **Gateway subnet size:**

   ```
   Minimum /29, recommended /27
   Named "GatewaySubnet"?
   ```

4. **NSG on Gateway subnet:**

   ```
   Gateway subnet should NOT have NSG
   Remove NSG if present
   ```

5. **Check connection status:**

   ```bash
   az network vpn-connection show \
       --name "OnPremConnection" \
       --query "connectionStatus"
   ```

6. **Check VPN device compatibility:**
   ```
   Is device on Azure validated list?
   Firmware up to date?
   ```

### Scenario 5: DNS Resolution Not Working

**Problem:**
VMs cannot resolve each other by name.

**Scenarios and solutions:**

**Within same VNet:**

```
Issue: VM1 can't ping VM2 by name
Check: Using Azure-provided DNS?
Solution: Should work by default, check NSG for port 53
```

**Between peered VNets:**

```
Issue: VM in VNet A can't resolve VM in VNet B
Cause: Azure-provided DNS doesn't work across VNets
Solution: Use Azure Private DNS with VNet links
```

**Custom domain (e.g., contoso.internal):**

```
Issue: Custom domain not resolving
Solution: Create Private DNS zone, link to VNets
```

**From on-premises:**

```
Issue: On-premises can't resolve Azure VMs
Solution:
- Deploy Azure Private DNS Resolver
- Configure conditional forwarding from on-prem DNS
```

**Implementation:**

```bash
# Create private DNS zone
az network private-dns zone create \
    --resource-group "RG-Network" \
    --name "contoso.internal"

# Link to VNets
az network private-dns link vnet create \
    --resource-group "RG-Network" \
    --zone-name "contoso.internal" \
    --name "VNetLink" \
    --virtual-network "VNet-Prod" \
    --registration-enabled true
```

### Scenario 6: Need to Isolate Production from Dev

**Problem:**
Production and Dev workloads in same VNet, need isolation.

**Solution: Hub-Spoke with NSGs**

**Architecture:**

```
Hub VNet (10.0.0.0/16)
├─ Azure Firewall: 10.0.0.0/26
└─ Gateway subnet: 10.0.255.0/27

Production VNet (10.1.0.0/16)
├─ Web: 10.1.1.0/24
├─ App: 10.1.2.0/24
└─ Database: 10.1.3.0/24

Dev VNet (10.2.0.0/16)
├─ Web: 10.2.1.0/24
└─ App: 10.2.2.0/24
```

**NSG Rules:**

```
Production subnets:
- Deny all from 10.2.0.0/16 (Dev VNet)
- Allow from 10.1.0.0/16 (internal)
- Allow from on-premises (via VPN)

Dev subnets:
- Deny all from 10.1.0.0/16 (Prod VNet)
- Allow from 10.2.0.0/16 (internal)
```

**UDR for inter-spoke:**

```
If Prod needs to access Dev:
Route: 10.2.0.0/16 → Next hop: Azure Firewall
Firewall rules control access
```

### Scenario 7: Application Gateway vs Load Balancer

**Problem:**
When to use which?

**Use Load Balancer when:**

- Layer 4 (TCP/UDP) load balancing needed
- No HTTP inspection required
- Simple round-robin distribution
- Cost-sensitive ($18/month vs $300/month)
- Internal load balancing for databases

**Example:**

```
SQL Database cluster:
- TCP traffic (port 1433)
- No URL routing needed
- Internal communication only

Solution: Internal Load Balancer
```

**Use Application Gateway when:**

- Layer 7 (HTTP/HTTPS) needed
- URL-based routing required
- Multi-site hosting (multiple domains)
- WAF protection needed
- SSL termination wanted
- Cookie-based session affinity

**Example:**

```
Web application:
- Multiple microservices (/api, /images, /videos)
- Need WAF protection
- HTTPS with SSL offload

Solution: Application Gateway with WAF
```

### Scenario 8: Network Performance Issues

**Problem:**
High latency between Azure resources.

**Troubleshooting:**

1. **Check resource location:**

   ```
   VM in East US → Storage in West Europe
   Solution: Move resources to same region
   ```

2. **Check if using VPN:**

   ```
   VPN bandwidth limited (max 10 Gbps)
   High latency over internet
   Solution: Consider ExpressRoute
   ```

3. **Check for network virtual appliances:**

   ```
   Traffic going through NVA/Firewall?
   NVA causing bottleneck?
   Solution: Check NVA sizing, bypass if possible
   ```

4. **Check VM SKU:**

   ```
   Small VM = limited network bandwidth
   Solution: Use larger VM or accelerated networking
   ```

5. **Use Network Watcher:**
   ```bash
   # Check latency
   az network watcher test-connectivity \
       --source-resource "VM1" \
       --dest-resource "VM2"
   ```

---

## Practice Questions

### Question 1

You have a VNet with address space 10.0.0.0/16. You create a subnet with address prefix 10.0.1.0/24. How many IP addresses are available for resources?

**A)** 256  
**B)** 254  
**C)** 251  
**D)** 250

**Answer:** C

**Explanation:**

- Total in /24: 256 addresses
- Azure reserves 5 addresses (network, gateway, 2x DNS, broadcast)
- Available: 256 - 5 = 251
- A incorrect: Doesn't account for reserved IPs
- B incorrect: Standard subnet math, but Azure reserves 5 not 2
- D incorrect: Wrong calculation

### Question 2

You need to connect two VNets in different regions. The VNets have the following address spaces:

- VNet A: 10.0.0.0/16
- VNet B: 10.0.0.0/16

What should you do first?

**A)** Create VNet peering  
**B)** Change address spaces to not overlap  
**C)** Create VPN Gateway  
**D)** Use Traffic Manager

**Answer:** B

**Explanation:**

- Cannot peer VNets with overlapping address spaces
- Must redesign address spaces first
- Then can create peering
- A incorrect: Cannot peer with overlapping addresses
- C incorrect: VPN Gateway doesn't solve overlap issue
- D incorrect: Traffic Manager is for DNS-based routing

### Question 3

You have an NSG with the following rules:

Priority 100: Allow HTTP from Internet
Priority 200: Allow HTTPS from Internet  
Priority 300: Deny all from Internet

A request from Internet to port 443 arrives. What happens?

**A)** Denied by priority 300  
**B)** Allowed by priority 200  
**C)** Denied by default rule  
**D)** Allowed by default rule

**Answer:** B

**Explanation:**

- Rules evaluated by priority (lowest first)
- Request matches priority 200 (HTTPS = 443)
- Once match found, evaluation stops
- Traffic allowed
- A incorrect: Priority 200 matches first, rule 300 not evaluated
- C/D incorrect: Default rules not reached

### Question 4

You need to load balance HTTP traffic across VMs in multiple Azure regions. Which solution should you use?

**A)** Azure Load Balancer  
**B)** Application Gateway  
**C)** Azure Front Door  
**D)** Traffic Manager

**Answer:** C

**Explanation:**

- Azure Front Door: Global Layer 7 load balancer with CDN
- Supports HTTP/HTTPS routing across regions
- A incorrect: Regional only
- B incorrect: Regional only
- D incorrect: DNS-based, doesn't handle HTTP traffic directly

### Question 5

You create VNet peering between VNet A and VNet B. VNet B is peered to VNet C. Can resources in VNet A communicate with VNet C?

**A)** Yes, peering is transitive  
**B)** Yes, if both peerings allow gateway transit  
**C)** No, must create direct peering from A to C  
**D)** Yes, if VNet B is configured as hub

**Answer:** C

**Explanation:**

- VNet peering is non-transitive
- A → B and B → C does NOT mean A → C
- Must create direct peering A → C for communication
- A incorrect: Peering is non-transitive by design
- B incorrect: Gateway transit doesn't enable transitivity
- D incorrect: Hub-spoke requires UDR for spoke-to-spoke

### Question 6

Your company requires that all outbound internet traffic from Azure VMs be inspected. What should you implement?

**A)** NSG with deny rules  
**B)** Azure Firewall with UDR  
**C)** VNet peering  
**D)** Private endpoints

**Answer:** B

**Explanation:**

- Azure Firewall inspects and filters traffic
- UDR forces all traffic through firewall (0.0.0.0/0 → Firewall)
- A incorrect: NSG doesn't inspect, only allows/denies
- C incorrect: Peering doesn't provide inspection
- D incorrect: Private endpoints for inbound to services

### Question 7

You have a VPN Gateway with Basic SKU. You need to enable Point-to-Site VPN. What should you do?

**A)** Enable P2S on existing gateway  
**B)** Upgrade to VpnGw1 SKU  
**C)** Change VPN type to Policy-Based  
**D)** Add another VPN Gateway

**Answer:** B

**Explanation:**

- Basic SKU doesn't support Point-to-Site VPN
- Cannot upgrade from Basic to VpnGw SKUs (must delete and recreate)
- Must create new gateway with VpnGw1 or higher
- A incorrect: Basic doesn't support P2S
- C incorrect: Policy-Based has even fewer features
- D incorrect: Cannot have two VPN Gateways in same VNet

### Question 8

You create an Azure Private DNS zone named contoso.internal and link it to your VNet with auto-registration enabled. What happens when you create a new VM?

**A)** VM gets public DNS record  
**B)** VM automatically gets A record in private zone  
**C)** Must manually create DNS record  
**D)** VM uses Azure-provided DNS only

**Answer:** B

**Explanation:**

- Auto-registration automatically creates A record for VM
- Record points VM name to private IP
- Automatically deleted when VM deleted
- A incorrect: Private DNS doesn't create public records
- C incorrect: Auto-registration is automatic
- D incorrect: VM uses both Azure DNS and Private DNS

---

## Key Takeaways for Exam

### Must-Know Concepts

1. **VNet and Subnets**

   - Azure reserves 5 IPs per subnet
   - CIDR notation (/16, /24, etc.)
   - Cannot change address space after peering
   - Subnet must be named "GatewaySubnet" for VPN Gateway

2. **NSG Rules**

   - Evaluated by priority (lowest first)
   - Stop evaluation after first match
   - Default rules cannot be deleted
   - Can associate with subnet or NIC (or both)
   - Service tags simplify rule management

3. **VNet Peering**

   - Non-transitive (A→B, B→C doesn't mean A→C)
   - Must be bidirectional
   - Address spaces cannot overlap
   - Gateway transit for hub-spoke

4. **Load Balancing**

   - Layer 4 regional: Azure Load Balancer
   - Layer 7 regional: Application Gateway
   - Layer 7 global: Azure Front Door
   - DNS-based: Traffic Manager

5. **VPN Gateway**

   - Requires /27 or larger Gateway subnet
   - Cannot have NSG on Gateway subnet
   - Takes 30-45 minutes to deploy
   - Cannot resize from Basic to VpnGw SKUs

6. **DNS**

   - Azure-provided DNS: Same VNet only
   - Private DNS zones: Cross-VNet with links
   - Auto-registration: Automatic A records for VMs

7. **Service Endpoints vs Private Endpoints**
   - Service Endpoints: Azure backbone, public IP remains
   - Private Endpoints: Private IP, no public IP needed

### Common Exam Tricks

**Trick 1: "VMs in peered VNets can't communicate"**

- Check if peering is bidirectional
- Check for NSG rules blocking traffic
- Check for overlapping address spaces

**Trick 2: "Only X IP addresses available in subnet"**

- Remember: Azure reserves 5 IPs
- /24 = 251 usable (not 254)

**Trick 3: "VPN not connecting"**

- Check Gateway subnet size (min /29)
- Check if NSG on Gateway subnet (should be none)
- Verify shared key matches

**Trick 4: "Need to inspect all traffic"**

- NSG only allows/denies, doesn't inspect
- Use Azure Firewall with UDR for inspection

**Trick 5: "DNS not resolving across VNets"**

- Azure-provided DNS doesn't work across VNets
- Use Private DNS zones with VNet links

**Trick 6: "Choose load balancer"**

- HTTP with URL routing = Application Gateway
- TCP/UDP = Azure Load Balancer
- Global HTTP = Azure Front Door
- DNS failover = Traffic Manager

**Trick 7: "Peering A→B→C means A can reach C"**

- No! Peering is non-transitive
- Must create direct peering A→C

**Trick 8: "Basic public IP with Standard Load Balancer"**

- Incompatible! Must use Standard public IP with Standard LB
- Basic with Basic, Standard with Standard

### Quick Reference Tables

**NSG Priority:**

| Priority | What Happens                           |
| -------- | -------------------------------------- |
| 100-4095 | Custom rules (lower = higher priority) |
| 65000    | Default allow VNet inbound             |
| 65001    | Default allow Azure Load Balancer      |
| 65500    | Default deny all                       |

**VNet Peering States:**

| State        | Meaning                       |
| ------------ | ----------------------------- |
| Initiated    | Peering request sent          |
| Connected    | Peering active, traffic flows |
| Disconnected | Peering broken                |

**VPN Gateway SKUs:**

| SKU    | Bandwidth | Tunnels | P2S | Use Case    |
| ------ | --------- | ------- | --- | ----------- |
| Basic  | 100 Mbps  | 10      | ❌  | Dev/test    |
| VpnGw1 | 650 Mbps  | 30      | ✅  | Small prod  |
| VpnGw2 | 1 Gbps    | 30      | ✅  | Medium prod |
| VpnGw3 | 1.25 Gbps | 30      | ✅  | Large prod  |

**Load Balancer Selection:**

| Need              | Solution                              |
| ----------------- | ------------------------------------- |
| TCP/UDP, regional | Azure Load Balancer                   |
| HTTP, URL routing | Application Gateway                   |
| HTTP, global      | Azure Front Door                      |
| DNS failover      | Traffic Manager                       |
| Internal API      | Internal Load Balancer                |
| WAF protection    | Application Gateway WAF or Front Door |

---

## Summary Checklist

Before moving to Module 5, ensure you can:

**Virtual Networks:**

- [ ] Explain CIDR notation and calculate available IPs
- [ ] Create and configure VNets and subnets
- [ ] Understand reserved IP addresses (5 per subnet)
- [ ] Design VNet address space planning

**Network Security:**

- [ ] Create and configure NSGs
- [ ] Understand NSG rule priority and evaluation
- [ ] Use service tags and ASGs
- [ ] Explain NSG vs Azure Firewall differences

**VNet Peering:**

- [ ] Create VNet peering (bidirectional)
- [ ] Understand non-transitive nature
- [ ] Configure gateway transit
- [ ] Design hub-spoke topology

**Load Balancing:**

- [ ] Choose appropriate load balancing solution
- [ ] Configure Azure Load Balancer
- [ ] Configure Application Gateway
- [ ] Understand Traffic Manager and Front Door

**VPN Connectivity:**

- [ ] Create and configure VPN Gateway
- [ ] Understand Site-to-Site vs Point-to-Site
- [ ] Configure Local Network Gateway
- [ ] Troubleshoot VPN connections

**DNS:**

- [ ] Create public and private DNS zones
- [ ] Add DNS records (A, CNAME, etc.)
- [ ] Link private DNS zones to VNets
- [ ] Configure auto-registration

**Network Monitoring:**

- [ ] Use Network Watcher tools
- [ ] Configure NSG flow logs
- [ ] Use IP flow verify and next hop
- [ ] Troubleshoot connectivity issues

---

## Additional Resources

**Microsoft Learn Modules:**

- [Configure virtual networks](https://learn.microsoft.com/training/modules/configure-virtual-networks/)
- [Configure network security groups](https://learn.microsoft.com/training/modules/configure-network-security-groups/)
- [Configure Azure Virtual Network peering](https://learn.microsoft.com/training/modules/configure-vnet-peering/)
- [Configure Azure Load Balancer](https://learn.microsoft.com/training/modules/configure-azure-load-balancer/)
- [Configure Application Gateway](https://learn.microsoft.com/training/modules/configure-azure-application-gateway/)
- [Configure VPN Gateway](https://learn.microsoft.com/training/modules/configure-vpn-gateway/)
- [Configure Azure DNS](https://learn.microsoft.com/training/modules/configure-azure-dns/)
- [Network Watcher](https://learn.microsoft.com/training/modules/configure-network-watcher/)

**Documentation:**

- [Virtual Network documentation](https://learn.microsoft.com/azure/virtual-network/)
- [NSG documentation](https://learn.microsoft.com/azure/virtual-network/network-security-groups-overview)
- [Load Balancer documentation](https://learn.microsoft.com/azure/load-balancer/)
- [Application Gateway documentation](https://learn.microsoft.com/azure/application-gateway/)
- [VPN Gateway documentation](https://learn.microsoft.com/azure/vpn-gateway/)
- [Azure DNS documentation](https://learn.microsoft.com/azure/dns/)

**Hands-on Labs:**

- Create hub-spoke network topology
- Configure NSG rules and test with IP flow verify
- Set up VNet peering across regions
- Deploy and configure Application Gateway
- Create Site-to-Site VPN connection
- Configure Azure Private DNS with auto-registration

---

## Next Steps

Congratulations on completing Module 4!

**You've mastered:**

- Virtual networks and subnets
- Network security (NSGs, Azure Firewall)
- Load balancing solutions
- VNet connectivity (peering, VPN, ExpressRoute)
- DNS services
- Network monitoring and troubleshooting

**Next Module:** Module 5: Monitoring and Backup

In Module 5, you'll learn:

- Azure Monitor and Log Analytics
- Application Insights
- Azure Backup
- Azure Site Recovery
- Alerts and action groups
- Cost management and optimization

---

**Study Tips for Module 4**

**Week 1 Focus:**

- Virtual Networks (address spaces, subnets, CIDR)
- NSG rules and priority
- Practice IP calculations
- Hands-on: Create VNets and subnets

**Week 2 Focus:**

- VNet peering and connectivity
- Load balancing solutions
- When to use each service
- Hands-on: Configure peering and load balancers

**Week 3 Focus:**

- VPN Gateway and ExpressRoute
- DNS services
- Network Watcher tools
- Hands-on: Set up VPN and DNS

**Week 4 Focus:**

- Practice scenarios
- Troubleshooting exercises
- Command proficiency
- Mock exam questions

**Common Mistakes to Avoid:**

1. Forgetting Azure reserves 5 IPs per subnet
2. Assuming VNet peering is transitive (it's not!)
3. Putting NSG on Gateway subnet (forbidden)
4. Using wrong load balancer (Layer 4 vs Layer 7)
5. Not creating bidirectional peering
6. Mixing Basic and Standard SKUs
7. Overlapping address spaces when peering
8. Forgetting to link Private DNS zones to VNets

**Red Flags in Exam Questions:**

- "VMs can communicate A→B→C via peering" → Non-transitive!
- "Subnet has 256 available IPs" → 251 (Azure reserves 5)
- "VPN not working" → Check Gateway subnet, NSG, shared key
- "NSG blocking but should allow" → Check priority order
- "Need HTTP routing" → Layer 7 (Application Gateway/Front Door)
- "Global load balancing" → Front Door or Traffic Manager
- "DNS across VNets not working" → Need Private DNS zones
- "Basic public IP with Standard LB" → Incompatible SKUs

---

## Top 10 PowerShell Commands

```powershell
# 1. Create VNet with subnet
New-AzVirtualNetwork -ResourceGroupName "RG" -Name "VNet" -Location "eastus" -AddressPrefix "10.0.0.0/16" -Subnet (New-AzVirtualNetworkSubnetConfig -Name "Subnet1" -AddressPrefix "10.0.1.0/24")

# 2. Create NSG with rule
$rule = New-AzNetworkSecurityRuleConfig -Name "Allow-HTTP" -Priority 100 -Protocol Tcp -Access Allow -DestinationPortRange 80
New-AzNetworkSecurityGroup -ResourceGroupName "RG" -Name "NSG" -Location "eastus" -SecurityRules $rule

# 3. Create VNet peering
Add-AzVirtualNetworkPeering -Name "VNet1-to-VNet2" -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.Id

# 4. Create public IP
New-AzPublicIpAddress -ResourceGroupName "RG" -Name "PIP" -Location "eastus" -Sku "Standard" -AllocationMethod "Static"

# 5. Create load balancer
New-AzLoadBalancer -ResourceGroupName "RG" -Name "LB" -Location "eastus" -Sku "Standard" -FrontendIpConfiguration $frontendIP -BackendAddressPool $backendPool

# 6. Create VPN Gateway
New-AzVirtualNetworkGateway -ResourceGroupName "RG" -Name "VPNGateway" -Location "eastus" -GatewayType "Vpn" -VpnType "RouteBased" -IpConfigurations $gwipconfig

# 7. Create DNS zone
New-AzDnsZone -ResourceGroupName "RG" -Name "contoso.com"

# 8. Add DNS A record
New-AzDnsRecordSet -ResourceGroupName "RG" -ZoneName "contoso.com" -Name "www" -RecordType A -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "20.50.100.5")

# 9. Test IP flow
Test-AzNetworkWatcherIPFlow -TargetVirtualMachineId $vm.Id -Direction Outbound -Protocol TCP -LocalIPAddress "10.0.1.4" -RemoteIPAddress "20.50.100.5"

# 10. Get VNet
Get-AzVirtualNetwork -ResourceGroupName "RG" -Name "VNet"
```

## Top 10 Azure CLI Commands

```bash
# 1. Create VNet with subnet
az network vnet create --resource-group "RG" --name "VNet" --address-prefix "10.0.0.0/16" --subnet-name "Subnet1" --subnet-prefix "10.0.1.0/24"

# 2. Create NSG with rule
az network nsg create --resource-group "RG" --name "NSG"
az network nsg rule create --resource-group "RG" --nsg-name "NSG" --name "Allow-HTTP" --priority 100 --destination-port-ranges 80

# 3. Create VNet peering
az network vnet peering create --resource-group "RG" --name "VNet1-to-VNet2" --vnet-name "VNet1" --remote-vnet $vnet2Id

# 4. Create public IP
az network public-ip create --resource-group "RG" --name "PIP" --sku "Standard" --allocation-method "Static"

# 5. Create load balancer
az network lb create --resource-group "RG" --name "LB" --sku "Standard" --public-ip-address "PIP" --frontend-ip-name "FrontEnd" --backend-pool-name "BackendPool"

# 6. Create VPN Gateway
az network vnet-gateway create --resource-group "RG" --name "VPNGateway" --gateway-type "Vpn" --vpn-type "RouteBased" --vnet "VNet" --public-ip-address "PIP"

# 7. Create DNS zone
az network dns zone create --resource-group "RG" --name "contoso.com"

# 8. Add DNS A record
az network dns record-set a add-record --resource-group "RG" --zone-name "contoso.com" --record-set-name "www" --ipv4-address "20.50.100.5"

# 9. Test IP flow
az network watcher test-ip-flow --vm "VM" --direction "Outbound" --protocol "TCP" --local "10.0.1.4:80" --remote "20.50.100.5:80"

# 10. Get VNet
az network vnet show --resource-group "RG" --name "VNet"
```

---

## Practice Scenarios - Advanced

### Scenario 9: Hub-Spoke Implementation

**Requirement:**
Design hub-spoke network for company with production, dev, and test environments.

**Solution:**

**Hub VNet (10.0.0.0/16):**

```
Subnets:
- AzureFirewallSubnet: 10.0.0.0/26 (64 IPs)
- GatewaySubnet: 10.0.255.0/27 (32 IPs)
- SharedServices: 10.0.1.0/24 (251 IPs) - AD, DNS
- AzureBastionSubnet: 10.0.2.0/26 (64 IPs)

Resources:
- Azure Firewall
- VPN Gateway
- Azure Bastion
- Domain Controllers
```

**Production Spoke (10.1.0.0/16):**

```
Subnets:
- Web: 10.1.1.0/24
- App: 10.1.2.0/24
- Database: 10.1.3.0/24

Peering:
- To Hub (allow gateway transit, use remote gateway)
```

**Dev Spoke (10.2.0.0/16):**

```
Subnets:
- Dev: 10.2.1.0/24

Peering:
- To Hub (allow gateway transit, use remote gateway)
```

**Routing (UDR on each spoke subnet):**

```
Route Table: Spoke-Routes
- 0.0.0.0/0 → Azure Firewall (10.0.0.4)
- 10.0.0.0/8 → Azure Firewall (for spoke-to-spoke)
- On-premises → VPN Gateway
```

**NSG Rules:**

```
Production subnets:
- Deny from 10.2.0.0/16 (dev)
- Allow from 10.1.0.0/16 (internal prod)
- Allow from on-premises

Dev subnets:
- Deny from 10.1.0.0/16 (prod)
- Allow from 10.2.0.0/16 (internal dev)
```

### Scenario 10: Multi-Region with Disaster Recovery

**Requirement:**
Web application in East US with DR in West US, 99.99% SLA needed.

**Solution:**

**Primary Region (East US):**

```
VNet: 10.0.0.0/16
- Web Subnet: 10.0.1.0/24 (VM Scale Set)
- App Subnet: 10.0.2.0/24 (VM Scale Set)
- Database Subnet: 10.0.3.0/24 (SQL with geo-replication)

Load Balancing:
- Standard Load Balancer (zone-redundant)
- Availability Zones: 1, 2, 3
```

**Secondary Region (West US):**

```
VNet: 10.1.0.0/16
- Web Subnet: 10.1.1.0/24 (VM Scale Set - standby)
- App Subnet: 10.1.2.0/24 (VM Scale Set - standby)
- Database Subnet: 10.1.3.0/24 (SQL replica)

Load Balancing:
- Standard Load Balancer (zone-redundant)
```

**Global Components:**

```
Traffic Manager:
- Primary endpoint: East US Load Balancer (priority 1)
- Secondary endpoint: West US Load Balancer (priority 2)
- Routing method: Priority (failover)
- Health probes: Check /health endpoint

OR

Azure Front Door:
- Backend pools: East US, West US
- Health probes on both regions
- Automatic failover
- Built-in CDN
```

**VNet Connectivity:**

```
Global VNet Peering: East US ↔ West US
- Allows management traffic
- Database replication
- Monitoring communication
```

---

## Network Design Patterns

### Pattern 1: Internet-Facing Web Application

**Scenario:** Public website with backend database

```
Internet
   ↓
Azure Application Gateway (WAF enabled)
   ↓
Web Tier (10.0.1.0/24)
- VM Scale Set (zones 1,2,3)
- NSG: Allow 443 from App Gateway only
   ↓
Internal Load Balancer
   ↓
App Tier (10.0.2.0/24)
- VM Scale Set
- NSG: Allow 8080 from Web subnet only
   ↓
Database Tier (10.0.3.0/24)
- SQL Database with Private Endpoint
- NSG: Allow 1433 from App subnet only
```

### Pattern 2: Hybrid Cloud Connectivity

**Scenario:** Connect on-premises datacenter to Azure

```
On-Premises (192.168.0.0/16)
   ↓
Site-to-Site VPN or ExpressRoute
   ↓
Hub VNet (10.0.0.0/16)
- VPN/ER Gateway
- Azure Firewall
- DNS Forwarders
   ↓ (Peering)
Spoke VNets (10.1.0.0/16, 10.2.0.0/16, ...)
- Application workloads
- Private endpoints for PaaS services

DNS:
- On-prem DNS forwards *.azure.internal → Azure Private DNS Resolver
- Azure DNS forwards *.company.local → On-prem DNS
```

### Pattern 3: Isolated Environments

**Scenario:** Complete isolation between prod and non-prod

```
Production VNet (10.1.0.0/16)
- No peering to non-prod
- Separate VPN Gateway (if needed)
- Dedicated Azure Firewall
- NSG: Deny all from 10.2.0.0/16 and 10.3.0.0/16

Development VNet (10.2.0.0/16)
- Separate infrastructure
- NSG: Deny all from 10.1.0.0/16

Test VNet (10.3.0.0/16)
- Can peer to Dev (not Prod)
- NSG: Allow from Dev, Deny from Prod
```

---

## Troubleshooting Flowcharts

### Cannot Connect to VM

```
Start: Cannot RDP/SSH to VM
   ↓
Does VM have public IP?
   NO → Assign public IP or use Azure Bastion
   YES ↓
   ↓
Is VM running?
   NO → Start VM
   YES ↓
   ↓
Check NSG on NIC: Is port 3389/22 allowed?
   NO → Add allow rule
   YES ↓
   ↓
Check NSG on subnet: Is port allowed?
   NO → Add allow rule
   YES ↓
   ↓
Check source IP in NSG rules: Is your IP allowed?
   NO → Update rule to allow your IP
   YES ↓
   ↓
Is Azure Firewall blocking?
   YES → Add allow rule in Firewall
   NO ↓
   ↓
Use Network Watcher IP Flow Verify
   Result: Shows which rule is blocking
```

### VNet Peering Not Working

```
Start: VMs in peered VNets can't communicate
   ↓
Is peering status "Connected" on both sides?
   NO → Create reverse peering
   YES ↓
   ↓
Do address spaces overlap?
   YES → Redesign address spaces (cannot peer)
   NO ↓
   ↓
Check NSG on source VM: Is destination subnet allowed?
   NO → Add allow rule
   YES ↓
   ↓
Check NSG on destination VM: Is source subnet allowed?
   NO → Add allow rule
   YES ↓
   ↓
Are you trying transitive routing (A→B→C)?
   YES → Create direct peering or use UDR with Hub
   NO ↓
   ↓
Use Network Watcher Next Hop
   Result: Shows routing path
```

---

## Cost Optimization Tips

### Network Cost Optimization

**1. VNet Peering vs VPN Gateway:**

```
Regional VNet Peering: ~$0.01 per GB
VPN Gateway: ~$150/month + $0.087 per GB

If <15 TB/month → Peering cheaper
If >15 TB/month → VPN may be cheaper
```

**2. Public IP Addresses:**

```
Deallocate VMs when not in use → Release dynamic IPs (free)
Use Azure Bastion → No public IPs needed on VMs
Use NAT Gateway → Multiple VMs share single public IP
```

**3. Load Balancer SKU:**

```
Dev/test: Basic Load Balancer (free, but limited)
Production: Standard Load Balancer (~$18/month)
Don't overprovision: Start small, scale as needed
```

**4. Application Gateway:**

```
Choose right size:
- Small instances: Test environments
- Medium instances: Production
- Large instances: High traffic only

Use autoscaling: Pay only for what you use
```

**5. Data Transfer:**

```
Most expensive: Cross-region data transfer
Free: Traffic within same zone
Optimize: Keep related services in same region
```

**6. ExpressRoute:**

```
Expensive: ~$500-5000/month
Consider VPN for: Dev/test, low bandwidth needs
Use ExpressRoute for: Production, high bandwidth, compliance
```

---

## Exam Weight Breakdown - Module 4

**Virtual Networks:** ~6-8 questions

- Address space planning
- Subnet configuration
- Service endpoints vs private endpoints

**Network Security:** ~8-10 questions

- NSG rules and priority
- Application Security Groups
- Azure Firewall scenarios

**Load Balancing:** ~5-7 questions

- Choosing correct load balancer
- Configuration scenarios
- Multi-tier applications

**VNet Connectivity:** ~6-8 questions

- VNet peering
- VPN Gateway
- ExpressRoute basics

**DNS:** ~3-4 questions

- Private DNS zones
- Name resolution scenarios

**Network Monitoring:** ~2-3 questions

- Network Watcher tools
- Troubleshooting scenarios

**Total Module 4 Questions:** ~30-40 out of ~60 total exam questions

---

## Final Preparation Checklist

### Theory Mastery

- [ ] Can calculate available IPs from CIDR notation
- [ ] Understand NSG rule evaluation order
- [ ] Know when to use each load balancing solution
- [ ] Explain VNet peering (including non-transitive nature)
- [ ] Understand VPN Gateway types and scenarios
- [ ] Know DNS resolution options in Azure
- [ ] Can design hub-spoke topology
- [ ] Understand difference between service and private endpoints

### Hands-On Skills

- [ ] Create VNet with multiple subnets
- [ ] Configure NSG with multiple rules
- [ ] Set up VNet peering between regions
- [ ] Deploy and configure Load Balancer
- [ ] Create Application Gateway with URL routing
- [ ] Configure VPN Gateway (Site-to-Site)
- [ ] Set up Azure Private DNS with auto-registration
- [ ] Use Network Watcher IP Flow Verify

### Command-Line Proficiency

- [ ] Create VNet (PowerShell + CLI)
- [ ] Configure NSG rules (PowerShell + CLI)
- [ ] Create VNet peering (PowerShell + CLI)
- [ ] Deploy load balancer (PowerShell + CLI)
- [ ] Troubleshoot with Network Watcher (PowerShell + CLI)

### Scenario Practice

- [ ] Complete all 10 practice scenarios
- [ ] Can troubleshoot connectivity issues
- [ ] Can design network topology for requirements
- [ ] Understand when to use hub-spoke vs alternatives

---

## Post-Module 4: What's Next?

**Congratulations!** You've completed the largest module of the AZ-104 exam.

**Progress Check:**

- Module 1: Identity & Governance ✅ (15-20%)
- Module 2: Storage ✅ (15-20%)
- Module 3: Compute ✅ (20-25%)
- Module 4: Networking ✅ (25-30%)
- Module 5: Monitoring & Backup ⏳ (10-15%)

**You've covered ~75% of the exam content!**

**Before Module 5:**

1. Review all networking concepts
2. Practice network design scenarios
3. Master NSG rule priority
4. Understand load balancing options
5. Complete hands-on labs

**Module 5 Preview:** Monitoring and Backup

- Azure Monitor and metrics
- Log Analytics workspaces
- Application Insights
- Azure Backup
- Azure Site Recovery
- Alerts and action groups
- Cost Management

**Estimated Study Time for Module 5:** 8-10 hours

---

## Final Exam Tips for Networking Questions

**Time Management:**

- Networking questions: ~25-30 out of 60 total
- Average time per question: 1.5 minutes
- Flag difficult questions, come back later

**Question Types:**

**1. NSG Priority Questions:**

```
Read carefully: Source/destination, protocol, port
Evaluate rules in priority order (lowest first)
Stop at first match
```

**2. Load Balancer Selection:**

```
Keywords to watch:
- "HTTP/HTTPS" → Layer 7 (App Gateway/Front Door)
- "TCP/UDP" → Layer 4 (Load Balancer)
- "global" → Front Door or Traffic Manager
- "regional" → Load Balancer or App Gateway
```

**3. VNet Peering:**

```
Always check:
- Bidirectional? (must be both ways)
- Overlapping addresses? (not allowed)
- Transitive? (no, must be direct)
```

**4. Troubleshooting:**

```
Systematic approach:
1. Physical connectivity (is resource running?)
2. Network layer (IP addresses correct?)
3. NSG rules (allowed or denied?)
4. Routing (correct next hop?)
5. Firewall (additional filtering?)
```

**Common Wrong Answer Patterns:**

- "Use NSG to inspect traffic" → NSG only allows/denies, doesn't inspect
- "Peering allows A→B→C" → Non-transitive, wrong
- "256 IPs available in /24" → 251 (Azure reserves 5)
- "Basic public IP with Standard LB" → Incompatible SKUs

---

**Good luck with your AZ-104 preparation!**

Networking is the most complex module, but mastering it gives you a huge advantage. The concepts you've learned here are fundamental to Azure administration and will serve you well beyond the exam.

**Remember:**

- Practice hands-on (theory + practice = mastery)
- Use Network Watcher for troubleshooting
- Design for high availability (zones, peering, multiple regions)
- Security first (NSGs, private endpoints, Azure Firewall)
- Cost awareness (choose appropriate SKUs and services)

---

**[← Back to Module 3: Compute](../Module-03-Compute/README.md) | [Next: Module 5: Monitoring & Backup →](../Module-05-Monitoring-Backup/README.md)**
