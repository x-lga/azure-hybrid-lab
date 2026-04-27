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
