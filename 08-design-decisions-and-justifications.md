# Step 08 - Design Decisions and Justifications

This document is the most important file in the repo for interview preparation.
Every architectural choice in this lab has a documented reason. These are the
questions Azure hiring managers and interviewers ask.

**The pattern:** "Why did you do X instead of Y?" - this document answers every
variant of that question before it is asked.

---

## Decision 1: Password Hash Sync (PHS) instead of Pass-Through Authentication (PTA)

**What was chosen:** Password Hash Sync

**What was not chosen:** Pass-Through Authentication (PTA) or Active Directory
Federation Services (AD FS)

**Why PHS was chosen:**

PHS synchronises a hash of the on-premises password hash to Entra ID. The actual
password never leaves on-premises - only a salted, hashed representation is synced.
When a user authenticates to a cloud service, Entra ID validates the password hash
locally in the cloud without needing to reach back to the on-premises DC.