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