# Azure Hybrid Lab — VM and Virtual Network Setup

## Azure Free Tier Note

All resources deployed in Azure free trial or M365 Developer Sandbox. Use B1s VMs (1 vCPU, 1GB RAM — free tier eligible).

---

## Create the Virtual Network

Azure Portal → Virtual Networks → Create

```
Resource Group  : rg-hybrid-lab
Name            : vnet-hybrid-lab
Region          : (nearest to you)
Address space   : 10.20.0.0/16

