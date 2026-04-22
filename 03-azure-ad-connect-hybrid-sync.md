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