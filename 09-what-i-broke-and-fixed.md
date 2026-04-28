# Step 09 - What I Broke and How I Fixed It

This document records the actual issues encountered during building this lab and
how they were diagnosed and resolved. This is what real IT work looks like.

Documenting what went wrong and how it was fixed is more valuable than pretending
everything worked on the first try. It demonstrates genuine troubleshooting experience
and practical problem-solving - exactly what employers want to see from a candidate
who claims hands-on experience.

---

## Issue 1: Azure AD Connect Sync Failing with "stopped-extension-dll-exception"

**What happened:**
After installing AD Connect and running the initial configuration, the first sync
cycle completed but subsequent delta syncs showed "stopped-extension-dll-exception"
in the Synchronisation Service Manager Operations tab. No users were being synced.

**Investigation:**
1. Opened: Start → Microsoft Entra Connect → Synchronisation Service
2. Operations tab showed the error: `stopped-extension-dll-exception`
3. Clicked on the failed run profile to expand the error details
4. Full error: `The server is not operational` - this was a misleading error message
5. Checked the event logs on the DC:
   ```powershell
   Get-EventLog -LogName Application -Source "ADSync" -Newest 20 |
       Select-Object TimeGenerated, EntryType, Message
   ```
6. Found: The ADSync service account did not have the required AD permissions

**Root cause:**
The express settings installation creates a service account (`MSOL_[hash]`) and
should automatically grant it the required AD permissions. However, in this lab
environment, a restrictive GPO was preventing the automatic delegation of the
"Replicate Directory Changes" permission.

**Fix:**
```powershell
# Grant the required permissions manually to the ADSync service account
# First, identify the service account name
Get-ADServiceAccount -Filter {Name -like "MSOL*"}
# Or check the ADSync connector configuration:
# Synchronisation Service → Connectors → [AD connector] → Properties →
#   Connect to Active Directory Forest → Account

# Grant "Replicate Directory Changes" to the service account
# This must be done on the domain root:
Import-Module ActiveDirectory
$ServiceAccount = "MSOL_abc123def456"  # Replace with actual account name
$DomainDN = (Get-ADDomain).DistinguishedName

# Use ADUC GUI:
# Active Directory Users and Computers →
# Right-click the domain root → Delegate Control →
# Add the MSOL account →
# Select: "Replicate directory changes"
```

After granting the permission, ran a full sync cycle:
```powershell
Start-ADSyncSyncCycle -PolicyType Initial
```

Sync completed successfully. Users appeared in Entra ID within 2 minutes.


**Lesson learned:**
The "stopped-extension-dll-exception" error in AD Connect is a generic catch-all
that can mean many things. Always check the full error details by clicking on the
failed run profile, not just the top-level status. The AD permissions issue was
buried in the expanded details.

---

## Issue 2: pfSense Firewall Blocking AD Connect Sync Traffic to Azure

**What happened:**
After initial sync completed successfully, AD Connect stopped syncing the following
day. The Synchronisation Service Manager showed "connectivity error" for the
Azure connector. The on-premises DC could not reach Azure Entra ID endpoints.

**Investigation:**
```powershell
# Test HTTPS connectivity to Azure AD endpoints
Test-NetConnection -ComputerName "login.microsoftonline.com" -Port 443
# Result: TcpTestSucceeded = False

Test-NetConnection -ComputerName "8.8.8.8" -Port 443
# Result: TcpTestSucceeded = True (internet itself was reachable)
```

This isolated the issue: port 443 to the internet worked generally, but the specific
Azure AD hostnames were being blocked. Checked the pfSense firewall logs.

**Root cause:**
pfSense had a DNS-based block rule targeting `microsoftonline.com` and related domains
(inherited from a DNS block list I had applied for ad-blocking purposes - the block
list incorrectly classified some Microsoft domains as advertising trackers).

**Fix:**
In pfSense:
```
DNS Resolver → Host Overrides → Add explicit allow for:
  login.microsoftonline.com
  login.windows.net
  aadcdn.msftauth.net
  graph.microsoft.com
  account.activedirectory.windowsazure.com

OR: Disable the DNS block list entirely for the DC's outbound traffic
```

Added the Azure AD endpoints to a DNS whitelist that overrides the block list.

After the fix:
```powershell
Test-NetConnection -ComputerName "login.microsoftonline.com" -Port 443
# Result: TcpTestSucceeded = True
```

Ran a manual sync cycle - completed successfully.

**Lesson learned:**
When AD Connect shows connectivity errors to Azure, check:
1. Direct network connectivity to Azure endpoints (Test-NetConnection)
2. DNS resolution for Azure hostnames (nslookup login.microsoftonline.com)
3. Any proxy or firewall rules that might be blocking based on domain rather than IP

---

## Issue 3: Splunk Universal Forwarder Not Sending Events to Log Analytics (Wrong Tool)

**What happened:**
(This issue involved a conceptual error, not a misconfiguration)

Initially attempted to forward Windows Security events to the Log Analytics Workspace
using the Splunk Universal Forwarder (which was already deployed in the lab for Repo #3).
The Splunk Forwarder was sending events to the Splunk Free instance, but I wanted
the same events in Azure Log Analytics for KQL querying.

**Root cause:**
Conceptual misunderstanding: the Splunk Universal Forwarder sends data to Splunk only.
Azure Log Analytics uses a completely different agent - the **Azure Monitor Agent (AMA)**
or the legacy **Log Analytics Agent (MMA/OMS agent)**.

**Fix:**
Installed the correct agent on the Windows DC:

```powershell
# Install the Azure Monitor Agent via the Azure Portal:
# Azure Portal → Virtual Machines → vm-win-server →
#   Extensions + Applications → Add →
#   Azure Monitor Agent → Install

# For on-premises machines (not Azure VMs), use Azure Arc:
# Azure Portal → Azure Arc → Servers → Add → Generate install script →
# Run the generated script on the on-premises DC
```

After installing the Azure Monitor Agent on the on-premises DC via Azure Arc,
the DC appeared in the Log Analytics Workspace as a connected data source and
Windows Security events began flowing within 15 minutes.

**Lesson learned:**
In a hybrid environment, there are multiple monitoring agents with different
destinations:
- **Splunk Universal Forwarder** → sends to Splunk
- **Azure Monitor Agent (AMA)** → sends to Log Analytics Workspace / Azure Monitor
- **Microsoft Defender for Endpoint sensor** → sends to Microsoft Defender portal
- **Sysmon** → writes to Windows Event Log (then forwarded by either agent above)

These are not interchangeable. The destination determines which agent you install.

---

## Issue 4: Intune Device Showing "Not Compliant" After Enrollment

**What happened:**
After enrolling the Windows 10 VM into Intune and applying the compliance policy
(compliance-windows-security-baseline), the device showed "Not Compliant" with
two failing requirements: BitLocker and Secure Boot.

**Investigation:**
```
Intune Portal → Devices → All Devices → [device name] → Device Compliance →
  Click on the policy → Shows which settings are non-compliant

Failing:
  - BitLocker: Not Configured
  - Secure Boot: Not Configured
```

