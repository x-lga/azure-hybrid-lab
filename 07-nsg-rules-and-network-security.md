# Step 07 - NSG Rules and Network Security

**AZ-104 Domain:** Implement and Manage Virtual Networking
**Last reviewed:** 2026-07

---

## NSG Architecture Overview

Network Security Groups (NSGs) are stateful packet filters applied at the subnet
or NIC level. In this lab, NSGs are applied at the subnet level - a single NSG
controls traffic for all VMs in subnet-servers.

**Stateful** means: if you allow inbound traffic on port 443 for a connection,
the corresponding outbound response traffic is automatically allowed - you do not
need a separate outbound rule for responses. This differs from traditional firewall
appliances where stateful inspection must be explicitly enabled.

**Rule processing:** Azure evaluates NSG rules in priority order, lowest priority
number first (e.g., 100 is evaluated before 200). The first rule that matches the
traffic is applied and no further rules are evaluated. This is the same "first match
wins" logic used by pfSense and most enterprise firewalls.

---

## NSG Rule Documentation: nsg-subnet-servers

### Inbound Rules

| Priority | Name | Port | Protocol | Source | Destination | Action | Justification |
|---------|------|------|---------|--------|------------|--------|---------------|
| 100 | Allow-RDP-MyIP | 3389 | TCP | [My Home IP] | Any | Allow | RDP from admin machine only (belt-and-suspenders alongside Bastion) |
| 200 | Allow-WinRM-VNet | 5985 | TCP | VirtualNetwork | VirtualNetwork | Allow | PowerShell remoting between VMs within the VNet |
| 210 | Allow-SSH-VNet | 22 | TCP | VirtualNetwork | VirtualNetwork | Allow | SSH management between VMs within the VNet |
| 300 | Allow-HTTPS-Inbound | 443 | TCP | AzureLoadBalancer | Any | Allow | Allow Azure infrastructure health probes |
| 4000 | Deny-All-Inbound | * | Any | Any | Any | Deny | Explicit default deny — documents security posture |

**Default rules (cannot be deleted, shown for reference):**

| Priority | Name | Action | What It Allows |
|---------|------|--------|---------------|
| 65000 | AllowVnetInBound | Allow | All intra-VNet traffic |
| 65001 | AllowAzureLoadBalancerInBound | Allow | Azure LB health probes |
| 65500 | DenyAllInBound | Deny | All other inbound traffic |


### Outbound Rules

| Priority | Name | Port | Protocol | Source | Destination | Action | Justification |
|---------|------|------|---------|--------|------------|--------|---------------|
| 100 | Allow-HTTPS-Outbound | 443 | TCP | Any | Internet | Allow | HTTPS to internet - Windows Update, Azure agents, AD Connect |
| 200 | Allow-HTTP-Outbound | 80 | TCP | Any | Internet | Allow | HTTP to internet - package managers, some update channels |
| 300 | Allow-DNS-Outbound | 53 | UDP | Any | Any | Allow | DNS resolution |
| 4000 | Deny-All-Outbound | * | Any | Any | Internet | Deny | Explicit default deny for internet outbound - only allowed ports above |

---

## Effective Security Rules - How to Check What Is Actually Applied

The "Effective Security Rules" view in the Azure Portal shows the combined NSG
rules as they actually apply to a specific VM's NIC - accounting for subnet-level
and NIC-level NSGs together. This is the definitive source of truth for what
traffic is allowed or denied to a specific VM.

```
Azure Portal → Virtual Machines → vm-win-server →
  Networking → [network interface name] →
  Effective security rules

This shows:
  - Inbound rules with their source
  - Outbound rules with their source
  - Which NSG each rule comes from (subnet or NIC)
  - The final action after all rules are evaluated
```

---

## IP Flow Verify - Testing Specific Traffic

IP Flow Verify answers the question: "Will Azure allow a packet from IP X to reach
VM Y on port Z?" without needing to actually send the traffic.

```
Azure Portal → Virtual Machines → vm-win-server →
  Networking → Network Watcher → IP Flow Verify

OR

Azure Portal → Network Watcher → IP Flow Verify

Settings:
  VM                  : vm-win-server
  Network interface   : [auto-populated]
  Protocol            : TCP
  Direction           : Inbound
  Local IP address    : 10.20.1.4 (VM private IP)
  Local port          : 3389
  Remote IP address   : [test IP address]
  Remote port         : 12345 (the source port — usually random)

Result: "Access allowed" or "Access denied" + which rule caused the result
```

This is faster than trying to connect and seeing if it works — and it tells you
exactly which NSG rule is responsible for the allow or deny.

---

## NSG Diagnostic Logging - NSG Flow Logs

NSG Flow Logs record every accepted and denied network flow through an NSG.
They are essential for security investigation and network traffic analysis.

```
Azure Portal → Network Security Groups → nsg-subnet-servers →
  Monitoring → NSG Flow Logs → Create

Version          : Version 2 (includes bytes transferred)
Storage Account  : Create new → stghybridlabflows[randomsuffix]
Retention days   : 7 (free tier - longer retention costs more)
Traffic analytics: Enable (optional - requires Log Analytics Workspace)
```

After flow logs are enabled, query them in Log Analytics:
```kql
// NSG denied inbound connections - last 24 hours
AzureNetworkAnalytics_CL
| where TimeGenerated > ago(24h)
    and SubType_s == "FlowLog"
    and Direction_s == "I"
    and FlowStatus_s == "D"
| summarize DeniedFlows = count() by
    NSGRule_s, SourceIP_s, DestinationPort_d
| order by DeniedFlows desc
| take 20
```

---

## Common NSG Troubleshooting Scenarios

### Scenario: VM cannot reach the internet for Windows Update

**Symptom:** Windows Update on vm-win-server shows "We can't connect to the update
service" or times out.

**Investigation:**
```powershell
# On the VM — test if HTTPS outbound is working
Test-NetConnection -ComputerName windowsupdate.microsoft.com -Port 443
# Expected: TcpTestSucceeded = True if NSG allows HTTPS outbound

Test-NetConnection -ComputerName 8.8.8.8 -Port 443
# Tests if HTTPS to a known external IP works
```