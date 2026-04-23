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

**What Reader cannot do:**
- Create, modify, or delete any resource
- View secrets in Key Vault (Key Vault has its own access policy)
- Access VM guest OS data

**How to assign:**
```
Azure Portal → Subscriptions → [your subscription] →
  Access Control (IAM) → Add → Add Role Assignment

Role   : Reader
Scope  : (the subscription scope is already set at this level)
Members: [auditor user or group]
```

**How to assign via PowerShell:**
```powershell
# Get the subscription ID
$SubID = (Get-AzSubscription).Id

# Get the auditor user
$Auditor = Get-AzADUser -UserPrincipalName "auditor@contosodemo.onmicrosoft.com"

# Assign Reader at subscription scope
New-AzRoleAssignment `
    -ObjectId           $Auditor.Id `
    -RoleDefinitionName "Reader" `
    -Scope              "/subscriptions/$SubID"

Write-Host "Reader role assigned at subscription scope for audit access"
```

---

### Scenario 3 - VM Operator: Virtual Machine Contributor at Resource Scope

**Business requirement:** An operations team member needs to be able to start, stop,
and restart a specific production VM (vm-win-server), but should have no access to
other VMs, the networking configuration, storage accounts, or any other resources.

**Role:** Virtual Machine Contributor
**Scope:** vm-win-server (individual resource)

**What Virtual Machine Contributor can do on the assigned VM:**
- Start, stop, restart, redeploy the VM
- Connect to the VM (RDP/SSH via Bastion)
- View VM metrics and diagnostics
- Resize the VM
- Add/remove data disks

**What Virtual Machine Contributor cannot do:**
- Manage the VNet or NSG the VM is connected to
- Access the storage account containing the VM disk
- Manage any other VM
- Assign roles

**How to assign:**
```
Azure Portal → Virtual Machines → vm-win-server →
  Access Control (IAM) → Add → Add Role Assignment

Role   : Virtual Machine Contributor
Scope  : (set at the resource level — vm-win-server)
Members: [VM operator user]
```

**How to assign via PowerShell:**
```powershell
# Get the VM resource ID
$VM = Get-AzVM -ResourceGroupName "rg-hybrid-lab" -Name "vm-win-server"

# Get the operator user
$Operator = Get-AzADUser -UserPrincipalName "vmoperator@contosodemo.onmicrosoft.com"

# Assign Virtual Machine Contributor at resource scope
New-AzRoleAssignment `
    -ObjectId           $Operator.Id `
    -RoleDefinitionName "Virtual Machine Contributor" `
    -Scope              $VM.Id

Write-Host "VM Contributor assigned to $($Operator.DisplayName) for vm-win-server only"
```

---

## RBAC Assignment Summary for This Lab

| User/Principal | Role | Scope | Business Justification |
|---------------|------|-------|----------------------|
| junioradmin | Contributor | rg-hybrid-lab (Resource Group) | Manages lab resources; cannot affect other RGs or subscription settings |
| auditor | Reader | Subscription | Read-only compliance review access across all resources |
| vmoperator | Virtual Machine Contributor | vm-win-server (Resource) | Operates one specific VM; no access to networking or other resources |

---