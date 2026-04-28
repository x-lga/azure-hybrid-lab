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