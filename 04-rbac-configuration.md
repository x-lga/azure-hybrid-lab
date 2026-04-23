# Step 04 - RBAC Role Assignment Configuration

**AZ-104 Domain:** Manage Azure Identities and Governance - Role-Based Access Control
**Last reviewed:** 2026-07

---

## RBAC Fundamentals: Why It Matters in Practice

RBAC (Role-Based Access Control) is the primary access control mechanism in Azure.
Every action you can take in Azure - creating VMs, reading logs, assigning policies -
is governed by an RBAC role assignment.

The principle of least privilege means: give each person or service exactly the
permissions they need, at the narrowest scope that achieves the objective, and nothing more.

**The four scope levels (broad to narrow):**
```
Management Group
  └── Subscription          ← broadest scope (affects all resource groups)
        └── Resource Group  ← mid scope (affects resources in this group only)
              └── Resource  ← narrowest scope (affects one specific resource)
```

A role assigned at the subscription level grants that role across ALL resource groups
and resources in the subscription. A role assigned at the resource group level grants
it only within that resource group. Always assign at the narrowest scope that
achieves the objective.

---
