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