# Step 03 - Azure AD Connect: Hybrid Identity Sync

**AZ-104 Domain:** Manage Azure Identities and Governance
**Last reviewed:** 2026-07

---

## Why Hybrid Identity Matters

Azure AD Connect (now Microsoft Entra Connect) is the bridge between on-premises
Active Directory and Azure Entra ID. It enables:

- **Single Sign-On:** Users log in with their on-premises AD credentials and can
  access Microsoft 365, Azure resources, and cloud apps without a separate cloud password
- **Unified identity:** IT admins manage users in one place (on-prem AD) and changes
  automatically propagate to the cloud
- **Hybrid access control:** On-premises groups can be used in Azure RBAC and M365
  security groups

This is the architecture used by the majority of Kenyan enterprises. Full cloud-only
identity (Entra ID only, no on-prem AD) is less common - most organisations with
existing Windows infrastructure use hybrid.

---

## Prerequisites

Before installing Azure AD Connect, verify:

**On-premises side:**
- [ ] Windows Server 2022 DC is running with AD DS promoted and contoso.local domain active
- [ ] You have Domain Admin credentials
- [ ] The DC can reach the internet (required for sync to Azure)
- [ ] The DC meets AD Connect requirements:
  - .NET Framework 4.7.2 or later (included in Server 2022)
  - PowerShell 5.1 or later (included in Server 2022)
  - Windows Server 2016 or later (Server 2022 ✓)

**Azure side:**
- [ ] You have a Global Administrator account in Entra ID
- [ ] The Entra ID tenant is available (Azure Portal → Entra ID shows tenant)
- [ ] You know the tenant's default domain (e.g., `contosodemo.onmicrosoft.com`)

---

## Step 1 - Prepare On-Premises AD

### Create a test OU and test users for sync validation
```powershell
# Run on the domain controller
Import-Module ActiveDirectory

# Create an OU for cloud-synced users
New-ADOrganizationalUnit -Name "Azure-Sync" -Path "DC=contoso,DC=local"

# Create test users in this OU
$SecurePW = ConvertTo-SecureString "Welcome@12345!" -AsPlainText -Force

New-ADUser `
    -Name "Test User One" `
    -GivenName "Test" `
    -Surname "User One" `
    -SamAccountName "testuser1" `
    -UserPrincipalName "testuser1@contoso.local" `
    -AccountPassword $SecurePW `
    -Enabled $true `
    -Path "OU=Azure-Sync,DC=contoso,DC=local" `
    -ChangePasswordAtLogon $false

New-ADUser `
    -Name "Test User Two" `
    -GivenName "Test" `
    -Surname "User Two" `
    -SamAccountName "testuser2" `
    -UserPrincipalName "testuser2@contoso.local" `
    -AccountPassword $SecurePW `
    -Enabled $true `
    -Path "OU=Azure-Sync,DC=contoso,DC=local" `
    -ChangePasswordAtLogon $false

Write-Host "Test users created in OU=Azure-Sync"
```

### Check UPN suffix alignment
AD Connect requires that the on-premises UPN suffix matches a verified domain in
Entra ID, OR the accounts use the default `onmicrosoft.com` domain.

For this lab, the on-premises UPN (`testuser1@contoso.local`) will sync to Entra ID
as `testuser1@contosodemo.onmicrosoft.com` (the tenant's default domain). This is the
expected behaviour in a lab with a `.local` domain that is not registered in Azure DNS.

In production, companies add their actual domain (`company.co.ke`) to Entra ID as a
verified custom domain and update UPN suffixes to match so users log in with their
real email address.

---
## Step 2 - Download and Install Azure AD Connect

1. On the **domain controller**, open a browser
2. Navigate to: `https://www.microsoft.com/en-us/download/details.aspx?id=47594`
   Or search Microsoft Download Center for "Microsoft Entra Connect"
3. Download the installer (`AzureADConnect.msi`)
4. Run the installer as Administrator

---

## Step 3 - Configure AD Connect with Express Settings

For this lab, Express Settings is appropriate. It configures:
- Password Hash Synchronisation (PHS)
- Automatic synchronisation every 30 minutes
- All users and groups in the domain synced (can be scoped later)

**Installation wizard walkthrough:**

```
Welcome screen → Use express settings

Connecting to Entra ID:
  Username : [Global Admin UPN for Azure tenant — e.g., admin@contosodemo.onmicrosoft.com]
  Password : [Global Admin password]
  → Next

Connecting to AD DS:
  Forest   : contoso.local
  Username : CONTOSO\Administrator  (or your domain admin account)
  Password : [Domain Admin password]
  → Add Directory → Next

Entra ID sign-in configuration:
  You will see a warning that contoso.local is not a verified domain in Azure.
  This is expected for a lab with a .local domain.
  Check: "Continue without matching all UPN suffixes to verified domains"
  → Next

Ready to configure:
  Review the summary:
    Synchronize all users and groups : Yes
    Enable Exchange hybrid deployment : No
    Password hash synchronization    : Yes
    Auto upgrade                     : Enabled
  → Install

Installation takes approximately 5–10 minutes.
```

---

## Step 4 - Verify Sync Completed

### Check sync status from the DC
```powershell
# Import the AD Connect Sync module
Import-Module ADSync

# Check scheduler status - should show NextSyncCycleStartTimeInUTC
Get-ADSyncScheduler

# Check last sync run results
Get-ADSyncRunProfileResult | Select-Object -Last 5 |
    Select-Object RunProfileName, RunResult, StartDate, EndDate

# Force a delta sync cycle (syncs only changes since last run)
Start-ADSyncSyncCycle -PolicyType Delta

# Force a full sync cycle (re-syncs all objects — use sparingly)
# Start-ADSyncSyncCycle -PolicyType Initial
```

### Verify users appear in Azure Portal
```
Azure Portal → Entra ID → Users → All users

Filter by:
  Sync status : Synced from on-premises

You should see testuser1 and testuser2 with:
  Source      : Windows Server AD
  Identity    : Synced
```

If users do not appear within 5 minutes of the sync cycle completing:
1. Check the AD Connect Synchronisation Service Manager (Start → Miicrosoft Entra Connect → Synchronization Service)
2. Look for errors in the Operations tab
3. Common issues: firewall blocking sync, wrong credentials, UPN mismatch

### Check the Synchronisation Service Manager for errors
```
Start Menu → Microsoft Entra Connect → Synchronization Service

Operations tab:
  All recent sync profiles should show Result: "success"
  Any "stopped-extension-dll-exception" or "stopped-server" = configuration issue

Connectors tab:
  Both connectors should show: Status "Idle" (not "Stopped" or "Error")
  - Azure Active Directory (contosodemo.onmicrosoft.com)
  - Active Directory Domain Services (contoso.local)
```

---

## Step 5 - Test Hybrid SSO

Verify that an on-premises user can authenticate to Azure/M365 cloud services:

1. In a browser (InPrivate/Incognito), navigate to `portal.office.com`
2. Sign in with: `testuser1@contosodemo.onmicrosoft.com`
   (the synced UPN format - `samaccountname@tenant.onmicrosoft.com`)
3. Enter the password set in Step 1 (`Welcome@12345!`)
4. The user should successfully authenticate and see the M365 portal
5. If prompted for MFA: the account has no MFA registered. This is expected in a lab.
   In production, MFA would be configured via Conditional Access.

**Successful sign-in confirms:**
- Password hash was successfully synced from on-prem AD to Entra ID
- The cloud account is active and usable
- Hybrid identity is functioning end-to-end

---

## Step 6 - Configure Scoped Sync (Optional - Demonstrates OU Filtering)

In production, you would not sync every user and group in the domain to the cloud.
You sync only the accounts that need cloud access. AD Connect supports OU-based
filtering to limit which objects are synced.

```powershell
# View current sync configuration
Get-ADSyncConnector | Select-Object Name, Type, ConnectivityParameters

# To configure OU filtering via the AD Connect wizard:
# Start → Microsoft Entra Connect → Microsoft Entra Connect →
# Change user sign-in → No (just change sync settings) →
# Customize synchronization options →
# Domain and OU filtering →
# Deselect OUs you do not want synced →
# Configure

# To check which OUs are currently in scope:
$ADConnector = Get-ADSyncConnector |
    Where-Object { $_.Type -eq "AD" }
$ADConnector.Partitions | ForEach-Object {
    Write-Host "Partition: $($_.Name)"
    $_.ContainerInclusionList | ForEach-Object {
        Write-Host "  Included OU: $_"
    }
}
```

---

## Key Concepts Validated by This Configuration

| Concept | How It Is Implemented in This Lab |
|---------|----------------------------------|
| Hybrid Identity | On-premises AD synced to Entra ID via AD Connect |
| Password Hash Sync (PHS) | User passwords hashed on-prem and synced to Entra ID - cloud auth works even if on-prem is offline |
| UPN matching | .local domain lab uses onmicrosoft.com UPN suffix - in production, verified custom domain is used |
| Sync scope | All objects by default; OU filtering demonstrated to show production scoping approach |
| Sync cycle | 30-minute automatic delta sync + manual delta and initial sync via PowerShell |
| Sync error resolution | Synchronisation Service Manager operations tab - documented in Step 4 |


---
