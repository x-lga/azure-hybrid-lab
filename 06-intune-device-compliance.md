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

**What enrollment establishes:**
- Intune can now push configuration profiles, compliance policies, and apps
- The device reports its compliance status to Intune
- Conditional Access can block/allow access based on compliance state

---

## Step 2 - Create a Windows Device Compliance Policy

A compliance policy defines the requirements a device must meet to be considered
"compliant." Non-compliant devices can be blocked from accessing M365 and Azure
resources via Conditional Access policies.

```
Intune Portal (endpoint.microsoft.com) →
  Devices → Compliance Policies → Create Policy →
  Platform: Windows 10 and later

Name: compliance-windows-security-baseline

Settings — Device Health:
  Require BitLocker encryption       : Require
  Require Secure Boot                : Require
  Require code integrity             : Require

Settings — Device Properties:
  Minimum OS version                 : 10.0.19041
  (Windows 10 May 2020 Update — baseline minimum)

Settings — System Security:
  Require password to unlock mobile  : Require
  Required password type             : Alphanumeric
  Minimum password length            : 10
  Password expiration (days)         : 90
  Number of previous passwords to    : 5
    prevent reuse
  Encryption of data storage         : Require
  Firewall                           : Require
  Antivirus (Windows Security)       : Require
  Antispyware (Windows Security)     : Require
  Microsoft Defender Antimalware     : Require

Settings — Microsoft Defender for Endpoint:
  Require machine risk score ≤       : Low
  (If Defender for Endpoint is licensed)

Assignments:
  Assign to: All Devices
  (or assign to a specific group for scoped rollout)
```

After creating the policy:
```
Intune Portal → Devices → All Devices → [your enrolled device]
Device status should show: Compliant or Not Compliant
```

If Non-Compliant, click the device to see which settings are failing.
Common causes:
- BitLocker not enabled on the test VM (expected — enable BitLocker or
  update the policy to not require it for the lab)
- OS version below minimum (update the machine or lower the requirement)
- Password policy not enforced (configure via Local Group Policy or Configuration Profile)

---

## Step 3 - Create a Device Configuration Profile

Configuration profiles push settings to devices — equivalent to Group Policy Objects
but managed from the cloud and applied to Entra ID-joined or Intune-enrolled devices.

```
Intune Portal → Devices → Configuration Profiles →
  Create Profile → Windows 10 and later → Settings Catalog

Name: config-security-baseline-windows

Add settings — search for and add each:

Category: Device Lock
  DevicePasswordEnabled                : Enabled
  DevicePasswordHistory                : 5
  DevicePasswordExpiration             : 90
  MinDevicePasswordLength              : 10
  AlphanumericDevicePasswordRequired   : Required

Category: Windows Defender
  AllowRealTimeMonitoring              : Allowed
  AllowCloudProtection                 : Allowed
  EnableNetworkProtection              : Enabled (Block mode)

Category: Windows Update
  AutoUpdateMode                       : Auto-install and restart at scheduled time
  ScheduledInstallDay                  : Every Sunday
  ScheduledInstallTime                 : 2 AM

Category: Browser (if Edge is in scope)
  PreventSmartScreenPromptOverride     : Enabled
  SmartScreenEnabled                   : Enabled

Assignments:
  Assign to: All Devices
```
After saving and assigning:
```
Intune Portal → Devices → Configuration Profiles →
  [your profile] → Device Status

Shows: Succeeded / Error / Conflict per device
```

---

## Step 4 - Deploy a Win32 Application Silently

Win32 app deployment through Intune allows silent installation of traditional
Windows applications (.exe, .msi) without user interaction.

**Prerequisites - Win32 Content Prep Tool:**
Download from: `https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool`
This tool converts your installer into an `.intunewin` package that Intune can deploy.

**Example: Deploy 7-Zip silently**

1. Download 7-Zip installer: `7z2301-x64.exe` from 7-zip.org
2. Package with the Content Prep Tool:
```cmd
IntuneWinAppUtil.exe -c C:\Apps\7zip -s 7z2301-x64.exe -o C:\Output
```
This creates: `7z2301-x64.intunewin`

3. Upload to Intune:
```
Intune Portal → Apps → Windows Apps → Add →
  App type: Windows app (Win32)

Upload: [select 7z2301-x64.intunewin]

App information:
  Name        : 7-Zip 23.01
  Description : [optional]
  Publisher   : Igor Pavlov

Program:
  Install command   : 7z2301-x64.exe /S
  Uninstall command : MsiExec.exe /X{23170F69-40C1-2702-2301-000001000000} /quiet
  Install behavior  : System

Detection rules:
  Rule type : File
  Path      : C:\Program Files\7-Zip
  File      : 7z.exe
  Detection method: File or folder exists

Assignments:
  Required: All Devices
  (Required means install automatically — user does not need to trigger it)
```

**Verify deployment:**
```
Intune Portal → Apps → All Apps → 7-Zip 23.01 → Device Install Status

Shows: Installed / Pending / Failed per device
```

---