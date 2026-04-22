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

## Step 3 — Configure AD Connect with Express Settings

For this lab, Express Settings is appropriate. It configures:
- Password Hash Synchronisation (PHS)
- Automatic synchronisation every 30 minutes
- All users and groups in the domain synced (can be scoped later)