# Azure Hybrid Lab - Environment Overview

## Purpose

To validate AZ-104 Azure Administrator competencies through a fully functional hybrid cloud environment connecting an on-premises Active Directory domain to Azure Entra ID.

---

## Components

| Component | Platform | Purpose |
|-----------|---------|---------|
| Windows Server 2022 DC | Proxmox VM | On-premises AD domain controller |
| Azure Virtual Machine (Windows) | Azure | Cloud-joined server VM |
| Azure Virtual Machine (Ubuntu) | Azure | Linux workload VM |
| Azure Virtual Network | Azure | Isolated VNet with segmented subnets |
| Azure Entra ID (AAD) | Azure | Cloud identity - synced from on-prem AD |
| Azure AD Connect | On-prem VM | Syncs on-prem AD objects to Entra ID |
| Microsoft Intune | Azure | MDM/endpoint management |
| Azure Monitor + Log Analytics | Azure | Monitoring and alerting |

---

## Network Architecture

```
On-premises (Proxmox lab):          Azure (Free Trial / M365 Dev):
┌─────────────────────────┐         ┌──────────────────────────────┐
│  Windows Server 2022    │         │  Virtual Network: 10.20.0.0/16│
│  contoso.local          │─AD ────►│  ├─ Subnet-Servers: 10.20.1.x │
│  192.168.10.10          │  Connect│  │   └─ Win Server VM           │
│                         │         │  └─ Subnet-Linux: 10.20.2.x    │
│  Azure AD Connect       │         │      └─ Ubuntu VM               │
│  (installed here)       │─Sync───►│                                 │
└─────────────────────────┘         │  Azure Entra ID (cloud)         │
                                    │  NSG rules per subnet           │
                                    │  Azure Monitor + Log Analytics   │
                                    │  Microsoft Intune                │
                                    └──────────────────────────────────┘
```
```

---