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