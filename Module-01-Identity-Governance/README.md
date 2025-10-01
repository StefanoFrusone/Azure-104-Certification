# Module 1: Identity and Governance

## Table of Contents

- [Introduction](#introduction)
- [Azure Active Directory (Entra ID)](#azure-active-directory-entra-id)
- [Users and Groups](#users-and-groups)
- [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
- [Azure Policy](#azure-policy)
- [Management Groups and Subscriptions](#management-groups-and-subscriptions)
- [Resource Groups](#resource-groups)
- [Resource Tags](#resource-tags)
- [Resource Locks](#resource-locks)
- [Managed Identities](#managed-identities)
- [PowerShell Commands Reference](#powershell-commands-reference)
- [Azure CLI Commands Reference](#azure-cli-commands-reference)
- [Common Exam Scenarios](#common-exam-scenarios)

---

## Introduction

**What is this module about?**

This module covers the foundation of Azure security and organization. It answers three critical questions:

1. **Who** can access Azure resources? (Identity)
2. **What** can they do? (Access Control)
3. **How** do we organize and govern everything? (Governance)

**Real-world analogy:**
Think of Azure like a large office building:

- **Azure AD** = The security desk with ID badges
- **RBAC** = Key cards that open specific doors
- **Azure Policy** = Building rules everyone must follow
- **Management Groups** = Different floors/departments
- **Resource Groups** = Individual offices/rooms

**Exam Weight:** 15-20% of the exam

---

## Azure Active Directory (Entra ID)

### What is Azure AD?

**Simple explanation:**
Azure Active Directory is Microsoft's cloud-based identity service. It's like a digital phonebook that stores:

- User accounts
- Passwords and authentication methods
- What applications and resources users can access

**Technical definition:**
Azure AD (now called Microsoft Entra ID) is a cloud-based identity and access management service that provides authentication and authorization for cloud resources, SaaS applications, and custom applications.

### Important: Azure AD vs Windows Server AD

| Feature        | Windows Server AD              | Azure AD                         |
| -------------- | ------------------------------ | -------------------------------- |
| **Location**   | On-premises servers            | Cloud-based                      |
| **Protocol**   | LDAP, Kerberos                 | REST APIs, OAuth, SAML           |
| **Structure**  | Forests, trees, domains        | Flat structure (tenants)         |
| **Main use**   | Windows machines, on-prem apps | Cloud apps, Microsoft 365, Azure |
| **Management** | Group Policy                   | Conditional Access, Policies     |

**Can they work together?** YES! Using **Azure AD Connect** to sync identities.

### Core Concepts

#### Tenant

**Simple:** Your organization's private space in Azure AD. Like your company's section in a shared building.

**Technical:** An isolated instance of Azure AD that represents a single organization. Each tenant has:

- A unique domain name (e.g., contoso.onmicrosoft.com)
- Its own set of users, groups, and applications
- Separate security boundary

**Key facts:**

- One organization can have multiple tenants (but unusual)
- A user can be a member/guest in multiple tenants
- Resources in a subscription belong to ONE tenant

#### Directory

**Simple:** Another name for "tenant" - they're the same thing.

#### Domain

**Simple:** The web address for your organization in Azure.

**Example:**

- Initial domain: `contoso.onmicrosoft.com` (automatic, always exists)
- Custom domain: `contoso.com` (you can add your own domain)

**Technical:** A DNS namespace associated with your Azure AD tenant for user principal names (UPNs) and authentication.

#### Subscription

**Simple:** Your Azure "account" where you create resources and get billed.

**Technical:** A billing and resource container linked to an Azure AD tenant. Provides:

- Authenticated and authorized access to Azure services
- Billing boundary
- Resource organization boundary

**Relationship:**

```
Azure AD Tenant (contoso.onmicrosoft.com)
  └─ Subscription 1 (Production)
  └─ Subscription 2 (Development)
  └─ Subscription 3 (Testing)
```

**Important:**

- One tenant can have multiple subscriptions
- One subscription belongs to ONE tenant only
- You can transfer subscriptions between tenants

### Azure AD Editions

| Feature                                  | Free       | Premium P1     | Premium P2     |
| ---------------------------------------- | ---------- | -------------- | -------------- |
| **Users**                                | 500,000    | Unlimited      | Unlimited      |
| **Price**                                | Free       | ~$6/user/month | ~$9/user/month |
| **SSO**                                  | 10 apps    | Unlimited      | Unlimited      |
| **Self-service password reset**          | Cloud only | ✅             | ✅             |
| **Conditional Access**                   | ❌         | ✅             | ✅             |
| **Dynamic Groups**                       | ❌         | ✅             | ✅             |
| **Identity Protection**                  | ❌         | ❌             | ✅             |
| **Privileged Identity Management (PIM)** | ❌         | ❌             | ✅             |

**For AZ-104 exam:** Focus on Free and P1 features. P2 features (PIM, Identity Protection) are more AZ-500 topics.

---

## Users and Groups

### User Types

#### 1. Cloud Identity (Cloud-only user)

**Simple:** A user account created directly in Azure AD. It exists only in the cloud, nowhere else.

**When to use:**

- New employees in cloud-first organizations
- No on-premises infrastructure
- External contractors (though Guest is often better)

**How to create:**

- Azure Portal
- PowerShell
- Azure CLI
- Bulk import (CSV file)

**Example:**

```
Username: john.doe@contoso.onmicrosoft.com
Display Name: John Doe
Password: Initial temporary password
Location: Cloud only (no on-premises object)
```

**Characteristics:**

- Managed entirely in Azure AD
- Password resets happen in cloud
- Can use MFA, Conditional Access
- No dependency on on-premises infrastructure

#### 2. Directory-Synchronized Identity (Hybrid user)

**Simple:** A user that exists in both on-premises Active Directory AND Azure AD. They're kept in sync automatically.

**When to use:**

- Organizations with existing on-premises AD
- Hybrid cloud strategy
- Need single sign-on across on-prem and cloud

**How it works:**

1. User exists in on-premises Active Directory
2. **Azure AD Connect** syncs the user to Azure AD (every 30 minutes by default)
3. User can authenticate to both environments

**Sync methods:**

**Password Hash Synchronization (PHS):**

- Most common and simplest
- Hashed password synced to Azure AD
- Users can sign in even if on-premises is down
- **Recommended by Microsoft**

**Pass-Through Authentication (PTA):**

- Password never leaves on-premises
- Authentication requests forwarded to on-premises AD
- On-premises must be reachable
- More secure but less resilient

**Federation (ADFS):**

- Uses federation server (ADFS)
- Most complex
- Required for smart cards, third-party MFA
- Being phased out (Microsoft recommends PHS or PTA)

**Important for exam:**

- Source of authority is on-premises
- Cannot reset password in Azure portal (must reset on-prem)
- User attributes managed on-premises, synced to cloud

#### 3. Guest User (External identity)

**Simple:** Someone from outside your organization invited to collaborate. They use their own email/identity.

**When to use:**

- External partners, contractors, consultants
- Customer collaboration
- Business-to-business (B2B) scenarios

**How it works:**

1. You invite external.user@partner.com
2. They receive an email invitation
3. They accept and can access specific resources
4. They authenticate with their own organization's credentials (or Microsoft account, Google, etc.)

**Example:**

```
Email: consultant@partnerfirm.com
User Type: Guest
Source: External Azure AD
Access: Limited to specific SharePoint site
```

**Characteristics:**

- Shows as "Guest" in Azure AD
- Limited permissions by default
- Cannot see other users in directory (unless given permission)
- Can be removed easily when collaboration ends

**Important:** Guest ≠ External user account. Guest uses their own identity from their organization.

### Groups

Groups let you manage permissions for multiple users at once. Instead of giving access to 50 people individually, you give access to one group.

#### Group Types

**1. Security Group**

**Purpose:** Control access to Azure resources, applications, and other objects.

**Use cases:**

- Assign RBAC roles (e.g., "All VMs in production")
- Control access to applications
- Manage device policies
- License assignment

**Can contain:**

- Users (cloud and synced)
- Devices
- Service principals
- Other groups (nested groups)

**Example:**

```
Group: IT-Administrators
Members:
  - john.doe@contoso.com
  - jane.smith@contoso.com
  - DevOps-Team (nested group)
Purpose: Full access to Production subscription
```

**2. Microsoft 365 Group**

**Purpose:** Collaboration - provides shared resources for teamwork.

**What you get:**

- Shared mailbox
- Shared calendar
- SharePoint site
- OneNote notebook
- Planner
- Teams (if enabled)

**Can contain:**

- Users only (no devices, no nested groups)

**Use cases:**

- Project teams
- Department collaboration
- Email distribution with shared resources

**Example:**

```
Group: Marketing-Team
Members: Marketing department employees
Resources:
  - marketing@contoso.com (shared mailbox)
  - Shared calendar
  - Marketing SharePoint site
  - Microsoft Teams channel
```

**Important difference:**

- Security groups = Access control
- M365 groups = Collaboration tools + access control

**Can a group be both?** No, but M365 groups can be used for security purposes.

#### Membership Types

**1. Assigned (Static Membership)**

**Simple:** You manually add and remove members.

**How it works:**

- Administrator adds users one-by-one
- Users stay in group until manually removed
- Full control over membership

**When to use:**

- Small teams (5-20 people)
- Special access groups
- When you need precise control
- Executive teams, project-specific groups

**Example:**

```
Group: Payroll-Access
Membership: Assigned
Members:
  - HR Manager
  - CFO
  - Payroll Specialist
Reason: Only specific people should have access
```

**Pros:**

- ✅ Simple and predictable
- ✅ Full control
- ✅ Works for small groups

**Cons:**

- ❌ Manual maintenance required
- ❌ Doesn't scale for large organizations
- ❌ Easy to forget to remove people (access creep)

**2. Dynamic User**

**Simple:** Azure automatically adds/removes users based on rules you define. Users are added when they match the rule and removed when they don't.

**How it works:**

1. You create a rule based on user attributes
2. Azure evaluates all users against the rule
3. Users matching the rule are automatically added
4. When attributes change, membership updates automatically

**When to use:**

- Large organizations
- Access based on job role/department
- Automatic onboarding/offboarding
- Reducing administrative overhead

**Rule examples:**

**Example 1: All Marketing employees**

```
user.department -eq "Marketing"
```

**Example 2: All developers in Milan**

```
(user.jobTitle -eq "Developer") -and (user.city -eq "Milan")
```

**Example 3: Multiple conditions**

```
(user.department -eq "IT") -and
(user.accountEnabled -eq true) -and
(user.country -eq "Italy")
```

**Available attributes:**

- department
- jobTitle
- city
- country
- accountEnabled
- userType
- employeeId
- And many more...

**Operators:**

- `-eq` (equals)
- `-ne` (not equals)
- `-startsWith`
- `-contains`
- `-match` (regex)
- `-and`, `-or`, `-not` (logical operators)

**Important for exam:**

- Requires Azure AD Premium P1 license
- Rule changes take time to process (not instant)
- Users are automatically removed when they no longer match
- Cannot manually add/remove members (rule-based only)

**Real-world scenario:**

```
Problem:
New marketing employees need access to Marketing SharePoint,
marketing@contoso.com mailbox, and Marketing Teams channel.

Solution:
Create dynamic M365 group: Marketing-Team
Rule: user.department -eq "Marketing"
Result: New marketing hire automatically gets all access on day 1
When they leave/transfer, access automatically removed
```

**3. Dynamic Device**

**Simple:** Same as dynamic user, but for devices instead of users.

**When to use:**

- Managing groups of similar devices
- Applying policies to device types
- Windows Autopilot scenarios

**Example:**

```
Rule: device.deviceOSType -eq "Windows"
Result: All Windows devices automatically in this group
```

**For AZ-104:** Focus mainly on user groups. Device groups are less tested.

### Administrative Units (AU)

**Simple:** A way to divide your Azure AD into smaller sections for delegated administration.

**Real-world analogy:** In a large university:

- IT Admin can manage everything
- "Engineering Dept AU Admin" can only manage engineering department users
- "Medical School AU Admin" can only manage medical school users

**When to use:**

- Large organizations (1000+ users)
- Delegated administration (regional admins, department admins)
- Restricting admin scope for security/compliance

**Example structure:**

```
Contoso Organization
  ├─ AU: Europe
  │   └─ AU Admin: Can only manage European users
  ├─ AU: North America
  │   └─ AU Admin: Can only manage North American users
  └─ AU: IT Department
      └─ AU Admin: Can only manage IT users
```

**Important for exam:**

- Requires Azure AD Premium P1
- AU admins cannot see/manage users outside their AU
- Good for compliance (data residency, regional laws)

---

## Role-Based Access Control (RBAC)

RBAC is THE most important topic in this module. It controls who can do what in Azure.

### The Three Components

Think of RBAC as a sentence with three parts:

**[WHO] can [DO WHAT] on [WHERE]**

1. **Security Principal (WHO)** - The identity getting access
2. **Role Definition (DO WHAT)** - The permissions granted
3. **Scope (WHERE)** - The resources they can access

### 1. Security Principal (WHO)

**User:**

- Individual person with Azure AD account
- Example: john.doe@contoso.com

**Group:**

- Collection of users
- **Best practice:** Always assign roles to groups, not individual users
- Why? Easier to manage. Add/remove users from group instead of reassigning roles.

**Service Principal:**

- Identity for applications, services, or automation tools
- Like a "robot user" for non-human entities
- Example: Your CI/CD pipeline needs to deploy resources

**Managed Identity:**

- Special type of service principal automatically managed by Azure
- No passwords to maintain
- Two types:
  - **System-assigned:** Created with resource, dies with resource
  - **User-assigned:** Created independently, can be assigned to multiple resources

**Example:**

```
User: john.doe@contoso.com (human)
Group: DevOps-Team (collection of humans)
Service Principal: GitHub-Actions-Deploy (automation tool)
Managed Identity: MyWebApp-Identity (the web app itself)
```

### 2. Role Definition (DO WHAT)

A role is a collection of permissions. Each permission is an "action" you can perform.

**Permission format:** `{provider}/{resource}/{action}`

**Examples:**

```
Microsoft.Compute/virtualMachines/read       (can view VMs)
Microsoft.Compute/virtualMachines/write      (can create/modify VMs)
Microsoft.Compute/virtualMachines/delete     (can delete VMs)
Microsoft.Compute/virtualMachines/*          (can do everything with VMs)
Microsoft.Storage/storageAccounts/read       (can view storage accounts)
```

**Wildcard actions:**

- `*` = All actions
- `*/read` = All read actions
- `Microsoft.Compute/*` = All compute actions

### Built-in Roles

Azure has 100+ built-in roles. Here are the most important for AZ-104:

#### Fundamental Roles (Know these COLD!)

**Owner**

- **Simple:** God mode - can do absolutely everything
- **Permissions:**
  - All actions (`*`)
  - Can manage access (grant roles to others)
- **Use case:** Subscription administrators, resource group owners
- **Warning:** Very powerful - use sparingly!

```
Can do:
✅ Create, modify, delete resources
✅ Grant access to others
✅ Change security settings
✅ View costs and billing
✅ Everything else
```

**Contributor**

- **Simple:** Can manage all resources but CANNOT give access to others
- **Permissions:**
  - All actions (`*`) EXCEPT role assignments
- **Use case:** Developers, DevOps engineers, resource managers
- **Key difference from Owner:** Cannot use RBAC (can't grant permissions)

```
Can do:
✅ Create, modify, delete resources
✅ Change configurations
✅ View everything

Cannot do:
❌ Grant access to others
❌ Assign roles
❌ Modify RBAC settings
```

**Reader**

- **Simple:** View-only access - cannot change anything
- **Permissions:**
  - `*/read` only
- **Use case:** Auditors, managers, read-only monitoring
- **Important:** Can see resource configurations, keys, connection strings (be careful!)

```
Can do:
✅ View all resources
✅ View configurations
✅ View metrics and logs

Cannot do:
❌ Create anything
❌ Modify anything
❌ Delete anything
```

**User Access Administrator**

- **Simple:** Can ONLY manage access (RBAC), cannot touch resources
- **Permissions:**
  - `*/read` (can view everything)
  - `Microsoft.Authorization/*` (can manage access)
- **Use case:** Security team, identity administrators
- **Unusual:** Can grant access but cannot use the resources themselves

```
Can do:
✅ Assign roles to users/groups
✅ View role assignments
✅ View all resources

Cannot do:
❌ Create resources
❌ Modify resources
❌ Delete resources
```

#### Common Resource-Specific Roles

**Virtual Machine Contributor**

- Manage VMs but not the network or storage they use
- Cannot assign roles

**Network Contributor**

- Manage networks (VNets, NSGs, Load Balancers, etc.)
- Cannot assign roles

**Storage Account Contributor**

- Manage storage accounts
- Cannot access data inside (need separate data-plane role)

**SQL DB Contributor**

- Manage SQL databases
- Cannot access data inside (need separate role for querying)

**Important concept:**

- **Management plane** = Creating/deleting/configuring resources (RBAC controls this)
- **Data plane** = Accessing data inside resources (separate permissions)

**Example:**

```
User has: Storage Account Contributor

Can do:
✅ Create storage account
✅ Delete storage account
✅ Change configuration

Cannot do:
❌ Upload blobs
❌ Download files
❌ Read data

Need additional role: Storage Blob Data Contributor (data-plane role)
```

### 3. Scope (WHERE)

Scope defines WHERE the permissions apply. It's hierarchical.

```
Management Group
  └─ Subscription
      └─ Resource Group
          └─ Resource (individual VM, storage account, etc.)
```

**Inheritance:** Permissions flow downward (from parent to child).

**Example:**

```
If you're Owner on Subscription:
  → You're Owner on all Resource Groups in that subscription
  → You're Owner on all Resources in those resource groups
```

#### Scope Levels Explained

**1. Management Group Scope**

- **Simple:** Highest level - can contain multiple subscriptions
- **Use case:** Enterprise-wide policies, C-level access
- **Example:** CTO is Owner on root management group = Owner of everything

**2. Subscription Scope**

- **Simple:** One billing account
- **Use case:** Subscription admins, environment separation
- **Example:** "Production subscription" vs "Development subscription"

**3. Resource Group Scope**

- **Simple:** Logical container for resources
- **Use case:** Project teams, application-specific access
- **Example:** "Web-App-RG" resource group for web application resources
- **Most common scope for AZ-104**

**4. Resource Scope**

- **Simple:** Individual resource (one VM, one storage account)
- **Use case:** Very specific access, least privilege
- **Example:** DBA has access to only one specific SQL database

#### Scope Examples

**Scenario 1: Developer Access**

```
Security Principal: DevOps-Team (group)
Role: Contributor
Scope: Resource Group "RG-Development"

Result:
DevOps team can create/modify/delete any resource
in RG-Development, but nothing outside that RG.
```

**Scenario 2: Read-Only Auditor**

```
Security Principal: auditor@contoso.com
Role: Reader
Scope: Subscription "Production"

Result:
Auditor can view everything in Production subscription,
but cannot make any changes.
```

**Scenario 3: Specific VM Access**

```
Security Principal: backup-service (service principal)
Role: Virtual Machine Contributor
Scope: VM "web-server-01"

Result:
Backup service can manage only that specific VM,
not other VMs or other resources.
```

**Scenario 4: Inherited Access**

```
Security Principal: john.doe@contoso.com
Role: Owner
Scope: Subscription "Production"

Result:
John is automatically Owner of:
- All resource groups in Production subscription
- All resources in those resource groups
(Inheritance flows down)
```

### Multiple Role Assignments

**Can someone have multiple roles?** YES!

**How are permissions combined?**

- **Additive model:** Permissions are ADDED together (union)
- If any role grants a permission, you have it
- There's no "deny" in RBAC (only "not granted")

**Example:**

```
User has two roles:
1. Reader on Subscription (can view everything)
2. Contributor on Resource Group "RG-Web" (can manage resources in RG-Web)

Effective permissions:
- Can VIEW everything in the subscription (from Reader)
- Can MANAGE resources in RG-Web (from Contributor)
- Cannot manage resources outside RG-Web
```

**Important for exam:**

```
User A: Reader on Subscription
User A: Contributor on Resource Group "RG-App"

Question: Can User A create a VM in RG-App?
Answer: YES (Contributor role grants this)

Question: Can User A create a VM in RG-Database?
Answer: NO (only has Reader there)
```

### Deny Assignments

**Special case:** Azure Blueprints can create "Deny Assignments" that override allow permissions.

**How it works:**

- Deny assignments are evaluated BEFORE allow assignments
- Even Owner role can be blocked by deny assignment
- Used to protect critical resources

**Example:**

```
Production resource group has deny assignment:
"Nobody can delete resources in this RG"

Even if you're Owner, you cannot delete resources.
```

**For AZ-104:** Know deny assignments exist, but they're not heavily tested. Focus on standard RBAC.

### Custom Roles

**When built-in roles aren't enough**, create a custom role.

**Structure of a role definition:**

```json
{
  "Name": "Custom VM Operator",
  "Description": "Can start and stop VMs but not create/delete",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/powerOff/action"
  ],
  "NotActions": [],
  "AssignableScopes": ["/subscriptions/{subscription-id}"]
}
```

**Components:**

- **Actions:** What you CAN do
- **NotActions:** Exclude specific actions from Actions (like exceptions)
- **DataActions:** Permissions for data plane (blob, queue operations)
- **NotDataActions:** Exclude specific data actions
- **AssignableScopes:** Where this role can be assigned

**Example use case:**

```
Need: Junior admin who can start/stop VMs for maintenance,
      but cannot create new VMs or delete existing ones

Solution: Custom role with limited actions:
- read (view VMs)
- start
- restart
- powerOff

Does NOT include:
- write (create/modify)
- delete
```

**Important:**

- Custom roles require specific permissions to create (Owner or User Access Administrator)
- Can be assigned at any scope like built-in roles
- Maximum 5000 custom roles per directory

### RBAC Best Practices (Important for exam!)

1. **Use groups, not individual users**

   - Easier management
   - Less role assignments to track
   - Cleaner audit logs

2. **Principle of least privilege**

   - Grant minimum permissions needed
   - Start with Reader, add more only if needed
   - Use resource-specific roles instead of Contributor

3. **Use built-in roles when possible**

   - Microsoft maintains them
   - Automatically updated with new services
   - Well-tested and documented

4. **Scope at resource group level**

   - More granular than subscription
   - Aligns with application/project boundaries
   - Easier to remove access (delete RG)

5. **Document custom roles**

   - Explain why it exists
   - Review periodically
   - Remove if no longer needed

6. **Regular access reviews**
   - Remove users who left
   - Remove excessive permissions
   - Use Azure AD access reviews (P2 feature)

### Common RBAC Scenarios (Exam favorites!)

**Scenario 1: Developer needs to deploy web app**

```
Role: Contributor
Scope: Resource Group "RG-WebApp-Dev"
Why: Can create/modify resources but not affect other resource groups
```

**Scenario 2: DBA needs SQL access but shouldn't create VMs**

```
Role: SQL DB Contributor
Scope: Resource Group "RG-Database"
Why: Limited to SQL, can't touch other resources
```

**Scenario 3: External auditor needs read-only access**

```
Role: Reader
Scope: Subscription "Production"
Why: Can view everything for audit, can't change anything
```

**Scenario 4: Backup service needs VM access**

```
Option 1: Create service principal, assign Virtual Machine Contributor
Option 2: Use managed identity for VM, assign Storage Blob Data Contributor
Why: Automated, no password management needed
```

**Scenario 5: User can see VMs but cannot start them**

```
Problem: User has Reader role
Solution: Assign "Virtual Machine Contributor" role
Or: Create custom role with start/stop actions only
```

---

## Azure Policy

Azure Policy enforces organizational rules and standards across your Azure environment.

### What is Azure Policy?

**Simple:** Automated rules that Azure checks and enforces. Like a company rulebook that's automatically enforced.

**Technical:** A service that evaluates resource properties against business rules. Policies can audit, deny, modify, or deploy resources based on defined conditions.

**Real-world analogy:**

```
Company rule: "All company cars must have insurance"
Azure Policy: "All VMs must have backup enabled"

Without Policy: Trust people to remember
With Policy: Azure automatically checks and enforces
```

### Why Use Azure Policy?

**Problems it solves:**

- ❌ Someone creates expensive VMs in wrong region
- ❌ Resources don't have proper tags for cost tracking
- ❌ Storage accounts exposed to public internet
- ❌ VMs without backup
- ❌ Non-compliant configurations

**With Azure Policy:**

- ✅ Automatically prevent non-compliant resource creation
- ✅ Auto-tag resources
- ✅ Audit existing resources for compliance
- ✅ Auto-remediate non-compliant resources

### Core Concepts

#### Policy Definition

**Simple:** A single rule describing what should or shouldn't happen.

**Structure:**

```json
{
  "if": {
    "condition": "What to check"
  },
  "then": {
    "effect": "What to do about it"
  }
}
```

**Example Policy:** "Allowed virtual machine sizes"

```json
{
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.Compute/virtualMachines"
      },
      {
        "not": {
          "field": "Microsoft.Compute/virtualMachines/sku.name",
          "in": ["Standard_B2s", "Standard_D2s_v3"]
        }
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

**Translation:** If someone tries to create a VM that's NOT B2s or D2s_v3, deny the creation.

#### Policy Assignment

**Simple:** Applying a policy to a specific scope (where it should be enforced).

**Example:**

```
Policy Definition: "Require tag 'CostCenter' on resources"
Policy Assignment: Applied to Resource Group "RG-Production"

Result: Every resource created in RG-Production must have CostCenter tag
```

**Assignment includes:**

- Which policy to enforce
- Where to enforce it (scope)
- Parameters (if policy has configurable values)
- Exclusions (specific resources to skip)

#### Initiative (Policy Set)

**Simple:** A group of related policies bundled together.

**Why?** Instead of assigning 20 policies one-by-one, assign one initiative.

**Example Initiative:** "ISO 27001 Compliance"
Contains:

- Require encryption for storage accounts
- Require HTTPS for web apps
- Require MFA for admin accounts
- Audit public IP usage
- And 50+ other related policies

**Built-in initiatives:**

- CIS Microsoft Azure Foundations Benchmark
- Azure Security Benchmark
- HIPAA HITRUST
- PCI DSS v3.2.1
- NIST SP 800-53

**Custom initiatives:**

- Create your own bundles
- Mix built-in and custom policies

### Policy Effects

Effects determine what happens when a policy is triggered.

#### 1. Deny

**Simple:** Blocks the action from happening.

**Use case:** Prevent non-compliant resources from being created.

**Example:**

```
Policy: "Storage accounts must use HTTPS only"
Effect: Deny
Result: If someone tries to create storage account without HTTPS,
        Azure shows error and blocks creation
```

**Important:**

- Applies only to NEW resources or UPDATES
- Does NOT affect existing resources
- User gets clear error message explaining why

#### 2. Audit

**Simple:** Allows the action but logs it as non-compliant.

**Use case:** Discover what's non-compliant without blocking anyone.

**Example:**

```
Policy: "VMs should have backup enabled"
Effect: Audit
Result: VM creation is allowed, but compliance state shows as "Non-compliant"
        in Azure Policy dashboard
```

**Perfect for:**

- Initial policy rollout (learn before you enforce)
- Reporting requirements
- Compliance audits

#### 3. AuditIfNotExists

**Simple:** Check if a related resource exists; if not, mark as non-compliant.

**Use case:** Verify dependent resources exist.

**Example:**

```
Policy: "VMs should have Microsoft Antimalware extension"
Effect: AuditIfNotExists
Checks: Does VM have the antimalware extension?
Result: If extension missing → mark VM as non-compliant
        If extension exists → mark as compliant
```

**Does NOT:**

- Block VM creation
- Automatically install extension
- Just audits/reports

#### 4. DeployIfNotExists (DINE)

**Simple:** If a required resource doesn't exist, automatically create it.

**Use case:** Auto-remediation - automatically fix non-compliance.

**Example:**

```
Policy: "Deploy diagnostic settings for VMs"
Effect: DeployIfNotExists
Result: When VM is created without diagnostic settings,
        Azure automatically creates and configures them
```

**Powerful because:**

- Fully automated compliance
- No manual intervention needed
- Consistent configurations

**Requires:**

- Managed identity for the policy assignment (to have permissions to create resources)
- User to explicitly trigger remediation OR auto-remediation enabled

#### 5. Modify

**Simple:** Automatically change resource properties to make them compliant.

**Use case:** Auto-fix specific properties (mainly tags and identity settings).

**Example:**

```
Policy: "Add CostCenter tag if missing"
Effect: Modify
Result: When resource created without CostCenter tag,
        Azure automatically adds it with default value
```

**Common uses:**

- Adding missing tags
- Enabling system-assigned managed identity
- Modifying specific settings

**Difference from DINE:**

- Modify = Change existing resource properties
- DINE = Create new related resources

#### 6. Append

**Simple:** Add additional settings to a resource during creation.

**Use case:** Enforce additional configurations.

**Example:**

```
Policy: "Append NSG rule to subnets"
Effect: Append
Result: When subnet is created, Azure adds required NSG rules
```

**For AZ-104:** Focus on Deny, Audit, and DeployIfNotExists. These are most tested.

### Policy Evaluation

**When are policies evaluated?**

1. **Resource creation** - Before the resource is created
2. **Resource update** - When you modify a resource
3. **Policy assignment** - When you first assign a policy (evaluates existing resources)
4. **Scheduled evaluation** - Every 24 hours (checks existing resources)

**Evaluation order:**

1. Disabled policies are skipped
2. Append/Modify effects processed first
3. Then Deny is evaluated (can block action)
4. Then Audit/AuditIfNotExists (for reporting)
5. Finally DeployIfNotExists (for remediation)

**Important timing:**

- New policy assignments take ~30 minutes to take effect
- Compliance results update every 24 hours
- Manual compliance scan can be triggered anytime

### Policy Parameters

**Simple:** Variables in policy definitions that you set when assigning.

**Why?** Makes policies reusable with different values.

**Example without parameters:**

```json
{
  "field": "location",
  "equals": "eastus"
}
```

Hard-coded to East US only.

**Example with parameters:**

```json
{
  "field": "location",
  "in": "[parameters('allowedLocations')]"
}
```

**When assigning:**

```json
{
  "allowedLocations": {
    "value": ["eastus", "westeurope", "northeurope"]
  }
}
```

**Benefits:**

- One policy definition, multiple assignments with different values
- Easier maintenance
- More flexible

### Compliance State

After policy evaluation, resources have compliance states:

| State              | Meaning                                         |
| ------------------ | ----------------------------------------------- |
| **Compliant**      | Resource meets policy requirements              |
| **Non-compliant**  | Resource violates policy                        |
| **Conflicting**    | Multiple policies with conflicting requirements |
| **Not started**    | Policy hasn't evaluated yet                     |
| **Not registered** | Resource provider not registered                |
| **Exempt**         | Resource explicitly exempted from policy        |

**Viewing compliance:**

- Azure Portal → Policy → Compliance
- See overall compliance percentage
- Drill down to non-compliant resources
- Export compliance reports

### Remediation

**Problem:** You assign a policy, but existing resources are already non-compliant. Policy only affects NEW resources.

**Solution:** Remediation tasks automatically fix existing resources.

**How it works:**

1. Assign policy with DeployIfNotExists or Modify effect
2. Run compliance scan (identifies non-compliant resources)
3. Create remediation task
4. Azure applies the policy to existing resources

**Example:**

```
1. Assign policy: "VMs should have backup enabled"
2. 50 existing VMs don't have backup (non-compliant)
3. Create remediation task
4. Azure enables backup on all 50 VMs
5. Resources become compliant
```

**Requires:**

- Managed identity for policy assignment
- Appropriate permissions for the managed identity
- Manual trigger OR auto-remediation enabled

**Auto-remediation:**

- Can be enabled during policy assignment
- Automatically creates remediation tasks for non-compliant resources
- No manual intervention needed

### Policy Scope and Inheritance

Policies flow downward (like RBAC):

```
Management Group
  └─ Subscription
      └─ Resource Group
          └─ Resource
```

**Example:**

```
Policy at Management Group level:
"All resources must have tag 'Environment'"

Effect:
- Applies to all subscriptions under that management group
- Applies to all resource groups in those subscriptions
- Applies to all resources in those resource groups
```

**Exclusions:**
You can exclude specific scopes from policy:

```
Policy: Applied to Subscription "Production"
Exclusion: Resource Group "RG-Legacy-App"

Result: Policy applies everywhere EXCEPT RG-Legacy-App
```

**Use cases for exclusions:**

- Legacy applications that can't be modified
- Temporary exceptions during migration
- Resources with special requirements

### Common Built-in Policies (Exam favorites!)

**1. Allowed locations**

```
Purpose: Restrict which Azure regions can be used
Effect: Deny
Why: Data residency, compliance, cost control
```

**2. Allowed virtual machine size SKUs**

```
Purpose: Prevent expensive VM sizes
Effect: Deny
Why: Cost control
```

**3. Require a tag on resources**

```
Purpose: Ensure all resources have specific tags
Effect: Deny
Why: Cost tracking, organization
```

**4. Inherit tag from resource group if missing**

```
Purpose: Auto-copy tags from RG to resources
Effect: Modify
Why: Consistent tagging without manual work
```

**5. Storage accounts should use private link**

```
Purpose: Ensure storage accounts use private endpoints
Effect: Audit
Why: Security compliance
```

**6. Deploy network watcher when virtual networks are created**

```
Purpose: Auto-enable network monitoring
Effect: DeployIfNotExists
Why: Automatic compliance
```

### Policy vs RBAC

**Common confusion:** What's the difference?

| Azure Policy                        | RBAC                             |
| ----------------------------------- | -------------------------------- |
| What resources SHOULD look like     | WHO can do WHAT                  |
| "All VMs must be in East US"        | "John can create VMs"            |
| Applies to properties               | Applies to actions               |
| Compliance and governance           | Access control                   |
| Can prevent non-compliant resources | Can prevent unauthorized actions |

**Example:**

```
John has "Contributor" role on RG-Production (RBAC)
Policy: "Only Standard_B2s VMs allowed" (Policy)

John tries to create Standard_D16s_v3 VM:
- RBAC says: "John is allowed to create VMs" ✅
- Policy says: "This VM size is not allowed" ❌
- Result: Creation blocked by policy
```

**They work together:**

- RBAC: Grants permissions
- Policy: Enforces rules on what's allowed even with permissions

---

## Management Groups and Subscriptions

### Management Groups

**Simple:** Containers that hold multiple subscriptions, allowing you to organize and apply governance at scale.

**Real-world analogy:**

```
Company (Root Management Group)
  ├─ Division A (Management Group)
  │   ├─ Subscription: Production
  │   └─ Subscription: Development
  └─ Division B (Management Group)
      ├─ Subscription: Production
      └─ Subscription: Development
```

**Why use management groups?**

- Apply policies to multiple subscriptions at once
- Assign RBAC roles across multiple subscriptions
- Organize subscriptions hierarchically (mirror company structure)
- Enforce compliance across the organization

### Management Group Hierarchy

**Structure:**

```
Root Management Group (automatic, one per tenant)
  └─ Management Group (up to 6 levels deep)
      └─ Management Group
          └─ Subscription
              └─ Resource Group
                  └─ Resource
```

**Rules:**

- Maximum 6 levels of depth (not counting root)
- Root management group cannot be deleted or moved
- Each child can have only one parent
- Each subscription can be in only one management group

**Example enterprise structure:**

```
Contoso (Root)
  ├─ Production (MG)
  │   ├─ North America (MG)
  │   │   ├─ Subscription: NA-Prod-1
  │   │   └─ Subscription: NA-Prod-2
  │   └─ Europe (MG)
  │       ├─ Subscription: EU-Prod-1
  │       └─ Subscription: EU-Prod-2
  └─ Non-Production (MG)
      ├─ Development (MG)
      │   └─ Subscription: Dev-Sub
      └─ Testing (MG)
          └─ Subscription: Test-Sub
```

**Inheritance:**

- Policies and RBAC assigned at MG level flow down to all children
- You cannot override a parent's deny policy (except with exemptions)

**Use cases:**

- Separate production from non-production
- Organize by geography
- Organize by department/business unit
- Apply different compliance standards (HIPAA, PCI-DSS, etc.)

### Subscriptions

**Simple:** A billing container and boundary for Azure resources.

**What a subscription provides:**

- Billing boundary (one bill per subscription)
- Access control boundary (RBAC at subscription level)
- Quota limits (resources per subscription)
- API rate limits

**Key facts:**

- Linked to one Azure AD tenant
- Can have multiple subscriptions per tenant
- Cannot merge subscriptions
- Can transfer subscriptions between tenants

**Subscription types:**

**Free Trial:**

- $200 credit for 30 days
- Popular services free for 12 months
- Always-free services
- Converts to pay-as-you-go after trial

**Pay-As-You-Go:**

- Standard subscription
- Pay for what you use
- No upfront commitment
- Most common for production

**Enterprise Agreement (EA):**

- Large organizations
- Negotiated pricing
- Minimum commitment
- Volume discounts

**CSP (Cloud Solution Provider):**

- Purchased through Microsoft partners
- Partner manages billing
- May include additional support/services

**Student:**

- $100 credit for students
- No credit card required
- Popular services free

### Subscription Limits

**Important limits to know:**

| Resource            | Limit per Subscription |
| ------------------- | ---------------------- |
| Resource Groups     | 980                    |
| VMs per region      | 25,000                 |
| VNets               | 1,000                  |
| Storage Accounts    | 250                    |
| Public IP addresses | 1,000                  |

**Note:** These are soft limits - you can request increases via support ticket.

**Why multiple subscriptions?**

**Billing separation:**

```
Subscription: Production (billed to Ops dept)
Subscription: Development (billed to Dev dept)
```

**Environment separation:**

```
Subscription: Prod (strict policies, limited access)
Subscription: Dev (relaxed policies, broad access)
```

**Quota requirements:**

```
Project needs 30,000 VMs
Solution: 2+ subscriptions (25K VMs per subscription limit)
```

**Organizational boundaries:**

```
Subscription per business unit/geography/project
Each with own admins and policies
```

### Moving Subscriptions

**You can:**

- Move subscription to different management group
- Transfer subscription to different Azure AD tenant
- Change subscription offer (e.g., trial → pay-as-you-go)

**Important during tenant transfer:**

- All RBAC assignments are LOST (must be recreated)
- Resources remain intact
- Downtime may occur
- Some resources don't support transfer (check docs)

---

## Resource Groups

**Simple:** Logical containers for grouping related Azure resources.

**Real-world analogy:** A folder on your computer that contains related files.

### What is a Resource Group?

**Technical:** A container that holds related resources for an Azure solution. Resources in a group share the same lifecycle, permissions, and policies.

**Key characteristics:**

- Every resource must be in exactly ONE resource group
- Resources can be in different regions than their resource group
- Resources can be moved between resource groups (with some exceptions)
- Deleting a resource group deletes all resources inside

**Example:**

```
Resource Group: RG-WebApp-Production
Contains:
  ├─ Virtual Machine (web server)
  ├─ Virtual Network
  ├─ Network Security Group
  ├─ Public IP address
  ├─ Storage Account (for logs)
  └─ SQL Database
```

### Organizing Resource Groups

**Common strategies:**

**1. By Application/Workload:**

```
RG-EcommerceWebsite
RG-CRMSystem
RG-DataWarehouse
```

**2. By Environment:**

```
RG-Production
RG-Staging
RG-Development
```

**3. By Lifecycle:**

```
RG-PermanentInfrastructure (VNet, etc.)
RG-TemporaryTestResources (can delete after test)
```

**4. By Resource Type (NOT recommended):**

```
RG-AllVirtualMachines
RG-AllStorageAccounts
(Hard to manage, doesn't reflect relationships)
```

**Best practice:** Group resources that share the same lifecycle.

**Example:**

```
Good: RG-ProjectAlpha
  - All resources for Project Alpha
  - When project ends, delete entire RG

Bad: RG-AllCompanyVMs
  - Mixed projects
  - Can't easily clean up
```

### Resource Group Properties

**Location:**

- Resource group has a location (region)
- Stores metadata about the resources
- Resources inside can be in different regions

**Example:**

```
Resource Group: Located in East US (metadata)
  ├─ VM in West Europe
  ├─ Storage in Southeast Asia
  └─ SQL DB in North Europe
```

**Tags:**

- Resource groups can have tags
- Tags are NOT inherited by resources (must tag separately)

**Locks:**

- Can lock resource group to prevent deletion/modification
- Locks ARE inherited by resources

### Moving Resources Between Resource Groups

**You can move most resources** but with considerations:

**General rules:**

- Source and target RG must be in same subscription (for most resources)
- Some resources cannot be moved (check Azure documentation)
- Resource ID changes after move
- Moving takes time (can be several hours)

**Resources that CANNOT be moved:**

- App Service Certificates
- Azure AD Domain Services
- Some VM configurations (with managed disks in different RGs)
- Recovery Services vaults (with backups)

**Best practice:** Plan your resource groups well from the start to avoid moving.

**How to check if moveable:**
Azure Portal → Resource → "Move" button

- If grayed out: Cannot move
- If available: Can move (but verify dependencies)

### Resource Group Locks

Prevent accidental deletion or modification.

**Lock Types:**

**1. CanNotDelete:**

- Can read and modify resources
- Cannot delete resource group or resources inside
- Use case: Protect production resources

**2. ReadOnly:**

- Can read resources
- Cannot modify or delete
- Even "write" operations are blocked
- Use case: Compliance, audit scenarios

**Example:**

```
Production Resource Group
Lock: CanNotDelete

Effect:
✅ Admins can update VM configurations
✅ Can add new resources to RG
❌ Cannot delete the RG
❌ Cannot delete resources inside RG
```

**Important inheritance:**

- Lock on resource group applies to ALL resources inside
- Lock on resource applies only to that resource
- Child locks inherit from parent

**Removing locks:**

- Must have Owner or User Access Administrator role
- Two-step process: Remove lock, then delete resource
- Good for protecting critical resources

**Common exam scenario:**

```
Question: User has Contributor role but cannot delete VM. Why?
Answer: Resource group likely has CanNotDelete lock
Solution: Remove lock (requires Owner role), then delete
```

---

## Resource Tags

**Simple:** Key-value pairs attached to resources for organization and automation.

**Think of it like:** Sticky labels you put on file folders.

### What are Tags?

**Format:** `Key: Value`

**Examples:**

```
Environment: Production
CostCenter: Marketing
Owner: john.doe@contoso.com
Project: ProjectAlpha
Department: IT
AppName: WebApp
ShutdownTime: 18:00
AutoStart: True
```

### Why Use Tags?

**1. Cost Management:**

```
Tag all resources with CostCenter
View billing report grouped by CostCenter
See how much each department spends
```

**2. Organization:**

```
Tag: Environment: Production
Filter portal to show only production resources
```

**3. Automation:**

```
Tag: ShutdownTime: 18:00
Automation script: Shut down all VMs with this tag at 6 PM
Saves money on dev/test environments
```

**4. Access Control:**

```
RBAC based on tags (advanced scenarios)
Example: Developers can only access resources with Environment: Development
```

**5. Compliance and Reporting:**

```
Tag: Compliance: HIPAA
Generate report of all HIPAA-compliant resources
```

### Tag Limitations

**Important rules:**

| Limit                 | Value                                 |
| --------------------- | ------------------------------------- |
| Tags per resource     | 50                                    |
| Tag name length       | 512 characters                        |
| Tag value length      | 256 characters                        |
| Tag name restrictions | No: `<`, `>`, `%`, `&`, `\`, `?`, `/` |
| Case sensitivity      | No (tags are case-insensitive)        |

**Not all resources support tags:**

- Some Azure services don't support tagging
- Classic resources (old deployment model) have limited tag support

**Tags don't inherit:**

```
Resource Group: RG-Production
Tags: Environment: Production

VM inside RG-Production:
Does NOT automatically get Environment: Production tag
Must tag explicitly
```

**Solution:** Use Azure Policy to enforce/auto-apply tags.

### Tagging Strategies

**Strategy 1: Required tags policy**

```
Azure Policy: "Require tag 'CostCenter' on all resources"
Effect: Deny
Result: Cannot create resource without CostCenter tag
```

**Strategy 2: Inherit tags from resource group**

```
Azure Policy: "Inherit tag 'Environment' from resource group"
Effect: Modify
Result: Automatically copies Environment tag from RG to resources
```

**Strategy 3: Default tag values**

```
Azure Policy: "Add tag 'Owner' with default value if missing"
Effect: Modify
Default: "unassigned"
Result: All resources get Owner tag (even if not specified)
```

**Common tagging schemes:**

**By Environment:**

```
Environment: Production
Environment: Staging
Environment: Development
Environment: Test
```

**By Cost:**

```
CostCenter: CC-12345
Department: Marketing
Project: ProjectAlpha
BillingCode: BC-67890
```

**By Ownership:**

```
Owner: john.doe@contoso.com
Team: DevOps
Approver: jane.smith@contoso.com
```

**By Technical:**

```
AppName: WebApp
AppTier: Frontend
AppVersion: 2.1.0
ManagedBy: Terraform
```

**By Compliance:**

```
DataClassification: Confidential
Compliance: GDPR
RetentionPeriod: 7years
```

**By Scheduling:**

```
AutoShutdown: 18:00
AutoStart: 08:00
BackupPolicy: Daily
```

### Viewing and Filtering by Tags

**Azure Portal:**

- Search for tag name
- Filter resources by tag
- Create saved views with tag filters

**Cost Management:**

- Group costs by tag
- Create budgets per tag
- Set alerts based on tag spending

**Azure Resource Graph:**

```
Query resources by tags across subscriptions
Example: "Show all VMs with Environment: Production across all subscriptions"
```

---

## Managed Identities

**Simple:** A way for Azure resources to authenticate to other Azure services without storing passwords in code.

**The problem:**

```
Traditional approach:
  Your web app needs to read from Azure SQL Database
  You store username/password in code or config file
  ❌ Security risk (passwords in code)
  ❌ Password rotation is manual and complex
  ❌ Credentials can leak
```

**The solution:**

```
Managed Identity approach:
  Azure automatically creates an identity for your web app
  Identity is managed by Azure (no passwords!)
  Grant that identity access to SQL Database
  ✅ No passwords to manage
  ✅ Automatic credential rotation
  ✅ Secure by design
```

### How Managed Identities Work

**Behind the scenes:**

1. You enable managed identity on a resource (e.g., VM, App Service)
2. Azure creates a service principal in Azure AD for that resource
3. You assign RBAC roles to that service principal
4. The resource can now authenticate using its identity (Azure handles tokens)

**From application perspective:**

```csharp
// Traditional (BAD):
var connectionString = "Server=...;User=admin;Password=P@ssw0rd";

// With Managed Identity (GOOD):
var credential = new DefaultAzureCredential(); // Automatically uses managed identity
var client = new BlobServiceClient(new Uri("https://storage.blob.core.windows.net"), credential);
```

### Types of Managed Identities

#### System-Assigned Managed Identity

**Simple:** Created automatically with the resource, dies with the resource. One-to-one relationship.

**Characteristics:**

- Created when you enable it on a resource
- Tied to the resource lifecycle (deleted when resource is deleted)
- Each resource has its own unique identity
- Cannot be shared across multiple resources

**When to use:**

- Single resource needs access to other Azure services
- Identity should be deleted with the resource
- Simple scenarios

**Example:**

```
VM "web-server-01"
  └─ System-assigned MI: "web-server-01" (same name)

When VM is deleted → MI is automatically deleted
```

**Lifecycle:**

```
1. Create VM
2. Enable system-assigned managed identity
3. Azure creates service principal in Azure AD
4. Assign RBAC role (e.g., "Storage Blob Data Reader" on storage account)
5. VM can now access storage using its identity
6. Delete VM → MI automatically deleted
```

#### User-Assigned Managed Identity

**Simple:** Created independently, can be assigned to multiple resources. Exists separately from resources.

**Characteristics:**

- Created as a standalone Azure resource
- Has its own lifecycle (independent of resources)
- Can be assigned to multiple resources
- Persists even if resources are deleted
- Must be explicitly deleted

**When to use:**

- Multiple resources need the same access
- Identity should survive resource deletion
- Need to grant access before creating resources
- Complex scenarios with shared access

**Example:**

```
User-assigned MI: "shared-app-identity"
  ├─ Assigned to: VM "web-server-01"
  ├─ Assigned to: VM "web-server-02"
  └─ Assigned to: App Service "api-app"

When VMs deleted → MI still exists and can be reused
```

**Lifecycle:**

```
1. Create user-assigned managed identity "app-identity"
2. Assign RBAC role to this identity
3. Create VM1, assign "app-identity" to it
4. Create VM2, assign "app-identity" to it
5. Both VMs use same identity for access
6. Delete VM1 → identity still exists for VM2
7. Must explicitly delete "app-identity" when no longer needed
```

### Comparison

| Feature        | System-Assigned           | User-Assigned          |
| -------------- | ------------------------- | ---------------------- |
| **Creation**   | With resource             | Standalone resource    |
| **Lifecycle**  | Tied to resource          | Independent            |
| **Sharing**    | One resource only         | Multiple resources     |
| **Deletion**   | Automatic (with resource) | Manual                 |
| **Use case**   | Simple, single resource   | Complex, shared access |
| **Management** | Easier (auto-cleanup)     | More control           |

### Common Scenarios (Exam favorites!)

**Scenario 1: VM needs to access Storage Account**

```
Solution:
1. Enable system-assigned MI on VM
2. Assign "Storage Blob Data Contributor" role to VM's MI on storage account
3. VM can now read/write blobs without passwords

PowerShell:
# Enable MI on VM
Update-AzVM -ResourceGroupName "RG-App" -VM $vm -IdentityType SystemAssigned

# Assign role
$vmIdentity = (Get-AzVM -ResourceGroupName "RG-App" -Name "MyVM").Identity.PrincipalId
New-AzRoleAssignment -ObjectId $vmIdentity -RoleDefinitionName "Storage Blob Data Contributor" -Scope "/subscriptions/{sub-id}/resourceGroups/RG-App/providers/Microsoft.Storage/storageAccounts/mystorageacct"
```

**Scenario 2: Multiple VMs need access to same Key Vault**

```
Solution:
1. Create user-assigned MI: "keyvault-reader-identity"
2. Assign "Key Vault Secrets User" role to this MI
3. Assign MI to all VMs that need access
4. All VMs can read secrets using shared identity

Benefit: Manage permissions in one place (the MI), not on each VM
```

**Scenario 3: App Service needs to pull container images from ACR**

```
Solution:
1. Enable system-assigned MI on App Service
2. Assign "AcrPull" role to App Service's MI on ACR
3. App Service can pull images without storing credentials
```

**Scenario 4: Azure Automation runbook needs to manage VMs**

```
Solution:
1. Automation account has system-assigned MI
2. Assign "Virtual Machine Contributor" role to automation account's MI
3. Runbooks can start/stop VMs automatically
```

### Supported Services

**Resources that CAN have managed identities:**

- Virtual Machines
- Virtual Machine Scale Sets
- App Service / Functions
- Azure Container Instances
- Azure Kubernetes Service (AKS)
- Azure Logic Apps
- Azure Data Factory
- Azure API Management
- Azure Automation
- Many more...

**Services that ACCEPT managed identities for authentication:**

- Azure Storage
- Azure SQL Database
- Azure Key Vault
- Azure Resource Manager (ARM)
- Azure Data Lake
- Azure Event Hubs
- Azure Service Bus
- Almost all Azure services with Azure AD integration

### Best Practices

1. **Prefer managed identities over service principals with passwords**

   - No credential management
   - Automatic rotation
   - More secure

2. **Use system-assigned for simple scenarios**

   - Auto-cleanup
   - Less management overhead

3. **Use user-assigned for:**

   - Multiple resources with same access
   - Pre-provisioning access before resource creation
   - Resources that get recreated frequently

4. **Principle of least privilege**

   - Grant minimum permissions needed
   - Use resource-specific roles

5. **Use managed identities for authentication to:**
   - Azure services (Storage, Key Vault, SQL)
   - NOT for authenticating users (that's Azure AD users/groups)

---

## PowerShell Commands Reference

### Azure AD / Entra ID Management

**Connect to Azure:**

```powershell
# Connect to Azure account
Connect-AzAccount

# Connect to specific tenant
Connect-AzAccount -TenantId "00000000-0000-0000-0000-000000000000"

# List subscriptions
Get-AzSubscription

# Set active subscription
Set-AzContext -SubscriptionId "00000000-0000-0000-0000-000000000000"
# Or by name
Set-AzContext -SubscriptionName "Production"
```

**User Management:**

```powershell
# Note: User management requires Azure AD PowerShell module
Install-Module -Name AzureAD

Connect-AzureAD

# Create new user
New-AzureADUser -DisplayName "John Doe" `
    -UserPrincipalName "john.doe@contoso.com" `
    -AccountEnabled $true `
    -MailNickName "johndoe" `
    -PasswordProfile $PasswordProfile

# Get user
Get-AzureADUser -ObjectId "john.doe@contoso.com"

# Get all users
Get-AzureADUser -All $true

# Filter users by department
Get-AzureADUser -Filter "department eq 'Marketing'"

# Delete user
Remove-AzureADUser -ObjectId "john.doe@contoso.com"

# Reset user password
Set-AzureADUserPassword -ObjectId "john.doe@contoso.com" -Password $newPassword
```

**Group Management:**

```powershell
# Create security group
New-AzureADGroup -DisplayName "IT-Admins" `
    -MailEnabled $false `
    -SecurityEnabled $true `
    -MailNickName "NotSet"

# Create M365 group
New-AzureADGroup -DisplayName "Marketing-Team" `
    -MailEnabled $true `
    -SecurityEnabled $false `
    -MailNickName "marketing"

# Get group
Get-AzureADGroup -SearchString "IT-Admins"

# Add member to group
Add-AzureADGroupMember -ObjectId $groupId -RefObjectId $userId

# Get group members
Get-AzureADGroupMember -ObjectId $groupId

# Remove member from group
Remove-AzureADGroupMember -ObjectId $groupId -MemberId $userId

# Get user's group memberships
Get-AzureADUserMembership -ObjectId $userId
```

### RBAC Management

**Role Assignments:**

```powershell
# Get all role definitions
Get-AzRoleDefinition

# Get specific role
Get-AzRoleDefinition -Name "Contributor"

# Search for roles
Get-AzRoleDefinition | Where-Object {$_.Name -like "*Virtual Machine*"}

# Assign role to user at subscription scope
New-AzRoleAssignment -SignInName "john.doe@contoso.com" `
    -RoleDefinitionName "Contributor" `
    -Scope "/subscriptions/00000000-0000-0000-0000-000000000000"

# Assign role at resource group scope
New-AzRoleAssignment -SignInName "john.doe@contoso.com" `
    -RoleDefinitionName "Contributor" `
    -ResourceGroupName "RG-Production"

# Assign role to group
New-AzRoleAssignment -ObjectId $groupObjectId `
    -RoleDefinitionName "Reader" `
    -ResourceGroupName "RG-Production"

# Assign role at resource scope
$vm = Get-AzVM -ResourceGroupName "RG-App" -Name "MyVM"
New-AzRoleAssignment -SignInName "john.doe@contoso.com" `
    -RoleDefinitionName "Virtual Machine Contributor" `
    -Scope $vm.Id

# List role assignments for a user
Get-AzRoleAssignment -SignInName "john.doe@contoso.com"

# List role assignments for a resource group
Get-AzRoleAssignment -ResourceGroupName "RG-Production"

# Remove role assignment
Remove-AzRoleAssignment -SignInName "john.doe@contoso.com" `
    -RoleDefinitionName "Contributor" `
    -ResourceGroupName "RG-Production"
```

**Custom Roles:**

```powershell
# Get role definition as JSON
Get-AzRoleDefinition -Name "Virtual Machine Contributor" | ConvertTo-Json

# Create custom role from JSON file
New-AzRoleDefinition -InputFile "C:\CustomRole.json"

# Create custom role from object
$role = Get-AzRoleDefinition "Virtual Machine Contributor"
$role.Id = $null
$role.Name = "Custom VM Operator"
$role.Description = "Can start and stop VMs"
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Compute/virtualMachines/read")
$role.Actions.Add("Microsoft.Compute/virtualMachines/start/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/restart/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/powerOff/action")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/00000000-0000-0000-0000-000000000000")
New-AzRoleDefinition -Role $role

# Update custom role
Set-AzRoleDefinition -Role $role

# Delete custom role
Remove-AzRoleDefinition -Name "Custom VM Operator"
```

### Azure Policy

**Policy Definitions:**

```powershell
# Get all built-in policy definitions
Get-AzPolicyDefinition | Where-Object {$_.Properties.PolicyType -eq "BuiltIn"}

# Get specific policy definition
Get-AzPolicyDefinition -Name "allowed-locations"

# Search for policies
Get-AzPolicyDefinition | Where-Object {$_.Properties.DisplayName -like "*virtual machine*"}

# Create custom policy definition
$policyDef = @{
    Name = "require-tag-costcenter"
    DisplayName = "Require CostCenter tag on resources"
    Description = "Enforces existence of CostCenter tag on all resources"
    Policy = '{
        "if": {
            "field": "tags[CostCenter]",
            "exists": "false"
        },
        "then": {
            "effect": "deny"
        }
    }'
    Parameter = '{}'
    Mode = "All"
}
New-AzPolicyDefinition @policyDef

# Create from JSON file
New-AzPolicyDefinition -Name "my-policy" `
    -DisplayName "My Custom Policy" `
    -Policy "C:\policy-rule.json" `
    -Parameter "C:\policy-parameters.json"
```

**Policy Assignments:**

```powershell
# Assign policy to subscription
$subscription = Get-AzSubscription -SubscriptionName "Production"
New-AzPolicyAssignment -Name "allowed-locations-assignment" `
    -DisplayName "Allowed Locations" `
    -Scope "/subscriptions/$($subscription.Id)" `
    -PolicyDefinition $policyDef `
    -PolicyParameterObject @{
        listOfAllowedLocations = @("eastus", "westeurope")
    }

# Assign policy to resource group
New-AzPolicyAssignment -Name "require-tags" `
    -DisplayName "Require Tags on Resources" `
    -Scope "/subscriptions/$subscriptionId/resourceGroups/RG-Production" `
    -PolicyDefinition $policyDef

# List policy assignments
Get-AzPolicyAssignment

# Get specific policy assignment
Get-AzPolicyAssignment -Name "allowed-locations-assignment"

# Remove policy assignment
Remove-AzPolicyAssignment -Name "allowed-locations-assignment" `
    -Scope "/subscriptions/$subscriptionId"

# Get policy compliance state
Get-AzPolicyState -ResourceGroupName "RG-Production"

# Trigger policy compliance scan
Start-AzPolicyComplianceScan -ResourceGroupName "RG-Production"
```

**Initiatives (Policy Sets):**

```powershell
# Get built-in initiatives
Get-AzPolicySetDefinition | Where-Object {$_.Properties.PolicyType -eq "BuiltIn"}

# Get specific initiative
Get-AzPolicySetDefinition -Name "CIS-Microsoft-Azure-Foundations-Benchmark"

# Create custom initiative
$policySet = @{
    Name = "tagging-initiative"
    DisplayName = "Tagging Requirements"
    Description = "Enforces tagging standards"
    PolicyDefinition = @(
        @{
            policyDefinitionId = "/subscriptions/$subscriptionId/providers/Microsoft.Authorization/policyDefinitions/require-tag-costcenter"
        },
        @{
            policyDefinitionId = "/subscriptions/$subscriptionId/providers/Microsoft.Authorization/policyDefinitions/require-tag-environment"
        }
    )
}
New-AzPolicySetDefinition @policySet

# Assign initiative
New-AzPolicyAssignment -Name "tagging-initiative-assignment" `
    -PolicySetDefinition $initiative `
    -Scope "/subscriptions/$subscriptionId"
```

**Remediation:**

```powershell
# Create remediation task
Start-AzPolicyRemediation -Name "remediate-backup" `
    -PolicyAssignmentId $assignmentId `
    -ResourceGroupName "RG-Production"

# Get remediation status
Get-AzPolicyRemediation -Name "remediate-backup" `
    -ResourceGroupName "RG-Production"

# List all remediations
Get-AzPolicyRemediation

# Cancel remediation
Stop-AzPolicyRemediation -Name "remediate-backup" `
    -ResourceGroupName "RG-Production"
```

### Management Groups

```powershell
# Get all management groups
Get-AzManagementGroup

# Get specific management group
Get-AzManagementGroup -GroupName "Contoso"

# Get management group with expanded details
Get-AzManagementGroup -GroupName "Contoso" -Expand

# Create management group
New-AzManagementGroup -GroupName "Production-MG" `
    -DisplayName "Production" `
    -ParentId "/providers/Microsoft.Management/managementGroups/Contoso"

# Update management group
Update-AzManagementGroup -GroupName "Production-MG" `
    -DisplayName "Production Environment"

# Move subscription to management group
New-AzManagementGroupSubscription -GroupName "Production-MG" `
    -SubscriptionId $subscriptionId

# Remove subscription from management group
Remove-AzManagementGroupSubscription -GroupName "Production-MG" `
    -SubscriptionId $subscriptionId

# Delete management group
Remove-AzManagementGroup -GroupName "Production-MG"
```

### Subscriptions

```powershell
# List all subscriptions
Get-AzSubscription

# Get current subscription context
Get-AzContext

# Set active subscription
Set-AzContext -SubscriptionId "00000000-0000-0000-0000-000000000000"

# Set subscription by name
Set-AzContext -SubscriptionName "Production"

# Get subscription details
Get-AzSubscription -SubscriptionId "00000000-0000-0000-0000-000000000000"

# List resource providers
Get-AzResourceProvider

# Register resource provider
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute

# Check registration status
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

### Resource Groups

```powershell
# Create resource group
New-AzResourceGroup -Name "RG-Production" -Location "eastus"

# Create with tags
New-AzResourceGroup -Name "RG-Production" `
    -Location "eastus" `
    -Tag @{Environment="Production"; CostCenter="IT"}

# Get resource group
Get-AzResourceGroup -Name "RG-Production"

# List all resource groups
Get-AzResourceGroup

# Filter resource groups by tag
Get-AzResourceGroup -Tag @{Environment="Production"}

# Update resource group (add/update tags)
Update-AzResourceGroup -Name "RG-Production" `
    -Tag @{Environment="Production"; Owner="john.doe@contoso.com"}

# List resources in resource group
Get-AzResource -ResourceGroupName "RG-Production"

# Move resources to another resource group
Move-AzResource -DestinationResourceGroupName "RG-NewGroup" `
    -ResourceId $resourceId

# Delete resource group (CAREFUL!)
Remove-AzResourceGroup -Name "RG-Test" -Force
```

### Resource Locks

```powershell
# Create CanNotDelete lock on resource group
New-AzResourceLock -LockName "DoNotDeleteProd" `
    -LockLevel CanNotDelete `
    -LockNotes "Production resources - do not delete" `
    -ResourceGroupName "RG-Production"

# Create ReadOnly lock
New-AzResourceLock -LockName "ReadOnlyLock" `
    -LockLevel ReadOnly `
    -ResourceGroupName "RG-Audit"

# Create lock on specific resource
$vm = Get-AzVM -ResourceGroupName "RG-App" -Name "CriticalVM"
New-AzResourceLock -LockName "VMProtection" `
    -LockLevel CanNotDelete `
    -ResourceId $vm.Id

# List locks
Get-AzResourceLock

# List locks for resource group
Get-AzResourceLock -ResourceGroupName "RG-Production"

# Get specific lock
Get-AzResourceLock -LockName "DoNotDeleteProd" `
    -ResourceGroupName "RG-Production"

# Remove lock
Remove-AzResourceLock -LockName "DoNotDeleteProd" `
    -ResourceGroupName "RG-Production" `
    -Force
```

### Resource Tags

```powershell
# Add tags to resource group
$tags = @{
    Environment = "Production"
    CostCenter = "IT"
    Owner = "john.doe@contoso.com"
}
Update-AzResourceGroup -Name "RG-Production" -Tag $tags

# Add tags to existing resource
$resource = Get-AzResource -ResourceName "MyVM" -ResourceGroupName "RG-Production"
Update-AzResource -ResourceId $resource.Id `
    -Tag @{Environment="Production"; AppName="WebApp"} `
    -Force

# Get resources by tag
Get-AzResource -Tag @{Environment="Production"}

# Get all resources with specific tag name (any value)
Get-AzResource -TagName "Environment"

# Get all tag names and values in subscription
(Get-AzResource).Tags

# Remove all tags from resource
$resource = Get-AzResource -ResourceName "MyVM" -ResourceGroupName "RG-Production"
Update-AzResource -ResourceId $resource.Id -Tag @{} -Force

# Add tag to existing tags (without overwriting)
$resource = Get-AzResource -ResourceName "MyVM" -ResourceGroupName "RG-Production"
$tags = $resource.Tags
$tags += @{NewTag="NewValue"}
Update-AzResource -ResourceId $resource.Id -Tag $tags -Force

# Bulk tag resources in resource group
$resources = Get-AzResource -ResourceGroupName "RG-Production"
foreach ($resource in $resources) {
    $tags = $resource.Tags
    if ($null -eq $tags) { $tags = @{} }
    $tags += @{BulkTagged="Yes"}
    Update-AzResource -ResourceId $resource.Id -Tag $tags -Force
}
```

### Managed Identities

```powershell
# Enable system-assigned managed identity on VM
$vm = Get-AzVM -ResourceGroupName "RG-App" -Name "MyVM"
Update-AzVM -ResourceGroupName "RG-App" -VM $vm -IdentityType SystemAssigned

# Get system-assigned identity principal ID
$vm = Get-AzVM -ResourceGroupName "RG-App" -Name "MyVM"
$principalId = $vm.Identity.PrincipalId

# Assign role to system-assigned identity
New-AzRoleAssignment -ObjectId $principalId `
    -RoleDefinitionName "Storage Blob Data Contributor" `
    -Scope "/subscriptions/$subscriptionId/resourceGroups/RG-App/providers/Microsoft.Storage/storageAccounts/mystorageacct"

# Create user-assigned managed identity
New-AzUserAssignedIdentity -ResourceGroupName "RG-App" `
    -Name "shared-app-identity" `
    -Location "eastus"

# Get user-assigned identity
$userIdentity = Get-AzUserAssignedIdentity -ResourceGroupName "RG-App" `
    -Name "shared-app-identity"

# Assign user-assigned identity to VM
$vm = Get-AzVM -ResourceGroupName "RG-App" -Name "MyVM"
Update-AzVM -ResourceGroupName "RG-App" -VM $vm `
    -IdentityType UserAssigned `
    -IdentityId $userIdentity.Id

# Assign role to user-assigned identity
New-AzRoleAssignment -ObjectId $userIdentity.PrincipalId `
    -RoleDefinitionName "Key Vault Secrets User" `
    -Scope "/subscriptions/$subscriptionId/resourceGroups/RG-App/providers/Microsoft.KeyVault/vaults/mykeyvault"

# Remove system-assigned identity
$vm = Get-AzVM -ResourceGroupName "RG-App" -Name "MyVM"
Update-AzVM -ResourceGroupName "RG-App" -VM $vm -IdentityType None

# Delete user-assigned identity
Remove-AzUserAssignedIdentity -ResourceGroupName "RG-App" `
    -Name "shared-app-identity"
```

---

## Azure CLI Commands Reference

### Connection and Context

```bash
# Login to Azure
az login

# Login to specific tenant
az login --tenant 00000000-0000-0000-0000-000000000000

# List subscriptions
az account list --output table

# Show current subscription
az account show

# Set active subscription
az account set --subscription "00000000-0000-0000-0000-000000000000"

# Set by name
az account set --subscription "Production"
```

### User Management

```bash
# Create user (requires Azure AD permissions)
az ad user create \
    --display-name "John Doe" \
    --user-principal-name "john.doe@contoso.com" \
    --password "P@ssw0rd123!" \
    --force-change-password-next-sign-in true

# Get user
az ad user show --id "john.doe@contoso.com"

# List all users
az ad user list

# Search users
az ad user list --filter "startswith(displayName,'John')"

# Update user
az ad user update --id "john.doe@contoso.com" \
    --set jobTitle="Senior Developer"

# Delete user
az ad user delete --id "john.doe@contoso.com"

# Reset password
az ad user update --id "john.doe@contoso.com" \
    --password "NewP@ssw0rd!"
```

### Group Management

```bash
# Create security group
az ad group create \
    --display-name "IT-Admins" \
    --mail-nickname "itadmins"

# Get group
az ad group show --group "IT-Admins"

# List groups
az ad group list

# Add member to group
az ad group member add \
    --group "IT-Admins" \
    --member-id <user-object-id>

# List group members
az ad group member list --group "IT-Admins"

# Remove member
az ad group member remove \
    --group "IT-Admins" \
    --member-id <user-object-id>

# Check if user is member
az ad group member check \
    --group "IT-Admins" \
    --member-id <user-object-id>

# Delete group
az ad group delete --group "IT-Admins"
```

### RBAC Management

```bash
# List all role definitions
az role definition list --output table

# Get specific role
az role definition list --name "Contributor"

# Search roles
az role definition list --custom-role-only false \
    --query "[?contains(roleName, 'Virtual Machine')]"

# Assign role to user at subscription scope
az role assignment create \
    --assignee "john.doe@contoso.com" \
    --role "Contributor" \
    --scope "/subscriptions/00000000-0000-0000-0000-000000000000"

# Assign role at resource group scope
az role assignment create \
    --assignee "john.doe@contoso.com" \
    --role "Contributor" \
    --resource-group "RG-Production"

# Assign role to group
az role assignment create \
    --assignee <group-object-id> \
    --role "Reader" \
    --resource-group "RG-Production"

# Assign role at resource scope
az role assignment create \
    --assignee "john.doe@contoso.com" \
    --role "Virtual Machine Contributor" \
    --scope "/subscriptions/{sub-id}/resourceGroups/RG-App/providers/Microsoft.Compute/virtualMachines/MyVM"

# List role assignments
az role assignment list --output table

# List role assignments for user
az role assignment list \
    --assignee "john.doe@contoso.com" \
    --output table

# List role assignments for resource group
az role assignment list \
    --resource-group "RG-Production" \
    --output table

# Remove role assignment
az role assignment delete \
    --assignee "john.doe@contoso.com" \
    --role "Contributor" \
    --resource-group "RG-Production"
```

**Custom Roles:**

```bash
# Create custom role from JSON file
az role definition create --role-definition role-definition.json

# Example role-definition.json:
{
  "Name": "Custom VM Operator",
  "Description": "Can start and stop VMs",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/powerOff/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}

# Update custom role
az role definition update --role-definition role-definition.json

# Delete custom role
az role definition delete --name "Custom VM Operator"
```

### Azure Policy

```bash
# List built-in policy definitions
az policy definition list --query "[?policyType=='BuiltIn']" --output table

# Get specific policy
az policy definition show --name "allowed-locations"

# Create custom policy definition
az policy definition create \
    --name "require-tag-costcenter" \
    --display-name "Require CostCenter tag" \
    --description "Enforces CostCenter tag on all resources" \
    --rules '{
        "if": {
            "field": "tags[CostCenter]",
            "exists": "false"
        },
        "then": {
            "effect": "deny"
        }
    }' \
    --mode All

# Create from JSON file
az policy definition create \
    --name "my-policy" \
    --display-name "My Custom Policy" \
    --rules @policy-rules.json \
    --params @policy-parameters.json

# Assign policy to subscription
az policy assignment create \
    --name "allowed-locations" \
    --display-name "Allowed Locations" \
    --policy "allowed-locations" \
    --params '{
        "listOfAllowedLocations": {
            "value": ["eastus", "westeurope"]
        }
    }'

# Assign policy to resource group
az policy assignment create \
    --name "require-tags" \
    --display-name "Require Tags" \
    --policy "require-tag-costcenter" \
    --resource-group "RG-Production"

# List policy assignments
az policy assignment list --output table

# Show specific policy assignment
az policy assignment show --name "allowed-locations"

# Delete policy assignment
az policy assignment delete --name "allowed-locations"

# Get compliance state
az policy state list --resource-group "RG-Production"

# Trigger compliance scan
az policy state trigger-scan --resource-group "RG-Production"
```

**Initiatives:**

```bash
# List built-in initiatives
az policy set-definition list --output table

# Get specific initiative
az policy set-definition show \
    --name "CIS_Microsoft_Azure_Foundations_Benchmark"

# Create custom initiative
az policy set-definition create \
    --name "tagging-initiative" \
    --display-name "Tagging Requirements" \
    --definitions '[
        {
            "policyDefinitionId": "/subscriptions/{sub-id}/providers/Microsoft.Authorization/policyDefinitions/require-tag-costcenter"
        },
        {
            "policyDefinitionId": "/subscriptions/{sub-id}/providers/Microsoft.Authorization/policyDefinitions/require-tag-environment"
        }
    ]'

# Assign initiative
az policy assignment create \
    --name "tagging-initiative" \
    --policy-set-definition "tagging-initiative"
```

**Remediation:**

```bash
# Create remediation task
az policy remediation create \
    --name "remediate-backup" \
    --policy-assignment "/subscriptions/{sub-id}/providers/Microsoft.Authorization/policyAssignments/backup-policy" \
    --resource-group "RG-Production"

# List remediations
az policy remediation list --resource-group "RG-Production"

# Show remediation details
az policy remediation show \
    --name "remediate-backup" \
    --resource-group "RG-Production"

# Cancel remediation
az policy remediation cancel \
    --name "remediate-backup" \
    --resource-group "RG-Production"

# Delete remediation
az policy remediation delete \
    --name "remediate-backup" \
    --resource-group "RG-Production"
```

### Management Groups

```bash
# List all management groups
az account management-group list

# Show specific management group
az account management-group show --name "Contoso"

# Create management group
az account management-group create \
    --name "Production-MG" \
    --display-name "Production" \
    --parent "Contoso"

# Update management group
az account management-group update \
    --name "Production-MG" \
    --display-name "Production Environment"

# Add subscription to management group
az account management-group subscription add \
    --name "Production-MG" \
    --subscription "00000000-0000-0000-0000-000000000000"

# Remove subscription from management group
az account management-group subscription remove \
    --name "Production-MG" \
    --subscription "00000000-0000-0000-0000-000000000000"

# Delete management group
az account management-group delete --name "Production-MG"
```

### Resource Groups

```bash
# Create resource group
az group create --name "RG-Production" --location "eastus"

# Create with tags
az group create \
    --name "RG-Production" \
    --location "eastus" \
    --tags Environment=Production CostCenter=IT

# Show resource group
az group show --name "RG-Production"

# List all resource groups
az group list --output table

# Filter by tag
az group list --tag Environment=Production

# Update resource group (tags)
az group update \
    --name "RG-Production" \
    --tags Environment=Production Owner=john.doe@contoso.com

# List resources in resource group
az resource list --resource-group "RG-Production" --output table

# Check if resource group exists
az group exists --name "RG-Production"

# Delete resource group
az group delete --name "RG-Test" --yes --no-wait
```

### Resource Locks

```bash
# Create CanNotDelete lock on resource group
az lock create \
    --name "DoNotDeleteProd" \
    --lock-type CanNotDelete \
    --notes "Production resources - do not delete" \
    --resource-group "RG-Production"

# Create ReadOnly lock
az lock create \
    --name "ReadOnlyLock" \
    --lock-type ReadOnly \
    --resource-group "RG-Audit"

# Create lock on specific resource
az lock create \
    --name "VMProtection" \
    --lock-type CanNotDelete \
    --resource-group "RG-App" \
    --resource-name "CriticalVM" \
    --resource-type "Microsoft.Compute/virtualMachines"

# List locks
az lock list --output table

# List locks for resource group
az lock list --resource-group "RG-Production"

# Show specific lock
az lock show \
    --name "DoNotDeleteProd" \
    --resource-group "RG-Production"

# Delete lock
az lock delete \
    --name "DoNotDeleteProd" \
    --resource-group "RG-Production"
```

### Resource Tags

```bash
# Add tags to resource group
az group update \
    --name "RG-Production" \
    --tags Environment=Production CostCenter=IT Owner=john.doe@contoso.com

# Show resource group tags
az group show --name "RG-Production" --query tags

# Add tags to resource
az resource tag \
    --tags Environment=Production AppName=WebApp \
    --resource-group "RG-Production" \
    --name "MyVM" \
    --resource-type "Microsoft.Compute/virtualMachines"

# List resources by tag
az resource list --tag Environment=Production --output table

# List all resources with specific tag name
az resource list --tag-name Environment --output table

# Remove all tags from resource
az resource tag \
    --tags "" \
    --resource-group "RG-Production" \
    --name "MyVM" \
    --resource-type "Microsoft.Compute/virtualMachines"

# Query resources using JMESPath
az resource list \
    --query "[?tags.Environment=='Production'].{Name:name, Type:type}" \
    --output table
```

### Managed Identities

```bash
# Enable system-assigned identity on VM
az vm identity assign \
    --resource-group "RG-App" \
    --name "MyVM"

# Get system-assigned identity details
az vm identity show \
    --resource-group "RG-App" \
    --name "MyVM"

# Assign role to system-assigned identity
# First get the principal ID
PRINCIPAL_ID=$(az vm identity show \
    --resource-group "RG-App" \
    --name "MyVM" \
    --query principalId --output tsv)

# Then assign role
az role assignment create \
    --assignee $PRINCIPAL_ID \
    --role "Storage Blob Data Contributor" \
    --scope "/subscriptions/{sub-id}/resourceGroups/RG-App/providers/Microsoft.Storage/storageAccounts/mystorageacct"

# Create user-assigned identity
az identity create \
    --resource-group "RG-App" \
    --name "shared-app-identity"

# Get user-assigned identity details
az identity show \
    --resource-group "RG-App" \
    --name "shared-app-identity"

# Assign user-assigned identity to VM
az vm identity assign \
    --resource-group "RG-App" \
    --name "MyVM" \
    --identities "/subscriptions/{sub-id}/resourcegroups/RG-App/providers/Microsoft.ManagedIdentity/userAssignedIdentities/shared-app-identity"

# Assign role to user-assigned identity
PRINCIPAL_ID=$(az identity show \
    --resource-group "RG-App" \
    --name "shared-app-identity" \
    --query principalId --output tsv)

az role assignment create \
    --assignee $PRINCIPAL_ID \
    --role "Key Vault Secrets User" \
    --scope "/subscriptions/{sub-id}/resourceGroups/RG-App/providers/Microsoft.KeyVault/vaults/mykeyvault"

# Remove system-assigned identity
az vm identity remove \
    --resource-group "RG-App" \
    --name "MyVM"

# Delete user-assigned identity
az identity delete \
    --resource-group "RG-App" \
    --name "shared-app-identity"
```

---

## Common Exam Scenarios

### Scenario 1: User Cannot Delete Resources

**Problem:**
John has Contributor role on resource group RG-Production but cannot delete a VM.

**Possible causes:**

1. ✅ **Resource lock** (CanNotDelete) on resource group or resource
2. Azure Policy with Deny effect
3. Resource has child resources that need to be deleted first

**Solution:**

```powershell
# Check for locks
Get-AzResourceLock -ResourceGroupName "RG-Production"

# If lock exists, remove it (requires Owner role)
Remove-AzResourceLock -LockName "DoNotDelete" -ResourceGroupName "RG-Production" -Force

# Then delete resource
```

**Key point:** Even with Contributor role, locks prevent deletion.

### Scenario 2: Policy Not Working on Existing Resources

**Problem:**
Policy assigned to deny VMs without backup, but existing VMs without backup still show as compliant.

**Cause:**
Policies only affect NEW resources or UPDATES by default. Existing resources are audited but not automatically remediated.

**Solution:**

```bash
# Create remediation task
az policy remediation create \
    --name "fix-vm-backup" \
    --policy-assignment "/subscriptions/{sub-id}/providers/Microsoft.Authorization/policyAssignments/require-backup" \
    --resource-group "RG-Production"
```

**Key point:** Use DeployIfNotExists effect + remediation for existing resources.

### Scenario 3: Multiple Role Assignments

**Problem:**
User has Reader role on subscription and Contributor role on RG-App. What can they do?

**Answer:**

- Can VIEW everything in subscription (Reader inheritance)
- Can MANAGE resources in RG-App (Contributor on that RG)
- Cannot manage resources in other resource groups (only Reader there)

**Key point:** Permissions are additive. Most permissive wins.

### Scenario 4: Tag Inheritance

**Problem:**
Resource group has Environment: Production tag. New VM created in that RG doesn't have the tag.

**Cause:**
Tags do NOT inherit by default.

**Solution:**
Use Azure Policy with Modify effect:

```json
{
  "if": {
    "field": "tags[Environment]",
    "exists": "false"
  },
  "then": {
    "effect": "modify",
    "details": {
      "operations": [
        {
          "operation": "add",
          "field": "tags[Environment]",
          "value": "[resourceGroup().tags[Environment]]"
        }
      ]
    }
  }
}
```

**Key point:** Use policy to enforce tag inheritance.

### Scenario 5: System vs User-Assigned Identity

**Problem:**
Need 5 VMs to access the same Key Vault. Which identity type?

**Answer:**
User-assigned managed identity is better:

- Create one user-assigned identity
- Assign it to all 5 VMs
- Manage permissions in one place
- Identity persists if VMs are recreated

**System-assigned would require:**

- 5 separate identities
- 5 separate role assignments
- Identities deleted when VMs deleted

**Key point:** User-assigned for shared access, system-assigned for single-resource access.

### Scenario 6: Management Group Policies

**Problem:**
Policy at Management Group denies VM creation in certain regions. Can subscription admin override it?

**Answer:**
NO. Policies inherit down and cannot be overridden (only exemptions possible).

**Hierarchy:**

```
Management Group (Deny VMs in westus)
  └─ Subscription (Admin CANNOT override)
      └─ Resource Group (CANNOT create VMs in westus)
```

**Key point:** Parent policies cannot be overridden by children.

### Scenario 7: Custom Role Scope

**Problem:**
Created custom role but cannot assign it to resource group RG-Test.

**Cause:**
AssignableScopes in role definition doesn't include that resource group.

**Solution:**

```powershell
$role = Get-AzRoleDefinition -Name "Custom VM Operator"
$role.AssignableScopes.Add("/subscriptions/{sub-id}/resourceGroups/RG-Test")
Set-AzRoleDefinition -Role $role
```

**Key point:** Custom roles have explicit assignable scopes.

### Scenario 8: Guest User Access

**Problem:**
External consultant needs access to specific SharePoint site. How to grant access?

**Answer:**

1. Invite as guest user (B2B)
2. They use their own organization's credentials
3. Assign to group with appropriate permissions

**Do NOT:**

- Create cloud-only user for them
- Share admin credentials

**Key point:** Use guest users for external collaboration.

---

## Key Takeaways for Exam

### Must-Know Concepts

1. **RBAC vs Azure Policy**

   - RBAC = WHO can do WHAT
   - Policy = WHAT resources SHOULD look like
   - They work together, not instead of each other

2. **Permission Inheritance**

   - RBAC: Flows down (Management Group → Subscription → RG → Resource)
   - Policy: Flows down (same hierarchy)
   - Tags: Do NOT inherit
   - Locks: DO inherit

3. **Built-in Roles**

   - Owner = Everything including access management
   - Contributor = Everything except access management
   - Reader = View only
   - User Access Administrator = Access management only

4. **Policy Effects Priority**

   - Deny > Append/Modify > Audit > DeployIfNotExists
   - Disabled policies are ignored
   - Deny can block Owner from performing actions

5. **Managed Identity Types**

   - System-assigned = One-to-one with resource, auto-cleanup
   - User-assigned = Standalone, shareable, manual cleanup
   - Both eliminate password management

6. **Resource Group Rules**

   - Every resource in exactly ONE RG
   - Resources can be in different regions than RG
   - Deleting RG deletes all resources inside
   - Locks prevent deletion even for Owners

7. **Tags Best Practices**

   - 50 tags maximum per resource
   - Use for cost tracking, automation, organization
   - Not inherited by default
   - Use policy to enforce tagging

8. **Policy Evaluation**
   - Applies to NEW resources automatically
   - Existing resources need remediation tasks
   - Compliance scan every 24 hours
   - 30 minutes for new assignments to take effect

### Common Exam Tricks

**Trick 1: "User has Contributor but can't delete"**

- Look for resource locks (CanNotDelete)

**Trick 2: "Policy not affecting existing resources"**

- Policy only affects new/updated resources by default
- Need remediation task for existing resources

**Trick 3: "Tags not showing on resources"**

- Tags don't inherit - need policy to enforce

**Trick 4: "Can view but not modify with Contributor role"**

- Check for ReadOnly lock on resource or RG

**Trick 5: "Policy at subscription but resource group admin overrides it"**

- Cannot override parent policy (only exemptions possible)

**Trick 6: "Multiple roles - what can user do?"**

- Permissions are additive (union, not intersection)

**Trick 7: "Custom role cannot be assigned"**

- Check AssignableScopes in role definition

**Trick 8: "Guest user needs access"**

- Use B2B invitation, not new cloud user

### Quick Reference Tables

**RBAC Scope Hierarchy:**

```
Management Group (highest)
  ↓
Subscription
  ↓
Resource Group
  ↓
Resource (lowest)
```

**Policy Effect Comparison:**

| Effect            | Blocks Creation? | Audits? | Auto-Fix? | Use Case                        |
| ----------------- | ---------------- | ------- | --------- | ------------------------------- |
| Deny              | ✅ Yes           | No      | No        | Prevent non-compliant resources |
| Audit             | No               | ✅ Yes  | No        | Discovery and reporting         |
| AuditIfNotExists  | No               | ✅ Yes  | No        | Check for related resources     |
| DeployIfNotExists | No               | ✅ Yes  | ✅ Yes    | Auto-remediation                |
| Modify            | No               | No      | ✅ Yes    | Auto-fix properties (tags)      |
| Append            | No               | No      | ✅ Yes    | Add configurations              |

**Lock Types:**

| Lock Type    | Read | Modify | Delete |
| ------------ | ---- | ------ | ------ |
| CanNotDelete | ✅   | ✅     | ❌     |
| ReadOnly     | ✅   | ❌     | ❌     |

**User Types:**

| Type   | Where Created | Authentication    | Use Case          |
| ------ | ------------- | ----------------- | ----------------- |
| Cloud  | Azure AD      | Azure AD          | Cloud-only orgs   |
| Synced | On-prem AD    | On-prem/Cloud     | Hybrid orgs       |
| Guest  | External      | External identity | B2B collaboration |

### Commands You Must Know

**PowerShell - Top 10:**

```powershell
# 1. Connect and set context
Connect-AzAccount
Set-AzContext -SubscriptionName "Production"

# 2. Create resource group
New-AzResourceGroup -Name "RG-App" -Location "eastus"

# 3. Assign RBAC role
New-AzRoleAssignment -SignInName "user@contoso.com" -RoleDefinitionName "Contributor" -ResourceGroupName "RG-App"

# 4. Create policy assignment
New-AzPolicyAssignment -Name "require-tags" -PolicyDefinition $policyDef -Scope "/subscriptions/{id}"

# 5. Create resource lock
New-AzResourceLock -LockName "DoNotDelete" -LockLevel CanNotDelete -ResourceGroupName "RG-Prod"

# 6. Add tags
Update-AzResourceGroup -Name "RG-App" -Tag @{Environment="Prod"; Owner="john@contoso.com"}

# 7. Enable managed identity
Update-AzVM -ResourceGroupName "RG-App" -VM $vm -IdentityType SystemAssigned

# 8. Create user-assigned identity
New-AzUserAssignedIdentity -ResourceGroupName "RG-App" -Name "app-identity"

# 9. List role assignments
Get-AzRoleAssignment -ResourceGroupName "RG-App"

# 10. Get policy compliance
Get-AzPolicyState -ResourceGroupName "RG-App"
```

**Azure CLI - Top 10:**

```bash
# 1. Login and set context
az login
az account set --subscription "Production"

# 2. Create resource group
az group create --name "RG-App" --location "eastus"

# 3. Assign RBAC role
az role assignment create --assignee "user@contoso.com" --role "Contributor" --resource-group "RG-App"

# 4. Create policy assignment
az policy assignment create --name "require-tags" --policy "policy-def-id" --resource-group "RG-App"

# 5. Create resource lock
az lock create --name "DoNotDelete" --lock-type CanNotDelete --resource-group "RG-Prod"

# 6. Add tags
az group update --name "RG-App" --tags Environment=Prod Owner=john@contoso.com

# 7. Enable managed identity
az vm identity assign --resource-group "RG-App" --name "MyVM"

# 8. Create user-assigned identity
az identity create --resource-group "RG-App" --name "app-identity"

# 9. List role assignments
az role assignment list --resource-group "RG-App"

# 10. Get policy compliance
az policy state list --resource-group "RG-App"
```

### Troubleshooting Flowcharts

**"User Cannot Perform Action" Flowchart:**

```
User cannot perform action
  ↓
Check RBAC role assignment
  ├─ No role? → Assign appropriate role
  └─ Has role? ↓
    Check resource lock
      ├─ Lock exists? → Remove lock (need Owner)
      └─ No lock? ↓
        Check Azure Policy
          ├─ Deny policy? → Remove policy or create exemption
          └─ No policy? ↓
            Check resource provider registration
              ├─ Not registered? → Register provider
              └─ Registered? → Check service limits/quotas
```

**"Policy Not Working" Flowchart:**

```
Policy not working
  ↓
Check policy assignment status
  ├─ Not assigned? → Assign policy
  └─ Assigned? ↓
    Wait 30 minutes for policy to take effect
      ↓
    Check resource creation/update time
      ├─ Resource created before policy? → Need remediation task
      └─ Resource created after policy? ↓
        Check policy effect
          ├─ Audit? → Check compliance dashboard
          ├─ Deny? → Should block creation
          └─ DeployIfNotExists? → Check managed identity permissions
```

### Study Tips

**Week 1 Focus:**

- Azure AD basics (users, groups)
- RBAC roles and assignments
- Practice assigning roles via portal, PowerShell, CLI

**Week 2 Focus:**

- Azure Policy (effects, assignments)
- Management groups and subscriptions
- Resource groups and organization

**Week 3 Focus:**

- Tags and metadata
- Resource locks
- Managed identities

**Week 4 Focus:**

- Practice scenarios
- Mix concepts together
- Time yourself on practice questions

**Practice Labs:**

1. Create users and groups, assign RBAC roles
2. Create policy to require tags, test it
3. Create management group hierarchy
4. Enable managed identity on VM, grant Key Vault access
5. Apply locks, try to delete resources
6. Bulk tag resources using PowerShell

**Red Flags in Questions:**

- "User has Owner but cannot..." → Look for LOCKS
- "Policy assigned but resources still non-compliant" → Need REMEDIATION
- "Tags not showing" → Tags DON'T INHERIT
- "Multiple roles" → Permissions are ADDITIVE
- "External user access" → Use GUEST user (B2B)
- "Many VMs need same access" → USER-ASSIGNED identity
- "Cannot assign custom role" → Check ASSIGNABLE SCOPES

---

## Practice Questions

### Question 1

You have a resource group named RG-Production with a CanNotDelete lock. A user has Owner role on the subscription. Can the user delete a VM in RG-Production?

**Answer:** No. The lock prevents deletion even for Owners. The user would need to remove the lock first (which requires Owner or User Access Administrator role), then delete the VM.

### Question 2

You assign a policy that requires all VMs to have backup enabled. Ten VMs already exist without backup. What happens to these VMs?

**Answer:** The policy will mark them as non-compliant but won't automatically fix them. If the policy uses DeployIfNotExists effect, you need to create a remediation task to enable backup on existing VMs.

### Question 3

A resource group has tag Environment: Production. You create a VM in that resource group. Does the VM automatically get the Environment tag?

**Answer:** No. Tags do not inherit by default. You would need to manually tag the VM or use an Azure Policy with Modify effect to inherit tags from the resource group.

### Question 4

User A has Reader role on subscription and Contributor role on RG-App. What can User A do in RG-Database (another resource group in the same subscription)?

**Answer:** User A can only VIEW resources in RG-Database (Reader role from subscription inheritance). They cannot create, modify, or delete resources there. Contributor role only applies to RG-App.

### Question 5

You need to grant 5 VMs access to the same Azure Key Vault. Should you use system-assigned or user-assigned managed identity?

**Answer:** User-assigned managed identity. Create one user-assigned identity, grant it Key Vault access, then assign it to all 5 VMs. This is easier to manage than 5 separate system-assigned identities and the identity persists even if VMs are deleted.

### Question 6

You create a custom role but cannot assign it to resource group RG-Test. What should you check?

**Answer:** Check the AssignableScopes property in the role definition. The role can only be assigned at scopes explicitly listed in AssignableScopes. Add the resource group (or its parent subscription) to AssignableScopes.

### Question 7

A policy is assigned at the management group level that denies all VM creation. Can a subscription admin in that management group override this policy to allow VMs?

**Answer:** No. Policies inherit down the hierarchy and cannot be overridden by child scopes. The only option is to create a policy exemption (which requires appropriate permissions).

### Question 8

You have Contributor role but cannot delete a storage account. You check and there are no locks. What else could prevent deletion?

**Answer:** Check for:

1. Azure Policy with Deny effect on delete operations
2. Storage account might contain data that requires separate permissions to delete (unlikely but possible)
3. Dependencies - other resources might depend on it
4. Your Contributor role might be at a different scope (check if role is actually on this resource group)

---

## Additional Resources

**Microsoft Learn Modules (Specific to Module 1):**

- [Manage identities and governance in Azure](https://learn.microsoft.com/training/paths/az-104-manage-identities-governance/)
- [Configure Azure Active Directory](https://learn.microsoft.com/training/modules/configure-azure-active-directory/)
- [Configure user and group accounts](https://learn.microsoft.com/training/modules/configure-user-group-accounts/)
- [Configure role-based access control](https://learn.microsoft.com/training/modules/configure-role-based-access-control/)
- [Configure Azure Policy](https://learn.microsoft.com/training/modules/configure-azure-policy/)

**Documentation:**

- [Azure AD documentation](https://learn.microsoft.com/azure/active-directory/)
- [Azure RBAC documentation](https://learn.microsoft.com/azure/role-based-access-control/)
- [Azure Policy documentation](https://learn.microsoft.com/azure/governance/policy/)
- [Managed identities documentation](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/)

**Video Resources:**

- [John Savill's AZ-104 Identity and Governance](https://www.youtube.com/watch?v=0Sq8AQux_O4)
- [Azure AD Fundamentals](https://www.youtube.com/watch?v=Ma7VAQE7ga4)

---

## Summary Checklist

Before moving to Module 2, ensure you can:

**Azure AD / Entra ID:**

- [ ] Explain the difference between tenant, directory, and domain
- [ ] Create and manage users (cloud, synced, guest)
- [ ] Create and manage groups (security and M365)
- [ ] Understand dynamic group membership rules
- [ ] Explain Azure AD editions and features

**RBAC:**

- [ ] Explain the three components of RBAC (who, what, where)
- [ ] Describe the 4 fundamental built-in roles
- [ ] Understand scope hierarchy and inheritance
- [ ] Assign roles at different scopes
- [ ] Create custom roles
- [ ] Troubleshoot role assignment issues

**Azure Policy:**

- [ ] Explain policy definition vs assignment
- [ ] Describe all 6 policy effects
- [ ] Create and assign policies
- [ ] Create policy initiatives
- [ ] Perform remediation tasks
- [ ] Understand policy evaluation and compliance

**Management & Organization:**

- [ ] Create and manage management groups
- [ ] Understand subscription organization
- [ ] Create and organize resource groups
- [ ] Apply and manage resource locks
- [ ] Implement tagging strategies

**Managed Identities:**

- [ ] Explain system-assigned vs user-assigned
- [ ] Enable managed identities on resources
- [ ] Assign RBAC roles to managed identities
- [ ] Understand when to use each type

**PowerShell & CLI:**

- [ ] Execute common identity management commands
- [ ] Execute common RBAC commands
- [ ] Execute common policy commands
- [ ] Execute common resource management commands

**Exam Readiness:**

- [ ] Complete practice questions with 80%+ accuracy
- [ ] Explain scenarios without looking at notes
- [ ] Perform hands-on labs for all topics
- [ ] Can troubleshoot common issues

---

## Next Steps

Congratulations on completing Module 1! 🎉

**You've mastered:**
✅ Identity and access management
✅ Governance and organization
✅ Security and compliance foundations

**Next Module:** [Module 2: Storage →](../Module-02-Storage/)

In Module 2, you'll learn:

- Storage accounts and services
- Blob storage and lifecycle management
- Azure Files and File Sync
- Storage security and access control
- Backup and disaster recovery

**Before moving on:**

1. ✅ Complete practice labs for Module 1
2. ✅ Take the Module 1 practice quiz
3. ✅ Review any weak areas
4. ✅ Ensure you understand PowerShell/CLI commands

**Estimated time for Module 2:** 8-10 hours

---

## Need Help?

**Common questions:**

**Q: Do I need to memorize all PowerShell/CLI commands?**
A: No, but you should understand the syntax pattern and be able to recognize them. The exam provides some hints, and you'll have access to `--help` in lab scenarios.

**Q: How deep should I know Azure AD?**
A: Focus on basics for AZ-104. Deep Azure AD topics (Conditional Access, PIM) are more relevant for AZ-500 (Security certification).

**Q: Are custom roles heavily tested?**
A: Not heavily, but you should know when to use them and how AssignableScopes work. Focus more on built-in roles.

**Q: How important are management groups?**
A: Moderately important. Understand the concept and hierarchy, but they're less tested than RBAC and policies.

**Q: Should I practice in portal or PowerShell/CLI?**
A: Both! Portal for understanding concepts, PowerShell/CLI for speed and automation. The exam tests both.

---

**Good luck with your studies! You're doing great! 💪**

Remember: Consistent practice is more important than cramming. Take breaks, do hands-on labs, and don't move forward until you're comfortable with the material.

**[← Back to Introduction](../README.md) | [Next: Module 2: Storage →](../Module-02-Storage/README.md)**
