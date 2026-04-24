# Step 05 - Azure Monitor, Log Analytics, and Alert Configuration

**AZ-104 Domain:** Monitor and Maintain Azure Resources
**Last reviewed:** 2026-07

---

## The Azure Monitoring Stack - Understanding the Components

Azure monitoring consists of three interconnected components that are frequently
confused with each other in exam questions and real-world discussions:

**Azure Monitor:** The overarching platform that collects metrics (numeric time-series
data like CPU %, memory %, disk I/O) and logs (structured event data). Think of it as
the collection and routing layer.

**Log Analytics Workspace:** The database where logs are stored and where KQL queries
run. Azure Monitor routes logs to the Log Analytics Workspace. You query the workspace
to investigate events.

**Diagnostic Settings:** The configuration that tells Azure Monitor what to collect
from each resource and where to send it. Without Diagnostic Settings, Azure Monitor
collects only a limited set of platform metrics. Diagnostic Settings enable rich log
collection (Windows Event Logs, Syslog, performance counters, activity logs).

**The relationship:**
```
Azure Resource (VM, NSG, App Service)
    │
    │ Diagnostic Settings route:
    ├── Metrics → Azure Monitor Metrics (short retention, real-time)
    └── Logs    → Log Analytics Workspace (long retention, queryable with KQL)
                        │
                        └── Queried with KQL
                        └── Used by Azure Sentinel (SIEM)
                        └── Used by Alert Rules
```

---

## Step 1 - Create a Log Analytics Workspace

```
Azure Portal → Log Analytics Workspaces → Create

Resource Group  : rg-hybrid-lab
Name            : law-hybrid-lab
Region          : (same region as VMs - minimises ingestion latency)
Pricing tier    : Pay-As-You-Go
                  (First 5GB/month free - more than enough for this lab)
```

Wait for deployment (1-2 minutes).

**Why a single workspace for the entire lab?**
In this lab, all resources are managed by the same team and have the same access
requirements. A single workspace simplifies querying - you can correlate events
from the VM, the NSG, and the activity log in one query. In production, workspace
design is more complex: some organisations use one workspace per environment (dev,
test, prod), others per team, others per data sensitivity level. The tradeoff is
between query simplicity (one workspace) and access isolation (multiple workspaces).

---

## Step 2 - Enable Diagnostic Settings on Both VMs

Diagnostic Settings must be configured on each resource individually. There is no
"enable for all VMs" option - it is per-resource.

### For vm-win-server (Windows VM):
```
Azure Portal → Virtual Machines → vm-win-server →
  Monitoring → Diagnostic settings → Enable guest-level monitoring

Destination:
  Send to Log Analytics workspace → law-hybrid-lab

Performance counters to collect:
  [All selected - CPU, Memory, Disk, Network]

Windows Event Logs to collect:
  Application: Critical, Error, Warning
  Security    : Audit Success, Audit Failure
  System      : Critical, Error, Warning
```

### For vm-ubuntu (Linux VM):
```
Azure Portal → Virtual Machines → vm-ubuntu →
  Monitoring → Insights → Enable

Workspace : law-hybrid-lab

This installs the Azure Monitor Agent and configures basic Linux metric collection.
```

### Enable Activity Log collection for the resource group:
```
Azure Portal → Monitor → Activity Log → Export Activity Logs

Subscription: [your subscription]
Destination : Log Analytics workspace → law-hybrid-lab
```

Activity logs record all control-plane operations - who created, modified, or
deleted Azure resources and when. This is essential for security investigation
and change tracking.

---

## Step 3 - Create a CPU Alert Rule

This alert demonstrates the full Azure Monitor alerting stack: metric collection →
threshold evaluation → action group notification.

### Create an Action Group first (who gets notified):
```
Azure Portal → Monitor → Alerts → Action Groups → Create

Resource Group  : rg-hybrid-lab
Name            : ag-it-alerts
Display Name    : IT Alerts

Notifications:
  Notification type : Email/SMS/Push/Voice
  Name              : Email-IT-Team
  Email             : [your email address]
```

### Create the Alert Rule:
```
Azure Portal → Monitor → Alerts → Create → Alert Rule

Scope:
  Resource scope : vm-win-server

Condition:
  Signal type    : Metrics
  Signal name    : Percentage CPU
  Threshold type : Static
  Operator       : Greater than
  Threshold      : 80
  Aggregation    : Average
  Aggregation granularity: 5 minutes
  Frequency of evaluation: 1 minute

Actions:
  Action group   : ag-it-alerts

Details:
  Alert rule name     : alert-cpu-high-vmwinserver
  Severity            : 2 - Warning
  Enable alert rule   : Yes
```

---

## Step 4 - Test the Alert Rule

Generate CPU load on the VM to trigger the alert and confirm the email is received.

```powershell
# Run this on vm-win-server (connect via Bastion)
# This script generates sustained CPU load for 6 minutes
# (enough to breach the 80% threshold and trigger the 5-minute aggregation window)

Write-Host "Generating CPU load — alert should fire in 5–7 minutes..."
$EndTime = (Get-Date).AddMinutes(6)
$x = 0
while ((Get-Date) -lt $EndTime) {
    $x = [math]::Sqrt($x + 1.0)
    # This tight loop consumes CPU without meaningful output
}
Write-Host "CPU stress test complete."
```

**What to expect:**
- After 5-6 minutes of sustained load above 80%, the alert should fire
- An email arrives at the address configured in the Action Group
- The email subject will be: "Alert: alert-cpu-high-vmwinserver fired"
- The body includes: current metric value, threshold, resource affected, time

**View the fired alert:**
```
Azure Portal → Monitor → Alerts → Alert History
```
The alert should show with state: Fired. After the CPU stress completes and CPU
drops below 80%, the alert resolves and another email is sent with state: Resolved.

---

## Step 5 - KQL Queries in Log Analytics

KQL (Kusto Query Language) is the query language for Log Analytics and Microsoft Sentinel.
These queries were tested in the lab after Diagnostic Settings were enabled and data
was ingested (allow 10–15 minutes for first data to appear).

```kql
// ── All events from the Windows VM in the last 1 hour ──────────────────
AzureDiagnostics
| where Resource == "VM-WIN-SERVER"
| where TimeGenerated > ago(1h)
| project TimeGenerated, Category, OperationName, ResultType, Level
| order by TimeGenerated desc
```

```kql
// ── Failed RDP / interactive login attempts (Security EventID 4625) ─────
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(24h)
| summarize FailureCount = count() by Account, IpAddress,
    bin(TimeGenerated, 15m)
| order by FailureCount desc
```

```kql
// ── CPU utilisation over time for all VMs ──────────────────────────────
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where TimeGenerated > ago(1h)
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart
```

```kql
// ── Disk free space below 20% ─────────────────────────────────────────
Perf
| where ObjectName == "LogicalDisk"
    and CounterName == "% Free Space"
    and InstanceName != "_Total"
| where CounterValue < 20
| project TimeGenerated, Computer, Drive = InstanceName,
    FreeSpacePct = round(CounterValue, 1)
| order by FreeSpacePct asc
```

```kql
// ── Azure activity log: all operations in the last 7 days ───────────────
AzureActivity
| where TimeGenerated > ago(7d)
| where ResourceGroup == "rg-hybrid-lab"
| project TimeGenerated, Caller, OperationNameValue,
    ActivityStatusValue, ResourceId
| order by TimeGenerated desc
```

```kql
// ── Azure activity log: only failed operations ───────────────────────
AzureActivity
| where TimeGenerated > ago(7d)
| where ActivityStatusValue == "Failure"
| project TimeGenerated, Caller, OperationNameValue, Properties
| order by TimeGenerated desc
```

```kql
// ── Inbound NSG rule hit counts (requires NSG flow logs enabled) ────────
AzureNetworkAnalytics_CL
| where TimeGenerated > ago(24h)
    and SubType_s == "FlowLog"
    and Direction_s == "I"
| summarize Flows = count() by
    NSGRule_s, SourceIP_s, DestinationPort_d
| order by Flows desc
| take 20
```

```kql
// ── Memory availability over time ───────────────────────────────────────
Perf
| where ObjectName == "Memory"
    and CounterName == "Available MBytes"
| where TimeGenerated > ago(1h)
| summarize AvgAvailMB = avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart
```

---

