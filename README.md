# azure-hybrid-lab

Fully documented Azure hybrid cloud lab validating AZ-104 Azure Administrator
competencies. Connects an on-premises Windows Server 2022 Active Directory domain
(contoso.local) to Azure Entra ID via Azure AD Connect with Password Hash Sync,
deploys VMs into a segmented VNet with NSG-enforced security, implements RBAC at
three different scopes demonstrating the principle of least privilege, configures
Azure Monitor with Log Analytics and KQL queries, and manages device compliance
through Microsoft Intune.

Every architectural decision is documented with a written justification - not just
what was deployed, but why it was deployed that way and what alternatives were
considered. The "What I Broke and Fixed" document records five real troubleshooting
scenarios encountered during the build, including the diagnostic steps taken and
the root causes found.

This is not a tutorial walkthrough. It is a documented, working environment.

---

## What this repo contains

| File | Purpose |
|------|---------|
| `01-environment-overview.md` | Lab component inventory, network architecture diagram, Azure free tier usage breakdown, deployment checklist |
| `02-vm-and-vnet-setup.md` | VNet design (why /16, why separate subnets), NSG rule documentation with justifications, Bastion deployment, VM deployment with no-public-IP security posture |
| `03-azure-ad-connect-hybrid-sync.md` | On-prem AD preparation, AD Connect install and express settings walkthrough, sync verification via PowerShell and Portal, hybrid SSO test, OU-based scoped sync |
| `04-rbac-configuration.md` | Three RBAC scenarios at three different scopes (subscription/RG/resource), PowerShell and Portal assignment methods, built-in role reference |
| `05-azure-monitor-and-alerts.md` | Log Analytics Workspace creation, Diagnostic Settings on both VMs, CPU alert rule creation and testing, 8 KQL queries tested and verified, disk space alert via KQL |
| `06-intune-device-compliance.md` | Device MDM enrollment, Windows compliance policy (BitLocker, Secure Boot, password), Configuration Profile via Settings Catalog, Win32 app silent deployment with IntuneWinAppUtil, Conditional Access integration |
| `07-nsg-rules-and-network-security.md` | Full NSG rule documentation table with justifications, default Azure rules explained, Effective Security Rules, IP Flow Verify, NSG Flow Logs, troubleshooting scenarios |
| `08-design-decisions-and-justifications.md` | Seven architectural decisions with full written justifications: PHS vs PTA vs ADFS, Bastion vs public IP, RBAC scope choice, VM size selection, Log Analytics vs metrics-only, VNet design, Intune licensing |
| `09-what-i-broke-and-fixed.md` | Five real troubleshooting scenarios: AD Connect sync error (stopped-extension-dll-exception), pfSense blocking sync traffic, wrong monitoring agent installed, Intune compliance hardware limitation, VM allocation failure and Stop vs Restart distinction |
| `diagrams/hybrid-architecture.md` | Full architecture ASCII diagram, component configuration summary table |

---

## AZ-104 Exam Domains Covered

| AZ-104 Domain | Coverage in This Lab |
|--------------|---------------------|
| Manage Azure Identities (20–25%) | Entra ID, AD Connect (PHS vs PTA), RBAC at three scopes, Intune device compliance, Conditional Access |
| Implement and Manage Virtual Networking (15–20%) | VNet/subnet design, NSG rules with justification, Effective Security Rules, IP Flow Verify, Azure Bastion |
| Deploy and Manage Azure Compute (20–25%) | VM deployment (Windows + Linux), no-public-IP configuration, VM connectivity via Bastion, Diagnostic Settings |
| Monitor and Maintain Azure Resources (10–15%) | Log Analytics Workspace, Diagnostic Settings, KQL queries, metric alert rules, Action Groups, NSG Flow Logs |
| Manage Azure Storage (10–15%) | Touched briefly - NSG Flow Logs stored in a storage account |
| Implement and Manage Azure Governance (15–20%) | RBAC assignments, principle of least privilege, resource group organisation |

---

## Skills demonstrated

**Azure networking:** VNet and subnet design, NSG rule configuration and justification,
Effective Security Rules, IP Flow Verify, Azure Bastion, network security posture (no
public management ports)

**Azure identity:** Entra ID, Azure AD Connect with PHS, hybrid SSO, UPN alignment,
RBAC at subscription/resource group/resource scope, least privilege principle

**Azure monitoring:** Log Analytics Workspace, Diagnostic Settings, KQL queries
(8 tested queries covering security events, performance, activity logs), metric alert
rules with Action Groups, NSG Flow Logs

**Microsoft Intune:** MDM enrollment, compliance policy design, Configuration Profile
via Settings Catalog, Win32 app packaging and silent deployment, Conditional Access
integration

**Architecture thinking:** Every component has a documented design decision with
justification and consideration of alternatives - answers the interview question
"why did you build it this way?"

**Real troubleshooting:** Five documented real issues encountered during the build
with root cause analysis - demonstrates that the lab was actually built and tested,
not just described

---

## Lab environment

```
On-premises:
  Hypervisor  : Proxmox VE 8.x
  DC hostname : dc01.contoso.local
  DC OS       : Windows Server 2022 Standard
  DC IP       : 192.168.10.10
  Domain      : contoso.local
  Client      : Windows 10 22H2 (domain-joined, Intune enrolled)
  AD Connect  : Microsoft Entra Connect v2.x (PHS mode)

Azure:
  Region      : UK South (chosen after AllocationFailed in East US 2)
  VNet        : 10.20.0.0/16
  VMs         : Standard_B1s (Windows Server 2022 + Ubuntu 22.04)
  Bastion     : Developer SKU
  Log Analytics: law-hybrid-lab (5GB free tier)
  Intune      : M365 Developer E5 Sandbox (free 90-day tenant)
```

---

## Quick start

```bash
# Clone this repo
git clone https://github.com/YOUR-USERNAME/azure-hybrid-lab.git
cd azure-hybrid-lab

# Start with the overview to understand the full architecture
cat 01-environment-overview.md

# Then follow the numbered steps in order:
# 01 → 02 → 03 → 04 → 05 → 06 → 07

# Read design decisions to understand why, not just what:
cat 08-design-decisions-and-justifications.md

# Read the troubleshooting log for real diagnostic examples:
cat 09-what-i-broke-and-fixed.md
```

---

## Impact

This lab documents a fully functional hybrid environment that mirrors the actual
architecture used by enterprises and Microsoft Partner clients. The AD
Connect PHS configuration, RBAC at correct scopes, Azure Monitor alerting, and
Intune device compliance are all components that an Azure cloud administrator
supports day-to-day in production.

The troubleshooting log demonstrates that the lab was actually built and tested —
not just described. The five issues documented represent real diagnostic scenarios:
AD Connect sync failures, DNS blocking sync traffic, wrong monitoring agent selection,
hardware compatibility constraints in Intune compliance, and VM allocation failures.
Each one demonstrates a real skill that goes beyond reading documentation.
