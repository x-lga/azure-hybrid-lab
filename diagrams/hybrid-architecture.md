# Hybrid Architecture - Complete Lab Diagram and Component Reference

---

## Full Architecture Diagram

```
╔══════════════════════════════════════════════════════════════════════════╗
║                        ON-PREMISES (Proxmox Lab)                         ║
║                                                                          ║
║  ┌─────────────────────────────────────────────────────────────────┐     ║
║  │  Windows Server 2022 - dc01.contoso.local                       │     ║
║  │  IP: 192.168.10.10                                              │     ║
║  │  Roles: AD DS, DNS, DHCP                                        │     ║
║  │  Azure AD Connect (PHS mode)                                    │     ║
║  │  Splunk Universal Forwarder                                     │     ║
║  │  Azure Monitor Agent (via Azure Arc)                            │     ║
║  └──────────────────┬────────────────────────────────────────────-─┘     ║
║                     │ AD Connect Sync (HTTPS/443 to Azure)               ║
║  ┌──────────────────▼──────────────────────────────────────────────┐     ║
║  │  Windows 10 Client - win10-client.contoso.local                 │     ║
║  │  IP: 192.168.10.50                                              │     ║
║  │  Domain-joined: contoso.local                                   │     ║
║  │  Intune enrolled: M365 Dev tenant                               │     ║
║  └─────────────────────────────────────────────────────────────────┘     ║
╚══════════════════════════════════════════════════════════════════════════╝
                     │
                     │ HTTPS (AD Connect sync, Azure Arc, Monitor Agent)
                     │
╔════════════════════▼═════════════════════════════════════════════════════╗
║                        MICROSOFT AZURE                                   ║
║                                                                          ║
║  ┌─────────────────────────────────────────────────────────────────┐     ║
║  │  Azure Entra ID (Tenant: contosodemo.onmicrosoft.com)           │     ║
║  │  ├─ testuser1 (Synced from contoso.local via AD Connect)        │     ║
║  │  ├─ testuser2 (Synced from contoso.local via AD Connect)        │     ║
║  │  ├─ RBAC: junioradmin → Contributor → rg-hybrid-lab             │     ║
║  │  ├─ RBAC: auditor → Reader → Subscription                       │     ║
║  │  └─ RBAC: vmoperator → VM Contributor → vm-win-server           │     ║
║  └─────────────────────────────────────────────────────────────────┘     ║
║                                                                          ║
║  ┌─────────────────────────────────────────────────────────────────┐     ║
║  │  Resource Group: rg-hybrid-lab                                  │     ║
║  │                                                                 │     ║
║  │  ┌──────────────────────────────────────────────────────────┐   │     ║
║  │  │  Virtual Network: vnet-hybrid-lab (10.20.0.0/16)         │   │     ║
║  │  │                                                          │   │     ║
║  │  │  ┌────────────────────────────────────────────────────┐  │   │     ║
║  │  │  │  subnet-servers: 10.20.1.0/24                      │  │   │     ║
║  │  │  │  NSG: nsg-subnet-servers                           │  │   │     ║
║  │  │  │                                                    │  │   │     ║
║  │  │  │  ┌──────────────────────────────────────────────┐  │  │   │     ║
║  │  │  │  │  vm-win-server                               │  │  │   │     ║
║  │  │  │  │  Windows Server 2022 — Standard_B1s          │  │  │   │     ║
║  │  │  │  │  Private IP: 10.20.1.4                       │  │  │   │     ║
║  │  │  │  │  No public IP                                │  │  │   │     ║
║  │  │  │  │  Access: Azure Bastion only                  │  │  │   │     ║
║  │  │  │  └──────────────────────────────────────────────┘  │  │   │     ║
║  │  │  └────────────────────────────────────────────────────┘  │   │     ║
║  │  │                                                          │   │     ║
║  │  │  ┌────────────────────────────────────────────────────┐  │   │     ║
║  │  │  │  subnet-linux: 10.20.2.0/24                        │  │   │     ║
║  │  │  │                                                    │  │   │     ║
║  │  │  │  ┌──────────────────────────────────────────────┐  │  │   │     ║
║  │  │  │  │  vm-ubuntu                                   │  │  │   │     ║
║  │  │  │  │  Ubuntu Server 22.04 — Standard_B1s          │  │  │   │     ║
║  │  │  │  │  Private IP: 10.20.2.4                       │  │  │   │     ║
║  │  │  │  │  No public IP                                │  │  │   │     ║
║  │  │  │  │  Access: Azure Bastion only                  │  │  │   │     ║
║  │  │  │  └──────────────────────────────────────────────┘  │  │   │     ║
║  │  │  └────────────────────────────────────────────────────┘  │   │     ║
║  │  │                                                          │   │     ║
║  │  │  ┌────────────────────────────────────────────────────┐  │   │     ║
║  │  │  │  AzureBastionSubnet: 10.20.254.0/27                │  │   │     ║
║  │  │  │  Azure Bastion (Developer SKU)                     │  │   │     ║
║  │  │  │  Public IP: bastion-pip                            │  │   │     ║
║  │  │  │  Access: HTTPS port 443 only                       │  │   │     ║
║  │  │  └────────────────────────────────────────────────────┘  │   │     ║
║  │  └──────────────────────────────────────────────────────────┘   │     ║
║  │                                                                 │     ║
║  │  Log Analytics Workspace: law-hybrid-lab                        │     ║
║  │  Azure Monitor Alert Rules + Action Group (ag-it-alerts)        │     ║
║  │  Microsoft Intune (M365 Dev Tenant) - device compliance         │     ║
║  └─────────────────────────────────────────────────────────────────┘     ║
╚══════════════════════════════════════════════════════════════════════════╝
```

---
