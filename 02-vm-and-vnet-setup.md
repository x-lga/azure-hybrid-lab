# Step 02 - Virtual Machine and Virtual Network Setup

**AZ-104 Domain:** Implement and Manage Virtual Networking, Deploy and Manage Azure Compute  
**Last reviewed:** 2026-07  

---

## Why This Step Matters

The virtual network is the foundation of the lab. Every other component - the VMs,
the NSGs, the Bastion, the monitoring - runs within this network. Getting the subnet
design and NSG rules right from the beginning is far easier than retrofitting them later.

The design here follows the Azure Security Benchmark principle: no public IPs on
management ports. RDP (3389) and SSH (22) are exposed to the internet on thousands
of Azure VMs, making them constant targets for brute-force attacks. This lab uses
Azure Bastion to provide browser-based RDP/SSH without any public IP on the VMs.

---

## Create the Resource Group

All lab resources are grouped together to simplify billing visibility, access control,
and cleanup.


Azure Portal → Resource Groups → Create

Subscription    : [Your free trial subscription]
Resource Group  : rg-hybrid-lab
Region          : (choose nearest to you - East Africa: South Africa North or UAE North)


> **Why a single resource group?**
> All lab resources share the same lifecycle - they are created together and deleted
> together. A single resource group makes it easy to assign a Contributor role scoped
> to the lab only, and to delete everything cleanly when the lab is no longer needed.

---

## Create the Virtual Network


Azure Portal → Virtual Networks → Create

Basics:
  Resource Group  : rg-hybrid-lab
  Name            : vnet-hybrid-lab
  Region          : (same as resource group)

IP Addresses:
  Address space   : 10.20.0.0/16
  Subnets:
    Name              : subnet-servers
    Address range     : 10.20.1.0/24
    (This is where the Windows Server VM lives)

    Name              : subnet-linux
    Address range     : 10.20.2.0/24
    (This is where the Ubuntu VM lives)

    Name              : AzureBastionSubnet
    Address range     : 10.20.254.0/27
    (REQUIRED name - Azure Bastion will not deploy without this exact name)
    (Minimum /27 required for Bastion)


**Why /16 address space?**
A /16 provides 65,536 addresses - far more than needed for this lab. This is intentional:
in a real enterprise Azure environment, you plan address space for growth, VNet peering,
and on-premises VPN connectivity. Designing a /16 from the start and carving /24 subnets
out of it is standard enterprise practice. Starting with a /24 VNet and running out of
space when you need to add a new subnet is a real operational headache.

**Why separate subnets for servers and Linux?**
Separate subnets allow separate NSG policies per subnet. In this lab, the servers subnet
has different inbound rules than the Linux subnet. In production, you might have database
servers, application servers, and management servers each in their own subnet with
separate NSGs enforcing traffic segmentation.

---

## Create the Network Security Group


Azure Portal → Network Security Groups → Create

Resource Group  : rg-hybrid-lab
Name            : nsg-subnet-servers
Region          : (same region)


After creation, configure inbound rules:


Azure Portal → nsg-subnet-servers → Inbound security rules → Add

Rule 1 - Allow RDP from your IP only (for Bastion use - this is belt-and-suspenders):
  Source               : IP Addresses
  Source IP            : [Your home/work public IP - check at whatismyip.com]
  Destination          : Any
  Destination port     : 3389
  Protocol             : TCP
  Action               : Allow
  Priority             : 100
  Name                 : Allow-RDP-MyIP

Rule 2 - Allow WinRM from within VNet (for PowerShell remoting between VMs):
  Source               : VirtualNetwork
  Destination          : VirtualNetwork
  Destination port     : 5985
  Protocol             : TCP
  Action               : Allow
  Priority             : 200
  Name                 : Allow-WinRM-VNet

Rule 3 - Allow SSH from within VNet (for management between VMs):
  Source               : VirtualNetwork
  Destination          : VirtualNetwork
  Destination port     : 22
  Protocol             : TCP
  Action               : Allow
  Priority             : 210
  Name                 : Allow-SSH-VNet

Rule 4 - Allow HTTPS outbound for Azure services:
  (This is an outbound rule - add under Outbound security rules)
  Source               : Any
  Destination          : Internet
  Destination port     : 443
  Protocol             : TCP
  Action               : Allow
  Priority             : 100
  Name                 : Allow-HTTPS-Outbound

Rule 5 - Deny all other inbound:
  Source               : Any
  Destination          : Any
  Destination port     : *
  Protocol             : Any
  Action               : Deny
  Priority             : 4000
  Name                 : Deny-All-Inbound


**Associate the NSG with subnet-servers:**

Azure Portal → vnet-hybrid-lab → Subnets → subnet-servers →
  Network Security Group → nsg-subnet-servers → Save  

**Why deny-all at the end?**
Azure NSGs have an implicit deny-all at the end. Adding an explicit Deny-All-Inbound
rule at priority 4000 makes this policy visible and intentional - it documents the
security posture rather than relying on implicit behaviour that a future engineer might
not be aware of.

---  