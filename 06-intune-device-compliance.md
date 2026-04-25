# Step 06 - Microsoft Intune Device Compliance and Endpoint Management

**AZ-104 Domain:** Manage Azure Identities and Governance (Intune is part of M365/Entra ecosystem)
**Platform:** Microsoft Intune via M365 Developer E5 Sandbox (free 90-day tenant)
**Portal:** endpoint.microsoft.com
**Last reviewed:** 2026-07

---

## Why Intune Matters for an Azure Administrator

Microsoft Intune is the cloud-based Mobile Device Management (MDM) and Mobile
Application Management (MAM) platform in the Microsoft stack. AZ-104 candidates are
expected to understand its role in the Azure ecosystem, how it integrates with Entra ID
and Conditional Access, and how to perform basic device management tasks.

For anyone supporting Microsoft environments in East Africa - particularly fintech,
banking, NGOs, and enterprises under Kenya Data Protection Act or CBK compliance
requirements - Intune is increasingly the standard for endpoint management,
replacing on-premises Group Policy for modern device management.

---

## Prerequisites

This lab uses the **M365 Developer E5 Sandbox**, which is free for 90 days and
includes Intune, Entra ID P2, Microsoft Defender, and the full Microsoft 365 suite.

Sign up at: `https://developer.microsoft.com/en-us/microsoft-365/dev-program`

The M365 Dev tenant is separate from your Azure free trial subscription. Intune
operations in this section use the dev tenant.

---

## Step 1 - Enroll a Windows Device into Intune MDM

Manual MDM enrollment from a domain-joined or Azure AD-joined Windows 10/11 machine:

```
Settings → Accounts → Access work or school →
  Connect → Enter your M365 dev tenant credentials:
  Username : testuser1@contosodemo.onmicrosoft.com
  Password : [password]
→ The device enrolls into Intune MDM

OR for a newer Windows 11 device:
Settings → Accounts → Access work or school →
  Enroll only in device management →
  Enter M365 credentials
```

**Verify enrollment in Intune:**
```
endpoint.microsoft.com → Devices → All Devices
The enrolled device should appear within 5–10 minutes.
```
