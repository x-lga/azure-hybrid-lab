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

## Lab RBAC Scenarios

This lab implements RBAC at three different scopes to demonstrate the principle
in practice, not just in theory.

---

### Scenario 1 - Junior Admin: Contributor at Resource Group Scope

**Business requirement:** A junior cloud administrator needs to be able to create,
manage, and delete VMs and other resources within the lab resource group, but should
not be able to affect other resource groups in the subscription, change subscription
settings, or assign roles to other users.

**Role:** Contributor
**Scope:** rg-hybrid-lab (Resource Group)

**What Contributor can do:**
- Create, modify, and delete all resources within rg-hybrid-lab
- Start, stop, restart VMs
- Modify NSG rules
- View all resources and their configurations
- Deploy resources from ARM templates

**What Contributor cannot do:**
- Create resource groups or subscriptions
- Assign roles to other users (requires Owner or User Access Administrator)
- Access billing information
- Modify subscription-level policies

**How to assign via Portal:**
```
Azure Portal → Resource Groups → rg-hybrid-lab →
  Access Control (IAM) → Add → Add Role Assignment

Role     : Contributor
Members  : [select the user or service principal]
Condition: None (no ABAC conditions for this lab)

Review + Assign
```

**How to assign via PowerShell (Az module):**
```powershell
# Authenticate to Azure
Connect-AzAccount

# Get the resource group object
$RG = Get-AzResourceGroup -Name "rg-hybrid-lab"

# Get the user object (replace with actual UPN)
$User = Get-AzADUser -UserPrincipalName "junioradmin@contosodemo.onmicrosoft.com"

# Create the role assignment
New-AzRoleAssignment `
    -ObjectId            $User.Id `
    -RoleDefinitionName  "Contributor" `
    -ResourceGroupName   "rg-hybrid-lab"

Write-Host "Contributor role assigned to $($User.DisplayName) at rg-hybrid-lab scope"
```

**Verification:**
```powershell
# List all role assignments in the resource group
Get-AzRoleAssignment -ResourceGroupName "rg-hybrid-lab" |
    Select-Object DisplayName, RoleDefinitionName, Scope |
    Format-Table -AutoSize
```

---

### Scenario 2 - Read-Only Auditor: Reader at Subscription Scope

**Business requirement:** An internal auditor or compliance officer needs to be able
to view all resources, configurations, and activity logs across the entire subscription
for compliance review - but must not be able to make any changes.

**Role:** Reader
**Scope:** Subscription

**What Reader can do:**
- View all resources and their configurations
- Read all resource properties and metadata
- View activity logs and diagnostic data
- Export resource inventory and configuration to CSV