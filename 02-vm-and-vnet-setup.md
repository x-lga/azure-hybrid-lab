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

Subnets:
  Name: subnet-servers   | Address: 10.20.1.0/24
  Name: subnet-linux     | Address: 10.20.2.0/24
  Name: AzureBastionSubnet | Address: 10.20.254.0/27  (required name for Bastion)
```

---

## Create NSG (Network Security Group)

Azure Portal → Network Security Groups → Create

```
Name: nsg-subnet-servers
Resource Group: rg-hybrid-lab

Inbound rules to ADD:
  Priority | Name              | Port  | Protocol | Source      | Action
  100      | Allow-RDP-Admin   | 3389  | TCP      | Your IP     | Allow
  200      | Allow-WinRM       | 5985  | TCP      | VNet        | Allow
  1000     | Deny-All-Inbound  | *     | *        | *           | Deny

Associate NSG with subnet-servers after creation.
```

---

## Deploy Windows Server VM