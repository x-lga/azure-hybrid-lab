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

## Network Architecture

```
On-premises (Proxmox home lab):         Microsoft Azure (Free Trial / M365 Dev Tenant):
┌──────────────────────────────┐         ┌───────────────────────────────────────────────┐
│  Windows Server 2022         │         │  Resource Group: rg-hybrid-lab                │
│  Hostname: dc01              │         │  Virtual Network: 10.20.0.0/16                │
│  IP: 192.168.10.10           │         │                                               │
│  Domain: contoso.local       │         │  ┌──────────────────────────────────────┐     │
│  Roles: AD DS, DNS, DHCP     │──AD ───►│  │  subnet-servers: 10.20.1.0/24        │     │
│                              │  Connect│  │  ├─ vm-win-server (10.20.1.4)        │     │
│  Azure AD Connect installed  │──Sync──►│  │  └─ AzureBastionSubnet adjacent      │     │
│  on this machine             │         │  │                                      │     │
│                              │         │  │  subnet-linux: 10.20.2.0/24          │     │
│  Windows 10 client           │         │  │  └─ vm-ubuntu (10.20.2.4)            │     │
│  Domain-joined               │         │  └──────────────────────────────────────┘     │
│  Intune enrolled             │         │                                               │
└──────────────────────────────┘         │  Azure Entra ID (synced from contoso.local)   │
                                         │  NSG: nsg-subnet-servers                      │
                                         │  Azure Bastion                                │
                                         │  Log Analytics Workspace: law-hybrid-lab       │
                                         │  Microsoft Intune (M365 Dev Tenant)           │
                                         └───────────────────────────────────────────────┘
```

---

## Azure Free Tier Usage

All resources in this lab were deployed within Azure free trial or M365 Developer
Sandbox limits to demonstrate that this environment is reproducible by anyone with
free Azure access:

| Resource | Free Tier Availability |
|----------|----------------------|
| B1s VM (Windows) | 750 hours/month free for 12 months |
| B1s VM (Linux) | 750 hours/month free for 12 months |
| Azure Bastion | Not free - used Developer SKU at ~$0.19/hr. Shutdown when not in use. |
| Log Analytics | First 5GB/month free |
| Azure Monitor | Basic metrics free |
| Microsoft Intune | Included in M365 Developer E5 Sandbox (free 90-day tenant) |
| Azure AD Connect | Free download - no Azure cost |
| Entra ID | Free tier sufficient for this lab |

**Cost management note:** All VMs are configured to auto-shutdown at 23:00 daily.
Azure Bastion was manually started/stopped to control cost. Total estimated cost
for running this lab intermittently over 30 days: under $15 USD.

---

## Lab Checklist - All Components Deployed and Verified

- [x] Windows Server 2022 DC deployed on Proxmox with AD DS, DNS, DHCP
- [x] Azure Virtual Network created with two subnets
- [x] NSG deployed and associated with subnet-servers
- [x] Azure Bastion deployed (Developer SKU)
- [x] Windows Server VM deployed (no public IP, Bastion access only)
- [x] Ubuntu VM deployed (no public IP, Bastion access only)
- [x] Azure AD Connect installed on Proxmox DC and configured with Password Hash Sync
- [x] User sync verified - on-premises users visible in Entra ID portal
- [x] RBAC assignments configured at three scopes (subscription, resource group, resource)
- [x] Log Analytics Workspace created and linked to Azure Monitor
- [x] Diagnostic Settings enabled on both VMs
- [x] KQL queries tested and verified returning data
- [x] CPU alert rule created, tested by generating load, alert email received
- [x] Intune device enrolled, compliance policy created, device shows Compliant
- [x] Win32 app deployed silently via Intune


---

