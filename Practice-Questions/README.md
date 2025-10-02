# AZ-104 Practice Questions - Complete Collection

## Overview

This collection contains 60 practice questions covering all AZ-104 exam objectives, organized by module. Each question includes detailed explanations and references to relevant documentation.

**Question Distribution:**
- Module 1: Identity and Governance (12 questions)
- Module 2: Storage (12 questions)
- Module 3: Compute (12 questions)
- Module 4: Networking (16 questions)
- Module 5: Monitoring and Backup (8 questions)

**How to use:**
1. Answer questions without looking at solutions
2. Check your answers and read explanations
3. Review weak areas in relevant module documentation
4. Retake questions you missed after studying

**Passing score:** 42+ correct (70%)

---

## Module 1: Identity and Governance

### Question 1

Your company has an Azure subscription. You need to ensure that users can create resources only in the East US region. What should you implement?

**A)** Management group policy  
**B)** Azure Policy at subscription scope  
**C)** Resource lock  
**D)** Azure RBAC role assignment

**Answer:** B

**Explanation:**
Azure Policy at subscription scope can restrict resource creation to specific regions using the "Allowed locations" built-in policy. Management groups would work but aren't necessary for a single subscription. Resource locks prevent deletion/modification but don't restrict creation location. RBAC controls who can create resources, not where.

---

### Question 2

You have assigned the Contributor role to a user at the resource group scope. The resource group contains a storage account. You then assign the Reader role to the same user at the storage account scope. What are the user's effective permissions on the storage account?

**A)** Reader only  
**B)** Contributor only  
**C)** Both Reader and Contributor  
**D)** No permissions

**Answer:** B

**Explanation:**
RBAC permissions are cumulative and inherit down the hierarchy. The Contributor role at resource group level includes all Reader permissions and more. The explicit Reader assignment at storage account level doesn't reduce the inherited Contributor permissions. The user has Contributor access (the highest permission level).

---

### Question 3

You need to prevent accidental deletion of a production resource group. The solution must allow modifications to resources within the group. What should you apply?

**A)** CanNotDelete lock at resource group scope  
**B)** ReadOnly lock at resource group scope  
**C)** Azure Policy with Deny effect  
**D)** RBAC role removal

**Answer:** A

**Explanation:**
CanNotDelete lock prevents deletion while allowing modifications. ReadOnly lock would prevent modifications too. Azure Policy Deny would need complex rules and isn't designed for this. RBAC removal affects all operations, not just deletion.

---

### Question 4

Your company uses Azure AD. A user named User1 left the company. You deleted the user account. Two days later, you discover that User1's account is needed for auditing. What should you do?

**A)** Create a new user account with the same name  
**B)** Restore the deleted user from Azure AD recycle bin  
**C)** Contact Microsoft Support  
**D)** Recover from backup

**Answer:** B

**Explanation:**
Deleted Azure AD users are retained for 30 days in a soft-deleted state and can be restored with all properties intact. Creating a new account would result in a different ObjectId and lose all history. No need for support or backup within 30 days.

---

### Question 5

You need to delegate the ability to create VMs in a resource group without granting access to other resources. What should you assign?

**A)** Owner role at resource group scope  
**B)** Contributor role at resource group scope  
**C)** Virtual Machine Contributor role at resource group scope  
**D)** Custom role with Microsoft.Compute/virtualMachines/write

**Answer:** C

**Explanation:**
Virtual Machine Contributor is a built-in role that allows VM management without access to networking or storage. Owner and Contributor give too many permissions. A custom role would work but the built-in role is simpler and follows best practices.

---

### Question 6

Your organization has multiple Azure subscriptions. You need to apply consistent policies across all subscriptions. What should you implement?

**A)** Azure Policy at each subscription  
**B)** Management group with Azure Policy  
**C)** Resource groups with policies  
**D)** Azure Blueprints at subscription level

**Answer:** B

**Explanation:**
Management groups allow policy assignment at a level above subscriptions, automatically applying to all child subscriptions. This is more efficient than assigning policies individually. Azure Blueprints could deploy policies but management groups are the correct governance structure.

---

### Question 7

You have an Azure subscription that contains a resource group named RG1. RG1 contains 100 virtual machines. You need to ensure that when a VM is created, it must have a tag named "Environment". What should you do?

**A)** Create an Azure Policy with Append effect  
**B)** Create an Azure Policy with Deny effect  
**C)** Create a resource lock  
**D)** Use Azure Blueprints

**Answer:** B

**Explanation:**
Azure Policy with Deny effect will prevent creation of resources that don't meet the requirement (missing Environment tag). Append effect adds tags but doesn't prevent creation if missing. Resource locks don't enforce tag requirements. Blueprints could include the policy but the policy itself is what enforces the requirement.

---

### Question 8

You need to provide a user with the ability to manage access to all resources in a subscription without being able to modify the resources themselves. Which role should you assign?

**A)** Owner  
**B)** Contributor  
**C)** User Access Administrator  
**D)** Reader

**Answer:** C

**Explanation:**
User Access Administrator can manage RBAC role assignments but cannot modify resources. Owner would give too many permissions. Contributor can modify resources but not manage access. Reader cannot manage access.

---

### Question 9

Your company requires that all resources created in Azure must be tagged with a cost center. Existing resources without the tag must be automatically tagged. What should you use?

**A)** Azure Policy with Deny effect  
**B)** Azure Policy with Append effect  
**C)** Azure Policy with Modify effect  
**D)** PowerShell script

**Answer:** C

**Explanation:**
Modify effect can add tags to existing resources that don't comply. Deny only prevents new resource creation. Append adds tags during creation but doesn't remediate existing resources. PowerShell would be manual and not automated.

---

### Question 10

You have an Azure subscription that contains a resource group named RG1. You apply a ReadOnly lock to RG1. A user with Contributor role tries to delete a VM in RG1. What happens?

**A)** The VM is deleted successfully  
**B)** The deletion fails  
**C)** The user needs Owner role  
**D)** The lock is removed automatically

**Answer:** B

**Explanation:**
ReadOnly lock prevents any modifications or deletions, even by users with Contributor or Owner roles. Locks override RBAC permissions. The lock must be explicitly removed before deletion can occur.

---

### Question 11

You need to review all RBAC role assignments in your subscription from the last 30 days. What should you use?

**A)** Azure Policy compliance report  
**B)** Activity log filtered by category "Administrative"  
**C)** Azure Cost Management  
**D)** Azure Advisor

**Answer:** B

**Explanation:**
Activity log contains all management operations including RBAC role assignments. Filter by category "Administrative" and operation "Create role assignment". Azure Policy tracks compliance, not role assignments. Cost Management and Advisor don't track RBAC changes.

---

### Question 12

Your company uses Azure AD Premium P2. You need to require multi-factor authentication for users accessing Azure Portal. What should you configure?

**A)** Azure AD Conditional Access policy  
**B)** Azure Policy  
**C)** Per-user MFA settings  
**D)** Azure AD Identity Protection

**Answer:** A

**Explanation:**
Conditional Access policies (Azure AD Premium P1/P2) allow granular access control based on conditions like application (Azure Portal) with required controls (MFA). Per-user MFA is less flexible. Azure Policy doesn't control authentication. Identity Protection focuses on risk detection.

---

## Module 2: Storage

### Question 13

You have a storage account configured with LRS replication. You need to ensure that data is replicated to a secondary region. What should you do?

**A)** Enable GRS replication  
**B)** Create a second storage account with GRS  
**C)** Use AzCopy to copy data  
**D)** Enable ZRS replication

**Answer:** A

**Explanation:**
You can change replication type from LRS to GRS directly in storage account settings (no recreation needed). ZRS replicates across zones in same region, not to secondary region. Creating a second account is unnecessary. AzCopy is manual and not automatic replication.

---

### Question 14

You have a blob storage container with 1 TB of data in Hot tier. The data is accessed frequently for the first 30 days, then rarely accessed. You need to minimize storage costs. What should you implement?

**A)** Move all blobs to Cool tier immediately  
**B)** Move all blobs to Archive tier  
**C)** Implement lifecycle management policy  
**D)** Delete old blobs

**Answer:** C

**Explanation:**
Lifecycle management policy can automatically transition blobs from Hot to Cool after 30 days of inactivity, optimizing costs without manual intervention. Moving all blobs to Cool immediately would reduce performance for new blobs. Archive has retrieval delays. Deleting loses data.

---

### Question 15

You need to access Azure Files from on-premises Windows servers. The solution must use SMB protocol and integrate with Active Directory. What should you configure?

**A)** Azure File Sync  
**B)** Azure Files with AD DS authentication  
**C)** Storage account key access  
**D)** SAS token

**Answer:** B

**Explanation:**
Azure Files supports AD DS authentication for SMB access with on-premises integration. Azure File Sync is for caching but doesn't provide authentication. Storage account keys and SAS tokens don't integrate with AD.

---

### Question 16

You have a storage account that contains a blob container with archived blobs. You need to read a blob. What must you do first?

**A)** Change blob tier to Hot  
**B)** Change blob tier to Cool  
**C)** Rehydrate the blob to Hot or Cool tier  
**D)** Download with AzCopy

**Answer:** C

**Explanation:**
Archive tier blobs must be rehydrated (changed to Hot or Cool) before they can be read. This process can take up to 15 hours with standard priority. Direct reading is not possible. AzCopy cannot download archived blobs without rehydration.

---

### Question 17

You need to grant a user read-only access to a specific blob for 7 days without granting access to other blobs. What should you create?

**A)** Storage account key  
**B)** SAS token with read permission at blob level  
**C)** Azure AD role assignment  
**D)** Access policy

**Answer:** B

**Explanation:**
SAS (Shared Access Signature) token provides granular, time-limited access to specific resources. Storage account key gives full access to everything. Azure AD roles apply to entire containers/accounts. Access policies are used with SAS but SAS token is what's given to the user.

---

### Question 18

Your company requires that all data in storage accounts must be encrypted at rest with customer-managed keys. What should you configure?

**A)** Storage Service Encryption with Microsoft-managed keys  
**B)** Storage Service Encryption with customer-managed keys in Azure Key Vault  
**C)** BitLocker encryption  
**D)** Transport Layer Security

**Answer:** B

**Explanation:**
Customer-managed keys (CMK) in Azure Key Vault allow you to control encryption keys. Microsoft-managed keys are default but don't give customer control. BitLocker is for VMs, not storage accounts. TLS is for data in transit, not at rest.

---

### Question 19

You have a storage account with blob versioning enabled. A blob was accidentally overwritten. You need to restore the previous version. What should you do?

**A)** Restore from Azure Backup  
**B)** Use AzCopy to copy from another storage account  
**C)** Promote the previous version to current version  
**D)** Enable soft delete and restore

**Answer:** C

**Explanation:**
Blob versioning keeps all previous versions. You can promote any previous version to become the current version. Soft delete is for deleted blobs, not overwritten ones (though both features work together). Backup and AzCopy aren't needed when versioning is enabled.

---

### Question 20

You need to ensure that blobs in a container are automatically deleted 365 days after creation. What should you implement?

**A)** Lifecycle management policy with Delete action  
**B)** Azure Automation runbook  
**C)** Logic App scheduled task  
**D)** Soft delete with 365-day retention

**Answer:** A

**Explanation:**
Lifecycle management policy is the built-in feature for automatic blob deletion based on age. It's efficient and requires no custom scripting. Automation runbook or Logic App would require custom code and scheduled execution. Soft delete is for retention after deletion, not automatic deletion.

---

### Question 21

You have two storage accounts in different regions. You need to replicate data from StorageAccount1 to StorageAccount2 automatically. What should you use?

**A)** GRS replication  
**B)** Object replication  
**C)** AzCopy scheduled task  
**D)** Azure File Sync

**Answer:** B

**Explanation:**
Object replication (blob replication) automatically copies blobs between storage accounts asynchronously. GRS replicates within same storage account to Microsoft-managed secondary. AzCopy requires manual/scheduled runs. Azure File Sync is for files, not blobs.

---

### Question 22

You need to allow public read access to all blobs in a container without allowing anonymous users to list blobs. What should you configure?

**A)** Container public access level: Private  
**B)** Container public access level: Blob  
**C)** Container public access level: Container  
**D)** SAS token

**Answer:** B

**Explanation:**
"Blob" level allows anonymous read of blobs by direct URL but doesn't allow listing blobs in the container. "Container" level allows both reading and listing. "Private" allows no anonymous access. SAS token is not needed for public access.

---

### Question 23

You have a storage account with 10 TB of blob data. You need to copy this data to another storage account in a different region. What is the most efficient method?

**A)** Azure Portal download and upload  
**B)** AzCopy with service-to-service copy  
**C)** PowerShell with Get-AzStorageBlob and Set-AzStorageBlob  
**D)** Storage Explorer

**Answer:** B

**Explanation:**
AzCopy with service-to-service copy transfers data directly between storage accounts on Azure backbone network (fast, no egress charges). Downloading and uploading through your machine is slow and incurs bandwidth costs. PowerShell cmdlets are slower than AzCopy. Storage Explorer uses your internet connection.

---

### Question 24

Your storage account has soft delete enabled with 7-day retention. A container was deleted 5 days ago. You need to recover it. What should you do?

**A)** Restore from Azure Backup  
**B)** Recover from soft delete in portal  
**C)** Containers cannot be recovered, only blobs  
**D)** Contact Microsoft Support

**Answer:** C

**Explanation:**
Soft delete applies to blobs, not containers. When a container is deleted, the blobs inside are soft-deleted (if enabled) but the container itself must be recreated. You can then restore the soft-deleted blobs into the new container.

---

## Module 3: Compute

### Question 25

You have a VM with a B2s (burstable) SKU. Users report that the VM is slow. You check metrics and see CPU is at 100% constantly. What is the most likely cause?

**A)** VM needs more memory  
**B)** VM has exhausted CPU credits  
**C)** Disk is slow  
**D)** Network bandwidth limit reached

**Answer:** B

**Explanation:**
B-series (burstable) VMs accumulate CPU credits when idle and consume them during high usage. Once credits are exhausted, CPU is throttled to baseline performance. Solution: Use a non-burstable SKU (D-series) or increase B-series size.

---

### Question 26

You need to ensure 99.99% SLA for a VM. What should you configure?

**A)** Premium SSD disk  
**B)** Deploy VM in Availability Zone  
**C)** Deploy two VMs in Availability Set  
**D)** Use VM Scale Set

**Answer:** B

**Explanation:**
Single VM with Premium SSD = 99.9%. Availability Set = 99.95%. Availability Zone = 99.99%. VM Scale Set doesn't automatically provide 99.99% without zones.

---

### Question 27

You have a VM that is stopped (deallocated). What charges do you incur?

**A)** Compute + storage + networking  
**B)** Storage only  
**C)** No charges  
**D)** Compute only

**Answer:** B

**Explanation:**
Deallocated VMs don't incur compute charges but you still pay for attached disks (storage), reserved public IPs (if static), and any other associated resources. Stopping from inside the OS leaves VM allocated (still charged).

---

### Question 28

You need to automatically scale VMs based on CPU usage (add when >70%, remove when <30%). What should you deploy?

**A)** Multiple standalone VMs with Azure Automation  
**B)** VM Scale Set with autoscale rules  
**C)** Availability Set  
**D)** Multiple VMs with Load Balancer

**Answer:** B

**Explanation:**
VM Scale Set has built-in autoscaling based on metrics or schedules. Standalone VMs don't have native autoscale. Availability Set is for availability, not scaling. Load Balancer distributes traffic but doesn't add/remove VMs.

---

### Question 29

You need to deploy a web application to Azure with automatic updates, SSL certificates, and custom domains without managing VMs. What should you use?

**A)** Virtual Machines  
**B)** Azure App Service  
**C)** Container Instances  
**D)** Azure Functions

**Answer:** B

**Explanation:**
App Service is PaaS for web apps with built-in SSL, custom domains, deployment slots, and automatic patching. VMs require management. Container Instances lack built-in SSL/domain features. Functions are for event-driven code, not full web apps.

---

### Question 30

Your App Service plan is Standard S1 with 3 instances. You need to add a staging deployment slot. What should you do first?

**A)** Create the slot (Standard tier supports slots)  
**B)** Upgrade to Premium tier  
**C)** Add more instances  
**D)** Enable App Service Environment

**Answer:** A

**Explanation:**
Standard tier and above support deployment slots (5 slots in Standard, 20 in Premium). Basic and Free tiers don't support slots. No upgrade needed.

---

### Question 31

You have a VM in East US. You need to move it to West US. What should you do?

**A)** Use Move operation in portal  
**B)** Create snapshot, create disk in West US, create VM  
**C)** Use Azure Site Recovery  
**D)** Use AzCopy

**Answer:** B

**Explanation:**
VMs cannot be directly moved between regions. Process: snapshot disk → copy to target region → create disk from snapshot → create VM from disk. Site Recovery could also work but is designed for DR, not migration. Portal Move works only within same region.

---

### Question 32

You deploy a containerized application that must run continuously and requires 2 vCPU and 4 GB memory. Which service provides the most cost-effective solution?

**A)** Azure Container Instances  
**B)** Azure Kubernetes Service  
**C)** Azure App Service (Containers)  
**D)** Virtual Machines

**Answer:** C

**Explanation:**
For single-container web apps running continuously, App Service (Containers) is most cost-effective with built-in features. ACI bills per second but continuous running costs more than App Service. AKS has overhead costs for node VMs. VMs require more management.

---

### Question 33

You have an App Service app that experiences high traffic during business hours (8 AM - 6 PM) and low traffic after hours. You need to minimize costs while maintaining performance. What should you configure?

**A)** Manual scaling  
**B)** Autoscale with schedule-based rules  
**C)** Autoscale with metric-based rules  
**D)** Reduce to Free tier after hours

**Answer:** B

**Explanation:**
Schedule-based autoscale can scale up before business hours and down after, optimizing costs with predictable patterns. Metric-based would work but schedule is more efficient for predictable patterns. Manual requires intervention. Can't change tiers automatically.

---

### Question 34

You need to run a batch job that processes data for 2 hours daily. The job is fault-tolerant and can be interrupted. What VM option minimizes costs?

**A)** Standard VMs  
**B)** Reserved Instances  
**C)** Spot VMs  
**D)** Dedicated Hosts

**Answer:** C

**Explanation:**
Spot VMs offer up to 90% discount for interruptible, fault-tolerant workloads. Short duration (2 hours) makes Reserved Instances cost-ineffective. Standard VMs are more expensive. Dedicated Hosts are most expensive.

---

### Question 35

Your VM runs SQL Server and requires consistent IOPS of 8,000. Which disk type should you use?

**A)** Standard HDD  
**B)** Standard SSD  
**C)** Premium SSD  
**D)** Ultra Disk

**Answer:** C

**Explanation:**
Premium SSD supports up to 20,000 IOPS, sufficient for 8,000 requirement. Standard HDD (500 IOPS) and Standard SSD (6,000 IOPS) are insufficient. Ultra Disk (160,000 IOPS) is overkill and most expensive. Premium SSD is the right balance.

---

### Question 36

You have an App Service app with a deployment slot named "staging". After testing, you need to swap staging to production with zero downtime. What happens during the swap?

**A)** Production app is stopped, staging replaces it  
**B)** Slots swap instantly, no downtime  
**C)** Staging content copied to production, takes several minutes  
**D)** DNS records updated, 30-minute TTL delay

**Answer:** B

**Explanation:**
Slot swap is instant (seconds) with zero downtime. App settings are swapped, worker processes are warmed up with target settings before swap, traffic routes to swapped slots immediately. No copying or DNS changes involved.

---

## Module 4: Networking

### Question 37

You have a VNet with address space 10.0.0.0/16. You create a subnet with prefix 10.0.1.0/24. How many IP addresses are available for resources?

**A)** 256  
**B)** 254  
**C)** 251  
**D)** 250

**Answer:** C

**Explanation:**
/24 = 256 total addresses. Azure reserves 5: network (10.0.1.0), gateway (10.0.1.1), DNS (10.0.1.2-3), broadcast (10.0.1.255). Available: 256 - 5 = 251.

---

### Question 38

You have an NSG with these rules:
- Priority 100: Allow HTTP (80) from Internet
- Priority 200: Deny HTTP (80) from Internet
- Priority 300: Allow all from VNet

A request comes from Internet to port 80. What happens?

**A)** Allowed by rule 100  
**B)** Denied by rule 200  
**C)** Allowed by rule 300  
**D)** Denied by default rule

**Answer:** A

**Explanation:**
NSG evaluates rules by priority (lowest first). Request matches priority 100 (allow), evaluation stops. Priority 200 is never reached. Order matters: once matched, no further evaluation.

---

### Question 39

You create VNet peering from VNet1 to VNet2. Can resources in VNet2 communicate with VNet1?

**A)** Yes, peering is automatically bidirectional  
**B)** No, must create reverse peering from VNet2 to VNet1  
**C)** Yes, but only if gateway transit is enabled  
**D)** Yes, if address spaces don't overlap

**Answer:** B

**Explanation:**
VNet peering must be explicitly created in both directions. Creating peering from VNet1 to VNet2 only allows VNet1 → VNet2 traffic. Must create separate peering VNet2 → VNet1 for bidirectional communication.

---

### Question 40

You need to distribute HTTP traffic across VMs in multiple Azure regions with automatic failover. Which service should you use?

**A)** Azure Load Balancer  
**B)** Application Gateway  
**C)** Azure Front Door  
**D)** Traffic Manager

**Answer:** C

**Explanation:**
Azure Front Door is global Layer 7 load balancer with automatic failover across regions. Load Balancer and Application Gateway are regional only. Traffic Manager is DNS-based (not direct HTTP routing) with slower failover due to DNS TTL.

---

### Question 41

You create a VPN Gateway in a VNet. The gateway deployment fails with "GatewaySubnet not found". What should you do?

**A)** Create any subnet and deploy gateway  
**B)** Create subnet named "GatewaySubnet"  
**C)** Use existing subnet  
**D)** Contact Microsoft Support

**Answer:** B

**Explanation:**
VPN Gateway requires a subnet named exactly "GatewaySubnet" (case-sensitive). Minimum size /29 (8 IPs), recommended /27 (32 IPs). Cannot use any other name or existing subnet with different name.

---

### Question 42

Your VPN connection status shows "Not connected". You verified shared keys match. What should you check next?

**A)** NSG rules on Gateway subnet  
**B)** VPN device configuration  
**C)** Remove NSG from Gateway subnet  
**D)** Increase Gateway SKU

**Answer:** C

**Explanation:**
Gateway subnet must NOT have NSG attached. NSG on Gateway subnet will block VPN traffic. This is a common mistake. First remove any NSG, then troubleshoot device configuration if still failing.

---

### Question 43

You need to allow VMs in two peered VNets to communicate. Both VNets have NSGs. VMs still cannot communicate. What could be the cause?

**A)** Peering not configured correctly  
**B)** NSG blocking traffic from peered VNet address space  
**C)** VNets must be in same region  
**D)** Need VPN Gateway

**Answer:** B

**Explanation:**
Peering provides connectivity, but NSG can still block traffic. Common mistake: NSG rules don't explicitly allow peered VNet address space. VNets can be in different regions. VPN Gateway not needed for peering.

---

### Question 44

You have Application Gateway with path-based routing:
- /api/* → API backend pool
- /images/* → Image backend pool
- /* → Web backend pool

A request to /api/users goes to which pool?

**A)** API backend pool  
**B)** Web backend pool  
**C)** Image backend pool  
**D)** Request fails

**Answer:** A

**Explanation:**
Path-based routing matches most specific path first. /api/* matches /api/users. If no specific match, defaults to /* (web pool). Order: specific paths first, then default.

---

### Question 45

You need to ensure VMs can resolve each other by name (e.g., "web01") within and across peered VNets. What should you configure?

**A)** Azure-provided DNS (automatic)  
**B)** Azure Private DNS zone linked to VNets  
**C)** Custom DNS server  
**D)** Public DNS zone

**Answer:** B

**Explanation:**
Azure Private DNS zones with VNet links provide name resolution across peered VNets. Azure-provided DNS only works within single VNet. Private DNS zone can auto-register VM names and resolve across linked VNets.

---

### Question 46

Your company requires all outbound internet traffic from VMs be inspected. What should you implement?

**A)** NSG with deny rules  
**B)** Azure Firewall with UDR (0.0.0.0/0 → Firewall)  
**C)** Application Gateway  
**D)** Service Endpoints

**Answer:** B

**Explanation:**
Azure Firewall with User-Defined Route forces all traffic (0.0.0.0/0) through firewall for inspection. NSG allows/denies but doesn't inspect content. Application Gateway is for inbound HTTP traffic. Service Endpoints are for Azure services, not internet.

---

### Question 47

You have a Standard Load Balancer with Basic Public IP. The deployment fails. Why?

**A)** Standard LB requires Standard Public IP SKU  
**B)** Need Premium Public IP  
**C)** Load Balancer SKU mismatch  
**D)** Region not supported

**Answer:** A

**Explanation:**
SKU matching required: Basic LB with Basic public IP, Standard LB with Standard public IP. Cannot mix SKUs. Standard LB requires Standard public IP for security and zone redundancy features.

---

### Question 48

You need DNS-based failover between Azure regions for any protocol (not just HTTP). Which service should you use?

**A)** Azure Load Balancer  
**B)** Application Gateway  
**C)** Traffic Manager  
**D)** Azure Front Door

**Answer:** C

**Explanation:**
Traffic Manager provides DNS-based routing for any protocol (HTTP, HTTPS, TCP, UDP). Works by returning different IP addresses based on routing method (priority, geographic, performance). Front Door is HTTP/HTTPS only. Load Balancer and Application Gateway are regional.

---

### Question 49

You create Azure Bastion in a VNet. You try to RDP to a VM but connection fails. What should you check first?

**A)** VM has public IP  
**B)** VM's NSG allows RDP from Internet  
**C)** VM's NSG allows RDP from AzureBastionSubnet  
**D)** Bastion's NSG rules

**Answer:** C

**Explanation:**
With Bastion, VMs don't need public IPs. VM's NSG must allow RDP (3389) from AzureBastionSubnet (10.x.x.x/26). Common mistake: NSG allows RDP only from specific public IPs, not Bastion subnet.

---

### Question 50

You have Hub VNet with Azure Firewall and Spoke VNets. Spokes can communicate with Hub but not with each other. What should you configure to allow Spoke-to-Spoke communication?

**A)** Direct VNet peering between spokes  
**B)** UDR on spokes routing to Firewall (next hop)  
**C)** Enable gateway transit  
**D)** Add NSG allow rules

**Answer:** B

**Explanation:**
VNet peering is non-transitive. Spoke-to-Hub doesn't enable Spoke-to-Spoke. Solution: UDR on spoke subnets routes traffic destined for other spokes to Azure Firewall (hub) as next hop. Firewall forwards between spokes. Direct peering would work but defeats hub-spoke model.

---

### Question 51

You need to create a VPN connection that requires 1.5 Gbps bandwidth. Which VPN Gateway SKU should you use?

**A)** Basic  
**B)** VpnGw1  
**C)** VpnGw2  
**D)** VpnGw3

**Answer:** D

**Explanation:**
VPN Gateway SKUs: Basic (100 Mbps), VpnGw1 (650 Mbps), VpnGw2 (1 Gbps), VpnGw3 (1.25 Gbps), VpnGw4 (5 Gbps). For 1.5 Gbps requirement, VpnGw3 (1.25 Gbps) is insufficient. Need VpnGw4. (Note: Closest available in list is VpnGw3, but actually need VpnGw4+)

---

### Question 52

You have two VNets with overlapping address spaces (both use 10.0.0.0/16). Can you peer them?

**A)** Yes, Azure handles routing automatically  
**B)** No, address spaces cannot overlap  
**C)** Yes, with UDR  
**D)** Yes, if in different regions

**Answer:** B

**Explanation:**
VNet peering requires non-overlapping address spaces. Azure cannot route between identical address ranges. Solution: Redesign one VNet with different address space (e.g., 10.1.0.0/16) before peering. UDR and regions don't overcome this fundamental limitation.

---

## Module 5: Monitoring and Backup

### Question 53

You need to be alerted when any VM in your subscription is deleted. What type of alert should you create?

**A)** Metric alert  
**B)** Log query alert  
**C)** Activity log alert  
**D)** Application Insights alert

**Answer:** C

**Explanation:**
Activity log alert monitors subscription-level operations like resource creation/deletion. Filter by operation "Delete Virtual Machine". Metric alerts are for performance metrics. Log query alerts need Log Analytics. Application Insights is for application telemetry.

---

### Question 54

You configure a metric alert for CPU > 80%. The CPU is at 85% but you receive no notification. What should you check first?

**A)** Alert rule is enabled  
**B)** Action group is attached to alert  
**C)** Time aggregation and window size  
**D)** All of the above

**Answer:** D

**Explanation:**
Common issues: alert disabled, no action group (no notification sent), or time aggregation (e.g., Average over 10 minutes may be <80% even if current is 85%). Check all configuration aspects systematically.

---

### Question 55

Your Log Analytics workspace query returns no results. Diagnostic logs are configured. What could be the issue?

**A)** Wrong table name  
**B)** Time range too narrow  
**C)** Data takes 5-10 minutes to appear  
**D)** All of the above

**Answer:** D

**Explanation:**
Common issues: table names are case-sensitive (AzureActivity not azureactivity), time range must include when logs were generated (ago(1h) might be too narrow), logs have ingestion delay. Check all aspects.

---

### Question 56

You need to analyze application performance including request rates, dependencies, and failures. Which service should you use?

**A)** Azure Monitor metrics  
**B)** Log Analytics  
**C)** Application Insights  
**D)** Azure Advisor

**Answer:** C

**Explanation:**
Application Insights is APM (Application Performance Management) service specifically for application telemetry. Provides request tracking, dependency mapping, failure analysis, and user behavior. Azure Monitor stores metrics but doesn't provide application-specific insights.

---

### Question 57

Your first VM backup fails with "Snapshot timeout". What is the most likely cause?

**A)** VM agent not installed  
**B)** VM has high disk I/O activity  
**C)** Wrong backup policy  
**D)** Storage account issue

**Answer:** B

**Explanation:**
Snapshot timeout typically occurs when VM has heavy disk I/O, making snapshot operation slow. Solutions: schedule backup during off-peak hours, reduce I/O during backup window, use enhanced backup for multiple attempts. VM agent would give different error.

---

### Question 58

You enabled backup for a VM but want to change from LRS to GRS replication in the Recovery Services vault. What should you do?

**A)** Change setting in vault properties  
**B)** Cannot change after first backup  
**C)** Delete backups, change setting, re-enable  
**D)** Create new vault

**Answer:** B

**Explanation:**
Storage replication type (LRS/GRS/ZRS) can only be set BEFORE first backup. After backups start, it's locked. To change: must create new vault with desired replication, disable backup on old vault, enable on new vault (loses old recovery points).

---

### Question 59

You need to restore a single file deleted yesterday from a VM. What's the fastest method?

**A)** Restore entire VM to new VM  
**B)** Restore disks and attach to VM  
**C)** Use file-level restore  
**D)** Create VM from snapshot

**Answer:** C

**Explanation:**
File-level restore mounts backup as iSCSI disk, browse files, copy what's needed. Takes minutes. Restoring entire VM takes hours and requires creating new VM or replacing disks. File-level restore is specifically designed for this scenario.

---

### Question 60

Your Azure bill is unexpectedly high. You need to identify which resources are consuming the most cost. What should you use?

**A)** Azure Advisor  
**B)** Azure Monitor metrics  
**C)** Cost Management + Billing cost analysis  
**D)** Activity log

**Answer:** C

**Explanation:**
Cost Management cost analysis shows spending by service, resource, tag, region, etc. Can identify top cost contributors. Azure Advisor provides recommendations. Azure Monitor tracks performance, not cost. Activity log shows operations, not costs.

---

## Exam Simulation - Complete Practice Test

### Instructions

This is a 60-question practice exam simulating the AZ-104 certification test.

**Time limit:** 100 minutes (not enforced here)  
**Passing score:** 700/1000 (approximately 42 correct answers)  
**Format:** Single answer and multiple answer questions

**Tips:**
- Read questions carefully (watch for "NOT", "EXCEPT", "LEAST")
- Eliminate obviously wrong answers first
- Flag difficult questions, return later
- First instinct often correct

---

## Answer Sheet

Use this to track your answers:

```
Module 1 (Q1-12):   
1.__ 2.__ 3.__ 4.__ 5.__ 6.__ 7.__ 8.__ 9.__ 10.__ 11.__ 12.__

Module 2 (Q13-24):  
13.__ 14.__ 15.__ 16.__ 17.__ 18.__ 19.__ 20.__ 21.__ 22.__ 23.__ 24.__

Module 3 (Q25-36):  
25.__ 26.__ 27.__ 28.__ 29.__ 30.__ 31.__ 32.__ 33.__ 34.__ 35.__ 36.__

Module 4 (Q37-52):  
37.__ 38.__ 39.__ 40.__ 41.__ 42.__ 43.__ 44.__ 45.__ 46.__ 47.__ 48.__ 49.__ 50.__ 51.__ 52.__

Module 5 (Q53-60):  
53.__ 54.__ 55.__ 56.__ 57.__ 58.__ 59.__ 60.__
```

---

## Answer Key

**Module 1:** 1-B, 2-B, 3-A, 4-B, 5-C, 6-B, 7-B, 8-C, 9-C, 10-B, 11-B, 12-A

**Module 2:** 13-A, 14-C, 15-B, 16-C, 17-B, 18-B, 19-C, 20-A, 21-B, 22-B, 23-B, 24-C

**Module 3:** 25-B, 26-B, 27-B, 28-B, 29-B, 30-A, 31-B, 32-C, 33-B, 34-C, 35-C, 36-B

**Module 4:** 37-C, 38-A, 39-B, 40-C, 41-B, 42-C, 43-B, 44-A, 45-B, 46-B, 47-A, 48-C, 49-C, 50-B, 51-D, 52-B

**Module 5:** 53-C, 54-D, 55-D, 56-C, 57-B, 58-B, 59-C, 60-C

---

## Scoring Guide

**Calculate your score:**
- Count correct answers
- Multiply by 16.67 (to get out of 1000)
- Example: 45 correct × 16.67 = 750 points

**Score interpretation:**

| Score | Result | Recommendation |
|-------|--------|----------------|
| 700+ (42+) | Pass | Ready for exam! Review flagged topics |
| 650-699 (39-41) | Close | Focus on weak areas, retake in 1 week |
| 600-649 (36-38) | Need work | Review modules thoroughly, retake in 2 weeks |
| <600 (<36) | Not ready | Comprehensive review needed, hands-on labs |

---

## Review by Topic

Based on your results, identify weak areas:

**Identity & Governance (Q1-12):**
- If <9 correct: Review RBAC, Azure Policy, locks, Azure AD
- Key topics: Role assignments, policy effects, management groups

**Storage (Q13-24):**
- If <9 correct: Review storage account types, replication, access methods
- Key topics: SAS tokens, lifecycle management, blob tiers

**Compute (Q25-36):**
- If <9 correct: Review VM sizing, availability, App Service, containers
- Key topics: SLA requirements, VM states, scaling options

**Networking (Q37-52):**
- If <9 correct: Review VNets, NSGs, peering, load balancing
- Key topics: CIDR notation, NSG priority, peering requirements, load balancer selection

**Monitoring & Backup (Q53-60):**
- If <6 correct: Review alerts, Log Analytics, backup, cost management
- Key topics: Alert types, KQL queries, backup restore options

---

## Common Mistakes Analysis

**Top mistakes by category:**

**1. NSG Priority (Q38)**
- Mistake: Thinking all rules are evaluated
- Reality: Evaluation stops at first match
- Tip: Remember lowest priority number = highest priority

**2. VNet Peering (Q39, Q52)**
- Mistake: Assuming peering is automatic/transitive
- Reality: Must create both directions, address spaces can't overlap
- Tip: Peering is explicit and non-transitive

**3. VM Billing (Q27)**
- Mistake: Thinking stopped VM doesn't cost anything
- Reality: Must deallocate to stop compute charges
- Tip: Stop from Azure (deallocate) not from inside OS

**4. Backup Storage Replication (Q58)**
- Mistake: Thinking you can change anytime
- Reality: Locked after first backup
- Tip: Choose carefully before first backup

**5. IP Addresses in Subnet (Q37)**
- Mistake: Forgetting Azure reserves 5 IPs
- Reality: Always subtract 5 from total
- Tip: /24 = 251 usable, not 256

**6. Load Balancer Selection (Q40, Q48)**
- Mistake: Choosing Load Balancer for HTTP/global scenarios
- Reality: Different tools for different scenarios
- Tip: Global + HTTP = Front Door, Regional + L4 = Load Balancer

**7. B-Series VMs (Q25)**
- Mistake: Not understanding CPU credits
- Reality: Credits exhaust = throttled performance
- Tip: B-series for burstable workloads only

**8. Alert Not Firing (Q54)**
- Mistake: Assuming configuration is correct
- Reality: Multiple things can prevent alerts
- Tip: Check enabled, action group, thresholds, time aggregation

---

## Next Steps

**If you scored 700+ (Pass):**
1. Review any questions you missed
2. Do hands-on labs for weak topics
3. Take one more practice exam
4. Schedule your AZ-104 exam
5. Review exam day tips in Module 5

**If you scored 600-699 (Close):**
1. Identify weak modules (score by section)
2. Review those modules thoroughly
3. Do hands-on labs
4. Retake this practice exam in 1 week
5. Take additional practice exams

**If you scored <600 (Need more study):**
1. Review all modules systematically
2. Focus on understanding, not memorization
3. Complete all hands-on labs
4. Practice PowerShell/CLI commands
5. Retake exam in 2-3 weeks

---

## Additional Practice Resources

**Microsoft Official:**
- [Microsoft Learn AZ-104 Learning Path](https://learn.microsoft.com/training/courses/az-104t00)
- [Microsoft Practice Assessment](https://learn.microsoft.com/certifications/exams/az-104/practice/assessment)

**Third-Party Practice Exams:**
- MeasureUp (official Microsoft partner)
- Whizlabs
- Udemy practice tests
- Exam Topics (free questions from community)

**Hands-On Practice:**
- Azure Free Account (12 months free services)
- Azure Sandbox (Microsoft Learn)
- Your own subscription (set budget alerts!)

**Study Groups:**
- Reddit: r/AzureCertification
- Discord: Azure certification communities
- LinkedIn: Azure study groups
- Local user groups

---

## Question Breakdown by Skill

**Skill: Manage Azure identities and governance (15-20%)**
Questions: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12

**Skill: Implement and manage storage (15-20%)**
Questions: 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24

**Skill: Deploy and manage Azure compute resources (20-25%)**
Questions: 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36

**Skill: Configure and manage virtual networking (25-30%)**
Questions: 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52

**Skill: Monitor and back up Azure resources (10-15%)**
Questions: 53, 54, 55, 56, 57, 58, 59, 60

---

## Exam Day Reminders

**Before the exam:**
- Review all Key Takeaways from each module
- Review Exam Tricks sections
- Get good sleep (8 hours)
- Eat a good breakfast
- Arrive 15 minutes early (online: test environment 30 min before)

**During the exam:**
- Read each question twice
- Watch for keywords: NOT, EXCEPT, LEAST, MOST, FIRST
- Eliminate wrong answers first
- Flag difficult questions, return later
- Manage time: 60 questions in 100 minutes
- Don't overthink - trust your preparation

**Question strategies:**
- "What should you do FIRST?" → Simplest/least invasive solution
- "Minimize cost" → Smallest size, Basic tier, LRS replication
- "Maximize availability" → Zones, redundancy, multiple instances
- "Minimize administrative effort" → Use built-in features, avoid custom scripts

**After the exam:**
- Pass: Celebrate! Update LinkedIn, consider next certification
- Fail: Review score report, identify weak areas, study, retake

---

## Final Thoughts

You've completed 60 practice questions covering all AZ-104 objectives. This practice exam is designed to:

1. **Test your knowledge** across all exam domains
2. **Identify weak areas** for focused study
3. **Build confidence** with exam-like questions
4. **Practice time management** (1.5 min per question)

**Remember:**
- Practice exams predict readiness but aren't exact
- Hands-on experience is crucial
- Understanding > Memorization
- Real exam may have different question styles
- Microsoft updates exams regularly

**Success factors:**
- Complete all module reviews
- Do hands-on labs
- Take multiple practice exams
- Understand WHY answers are correct
- Stay calm and confident

**Good luck with your AZ-104 certification exam!**

You've prepared thoroughly, practiced extensively, and you're ready to succeed.

---

**[← Back to Main README](../README.md) | [Next: Cheat Sheets →](../Cheat-Sheets/README.md)**