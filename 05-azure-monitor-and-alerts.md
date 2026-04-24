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