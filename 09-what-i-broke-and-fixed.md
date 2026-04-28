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
