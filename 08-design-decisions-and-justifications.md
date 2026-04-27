# Step 08 - Design Decisions and Justifications

This document is the most important file in the repo for interview preparation.
Every architectural choice in this lab has a documented reason. These are the
questions Azure hiring managers and interviewers ask.

**The pattern:** "Why did you do X instead of Y?" - this document answers every
variant of that question before it is asked.

---

## Decision 1: Password Hash Sync (PHS) instead of Pass-Through Authentication (PTA)

**What was chosen:** Password Hash Sync

**What was not chosen:** Pass-Through Authentication (PTA) or Active Directory
Federation Services (AD FS)

**Why PHS was chosen:**

PHS synchronises a hash of the on-premises password hash to Entra ID. The actual
password never leaves on-premises - only a salted, hashed representation is synced.
When a user authenticates to a cloud service, Entra ID validates the password hash
locally in the cloud without needing to reach back to the on-premises DC.

**Operational advantage for a lab environment:**
If the Proxmox host is shut down (as it often is in a lab to save electricity),
users can still authenticate to cloud services using PHS. The cloud has the hash
and can validate it independently. PTA, by contrast, requires the on-premises DC
to be reachable for every authentication - if the Proxmox VM is off, nobody can log in.

**Security considerations:**
PHS has been the subject of concern that "password hashes in the cloud" is a risk.
Microsoft's response is that the hashes are stored using the same cryptographic
protection as AD itself, and the hash that is synced is already a derived hash -
not the NT hash that an attacker could use for Pass-the-Hash attacks directly.
In practice, PHS is considered acceptable by most security frameworks including
ISO 27001 and NIST.

**When to choose PTA instead:**
PTA is the right choice when an organisation has a compliance requirement that
passwords (including their hashes) must never leave the on-premises network.
Common in government, banking, and heavily regulated industries. PTA sends
authentication requests to an on-premises agent that validates them against AD
and returns a yes/no to Entra ID. No password material leaves the network.
The tradeoff is the operational dependency on the on-premises infrastructure
being available.

**When to choose AD FS instead:**
AD FS (Active Directory Federation Services) provides full federation with claims-based
authentication. It is the most complex and expensive option. It is required for
specific scenarios like smart card authentication, on-premises MFA solutions, or
custom claims transformation. Most organisations have moved away from AD FS toward
PHS or PTA plus Entra ID Conditional Access.

---

## Decision 2: Azure Bastion instead of Public IP + RDP

**What was chosen:** Azure Bastion (Developer SKU) with no public IP on any VM

**What was not chosen:** Public IP address on each VM with RDP/SSH open to the internet

**Why Bastion was chosen:**

RDP (port 3389) and SSH (port 22) exposed on a public IP receive automated
brute-force attacks within minutes of being created. This is not theoretical -
Azure Security Centre reports show that internet-facing VMs with RDP exposed are
probed thousands of times per day. Common passwords are tried automatically by
credential stuffing bots.

Bastion provides RDP and SSH access through the browser over HTTPS (port 443),
which is almost always already open and does not expose management ports to the
internet. The VMs have no public IP at all - they are entirely inaccessible from
the internet.

**Additional security benefit:**
Bastion requires Azure AD authentication before you can connect to any VM behind it.
This means MFA, Conditional Access, and Entra ID identity controls apply to VM access.
A public IP with RDP has no such protection - it is only protected by the local
Windows credentials on the VM.

**Cost tradeoff:**
Bastion has a cost - the Developer SKU is approximately $0.19/hour when active.
For this lab, Bastion is started when needed and stopped when not in use to minimise
cost. In production, the cost of Bastion (approximately $140/month for Standard SKU)
is far less than the cost of a ransomware incident from a compromised internet-facing RDP.

**Alternative for labs:** Azure JIT (Just-In-Time) VM access via Microsoft Defender
for Cloud is a lower-cost alternative for temporary port opening. It requires Defender
for Cloud (paid) and opens the port for a limited time window only. Not used in this
lab to keep costs zero where possible.

---

## Decision 3: RBAC at Resource Group scope for the Junior Admin, not Subscription scope

**What was chosen:** Contributor role at Resource Group scope (rg-hybrid-lab)

**What was not chosen:** Contributor at Subscription scope

**Why Resource Group scope was chosen:**

Assigning Contributor at the subscription level would give the junior admin
Contributor access to every resource group in the subscription - including any
other labs, test environments, or production resources that might exist in the
same subscription.

Contributor at the Resource Group level gives them everything they need (full
management of all resources within rg-hybrid-lab) without the risk of accidentally
affecting other resources.

**The least privilege principle in action:**
The question to ask is: "What is the narrowest scope at which this person can do
their job?" For a junior admin working on this specific lab, the answer is Resource
Group scope. Subscription scope is broader than necessary.

**When to assign at Subscription scope:**
Subscription-level Contributor is appropriate for a full subscription administrator —
someone responsible for all resources in the subscription. Examples: a senior cloud
architect managing the entire Azure environment, or a service principal used by a
CI/CD pipeline that deploys to any resource group.

---

## Decision 4: B1s VM size instead of larger

**What was chosen:** Standard_B1s (1 vCPU, 1GB RAM)

**What was not chosen:** D-series, E-series, or other production-grade VM sizes

**Why B1s was chosen:**

The B-series (Burstable) VMs are specifically designed for workloads that do not
need continuous full CPU performance. The B1s accrues CPU credits when idle and
spends them during bursts. For a lab VM that is mostly idle with occasional bursts
of activity, this is exactly the right fit - and B1s is free-tier eligible for 750
hours/month for the first 12 months.

**The limitation:**
B1s has only 1GB RAM. Windows Server 2022 runs acceptably with 1GB for basic
domain controller functions in a lab, but would be unsuitable for any production
workload. In production, domain controllers typically have a minimum of 4–8GB RAM
(D2s_v3 or equivalent).

**In an interview:** If asked "why B1s instead of a larger VM?", the answer is:
"This is a lab for learning and demonstrating concepts - the B1s is free-tier eligible
and sufficient for the workloads involved. For production, I would size based on the
actual workload requirements and Microsoft's DC sizing guidance."

---

## Decision 5: Log Analytics Workspace instead of just Azure Monitor Metrics

**What was chosen:** Log Analytics Workspace with Diagnostic Settings routing both
metrics and logs to the workspace

**What was not chosen:** Azure Monitor Metrics only (without Log Analytics)

**Why Log Analytics was chosen:**

Azure Monitor Metrics provides numeric time-series data - CPU %, disk I/O,
network bytes. This is sufficient for dashboards and threshold-based alerts.
It does not support investigation queries.

Log Analytics adds queryable log data: Windows Event Logs (Security, System,
Application), Syslog from Linux, performance counters over time, and all Azure
activity operations. The combination of metrics (for alerting) and logs
(for investigation) is what real security monitoring requires.

**Critical additional benefit:**
KQL queries in Log Analytics are exactly the same language used in Microsoft Sentinel
(Azure's cloud SIEM). Learning KQL in Log Analytics directly transfers to Sentinel
skills. This lab builds the same query skills used by SOC analysts working in
enterprise Azure environments.

---

## Decision 6: Single VNet with two subnets instead of VNet peering

**What was chosen:** Single VNet (10.20.0.0/16) with subnet-servers and subnet-linux

**What was not chosen:** Two separate VNets with VNet peering

**Why single VNet was chosen:**

VMs in the same VNet can communicate with each other by default (subject to NSG rules).
No additional configuration is needed. Deploying both VMs in one VNet keeps the
lab simple and costs nothing extra.