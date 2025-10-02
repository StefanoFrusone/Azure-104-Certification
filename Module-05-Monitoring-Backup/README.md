# Module 5: Monitoring and Backup

## Table of Contents

- [Introduction](#introduction)
- [Azure Monitor](#azure-monitor)
- [Log Analytics](#log-analytics)
- [Application Insights](#application-insights)
- [Alerts and Action Groups](#alerts-and-action-groups)
- [Azure Backup](#azure-backup)
- [Azure Site Recovery](#azure-site-recovery)
- [Cost Management](#cost-management)
- [PowerShell Commands Reference](#powershell-commands-reference)
- [Azure CLI Commands Reference](#azure-cli-commands-reference)
- [Common Exam Scenarios](#common-exam-scenarios)
- [Practice Questions](#practice-questions)

---

## Introduction

**What is this module about?**

This module covers monitoring, alerting, backup, and disaster recovery in Azure - the essential tools for keeping your infrastructure healthy and recoverable.

**Real-world analogy:**
Think of monitoring and backup like maintaining a house:

- **Azure Monitor** = Security cameras watching everything
- **Log Analytics** = Recording and analyzing all footage
- **Alerts** = Alarm system that notifies you of problems
- **Azure Backup** = Regular photos of your belongings
- **Site Recovery** = Insurance policy with quick rebuild plan
- **Cost Management** = Budget tracking and expense reports

**Key questions this module answers:**

1. **How** do I monitor resource health and performance? (Azure Monitor)
2. **How** do I collect and analyze logs? (Log Analytics)
3. **How** do I get notified when problems occur? (Alerts)
4. **How** do I protect data with backups? (Azure Backup)
5. **How** do I ensure business continuity? (Site Recovery)
6. **How** do I control and optimize costs? (Cost Management)

**Exam Weight:** 10-15% of the exam

---

## Azure Monitor

### What is Azure Monitor?

**Simple explanation:**
Azure Monitor collects, analyzes, and acts on telemetry data from your Azure and on-premises environments.

**Technical definition:**
Azure Monitor is a comprehensive monitoring solution for collecting, analyzing, and responding to monitoring data from cloud and on-premises environments.

**What does Azure Monitor collect?**

```
Azure Resources
   ↓
Metrics + Logs
   ↓
Azure Monitor
   ↓
├─ Visualize (dashboards, workbooks)
├─ Analyze (metrics explorer, log queries)
├─ Respond (alerts, autoscale)
└─ Integrate (export, APIs)
```

### Monitoring Data Types

**1. Metrics**

**Simple:** Numerical values collected at regular intervals.

**Characteristics:**
- Lightweight (efficient)
- Near real-time (1-minute intervals)
- Time-series data
- Stored for 93 days
- Free (platform metrics)

**Examples:**
```
VM metrics:
- CPU percentage: 45%
- Memory usage: 2.3 GB
- Network in: 150 Mbps
- Disk IOPS: 500

Storage metrics:
- Transactions: 1,500/minute
- Availability: 99.99%
- Latency: 25ms
```

**Platform metrics (automatic, no configuration):**
- VM: CPU, disk, network
- Storage: Transactions, latency, availability
- SQL Database: DTU percentage, connections
- App Service: CPU time, memory, HTTP requests

**Custom metrics (you send):**
- Application-specific metrics
- Business KPIs
- Custom counters

**2. Logs**

**Simple:** Detailed records of events and operations.

**Characteristics:**
- Rich data (detailed information)
- Queried on-demand (KQL queries)
- Stored in Log Analytics workspace
- Retention: 30 days to 2 years (configurable)
- Costs for ingestion and retention

**Examples:**
```
Activity logs:
- "User john@contoso.com created VM 'WebServer01'"
- "Resource group 'RG-Prod' deleted"
- "Role assignment added to 'alice@contoso.com'"

Resource logs:
- NSG flow logs
- VPN Gateway diagnostics
- SQL query performance
- Application errors
```

**Log types:**

**Activity Log:**
- Subscription-level events
- Who did what, when
- Automatic collection (free)
- 90-day retention

**Resource Logs (Diagnostic Logs):**
- Resource-specific logs
- Must enable per resource
- Sent to Log Analytics, Storage, or Event Hub

**Azure Active Directory Logs:**
- Sign-ins
- Audit logs
- Requires Azure AD Premium P1/P2

### Metrics Explorer

**Simple:** Visual tool to analyze and chart metrics.

**How to use:**

1. Select resource (VM, Storage, etc.)
2. Select metric (CPU, Memory, etc.)
3. Select aggregation (Average, Max, Min, Sum)
4. Select time range (Last hour, Last 24 hours, etc.)
5. Optional: Add filters, split by dimensions

**Example analysis:**

```
Question: Is my VM CPU overloaded?

Metrics Explorer:
Resource: WebServer01
Metric: Percentage CPU
Aggregation: Average
Time range: Last 24 hours
Split by: None

Result: Average CPU = 85%
Conclusion: Yes, consider larger VM size
```

**Splitting metrics:**

```
Multiple VMs in same chart:
Resource: All VMs in resource group
Metric: Percentage CPU
Split by: Virtual machine name

Shows: Comparison of CPU across all VMs
```

### Diagnostic Settings

**Simple:** Configuration that routes resource logs to destinations.

**Components:**

**1. Source:** Which logs to collect
- All logs
- Specific log categories
- Metrics

**2. Destination:** Where to send logs
- **Log Analytics workspace** (query and analyze)
- **Storage Account** (archive, long-term retention)
- **Event Hub** (stream to external systems)
- **Partner solutions** (third-party SIEM)

**Common patterns:**

**Pattern 1: Analysis**
```
Source: VM, NSG, Load Balancer
Logs: All logs
Destination: Log Analytics workspace
Use case: Query and analyze, create alerts
```

**Pattern 2: Archive**
```
Source: All resources
Logs: Activity log, resource logs
Destination: Storage Account
Use case: Long-term retention, compliance
Retention: 365 days+
```

**Pattern 3: Real-time streaming**
```
Source: Application
Logs: Application logs
Destination: Event Hub
Use case: Stream to Splunk, QRadar, third-party SIEM
```

**Pattern 4: Multi-destination**
```
Source: SQL Database
Logs: Query performance
Destinations:
  - Log Analytics (analysis, alerts)
  - Storage Account (archive)
Use case: Best of both worlds
```

### Activity Log

**Simple:** Subscription-level audit trail.

**What's logged:**
- Resource operations (create, delete, update)
- Who performed the action
- When it happened
- Status (succeeded, failed)
- IP address

**Examples:**
```
Event: Virtual machine created
Who: john@contoso.com
When: 2024-01-15 10:30:00 UTC
Resource: WebServer01
Status: Succeeded

Event: Network security group rule modified
Who: alice@contoso.com
When: 2024-01-15 11:15:00 UTC
Resource: NSG-Web
Status: Succeeded

Event: Resource group deleted
Who: admin@contoso.com
When: 2024-01-15 14:00:00 UTC
Resource: RG-Test
Status: Failed (resources still exist)
```

**Activity Log categories:**
- Administrative (management operations)
- Service Health (Azure service issues)
- Resource Health (resource-specific health)
- Alert (fired alerts)
- Autoscale (scaling operations)
- Security (Azure Security Center recommendations)

**Retention:**
- Portal view: 90 days
- For longer: Export to Log Analytics or Storage

### Workbooks

**Simple:** Interactive reports combining metrics, logs, and text.

**Use cases:**
- Custom dashboards
- Troubleshooting guides
- Performance reports
- Capacity planning

**Components:**
- Text (markdown)
- Parameters (filters, dropdowns)
- Queries (KQL)
- Metrics charts
- Visualizations (grids, charts, maps)

**Example workbook:**

```
VM Performance Report
───────────────────────────────────────
Parameters:
- Subscription: [Production]
- Resource Group: [RG-Web]
- Time Range: [Last 24 hours]

Section 1: CPU Usage
[Chart showing CPU percentage for all VMs]

Section 2: Memory Usage
[Chart showing available memory]

Section 3: Disk IOPS
[Grid showing disk performance per VM]

Section 4: Top Errors (from logs)
[Table of most common errors]
```

**Built-in workbooks:**
- VM Insights
- Storage Insights
- Network Insights
- Application Insights
- Azure Backup

**Custom workbooks:**
- Create from scratch
- Start from template
- Save and share with team

---

## Log Analytics

### What is Log Analytics?

**Simple:** Centralized log storage and powerful query engine.

**Real-world analogy:**
Log Analytics = Database for all your logs with SQL-like queries

**Architecture:**

```
Data Sources → Log Analytics Workspace → Queries → Insights

Data Sources:
├─ Azure resources (VMs, NSGs, etc.)
├─ On-premises servers (Log Analytics agent)
├─ Applications (Application Insights)
└─ Custom data (API)

Queries:
├─ KQL (Kusto Query Language)
├─ Pre-built queries
└─ Custom queries

Insights:
├─ Visualizations
├─ Alerts
├─ Workbooks
└─ Dashboards
```

### Log Analytics Workspace

**Simple:** Container for log data.

**Key concepts:**

**1. Workspace Design:**

**Single workspace (simple):**
```
One workspace for entire organization
Pros: Simple, easy management, cross-resource queries
Cons: All users see all data, harder to control costs
Use for: Small organizations, dev/test
```

**Multiple workspaces (recommended):**
```
Workspace per environment:
- Workspace-Prod
- Workspace-Dev
- Workspace-Test

OR workspace per region:
- Workspace-EastUS
- Workspace-WestEurope

Pros: Data isolation, cost tracking, access control
Cons: More complex, queries across workspaces harder
Use for: Large organizations, compliance requirements
```

**2. Data Retention:**
- Default: 30 days (free)
- Can increase: Up to 2 years (additional cost)
- Archive tier: Long-term retention (lower cost, slower access)

**3. Pricing:**
- Pay-as-you-go: Per GB ingested
- Commitment tiers: 100 GB/day, 200 GB/day, etc. (cheaper per GB)
- Data retention: Free for first 30 days, then per GB/month

### Kusto Query Language (KQL)

**Simple:** SQL-like language for querying logs.

**Basic structure:**
```kql
TableName
| where condition
| project columns
| summarize aggregation
| order by column
| take number
```

**Common tables:**
- `AzureActivity` - Activity log
- `AzureDiagnostics` - Resource diagnostic logs
- `AzureMetrics` - Metrics
- `Heartbeat` - Agent health
- `Perf` - Performance counters
- `Syslog` - Linux syslogs
- `Event` - Windows events
- `SecurityEvent` - Security events

### KQL Query Examples

**Example 1: Failed Azure operations (last 24 hours)**
```kql
AzureActivity
| where TimeGenerated > ago(24h)
| where ActivityStatusValue == "Failure"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup
| order by TimeGenerated desc
```

**Example 2: VM CPU over 80%**
```kql
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where CounterValue > 80
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| order by TimeGenerated desc
```

**Example 3: Count of operations by user**
```kql
AzureActivity
| where TimeGenerated > ago(7d)
| summarize Count = count() by Caller
| order by Count desc
| take 10
```

**Example 4: NSG flows (top talkers)**
```kql
AzureDiagnostics
| where Category == "NetworkSecurityGroupFlowEvent"
| where TimeGenerated > ago(1h)
| summarize TotalBytes = sum(BytesSent) by SourceIP = SrcIP_s
| order by TotalBytes desc
| take 10
```

**Example 5: VM disk space low**
```kql
Perf
| where ObjectName == "LogicalDisk" and CounterName == "% Free Space"
| where CounterValue < 20
| where InstanceName != "_Total"
| project Computer, InstanceName, CounterValue
| order by CounterValue asc
```

**Example 6: Application errors (last hour)**
```kql
AppExceptions
| where TimeGenerated > ago(1h)
| summarize Count = count() by ProblemId, ExceptionType
| order by Count desc
```

### KQL Operators

**Filtering:**
```kql
// where - filter rows
| where Computer == "WebServer01"
| where CounterValue > 80
| where TimeGenerated > ago(1h)

// between - range filter
| where CounterValue between (70 .. 90)

// in - multiple values
| where ResourceGroup in ("RG-Prod", "RG-Stage")
```

**Selecting columns:**
```kql
// project - select columns
| project TimeGenerated, Computer, CounterValue

// project-away - exclude columns
| project-away TenantId, SourceSystem
```

**Aggregating:**
```kql
// summarize - aggregate data
| summarize count() by Computer
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)
| summarize max(CounterValue), min(CounterValue) by Computer
```

**Sorting:**
```kql
// order by - sort results
| order by TimeGenerated desc
| order by CounterValue asc
| sort by Computer asc, TimeGenerated desc
```

**Limiting:**
```kql
// take/limit - limit rows
| take 100
| limit 10
```

**Joining:**
```kql
// join - combine tables
Heartbeat
| where TimeGenerated > ago(1h)
| join kind=inner (
    Perf
    | where ObjectName == "Processor"
) on Computer
```

**Time functions:**
```kql
ago(1h)          // 1 hour ago
ago(1d)          // 1 day ago
now()            // Current time
startofday(now()) // Today at 00:00
bin(TimeGenerated, 5m) // Round to 5-minute buckets
```

### Saved Queries

**Simple:** Reusable KQL queries.

**Benefits:**
- Don't rewrite common queries
- Share with team
- Quick access to important queries
- Include in workbooks

**Example saved queries:**
```
Name: "Failed Operations Today"
Query: AzureActivity | where ActivityStatusValue == "Failure" | where TimeGenerated > startofday(now())
Category: Troubleshooting

Name: "High CPU VMs"
Query: Perf | where CounterName == "% Processor Time" | where CounterValue > 80 | summarize avg(CounterValue) by Computer
Category: Performance

Name: "Recent NSG Changes"
Query: AzureActivity | where OperationNameValue contains "SecurityRule" | where TimeGenerated > ago(7d)
Category: Security
```

---

## Application Insights

### What is Application Insights?

**Simple:** Application Performance Management (APM) service for developers.

**What it monitors:**
- Request rates, response times, failure rates
- Dependency calls (SQL, HTTP, etc.)
- Exceptions and stack traces
- Page views and load performance
- AJAX calls from web pages
- User and session counts
- Performance counters (CPU, memory, network)

**How it works:**

```
Application
   ↓ (Telemetry)
Application Insights SDK or Agent
   ↓
Application Insights Resource
   ↓
├─ Live Metrics (real-time)
├─ Application Map (dependencies)
├─ Performance (response times)
├─ Failures (exceptions, errors)
└─ Usage (users, sessions, events)
```

### Application Insights Features

**1. Live Metrics**

Real-time telemetry (1-second refresh).

**Shows:**
- Incoming requests per second
- Outgoing requests (dependencies)
- Overall health
- Exceptions
- Sample telemetry

**Use for:**
- Deployment validation
- Load testing
- Immediate issue detection

**2. Application Map**

Visual representation of application architecture.

**Shows:**
- Application components
- Dependencies (databases, APIs, queues)
- Call rates and response times
- Failed calls
- Bottlenecks

**Example:**
```
[Web App] 
   ↓ (50 req/s, 200ms avg)
[App Service]
   ├─→ [SQL Database] (30 req/s, 50ms avg)
   ├─→ [Redis Cache] (40 req/s, 5ms avg)
   └─→ [External API] (10 req/s, 500ms avg) ← Bottleneck!
```

**3. Performance**

Analyze request performance.

**Views:**
- Operations: Slowest operations
- Dependencies: Slowest dependencies
- Timeline: Performance over time

**Example investigation:**
```
Issue: Application slow

Performance view:
- Operation: "GET /api/products"
- Average duration: 3.2 seconds
- Slowest duration: 8.5 seconds

Drill down:
- SQL query taking 2.8 seconds
- Fetching 10,000 rows

Solution: Add pagination, optimize query
```

**4. Failures**

Track exceptions and failed requests.

**Shows:**
- Exception types
- Failed operations
- Stack traces
- Affected users

**Example:**
```
Exception: NullReferenceException
Count: 157 occurrences (last 24h)
Operation: "POST /api/orders"
Stack trace: [Full stack trace]
First occurred: 2024-01-15 08:00 UTC
Affected users: 23
```

**5. Usage Analytics**

Understand user behavior.

**Metrics:**
- Users (unique users)
- Sessions (user sessions)
- Events (custom events)
- Page views
- Funnels (conversion paths)
- Cohorts (user groups)
- User flows

**Example funnel:**
```
Shopping funnel:
1. Visit homepage: 1,000 users (100%)
2. View product: 600 users (60%)
3. Add to cart: 300 users (30%)
4. Checkout: 150 users (15%)
5. Complete order: 120 users (12%)

Drop-off analysis: Identify where users leave
```

### Instrumenting Applications

**Option 1: SDK (code-based)**

Add Application Insights SDK to application code.

**Advantages:**
- Full telemetry
- Custom events
- Custom metrics
- Detailed control

**.NET Example:**
```csharp
// Install: Microsoft.ApplicationInsights.AspNetCore

// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddApplicationInsightsTelemetry();
}

// Custom tracking
private readonly TelemetryClient telemetryClient;

telemetryClient.TrackEvent("OrderPlaced", new Dictionary<string, string>
{
    { "OrderId", orderId },
    { "Amount", amount.ToString() }
});
```

**Option 2: Agent-based (no code changes)**

Install Application Insights agent on server.

**Advantages:**
- No code changes
- Works with existing apps
- Auto-instrumentation

**Supported:**
- App Service (enable in portal)
- VMs (install agent)
- AKS (install agent)

**Option 3: Auto-instrumentation (Azure services)**

Built-in for some Azure services.

**Examples:**
- Azure Functions (automatic)
- App Service (enable in portal)
- Container Apps (enable in portal)

### Application Insights Pricing

**Data volume-based:**
- First 5 GB/month: Free
- Additional data: ~$2.30 per GB

**Typical costs:**
- Small app: < 1 GB/month = Free
- Medium app: 10 GB/month = ~$12/month
- Large app: 100 GB/month = ~$220/month

**Cost optimization:**
- Sampling (reduce data collected)
- Filter out unnecessary telemetry
- Set daily cap
- Adaptive sampling (automatic)

---

## Alerts and Action Groups

### What are Alerts?

**Simple:** Notifications when conditions are met.

**Real-world analogy:**
Alert = Smoke detector that sounds alarm and calls fire department

**Alert components:**

```
Alert Rule
├─ Scope: What to monitor (resource)
├─ Condition: When to alert (metric/log query)
├─ Action Group: What to do (notify, run action)
└─ Alert Details: Name, description, severity
```

### Alert Types

**1. Metric Alerts**

Based on metric values.

**Example:**
```
Alert: VM High CPU
Scope: WebServer01
Condition: CPU Percentage > 80%
Time aggregation: Average over 5 minutes
Evaluation frequency: Every 1 minute
Action: Email admin, create ticket
```

**Characteristics:**
- Near real-time (1-5 minute evaluation)
- Stateful (alerts when condition met, resolves when condition cleared)
- Multiple dimensions (alert on multiple resources)

**2. Log Query Alerts**

Based on log query results.

**Example:**
```
Alert: Failed Logins
Scope: Log Analytics workspace
Condition: 
  SecurityEvent
  | where EventID == 4625
  | where TimeGenerated > ago(5m)
  | count
  | where Count > 10
Action: Email security team, run playbook
```

**Characteristics:**
- Flexible (any KQL query)
- Can aggregate multiple resources
- Scheduled evaluation (1-60 minutes)

**3. Activity Log Alerts**

Based on activity log events.

**Example:**
```
Alert: VM Deleted
Scope: Subscription
Condition: 
  Operation: Delete Virtual Machine
  Status: Succeeded
Action: Email admin, create incident ticket
```

**Use cases:**
- Resource deletions
- Configuration changes
- Service health events
- Security recommendations

**4. Smart Detection Alerts (Application Insights)**

Machine learning-based anomaly detection.

**Examples:**
- Abnormal rise in failure rate
- Abnormal rise in slow page load time
- Degradation in response time
- Potential memory leak
- Potential security issue

**Advantages:**
- No configuration needed
- Learns normal patterns
- Detects anomalies automatically

### Alert Severity

**Severity levels:**

| Severity | Level | Use Case |
|----------|-------|----------|
| Sev 0 | Critical | Production down, immediate action required |
| Sev 1 | Error | Major functionality impaired |
| Sev 2 | Warning | Potential issue, monitor closely |
| Sev 3 | Informational | FYI, no action required |
| Sev 4 | Verbose | Detailed information |

**Example assignments:**
```
Sev 0: Production database offline
Sev 1: API response time > 3 seconds
Sev 2: CPU > 80% for 10 minutes
Sev 3: Scale operation completed
```

### Action Groups

**Simple:** Define who to notify and what actions to take.

**Action types:**

**1. Notifications:**
- Email/SMS/Push/Voice
- Azure mobile app notification
- Email Azure Resource Manager role

**2. Actions:**
- Run Azure Function
- Run Logic App
- Run Automation Runbook
- Call Webhook
- Create ITSM ticket (ServiceNow, etc.)

**Example action group:**
```
Name: "Production-Critical"
Actions:
1. Email: ops-team@contoso.com
2. SMS: +1-555-0100 (on-call engineer)
3. Push: Azure mobile app
4. Webhook: https://slack.com/webhook (Slack channel)
5. Automation Runbook: Restart-WebServer
```

**Multiple action groups:**
```
Sev 0 alerts → "Production-Critical" action group
Sev 1 alerts → "Production-Standard" action group  
Sev 2 alerts → "Monitoring-Team" action group
Sev 3 alerts → No action group (informational)
```

### Alert Rules Best Practices

**1. Tune thresholds:**
```
Bad: CPU > 50% (too sensitive, too many alerts)
Good: CPU > 80% for 15 minutes (actual problem)
```

**2. Use appropriate evaluation frequency:**
```
Critical metrics: Every 1 minute
Standard metrics: Every 5 minutes
Log queries: Every 15-30 minutes
```

**3. Group related alerts:**
```
Instead of: 10 separate "VM CPU high" alerts
Use: 1 alert with dimension splitting (all VMs)
```

**4. Set proper severity:**
```
Critical (Sev 0): Wake someone up at 3 AM
Warning (Sev 2): Review during business hours
```

**5. Avoid alert fatigue:**
```
Too many alerts → People ignore them → Real issues missed
Solution: Fine-tune thresholds, suppress non-critical alerts
```

### Alert States

**States:**
- **New:** Alert fired (condition met)
- **Acknowledged:** Someone acknowledged the alert
- **Closed:** Issue resolved (condition cleared)

**State transitions:**
```
Condition met → New
User clicks "Acknowledge" → Acknowledged
Condition cleared → Closed (automatic)
```

---

## Azure Backup

### What is Azure Backup?

**Simple:** Cloud-based backup solution for Azure and on-premises resources.

**What can be backed up:**
- Azure VMs
- SQL Server in Azure VMs
- SAP HANA in Azure VMs
- Azure Files
- Azure Blobs
- On-premises files/folders
- On-premises VMs (Hyper-V, VMware)
- On-premises applications (SQL, Exchange, SharePoint)

**Why Azure Backup:**
- No infrastructure needed
- Unlimited data transfer (no egress charges for restore)
- Encrypted backups
- Long-term retention
- Application-consistent backups
- Ransomware protection

### Backup Architecture

```
Backup Sources
├─ Azure VMs
├─ On-premises servers
├─ Azure Files
└─ Databases
   ↓
Recovery Services Vault
├─ Backup policies
├─ Backup data
├─ Recovery points
└─ Configuration
   ↓
Backup Storage (Azure Storage)
├─ Locally Redundant (LRS)
├─ Geo-Redundant (GRS)
└─ Zone-Redundant (ZRS)
```

### Recovery Services Vault

**Simple:** Container for backup data and configuration.

**Key settings:**

**1. Storage Replication:**
- **Locally Redundant (LRS):** 3 copies in same datacenter (cheaper)
- **Geo-Redundant (GRS):** 6 copies across regions (recommended)
- **Zone-Redundant (ZRS):** Copies across availability zones

**Important:**
- Can only change BEFORE first backup
- Cannot change after backups started

**2. Soft Delete:**
- Deleted backups retained for 14 days
- Protection against accidental deletion
- Protection against ransomware
- Can be disabled (not recommended)

**3. Cross-Region Restore (GRS only):**
- Restore to secondary region
- Disaster recovery capability
- Additional cost

### VM Backup

**How it works:**

1. Backup extension installed on VM
2. Snapshot taken (VM continues running)
3. Data transferred to vault
4. Incremental backups (only changed blocks)

**Backup types:**

**Standard backup:**
- Once per day
- Application-consistent (Windows with VSS)
- File-system consistent (Linux)
- Recovery points: 1-9999 days

**Enhanced backup:**
- Multiple backups per day (up to every 4 hours)
- Lower RPO (Recovery Point Objective)
- Same recovery options

**Backup policies:**

**Default policy:**
```
Schedule: Daily at 10:00 PM
Retention:
- Daily: 30 days
- Weekly: 12 weeks (Sunday)
- Monthly: 12 months (First Sunday)
- Yearly: 10 years (First Sunday of January)
```

**Custom policy example:**
```
Schedule: Daily at 2:00 AM
Retention:
- Daily: 7 days
- Weekly: 4 weeks
- Monthly: 12 months
- Yearly: 7 years
```

**Important:** First backup is full, subsequent are incremental

### VM Restore Options

**1. Create new VM:**
```
Restore to new VM in same or different region
Original VM unchanged
Good for: Testing, DR scenarios
```

**2. Replace existing disk:**
```
Replace VM's disks with backup
VM configuration unchanged
Good for: Quick recovery
Caution: Overwrites existing data
```

**3. Restore disks only:**
```
Just restore VHD files
Attach to any VM manually
Good for: Flexibility, file-level recovery
```

**4. File-level restore:**
```
Mount backup as iSCSI disk
Browse and copy specific files
No full VM restore needed
Good for: Recovering individual files
```

**5. Cross-region restore (GRS):**
```
Restore VM to secondary region
Disaster recovery
Requires GRS vault
```

### Backup for Azure Files

**How it works:**
- Share-level snapshots (not file-level)
- Stored with Azure Files (not in vault)
- Incremental snapshots
- No impact on performance

**Backup policy:**
```
Schedule: Daily, weekly, or monthly
Retention: Up to 200 snapshots
Instant restore: Snapshots immediately available
```

**Restore options:**
- Full share restoration
- Individual file/folder restoration
- Restore to alternate location

### Backup Center

**Simple:** Centralized management for all backups.

**Features:**
- View all backups across subscriptions
- Monitor backup jobs
- Configure new backups
- Restore data
- View backup reports
- Manage backup policies

**Dashboard shows:**
- Total backup items
- Failed backup jobs
- Storage consumption
- Compliance status

---

## Azure Site Recovery

### What is Azure Site Recovery (ASR)?

**Simple:** Disaster recovery solution for business continuity.

**Capabilities:**
- Replicate Azure VMs between regions
- Replicate on-premises VMs to Azure
- Replicate on-premises to on-premises (with VMM)
- Orchestrate failover and failback

**Use cases:**
- Disaster recovery (region outage)
- Migration to Azure
- Datacenter migration

### Site Recovery vs Backup

| Feature | Azure Backup | Site Recovery |
|---------|--------------|---------------|
| **Purpose** | Data protection | Disaster recovery |
| **RPO** | Hours (24h default) | Minutes (30s-5min) |
| **RTO** | Hours | Hours |
| **Recovery** | Point-in-time | Near real-time replica |
| **Use case** | File/folder recovery | Full site failover |
| **Cost** | Lower | Higher |

**Best practice:** Use both together
- Backup: Daily protection, point-in-time recovery
- Site Recovery: Regional DR, business continuity

### Azure VM Replication

**Architecture:**

```
Primary Region (East US)
VM running production workload
   ↓ (Continuous replication)
Secondary Region (West US)
Replica VM (powered off)
Managed disks synced
   ↓ (Failover when needed)
Secondary VM powered on
Takes over production
```

**How it works:**

1. **Initial replication:**
   - Full copy of VM disks
   - Can take several hours
   - No impact on VM performance

2. **Delta replication:**
   - Continuous synchronization
   - Only changed blocks replicated
   - RPO

   **How it works:**

1. **Initial replication:**
   - Full copy of VM disks
   - Can take several hours
   - No impact on VM performance

2. **Delta replication:**
   - Continuous synchronization
   - Only changed blocks replicated
   - RPO: 30 seconds to 5 minutes

3. **Recovery points:**
   - Crash-consistent: Every 5 minutes
   - App-consistent: Every 1-12 hours (Windows with VSS)

**Configuration:**

```
Enable replication:
1. Select source VM
2. Choose target region
3. Configure replication settings:
   - Resource group
   - Virtual network
   - Storage account
   - Availability options (zones, sets)
4. Customize settings:
   - Replication policy (RPO)
   - VM properties (size, disks)
5. Enable replication
```

**Replication policy:**
```
RPO: 30 seconds
Crash-consistent snapshots: Every 5 minutes
App-consistent snapshots: Every 4 hours
Retention: 24 hours (crash), 7 days (app-consistent)
```

### Failover Types

**1. Test Failover**

Test DR plan without affecting production.

**Process:**
```
1. Select recovery point
2. Start test failover
3. VM created in isolated network
4. Test application functionality
5. Clean up test resources
6. No impact on replication
```

**Best practice:** Test every 6 months

**2. Planned Failover**

Graceful failover for planned maintenance.

**Characteristics:**
- No data loss
- Source VM shut down
- Final sync performed
- Target VM started
- Used for: Planned maintenance, datacenter migration

**Process:**
```
1. Notify users of downtime
2. Stop source VM
3. Final synchronization
4. Start target VM
5. Update DNS/traffic routing
```

**3. Unplanned Failover**

Emergency failover during disaster.

**Characteristics:**
- May have data loss (depends on last recovery point)
- Source VM not accessible
- Target VM started immediately
- Used for: Region outage, disaster

**Process:**
```
1. Disaster detected
2. Select latest recovery point
3. Start failover (manual or automated)
4. Target VM powered on
5. Update DNS/routing
6. Application validation
```

**Recovery points:**
```
Latest (lowest RPO): Most recent sync
Latest processed: Last successfully processed
Latest app-consistent: Last VSS snapshot
Custom: Specific point in time
```

### Failback

**Simple:** Return to primary region after disaster resolved.

**Process:**

```
1. Primary region restored
2. Enable reverse replication (Secondary → Primary)
3. Planned failover back to primary
4. Resume normal replication (Primary → Secondary)
```

**Important:**
- Requires re-protection (reverse replication)
- Can take time to sync data back
- Plan during maintenance window

### Recovery Plans

**Simple:** Orchestrate failover of multiple VMs.

**Components:**

**1. Groups:**
Organize VMs in logical tiers.

```
Group 1: Database servers (fail over first)
  Wait 5 minutes for DB startup

Group 2: Application servers
  Wait 3 minutes for app startup

Group 3: Web servers
  Update load balancer configuration
```

**2. Pre/Post scripts:**
Automate tasks during failover.

```
Pre-script (before failover):
- Notify operations team
- Backup current state
- Export configuration

Post-script (after failover):
- Update DNS records
- Configure load balancers
- Run health checks
- Notify users
```

**3. Manual actions:**
Steps requiring human intervention.

```
Manual action: Verify database connection strings
Manual action: Update external API endpoints
Manual action: Test application functionality
```

**Example recovery plan:**

```
Plan: Three-Tier-App-DR

Group 1: Database
├─ SQL-Server-01
├─ SQL-Server-02
└─ Wait: 5 minutes

Group 2: Application
├─ App-Server-01
├─ App-Server-02
├─ Script: Update-ConnectionStrings.ps1
└─ Wait: 3 minutes

Group 3: Web
├─ Web-Server-01
├─ Web-Server-02
├─ Manual: Update load balancer
└─ Script: Notify-Users.ps1
```

### Site Recovery Pricing

**Per replicated instance:**
- ~$25/month per VM
- Includes: Replication, orchestration, testing

**Additional costs:**
- Storage (replica disks)
- Network egress (during failover)
- Snapshots

**Example:**
```
10 VMs with Site Recovery:
- License: 10 × $25 = $250/month
- Storage: 10 × 256 GB × $0.05 = $128/month
Total: ~$378/month

Compare to region outage cost:
- Downtime: 4 hours
- Revenue: $10,000/hour
- Loss: $40,000
- ROI: Worth it!
```

---

## Cost Management

### Azure Cost Management

**Simple:** Tools to monitor, analyze, and optimize Azure spending.

**Key features:**
- Cost analysis (view spending)
- Budgets (set spending limits)
- Recommendations (optimize costs)
- Exports (integrate with external tools)

### Cost Analysis

**Views:**

**1. Accumulated costs:**
```
Shows: Total spending over time
Example: 
- Week 1: $1,000
- Week 2: $2,500 (cumulative)
- Week 3: $4,200 (cumulative)
```

**2. Daily costs:**
```
Shows: Spending per day
Example:
- Jan 1: $150
- Jan 2: $145
- Jan 3: $180
```

**3. Forecast:**
```
Shows: Predicted spending based on trends
Example: If current trend continues, month total: $5,200
```

**Group by:**
- Resource group
- Resource type (VM, Storage, Network)
- Service (Compute, Storage, Networking)
- Location (region)
- Tags

**Example analysis:**

```
Question: Why is my bill high?

Group by: Service
Result:
- Compute: $2,000 (60%)
- Storage: $800 (24%)
- Networking: $500 (15%)
- Other: $50 (1%)

Drill into Compute:
Group by: Resource
- Large VMs running 24/7: $1,500
- Stopped but allocated VMs: $300
- VM Scale Sets: $200

Action:
- Deallocate unused VMs: Save $300/month
- Resize oversized VMs: Save $400/month
- Total savings: $700/month
```

### Budgets

**Simple:** Set spending limits and get alerts.

**Configuration:**

```
Budget: Production-Monthly
Scope: Resource group "RG-Prod"
Amount: $5,000
Period: Monthly (resets each month)

Alerts:
- 50% of budget ($2,500) → Email finance team
- 80% of budget ($4,000) → Email manager
- 100% of budget ($5,000) → Email everyone, trigger automation
- 110% of budget ($5,500) → Critical alert
```

**Budget periods:**
- Monthly (most common)
- Quarterly
- Annual
- Custom

**Action groups with budgets:**

```
Budget exceeded → Trigger action:
- Email notifications
- Webhook to ticketing system
- Automation runbook to shut down non-prod resources
- Logic App to request budget increase approval
```

### Cost Optimization Recommendations

**Azure Advisor cost recommendations:**

**1. Resize underutilized VMs:**
```
Recommendation: VM "WebServer03" CPU < 5%
Action: Resize from D4s_v3 to D2s_v3
Savings: $100/month
```

**2. Stop/deallocate idle VMs:**
```
Recommendation: VM "DevServer" not used
Action: Stop and deallocate
Savings: $150/month
```

**3. Delete unattached disks:**
```
Recommendation: 15 disks not attached to any VM
Action: Delete unused disks
Savings: $75/month
```

**4. Right-size or shutdown SQL databases:**
```
Recommendation: SQL Database underutilized
Action: Lower tier from Premium to Standard
Savings: $200/month
```

**5. Use Reserved Instances:**
```
Recommendation: VMs running 24/7 for 12+ months
Action: Purchase 1-year or 3-year reservation
Savings: 30-70% discount
```

### Cost-Saving Strategies

**1. Deallocate VMs when not needed:**
```
Dev/test VMs:
- Stop at 6 PM weekdays
- Stop all weekend
- Savings: 70 hours/week = 41% reduction
```

**2. Use Azure Hybrid Benefit:**
```
If you have Windows Server licenses with Software Assurance:
- Use existing licenses in Azure
- Savings: Up to 40% on Windows VMs
```

**3. Use Reserved Instances/Savings Plans:**
```
1-year commitment: 30-40% discount
3-year commitment: 50-70% discount
Best for: Predictable workloads
```

**4. Use Spot VMs:**
```
Unused Azure capacity
Discount: Up to 90% off
Risk: Can be evicted with 30-second notice
Best for: Batch processing, fault-tolerant workloads
```

**5. Right-size resources:**
```
Don't use D16s_v3 if D4s_v3 is sufficient
Review metrics, scale down
Savings: Can be 50-75%
```

**6. Use Azure Storage lifecycle management:**
```
Move blobs to:
- Cool tier after 30 days (cheaper)
- Archive tier after 90 days (cheapest)
- Delete after 365 days
Savings: 50-90% on storage costs
```

**7. Delete unused resources:**
```
Regular audit:
- Unattached disks
- Unused snapshots
- Old backups
- Stopped (but not deallocated) VMs
- Orphaned public IPs
```

### Tags for Cost Management

**Use tags to track costs:**

```
Common tags:
- Environment: Production, Development, Test
- CostCenter: IT, Marketing, Sales
- Project: ProjectA, ProjectB
- Owner: john@contoso.com
- Department: Engineering, Finance

Example:
Resource: WebServer01
Tags:
  Environment: Production
  CostCenter: IT-001
  Project: E-Commerce
  Owner: john@contoso.com

Cost analysis: Group by tag "CostCenter"
Result: IT-001 spent $5,000 this month
```

**Tag policies:**
```
Policy: Require "CostCenter" tag on all resources
Effect: Cannot create resources without this tag
Benefit: Complete cost tracking
```

---

## PowerShell Commands Reference

### Azure Monitor

```powershell
# Get metric definitions for a resource
$vm = Get-AzVM -ResourceGroupName "RG-Monitor" -Name "WebServer01"
Get-AzMetricDefinition -ResourceId $vm.Id

# Get metrics
Get-AzMetric -ResourceId $vm.Id `
    -MetricName "Percentage CPU" `
    -TimeGrain 00:05:00 `
    -StartTime (Get-Date).AddHours(-24) `
    -EndTime (Get-Date)

# Create diagnostic settings
$workspaceId = "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.OperationalInsights/workspaces/LogAnalytics"
$storageAccountId = "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.Storage/storageAccounts/diagstorage"

Set-AzDiagnosticSetting -ResourceId $vm.Id `
    -Name "DiagSettings" `
    -WorkspaceId $workspaceId `
    -StorageAccountId $storageAccountId `
    -Enabled $true `
    -Category "Administrative"

# Get diagnostic settings
Get-AzDiagnosticSetting -ResourceId $vm.Id

# Remove diagnostic settings
Remove-AzDiagnosticSetting -ResourceId $vm.Id -Name "DiagSettings"
```

### Log Analytics

```powershell
# Create Log Analytics workspace
New-AzOperationalInsightsWorkspace -ResourceGroupName "RG-Monitor" `
    -Name "LogAnalytics-Prod" `
    -Location "eastus" `
    -Sku "PerGB2018"

# Get workspace
Get-AzOperationalInsightsWorkspace -ResourceGroupName "RG-Monitor" `
    -Name "LogAnalytics-Prod"

# Get workspace keys (for agent configuration)
Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName "RG-Monitor" `
    -Name "LogAnalytics-Prod"

# Run query
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "RG-Monitor" `
    -Name "LogAnalytics-Prod"

$query = @"
AzureActivity
| where TimeGenerated > ago(24h)
| where ActivityStatusValue == "Failure"
| project TimeGenerated, Caller, OperationNameValue
| order by TimeGenerated desc
"@

Invoke-AzOperationalInsightsQuery -WorkspaceId $workspace.CustomerId `
    -Query $query

# Create saved search
New-AzOperationalInsightsSavedSearch -ResourceGroupName "RG-Monitor" `
    -WorkspaceName "LogAnalytics-Prod" `
    -SavedSearchId "HighCPU" `
    -DisplayName "High CPU VMs" `
    -Category "Performance" `
    -Query "Perf | where CounterName == '% Processor Time' | where CounterValue > 80"

# Delete workspace
Remove-AzOperationalInsightsWorkspace -ResourceGroupName "RG-Monitor" `
    -Name "LogAnalytics-Prod" `
    -Force
```

### Application Insights

```powershell
# Create Application Insights
New-AzApplicationInsights -ResourceGroupName "RG-Monitor" `
    -Name "AppInsights-WebApp" `
    -Location "eastus" `
    -Kind "web" `
    -ApplicationType "web"

# Get Application Insights
Get-AzApplicationInsights -ResourceGroupName "RG-Monitor" `
    -Name "AppInsights-WebApp"

# Get instrumentation key
$ai = Get-AzApplicationInsights -ResourceGroupName "RG-Monitor" `
    -Name "AppInsights-WebApp"
$ai.InstrumentationKey

# Set daily cap
Update-AzApplicationInsights -ResourceGroupName "RG-Monitor" `
    -Name "AppInsights-WebApp" `
    -IngestionMode "LogAnalytics" `
    -RetentionInDays 90

# Run query
$query = @"
requests
| where timestamp > ago(24h)
| summarize count() by resultCode
| order by count_ desc
"@

Invoke-AzOperationalInsightsQuery -WorkspaceId $ai.WorkspaceResourceId `
    -Query $query
```

### Alerts

```powershell
# Create action group
$email = New-AzActionGroupReceiver -Name "EmailAdmin" `
    -EmailReceiver `
    -EmailAddress "admin@contoso.com"

$sms = New-AzActionGroupReceiver -Name "SMSOnCall" `
    -SmsReceiver `
    -CountryCode "1" `
    -PhoneNumber "5550100"

Set-AzActionGroup -ResourceGroupName "RG-Monitor" `
    -Name "Production-Critical" `
    -ShortName "ProdCrit" `
    -Receiver $email, $sms

# Get action group
Get-AzActionGroup -ResourceGroupName "RG-Monitor" `
    -Name "Production-Critical"

# Create metric alert rule
$vm = Get-AzVM -ResourceGroupName "RG-Monitor" -Name "WebServer01"
$actionGroup = Get-AzActionGroup -ResourceGroupName "RG-Monitor" `
    -Name "Production-Critical"

$condition = New-AzMetricAlertRuleV2Criteria -MetricName "Percentage CPU" `
    -TimeAggregation Average `
    -Operator GreaterThan `
    -Threshold 80

Add-AzMetricAlertRuleV2 -ResourceGroupName "RG-Monitor" `
    -Name "HighCPU-WebServer01" `
    -Description "Alert when CPU > 80%" `
    -Severity 2 `
    -WindowSize 00:05:00 `
    -Frequency 00:01:00 `
    -TargetResourceId $vm.Id `
    -Condition $condition `
    -ActionGroupId $actionGroup.Id

# Create activity log alert
$condition = New-AzActivityLogAlertCondition -Field "operationName" `
    -Equal "Microsoft.Compute/virtualMachines/delete"

$actionGroup = Get-AzActionGroup -ResourceGroupName "RG-Monitor" `
    -Name "Production-Critical"

Set-AzActivityLogAlert -ResourceGroupName "RG-Monitor" `
    -Name "VMDeleted" `
    -Condition $condition `
    -Scope "/subscriptions/{subscription-id}" `
    -ActionGroupId $actionGroup.Id `
    -Enabled $true

# Disable alert
Update-AzMetricAlertRuleV2 -ResourceGroupName "RG-Monitor" `
    -Name "HighCPU-WebServer01" `
    -DisableRule

# Delete alert
Remove-AzMetricAlertRuleV2 -ResourceGroupName "RG-Monitor" `
    -Name "HighCPU-WebServer01"
```

### Azure Backup

```powershell
# Create Recovery Services vault
New-AzRecoveryServicesVault -ResourceGroupName "RG-Backup" `
    -Name "BackupVault" `
    -Location "eastus"

# Get vault
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "RG-Backup" `
    -Name "BackupVault"

# Set vault context
Set-AzRecoveryServicesVaultContext -Vault $vault

# Set storage redundancy (before first backup)
Set-AzRecoveryServicesBackupProperty -Vault $vault `
    -BackupStorageRedundancy GeoRedundant

# Get default policy
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"

# Create custom backup policy
$schedule = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType AzureVM
$schedule.ScheduleRunTimes.Clear()
$schedule.ScheduleRunTimes.Add((Get-Date -Hour 2 -Minute 0 -Second 0).ToUniversalTime())

$retention = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType AzureVM
$retention.DailySchedule.DurationCountInDays = 7
$retention.WeeklySchedule.DurationCountInWeeks = 4
$retention.MonthlySchedule.DurationCountInMonths = 12
$retention.YearlySchedule.DurationCountInYears = 5

New-AzRecoveryServicesBackupProtectionPolicy -Name "CustomVMPolicy" `
    -WorkloadType AzureVM `
    -RetentionPolicy $retention `
    -SchedulePolicy $schedule

# Enable backup for VM
$vm = Get-AzVM -ResourceGroupName "RG-VMs" -Name "WebServer01"
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "CustomVMPolicy"

Enable-AzRecoveryServicesBackupProtection -ResourceGroupName "RG-VMs" `
    -Name "WebServer01" `
    -Policy $policy

# Trigger backup immediately
$container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM `
    -FriendlyName "WebServer01"
$item = Get-AzRecoveryServicesBackupItem -Container $container `
    -WorkloadType AzureVM

Backup-AzRecoveryServicesBackupItem -Item $item

# Get backup jobs
Get-AzRecoveryServicesBackupJob -Status InProgress

# Get recovery points
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $item

# Restore VM disks
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] `
    -StorageAccountName "restorestorage" `
    -StorageAccountResourceGroupName "RG-Restore"

# Monitor restore job
Wait-AzRecoveryServicesBackupJob -Job $restorejob -Timeout 43200

# Disable backup
Disable-AzRecoveryServicesBackupProtection -Item $item `
    -RemoveRecoveryPoints `
    -Force
```

### Azure Site Recovery

```powershell
# Create Recovery Services vault (if not exists)
$vault = New-AzRecoveryServicesVault -ResourceGroupName "RG-DR" `
    -Name "DRVault" `
    -Location "westus"

Set-AzRecoveryServicesAsrVaultContext -Vault $vault

# Get replication fabrics
$fabrics = Get-AzRecoveryServicesAsrFabric

# Get protection containers
$container = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $fabrics[0]

# Get replication policy
$policy = Get-AzRecoveryServicesAsrPolicy

# Enable replication for VM
$vm = Get-AzVM -ResourceGroupName "RG-Prod" -Name "WebServer01"

New-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $container `
    -Name "WebServer01" `
    -RecoveryAzureStorageAccountId "/subscriptions/{sub-id}/resourceGroups/RG-DR/providers/Microsoft.Storage/storageAccounts/drstorage" `
    -RecoveryResourceGroupId "/subscriptions/{sub-id}/resourceGroups/RG-DR"

# Get replicated items
$replicatedItem = Get-AzRecoveryServicesAsrReplicationProtectedItem `
    -ProtectionContainer $container `
    -Name "WebServer01"

# Test failover
$recoveryPoint = Get-AzRecoveryServicesAsrRecoveryPoint -ReplicationProtectedItem $replicatedItem | Select-Object -First 1

Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $replicatedItem `
    -Direction PrimaryToRecovery `
    -RecoveryPoint $recoveryPoint

# Clean up test failover
Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $replicatedItem

# Planned failover
Start-AzRecoveryServicesAsrPlannedFailoverJob -ReplicationProtectedItem $replicatedItem `
    -Direction PrimaryToRecovery `
    -RecoveryPoint $recoveryPoint

# Unplanned failover
Start-AzRecoveryServicesAsrUnplannedFailoverJob -ReplicationProtectedItem $replicatedItem `
    -Direction PrimaryToRecovery `
    -RecoveryPoint $recoveryPoint

# Commit failover
Start-AzRecoveryServicesAsrCommitFailoverJob -ReplicationProtectedItem $replicatedItem
```

### Cost Management

```powershell
# Get current subscription
$subscription = Get-AzSubscription -SubscriptionName "Production"
Set-AzContext -Subscription $subscription

# Get cost for current month
$startDate = (Get-Date -Day 1).ToString("yyyy-MM-dd")
$endDate = (Get-Date).ToString("yyyy-MM-dd")

# Note: Cost Management cmdlets require Az.CostManagement module
# Install: Install-Module -Name Az.CostManagement

# Get cost
$costs = Get-AzCostManagementCostData -Scope "/subscriptions/$($subscription.Id)" `
    -Timeframe "Custom" `
    -TimeStart $startDate `
    -TimeEnd $endDate

# Create budget
New-AzConsumptionBudget -Name "Monthly-Budget" `
    -Amount 5000 `
    -Category Cost `
    -TimeGrain Monthly `
    -StartDate (Get-Date -Day 1) `
    -EndDate (Get-Date -Day 1).AddYears(1) `
    -ContactEmail "finance@contoso.com" `
    -NotificationThreshold 80, 100

# Get budgets
Get-AzConsumptionBudget

# Remove budget
Remove-AzConsumptionBudget -Name "Monthly-Budget"
```

---

## Azure CLI Commands Reference

### Azure Monitor

```bash
# Get metric definitions
az monitor metrics list-definitions \
    --resource "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.Compute/virtualMachines/WebServer01"

# Get metrics
az monitor metrics list \
    --resource "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.Compute/virtualMachines/WebServer01" \
    --metric "Percentage CPU" \
    --start-time 2024-01-01T00:00:00Z \
    --end-time 2024-01-02T00:00:00Z

# Create diagnostic settings
az monitor diagnostic-settings create \
    --resource "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.Compute/virtualMachines/WebServer01" \
    --name "DiagSettings" \
    --workspace "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.OperationalInsights/workspaces/LogAnalytics" \
    --logs '[{"category": "Administrative","enabled": true}]' \
    --metrics '[{"category": "AllMetrics","enabled": true}]'

# Get diagnostic settings
az monitor diagnostic-settings show \
    --resource "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.Compute/virtualMachines/WebServer01" \
    --name "DiagSettings"

# List diagnostic settings
az monitor diagnostic-settings list \
    --resource "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.Compute/virtualMachines/WebServer01"

# Delete diagnostic settings
az monitor diagnostic-settings delete \
    --resource "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.Compute/virtualMachines/WebServer01" \
    --name "DiagSettings"
```

### Log Analytics

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
    --resource-group "RG-Monitor" \
    --workspace-name "LogAnalytics-Prod" \
    --location "eastus" \
    --sku "PerGB2018"

# Get workspace
az monitor log-analytics workspace show \
    --resource-group "RG-Monitor" \
    --workspace-name "LogAnalytics-Prod"

# List workspaces
az monitor log-analytics workspace list \
    --resource-group "RG-Monitor"

# Get workspace keys
az monitor log-analytics workspace get-shared-keys \
    --resource-group "RG-Monitor" \
    --workspace-name "LogAnalytics-Prod"

# Run query
az monitor log-analytics query \
    --workspace "customer-id-here" \
    --analytics-query "AzureActivity | where TimeGenerated > ago(24h) | where ActivityStatusValue == 'Failure' | project TimeGenerated, Caller, OperationNameValue | order by TimeGenerated desc" \
    --timespan "P1D"

# Update retention
az monitor log-analytics workspace update \
    --resource-group "RG-Monitor" \
    --workspace-name "LogAnalytics-Prod" \
    --retention-time 90

# Delete workspace
az monitor log-analytics workspace delete \
    --resource-group "RG-Monitor" \
    --workspace-name "LogAnalytics-Prod" \
    --yes
```

### Application Insights

```bash
# Create Application Insights
az monitor app-insights component create \
    --app "AppInsights-WebApp" \
    --location "eastus" \
    --resource-group "RG-Monitor" \
    --application-type "web" \
    --kind "web"

# Get Application Insights
az monitor app-insights component show \
    --app "AppInsights-WebApp" \
    --resource-group "RG-Monitor"

# Get instrumentation key
az monitor app-insights component show \
    --app "AppInsights-WebApp" \
    --resource-group "RG-Monitor" \
    --query "instrumentationKey" \
    --output tsv

# Update retention
az monitor app-insights component update \
    --app "AppInsights-WebApp" \
    --resource-group "RG-Monitor" \
    --retention-time 90

# Query Application Insights
az monitor app-insights metrics show \
    --app "AppInsights-WebApp" \
    --resource-group "RG-Monitor" \
    --metric "requests/count" \
    --aggregation "count"

# Delete Application Insights
az monitor app-insights component delete \
    --app "AppInsights-WebApp" \
    --resource-group "RG-Monitor"
```

### Alerts

```bash
# Create action group
az monitor action-group create \
    --resource-group "RG-Monitor" \
    --name "Production-Critical" \
    --short-name "ProdCrit" \
    --email-receiver name="EmailAdmin" email-address="admin@contoso.com" \
    --sms-receiver name="SMSOnCall" country-code="1" phone-number="5550100"

# Get action group
az monitor action-group show \
    --resource-group "RG-Monitor" \
    --name "Production-Critical"

# List action groups
az monitor action-group list --resource-group "RG-Monitor"

# Create metric alert
az monitor metrics alert create \
    --resource-group "RG-Monitor" \
    --name "HighCPU-WebServer01" \
    --description "Alert when CPU > 80%" \
    --severity 2 \
    --window-size 5m \
    --evaluation-frequency 1m \
    --scopes "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/Microsoft.Compute/virtualMachines/WebServer01" \
    --condition "avg Percentage CPU > 80" \
    --action "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/microsoft.insights/actionGroups/Production-Critical"

# Create activity log alert
az monitor activity-log alert create \
    --resource-group "RG-Monitor" \
    --name "VMDeleted" \
    --description "Alert when VM is deleted" \
    --scope "/subscriptions/{subscription-id}" \
    --condition category=Administrative and operationName=Microsoft.Compute/virtualMachines/delete \
    --action-group "/subscriptions/{sub-id}/resourceGroups/RG-Monitor/providers/microsoft.insights/actionGroups/Production-Critical"

# List alerts
az monitor metrics alert list --resource-group "RG-Monitor"

# Update alert (disable)
az monitor metrics alert update \
    --resource-group "RG-Monitor" \
    --name "HighCPU-WebServer01" \
    --enabled false

# Delete alert
az monitor metrics alert delete \
    --resource-group "RG-Monitor" \
    --name "HighCPU-WebServer01"
```

### Azure Backup

```bash
# Create Recovery Services vault
az backup vault create \
    --resource-group "RG-Backup" \
    --name "BackupVault" \
    --location "eastus"

# Get vault
az backup vault show \
    --resource-group "RG-Backup" \
    --name "BackupVault"

# Set storage redundancy
az backup vault backup-properties set \
    --resource-group "RG-Backup" \
    --name "BackupVault" \
    --backup-storage-redundancy "GeoRedundant"

# Get default backup policy
az backup policy show \
    --resource-group "RG-Backup" \
    --vault-name "BackupVault" \
    --name "DefaultPolicy"

# Enable backup for VM
az backup protection enable-for-vm \
    --resource-group "RG-Backup" \
    --vault-name "BackupVault" \
    --vm "WebServer01" \
    --policy-name "DefaultPolicy"

# Trigger backup immediately
az backup protection backup-now \
    --resource-group "RG-Backup" \
    --vault-name "BackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VMs;WebServer01" \
    --item-name "vm;iaasvmcontainerv2;RG-VMs;WebServer01" \
    --retain-until "31-12-2024"

# List backup jobs
az backup job list \
    --resource-group "RG-Backup" \
    --vault-name "BackupVault" \
    --output table

# List recovery points
az backup recoverypoint list \
    --resource-group "RG-Backup" \
    --vault-name "BackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VMs;WebServer01" \
    --item-name "vm;iaasvmcontainerv2;RG-VMs;WebServer01" \
    --output table

# Restore VM
az backup restore restore-disks \
    --resource-group "RG-Backup" \
    --vault-name "BackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VMs;WebServer01" \
    --item-name "vm;iaasvmcontainerv2;RG-VMs;WebServer01" \
    --rp-name "<recovery-point-name>" \
    --storage-account "restorestorage" \
    --target-resource-group "RG-Restore"

# Disable backup
az backup protection disable \
    --resource-group "RG-Backup" \
    --vault-name "BackupVault" \
    --container-name "iaasvmcontainer;iaasvmcontainerv2;RG-VMs;WebServer01" \
    --item-name "vm;iaasvmcontainerv2;RG-VMs;WebServer01" \
    --delete-backup-data true \
    --yes
```

### Cost Management

```bash
# Get current subscription
az account show --query name

# Get cost for current month
az consumption usage list \
    --start-date 2024-01-01 \
    --end-date 2024-01-31

# Create budget
az consumption budget create \
    --budget-name "Monthly-Budget" \
    --amount 5000 \
    --time-grain Monthly \
    --start-date 2024-01-01 \
    --end-date 2025-01-01 \
    --category Cost \
    --notifications "Actual_GreaterThan_80_Percent={\"enabled\":true,\"operator\":\"GreaterThan\",\"threshold\":80,\"contactEmails\":[\"finance@contoso.com\"],\"contactRoles\":[\"Owner\"],\"thresholdType\":\"Actual\"}"

# List budgets
az consumption budget list

# Delete budget
az consumption budget delete \
    --budget-name "Monthly-Budget"
```

---

## Common Exam Scenarios

### Scenario 1: High CPU Alerts Not Triggering

**Problem:**
Created metric alert for CPU > 80%, but not receiving notifications even though CPU is high.

**Troubleshooting:**

1. **Check alert rule configuration:**
   ```
   Time aggregation: Average (not Maximum)
   Window size: 5 minutes
   Evaluation frequency: 1 minute
   Threshold: 80%
   
   Issue: CPU spikes to 90% for 2 minutes, but 5-minute average is only 75%
   Solution: Change time aggregation to Maximum or reduce window size
   ```

2. **Check action group:**
   ```
   Is action group attached to alert rule?
   Are email addresses correct?
   Check spam folder
   Verify SMS number format
   ```

3. **Check alert state:**
   ```
   State: Resolved (condition cleared before evaluation)
   Solution: Alert working correctly, CPU returned to normal
   ```

4. **Check metric availability:**
   ```
   Metrics delayed by 2-3 minutes
   Alert evaluates before metrics arrive
   Solution: Normal behavior, wait for next evaluation
   ```

### Scenario 2: Log Analytics Query Returns No Results

**Problem:**
KQL query returns no results, but data should exist.

**Troubleshooting:**

1. **Check time range:**
   ```kql
   // Too restrictive
   | where TimeGenerated > ago(1h)
   
   // Correct
   | where TimeGenerated > ago(24h)
   ```

2. **Check table name:**
   ```kql
   // Wrong
   AzureActivityLog
   
   // Correct
   AzureActivity
   ```

3. **Check diagnostic settings:**
   ```
   Are logs being sent to Log Analytics workspace?
   Check diagnostic settings on resource
   Wait 5-10 minutes for first logs to appear
   ```

4. **Check workspace:**
   ```
   Querying correct workspace?
   Data in different workspace?
   Check workspace permissions
   ```

5. **Check syntax:**
   ```kql
   // Wrong (case sensitive)
   | where caller == "John@Contoso.com"
   
   // Correct
   | where Caller == "john@contoso.com"
   ```

### Scenario 3: Backup Failing

**Problem:**
VM backup consistently fails with error.

**Common causes and solutions:**

**1. VM agent not responding:**
```
Cause: VM agent not installed or not running
Solution: 
- Install/update VM agent
- Restart VM
- Check agent status in VM properties
```

**2. Insufficient permissions:**
```
Cause: Backup service lacks permissions
Solution:
- Verify Recovery Services vault has proper role assignments
- Ensure VM has system-assigned managed identity
```

**3. Snapshot timeout:**
```
Cause: VM busy, snapshot taking too long
Solution:
- Reduce disk I/O during backup window
- Change backup schedule to off-peak hours
- Use enhanced backup policy
```

**4. Storage account issues:**
```
Cause: Snapshot storage account inaccessible
Solution:
- Check storage account firewall rules
- Verify network connectivity
- Ensure storage account in same region
```

### Scenario 4: Application Insights Not Collecting Data

**Problem:**
Application deployed but no telemetry in Application Insights.

**Troubleshooting:**

**1. Instrumentation key:**
```
Check: Is correct instrumentation key configured?
Location: appsettings.json, environment variables, or portal
Test: Log instrumentation key from application
```

**2. SDK installed:**
```
Check: Is Application Insights SDK installed?
.NET: Microsoft.ApplicationInsights.AspNetCore package
Node.js: applicationinsights package
```

**3. Firewall/proxy:**
```
Application Insights endpoints:
- dc.services.visualstudio.com
- dc.applicationinsights.microsoft.com

Check: Can application reach these endpoints?
Test: curl or wget from application server
```

**4. Sampling:**
```
Check: Is sampling enabled?
Default: 100% sampling
If custom: May be dropping telemetry
Solution: Adjust sampling settings
```

**5. Data delay:**
```
Normal: 2-3 minutes delay
During high load: Up to 5-10 minutes
Solution: Wait and refresh
```

### Scenario 5: Cost Unexpectedly High

**Problem:**
Azure bill much higher than expected.

**Investigation steps:**

**1. Cost analysis by service:**
```
Portal → Cost Management → Cost Analysis
Group by: Service name

Example findings:
- Compute: $2,500 (expected: $1,000)
- Storage: $800 (expected: $200)
```

**2. Drill into expensive services:**
```
Compute → Group by: Resource
Findings:
- 10 VMs stopped but not deallocated: $1,500
- Oversized VMs: $500

Actions:
- Deallocate stopped VMs: Save $1,500/month
- Right-size VMs: Save $300/month
```

**3. Check for orphaned resources:**
```
Unattached disks: 50 disks × $5 = $250
Unused snapshots: 100 snapshots × $2 = $200
Old backups: $150

Action: Clean up unused resources: Save $600/month
```

**4. Review recent changes:**
```
Activity Log → Last 30 days
- New VM scale set created (accidental): $500/month
- Database tier upgraded (forgotten): $400/month

Actions: Remove/downgrade
```

### Scenario 6: Site Recovery Failover Not Working

**Problem:**
Test failover failing.

**Common issues:**

**1. Replication not complete:**
```
Check: Replication status
Status: Initial replication in progress (45% complete)
Solution: Wait for initial replication to complete (can take hours)
```

**2. Target network not configured:**
```
Error: No virtual network selected for target
Solution: Configure target network settings in replication configuration
```

**3. Insufficient capacity in target region:**
```
Error: VM size not available in target region
Solution: 
- Choose different target region
- Select different VM size
- Request quota increase
```

**4. NSG blocking replication:**
```
Check: NSG rules on source and target
Ensure: Port 443 outbound allowed for Site Recovery traffic
Solution: Update NSG rules
```

### Scenario 7: Alert Fatigue

**Problem:**
Too many alerts, team ignoring them.

**Solutions:**

**1. Adjust thresholds:**
```
Current: CPU > 70% → 500 alerts/day
Better: CPU > 85% for 10 minutes → 10 alerts/day
```

**2. Use dynamic thresholds:**
```
Instead of: Static threshold (80%)
Use: Machine learning-based dynamic threshold
Benefit: Adapts to normal patterns, fewer false positives
```

**3. Group related alerts:**
```
Before: 50 separate alerts (one per VM)
After: 1 alert with multi-resource scope
Result: One notification covering all VMs
```

**4. Adjust severity:**
```
Review: Which alerts truly need immediate attention?
Downgrade: Informational alerts to Sev 3 (no notification)
Keep Sev 0: Only for production-down scenarios
```

**5. Use alert suppression:**
```
During maintenance windows: Suppress alerts
During deployments: Suppress alerts for 30 minutes
Scheduled downtime: Disable alert rules temporarily
```

### Scenario 8: Choose Monitoring Solution

**Problem:**
Which monitoring service to use?

**Decision matrix:**

**Scenario A: Infrastructure monitoring**
```
Resources: VMs, storage, network
Need: CPU, memory, disk metrics
Solution: Azure Monitor + Log Analytics
Cost: Low
```

**Scenario B: Application performance**
```
Resources: Web app, API
Need: Request rates, response times, dependencies
Solution: Application Insights
Cost: Medium
```

**Scenario C: Custom metrics**
```
Resources: Custom application
Need: Business metrics, custom counters
Solution: Application Insights (custom metrics)
Alternative: Azure Monitor custom metrics API
Cost: Based on volume
```

**Scenario D: Log aggregation**
```
Resources: Multiple services, on-premises servers
Need: Centralized log storage and analysis
Solution: Log Analytics workspace
Cost: Based on data ingestion (GB/day)
```

**Scenario E: Cost tracking**
```
Resources: All Azure resources
Need: Cost analysis, budgets, forecasts
Solution: Azure Cost Management
Cost: Free
```

---

## Practice Questions

### Question 1

You need to receive an email when CPU usage exceeds 80% for more than 5 minutes. What should you configure?

**A)** Log Analytics query alert  
**B)** Metric alert rule  
**C)** Activity log alert  
**D)** Application Insights smart detection

**Answer:** B

**Explanation:**
- Metric alert rule monitors metric values (CPU percentage)
- Can specify time aggregation (5 minutes)
- Triggers action group (email notification)
- A incorrect: Log query alerts for complex KQL queries, not simple metrics
- C incorrect: Activity log for management operations, not metrics
- D incorrect: Smart detection for Application Insights anomalies

### Question 2

Your company requires 90 days of log retention for compliance. Where should you store diagnostic logs?

**A)** Activity log (default 90-day retention)  
**B)** Log Analytics workspace (default 30 days)  
**C)** Storage Account with lifecycle management  
**D)** Event Hub

**Answer:** C

**Explanation:**
- Storage Account provides long-term retention at low cost
- Can configure retention period (90+ days)
- Lifecycle management for cost optimization
- A incorrect: Activity log is only subscription-level, not resource logs
- B incorrect: Default is 30 days, need to increase (additional cost)
- D incorrect: Event Hub for streaming, not long-term storage

### Question 3

You enabled Azure Backup for a VM but the first backup failed with "VM agent not responding". What should you do first?

**A)** Reinstall the VM agent  
**B)** Restart the VM  
**C)** Change backup policy  
**D)** Create a new Recovery Services vault

**Answer:** B

**Explanation:**
- VM agent issues often resolved by VM restart
- Simplest solution to try first
- No data loss, minimal downtime
- A incorrect: Restart first before reinstalling
- C incorrect: Policy not related to agent issues
- D incorrect: Vault configuration not the problem

### Question 4

You need to monitor application dependencies and identify bottlenecks. Which service should you use?

**A)** Azure Monitor metrics  
**B)** Log Analytics  
**C)** Application Insights  
**D)** Network Watcher

**Answer:** C

**Explanation:**
- Application Insights provides Application Map showing dependencies
- Tracks dependency calls (databases, APIs, etc.)
- Shows response times and failures
- A incorrect: Metrics don't show application dependencies
- B incorrect: Log Analytics stores logs but no built-in dependency visualization
- D incorrect: Network Watcher for network troubleshooting, not application performance

### Question 5

Your backup costs are high. What can you do to reduce costs while maintaining protection?

**A)** Disable soft delete  
**B)** Reduce retention period  
**C)** Use Locally Redundant storage instead of Geo-Redundant  
**D)** All of the above

**Answer:** D

**Explanation:**
- All options reduce backup costs
- Soft delete: Saves storage by not keeping deleted backups 14 extra days
- Shorter retention: Less storage needed
- LRS vs GRS: LRS is cheaper (but less resilient)
- Trade-off: Cost vs protection level (evaluate based on requirements)

### Question 6

You want to automatically scale VMs based on CPU usage. What should you configure?

**A)** Metric alert with autoscale action  
**B)** VM Scale Set with autoscale rules  
**C)** Log Analytics query with automation runbook  
**D)** Application Insights availability test

**Answer:** B

**Explanation:**
- VM Scale Set has built-in autoscaling
- Can scale based on CPU, memory, or custom metrics
- Automatically adds/removes VM instances
- A incorrect: Alerts notify, don't automatically scale
- C incorrect: Complex, not built-in autoscaling
- D incorrect: Availability tests for monitoring, not scaling

### Question 7

You need to restore a single file from yesterday's VM backup. What's the fastest method?

**A)** Restore entire VM, then copy file  
**B)** Restore disks, attach to VM, copy file  
**C)** Use file-level restore feature  
**D)** Create new VM from backup

**Answer:** C

**Explanation:**
- File-level restore mounts backup as iSCSI drive
- Browse and copy specific files
- No full VM restore needed (fastest)
- A incorrect: Restoring entire VM takes hours
- B incorrect: Slower than file-level restore
- D incorrect: Same as A, unnecessary full restore

### Question 8

Your Application Insights bill is high. What can reduce costs without losing important data?

**A)** Increase sampling rate  
**B)** Disable telemetry collection  
**C)** Enable adaptive sampling  
**D)** Delete Application Insights resource

**Answer:** C

**Explanation:**
- Adaptive sampling automatically adjusts data collection
- Reduces volume during high traffic
- Keeps important telemetry (errors, slow requests)
- A incorrect: Increasing sampling means MORE data (higher cost)
- B incorrect: Loses all telemetry
- D incorrect: Loses all monitoring capability

---

## Key Takeaways for Exam

### Must-Know Concepts

**1. Monitoring Data Types**
- Metrics: Numerical, near real-time, 93-day retention
- Logs: Detailed records, KQL queries, configurable retention
- Activity Log: Subscription-level, 90-day retention
- Resource Logs: Must enable diagnostic settings

**2. Log Analytics**
- Workspace: Container for log data
- KQL: Query language for logs
- Tables: AzureActivity, AzureDiagnostics, Perf, Event
- Retention: 30 days default, configurable

**3. Application Insights**
- APM for applications
- Request rates, response times, dependencies
- Application Map, Live Metrics, Failures
- SDK or agent-based instrumentation

**4. Alerts**
- Types: Metric, Log Query, Activity Log, Smart Detection
- Components: Scope, Condition, Action Group
- Severity: 0 (critical) to 4 (verbose)
- States: New, Acknowledged, Closed

**5. Azure Backup**
- Recovery Services vault
- VM backup: Application-consistent (Windows), file-system consistent (Linux)
- Retention: Daily, weekly, monthly, yearly
- Restore options: New VM, replace disk, disks only, file-level

**6. Site Recovery**
- Disaster recovery between regions
- RPO: 30 seconds - 5 minutes
- Failover types: Test, Planned, Unplanned
- Recovery plans: Orchestrate multi-VM failover

**7. Cost Management**
- Cost Analysis: View spending patterns
- Budgets: Set limits and alerts
- Recommendations: Azure Advisor
- Tags: Track costs by department/project

### Common Exam Tricks

**Trick 1: "Backup failed, what to try first?"**
- Always try simplest solution: Restart VM
- Don't jump to complex solutions (reinstall agent, recreate vault)

**Trick 2: "Alert not triggering"**
- Check time aggregation (Average vs Maximum)
- Check window size vs evaluation frequency
- Verify action group attached

**Trick 3: "Need 90-day log retention"**
- Default is 30 days (Log Analytics)
- Must configure longer retention (additional cost)
- Or use Storage Account

**Trick 4: "High backup costs"**
- Soft delete adds 14 days retention
- GRS more expensive than LRS
- Shorter retention = lower cost

**Trick 5: "KQL query returns nothing"**
- Check time range (ago(1h) vs ago(24h))
- Check table name (case-sensitive)
- Check diagnostic settings enabled

**Trick 6: "Choose monitoring service"**
- Metrics = Azure Monitor
- Application performance = Application Insights
- Logs = Log Analytics
- Costs = Cost Management

**Trick 7: "Restore single file"**
- Don't restore entire VM
- Use file-level restore (fastest)

**Trick 8: "Site Recovery vs Backup"**
- Backup: Point-in-time, hours RPO
- Site Recovery: Near real-time, minutes RPO
- Use both for comprehensive protection

### Quick Reference Tables

**Monitoring Services:**

| Need | Service |
|------|---------|
| Infrastructure metrics | Azure Monitor |
| Log aggregation | Log Analytics |
| Application performance | Application Insights |
| Cost tracking | Cost Management |
| Network diagnostics | Network Watcher |

**Alert Types:**

| Type | Use Case | Evaluation |
|------|----------|------------|
| Metric | CPU, memory, disk | 1-5 minutes |
| Log Query | Complex KQL queries | 1-60 minutes |
| Activity Log | Resource operations | Near real-time |
| Smart Detection | Application anomalies | Continuous |

**Backup vs Site Recovery:**

| Feature | Azure Backup | Site Recovery |
|---------|--------------|---------------|
| Purpose | Data protection | Disaster recovery |
| RPO | Hours (24h default) | Minutes (30s-5m) |
| RTO | Hours | Hours |
| Cost | Lower | Higher |
| Use case | File recovery | Region failover |

**Log Retention:**

| Service | Default | Maximum | Cost |
|---------|---------|---------|------|
| Activity Log | 90 days | 90 days | Free |
| Log Analytics | 30 days | 730 days | Pay per GB |
| Application Insights | 90 days | 730 days | Pay per GB |
| Storage Account | Unlimited | Unlimited | Pay per GB |

---

## Summary Checklist

Before completing your AZ-104 preparation, ensure you can:

**Azure Monitor:**
- [ ] Explain metrics vs logs
- [ ] Configure diagnostic settings
- [ ] Use Metrics Explorer
- [ ] Understand Activity Log

**Log Analytics:**
- [ ] Create workspace
- [ ] Write basic KQL queries
- [ ] Understand common tables
- [ ] Configure data retention

**Application Insights:**
- [ ] Instrument applications
- [ ] Use Application Map
- [ ] Analyze performance and failures
- [ ] Understand sampling

**Alerts:**
- [ ] Create metric alert rules
- [ ] Create log query alerts
- [ ] Configure action groups
- [ ] Set appropriate severity levels

**Azure Backup:**
- [ ] Create Recovery Services vault
- [ ] Configure VM backup
- [ ] Understand backup policies
- [ ] Perform file-level restore

**Site Recovery:**
- [ ] Enable VM replication
- [ ] Understand RPO and RTO
- [ ] Perform test failover
- [ ] Create recovery plans

**Cost Management:**
- [ ] Use Cost Analysis
- [ ] Create budgets
- [ ] Apply cost-saving recommendations
- [ ] Use tags for cost tracking

---

## Additional Resources

**Microsoft Learn Modules:**
- [Monitor Azure resources](https://learn.microsoft.com/training/modules/monitor-azure-resources/)
- [Configure Log Analytics](https://learn.microsoft.com/training/modules/configure-log-analytics/)
- [Monitor applications with Application Insights](https://learn.microsoft.com/training/modules/monitor-app-performance/)
- [Configure Azure Backup](https://learn.microsoft.com/training/modules/configure-virtual-machine-backups/)
- [Configure Azure Site Recovery](https://learn.microsoft.com/training/modules/protect-virtual-machines-with-azure-site-recovery/)
- [Cost management and optimization](https://learn.microsoft.com/training/modules/plan-manage-azure-costs/)

**Documentation:**
- [Azure Monitor](https://learn.microsoft.com/azure/azure-monitor/)
- [Log Analytics](https://learn.microsoft.com/azure/azure-monitor/logs/log-analytics-overview)
- [Application Insights](https://learn.microsoft.com/azure/azure-monitor/app/app-insights-overview)
- [Azure Backup](https://learn.microsoft.com/azure/backup/)
- [Site Recovery](https://learn.microsoft.com/azure/site-recovery/)
- [Cost Management](https://learn.microsoft.com/azure/cost-management-billing/)

**Hands-on Labs:**
- Create Log Analytics workspace and run KQL queries
- Configure VM backup and perform restore
- Set up Application Insights for web app
- Create metric and log query alerts
- Configure Site Recovery replication
- Analyze costs and create budgets

---

## Final Exam Preparation

**Congratulations!** You've completed all 5 modules of the AZ-104 study guide.

**Complete Coverage:**
- Module 1: Identity & Governance ✅ (15-20%)
- Module 2: Storage ✅ (15-20%)
- Module 3: Compute ✅ (20-25%)
- Module 4: Networking ✅ (25-30%)
- Module 5: Monitoring & Backup ✅ (10-15%)

**Total: 100% of exam content covered!**

### Final Week Study Plan

**Day 1-2: Identity & Governance + Storage**
- Review Azure AD, RBAC, subscriptions
- Review storage accounts, blob storage, Azure Files
- Practice questions from both modules

**Day 3-4: Compute + Networking**
- Review VMs, availability, scaling, App Service
- Review VNets, NSGs, load balancing, VPN
- Practice questions from both modules

**Day 5: Monitoring + Backup**
- Review Azure Monitor, Log Analytics, alerts
- Review backup and Site Recovery
- Practice questions

**Day 6: Hands-on Labs**
- Create complete 3-tier application
- Configure monitoring and alerts
- Set up backup and DR
- Practice troubleshooting scenarios

**Day 7: Final Review**
- Review all "Key Takeaways" sections
- Review all "Exam Tricks"
- Take practice exam
- Review weak areas

### Exam Day Tips

**Before the exam:**
- Good night's sleep
- Eat breakfast
- Arrive early (online: test environment 30 min before)
- Have water and snacks ready

**During the exam:**
- Read questions carefully (watch for "NOT", "EXCEPT")
- Eliminate obviously wrong answers first
- Flag difficult questions, come back later
- Manage time: 60 questions in 100 minutes = 1.5 min/question
- Don't overthink - first instinct often correct

**Question types:**
- Multiple choice (one answer)
- Multiple response (multiple answers)
- Drag-and-drop (order or match items)
- Case studies (scenario with multiple questions)

**Common question patterns:**
- "What should you do FIRST?" → Simplest solution
- "Least cost" → Basic tier, smaller size, LRS
- "Highest availability" → Zones, redundancy, multiple regions
- "Minimize downtime" → Blue-green deployment, test failover

### Passing the Exam

**Passing score:** 700 out of 1000 (approximately 70%)

**Exam structure:**
- 40-60 questions
- 100 minutes (adjusted for non-native English speakers)
- Can mark questions for review
- Case studies count for multiple questions

**If you don't pass:**
- Wait 24 hours to retake
- Review score report (shows weak areas)
- Focus study on weak areas
- Take more practice exams

### After Passing

**Next steps:**
- Update LinkedIn with certification
- Share accomplishment
- Real-world experience (most important!)
- Consider advanced certifications:
  - AZ-305: Azure Solutions Architect Expert
  - AZ-400: DevOps Engineer Expert
  - AZ-500: Azure Security Engineer

---

## Final Words

You've covered all essential topics for the AZ-104 exam. The key to success is:

1. **Understand concepts** (not just memorize)
2. **Practice hands-on** (theory + practice = mastery)
3. **Review regularly** (spaced repetition)
4. **Take practice exams** (identify weak areas)
5. **Stay calm** (you're well prepared!)

**Remember:**
- Azure is constantly evolving - keep learning
- Certification proves knowledge, but experience proves capability
- The learning doesn't stop after the exam
- Join Azure communities, read blogs, watch videos

**Good luck with your AZ-104 exam!**

You've put in the work, you know the material, and you're ready to succeed.

---

**[← Back to Module 4: Networking](../Module-04-Networking/README.md) | [🏠 Back to Main README](../README.md)**
