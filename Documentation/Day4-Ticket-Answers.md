# Day 4: Ticket NMG-0052 Questions

Full write up: `/Incident-Reports/NMG-0052-Resolution.txt`

**1. Why did fixing only the GPO permissions leave Michelle still broken?**
Fixing the delegation only fixed WS01 not being able to read the HR GPO. WS01 is a member of Authenticated Users, so giving that group Read is what let the policy download. But Michelle was in the default Users container in ADUC, not in any OU. GPOs cannot be linked to the Users container, so no department policy could reach her. That is why she still had no S: drive and the default wallpaper after the delegation fix. Once she was moved into the HR OU, the policies applied.

**2. What's the difference between Read and Read + Apply Group Policy?**
Read lets an account see and download the GPO. Apply Group Policy makes the settings actually take effect on that account. A user needs both to get the policy. Computers only need Read, because they download the GPO for the user. In the fix, Authenticated Users got Read only so WS01 could download the GPO, while HR-Users kept Read + Apply so the settings only reach HR.

**3. Why was checking storres and dchen in Phase 1 worth the extra two minutes?**
storres was in the correct OU and still broken, so OU placement could not be the only cause. That pointed the search at the policy itself, not just one user. dchen getting her Finance policies confirmed Group Policy was working overall and the break was limited to HR. Both checks pointed toward the root cause.

**4. How does this ticket compare to NMG-0047 (Jane Cooper)?**
Both were compounded issues that took more than one fix and extra checks to find the root cause. Jane's issue was only account level: she was in the wrong OU and the wrong security group, so she had the wrong user experience and the wrong permissions. This ticket was account level (mgrant not in an OU at all) and GPO level (the delegation was wrong, so WS01 could not read the policy for the whole department). Same lesson in both: OU placement, group membership, and GPO permissions are three separate layers, and each one has to be checked.
