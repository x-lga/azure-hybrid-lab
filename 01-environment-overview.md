# Azure Hybrid Lab — Environment Overview

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
| Azure Entra ID (AAD) | Azure | Cloud identity — synced from on-prem AD |
| Azure AD Connect | On-prem VM | Syncs on-prem AD objects to Entra ID |