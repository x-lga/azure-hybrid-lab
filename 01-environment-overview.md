# Azure Hybrid Lab - Environment Overview

## Purpose

This lab validates AZ-104 Azure Administrator competencies through a fully functional
hybrid cloud environment. It is not a theoretical exercise - every component described
here was deployed, tested, and documented in a working Azure subscription connected to
a Proxmox-hosted on-premises Active Directory domain.

The environment mirrors what small-to-medium enterprises and Microsoft Partner clients
actually run. Most companies cannot do a full cloud migration overnight - they have on-premises servers, AD-joined machines, and years of investment in Windows infrastructure. The hybrid model (on-premises AD + Azure Entra ID via AD Connect) is the real-world architecture these companies need engineers to understand
and support.

---

## Lab Components

| Component | Platform | Purpose | AZ-104 Domain |
|-----------|---------|---------|--------------|
| Windows Server 2022 DC | Proxmox VM | On-premises AD DS, DNS, DHCP for contoso.local | - (on-prem foundation) |
| Azure AD Connect | On-prem VM | Syncs on-prem AD users/groups to Entra ID | Identity (Manage Azure Identities) |
| Azure Entra ID | Azure | Cloud identity - synced from contoso.local | Identity |
| Azure Virtual Network | Azure | Isolated VNet with segmented subnets | Virtual Networking |
| Network Security Group | Azure | Subnet-level traffic control | Virtual Networking |
| Azure Bastion | Azure | Secure browser-based RDP/SSH (no public IP) | Virtual Networking |
| Azure VM - Windows Server | Azure | Cloud-side server workload | Virtual Machines |
| Azure VM - Ubuntu 22.04 | Azure | Linux workload VM | Virtual Machines |
| RBAC assignments | Azure | Least-privilege role assignments at 3 scopes | Governance |
| Log Analytics Workspace | Azure | Central log repository for all lab resources | Monitoring |
| Diagnostic Settings | Azure | Routes VM metrics and logs to Log Analytics | Monitoring |
| Azure Monitor Alert Rules | Azure | Metric-based alerts (CPU, disk) with Action Group | Monitoring |
| Microsoft Intune | Azure / M365 Dev | MDM enrollment, compliance policy, app deploy | Management |

---
