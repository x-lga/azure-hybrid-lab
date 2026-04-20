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