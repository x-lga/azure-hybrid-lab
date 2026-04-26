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