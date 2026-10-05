# Day 1: First Policy

- Client name: NMG-WS01 (Windows 11 Enterprise)
- IP address: 192.168.187.50 (static), DNS pointed to NMG-DC1 at 192.168.187.150
- OU: Workstations
- GPO: NMG-Workstation-Logon-Banner, linked to the Workstations OU
- Settings: Interactive logon message title "Northstar Medical Group" and message text "Authorized use only. Activity on this system is monitored and may contain protected health information."
- Verified with `gpresult /r /scope:computer` (banner GPO and Default Domain Policy applied) and the banner showing at sign in

New computers land in the default Computers container. That is a container, not an OU, so GPOs cannot be linked to it. I created the Workstations OU and moved NMG-WS01 into it so computer policies could reach it.

The banner tells users this device is for work only, that activity is monitored, and that they may come across protected health information. It also helps with compliance, since users cannot claim they were never told.

## Understand what you built

**1. Why did the client need its DNS pointed at the DC instead of your router?**
The client needs DNS pointed at the DC to join the domain. NMG.com only exists on the local DC and is not on the internet, so the router cannot resolve it. Only the DC's DNS has the SRV records that tell a client where the domain controllers for NMG.com are.

**2. Is the logon banner a Computer or User setting? Why does that matter for where you link it?**
It's a computer setting. Computer settings only apply to computer objects, so if it were linked to an OU that only has users, nothing would happen. Linking it to the Workstations OU makes sure anyone who signs in to a workstation sees the message.

**3. What is the difference between creating a GPO and linking it?**
Creating a GPO makes the object and stores it in the Group Policy Objects container. On its own it does not apply to anything. Linking it to a site, domain, or OU is what makes it apply to the users or computers there. Deleting a link does not delete the GPO, it just stops it from applying there. "Create a GPO in this domain, and Link it here" does both at once.

**4. What does gpupdate /force do, and when would policy apply without it?**
`gpupdate /force` pulls policy from the domain and reapplies all of it, including settings that have not changed. Without it, computer policy applies at startup and user policy applies at logon. Windows also refreshes policy in the background about every 90 minutes.

Date completed: October 4, 2026
