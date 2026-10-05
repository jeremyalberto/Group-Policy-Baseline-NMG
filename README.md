# Group Policy Baseline (NMG)

Part 2 of the Northstar Medical Group lab series. Part 1: [Basic Employee Onboarding (AD)(RBAC)](https://github.com/jeremyalberto/Basic-Employee-Onboarding-AD-RBAC-)

## Problem Statement
After the Active Directory rebuild, Northstar Medical Group had structure but nothing was being enforced. A HIPAA review flagged weak passwords, no account lockout, and no audit logging, so nobody could answer who logged in or who changed what. On top of that, new hires were mapping their own drives on day one, and nothing stopped staff from copying patient or payroll data onto a personal USB drive.

## Solution Overview
I joined a Windows 11 workstation (NMG-WS01) to NMG.com and moved it into its own Workstations OU so computer policies could actually reach it. At the domain level I set a 14 character password policy, a 5 attempt account lockout, and an audit baseline, then tested it by locking out a user and tracing the lockout and unlock in Event Viewer (4740 and 4767). I built a GPO for each department that maps their S: drive, sets their wallpaper, and blocks USB storage for Finance, HR, and Operations since they handle PHI. Access to each department share is controlled by NTFS permissions on the department security group, so OUs decide policy and groups decide access. Finally, I worked ticket NMG-0052, where an untested "security tightening" change broke the HR GPO for the whole department.

## Tools Used
* Windows Server 2022
* Windows 11 Enterprise (client)
* Active Directory Domain Services
* Group Policy Management Console
* Group Policy Preferences (Drive Maps)
* Event Viewer and auditpol
* gpresult and Group Policy Results (RSoP)
* Windows Defender Firewall (netsh)
* VMware Workstation

## Project Timeline
* Day 1: Client domain join, Workstations OU, logon banner GPO
* Day 2: Password, lockout, and audit baseline with a lockout trace in Event Viewer
* Day 3: Department shares, drive maps, wallpapers, and USB blocking
* Day 4: Troubleshooting ticket NMG-0052
* Day 5: Documentation and GitHub packaging

## Key Accomplishments
* Joined a Windows 11 client to NMG.com and organized computers into their own OU
* Enforced a 14 character password policy and 5 attempt lockout domain wide, and proved it by getting a weak password rejected
* Enabled advanced audit logging and traced a lockout through Event IDs 4740 and 4767, including the Caller Computer Name
* Built a GPO per department for drive mapping and wallpaper, with share access controlled by security groups
* Blocked USB storage for every department that handles PHI, with IT as the only exception
* Resolved a two cause GPO outage: the GPO could not be read by workstations, and a user was in the default Users container
* Diagnosed an unplanned "RPC server is unavailable" error during remote troubleshooting and fixed it by opening the WMI firewall rules

## GPO Summary
| GPO | Linked to | What it does |
| :--- | :--- | :--- |
| Default Domain Policy | NMG.com | 14 character passwords, complexity, 24 password history, no expiration (NIST), 5 attempt lockout for 15 minutes |
| NMG-Audit-Baseline | NMG.com | Advanced audit policy: logons, lockouts, account and group changes, audit policy changes |
| NMG-Workstation-Logon-Banner | Workstations OU | Authorized use and PHI warning before sign in |
| NMG-Finance-User-Policy | Finance OU | S: drive to \\NMG-DC1\Finance, Finance wallpaper, USB storage blocked |
| NMG-HR-User-Policy | HR OU | S: drive to \\NMG-DC1\HR, HR wallpaper, USB storage blocked, filtered to HR-Users |
| NMG-IT-User-Policy | IT OU | S: drive to \\NMG-DC1\IT, IT wallpaper, USB allowed for imaging and troubleshooting tools |
| NMG-Operations-User-Policy | Operations OU | S: drive to \\NMG-DC1\Operations, Operations wallpaper, USB storage blocked, Control Panel blocked |

## Lab Notes
* Lab accounts use a weak shared password that was set before the password policy existed. That shows password policy only applies when a password is changed, not to passwords that already exist. A new weak password was rejected (see Day2-Password-Rejected.png).
* Users can see every department share name at \\NMG-DC1 even though NTFS blocks access. In production I would put the shares under one Departments share with Access-Based Enumeration so users only see what they can open.
* Only the HR GPO uses security filtering, because the NMG-0052 scenario added it. The fix kept the filtering, so the original change's intent was preserved, and restored Read for Authenticated Users. The other department GPOs are scoped by OU alone, so security restrictions like the USB block apply to everyone in the OU. In production I'd raise a follow-up change to decide whether HR should match the others.
* CREATOR OWNER was removed from the department folders, so file creators can't change permissions and access only comes from the department group.
* Workstations keep their own sign in logs locally. The next step would be forwarding logs to one place with Windows Event Forwarding or a SIEM.

## Screenshots
| Day | File | Shows |
| :--- | :--- | :--- |
| 1 | Day1-Workstations-OU.png | Workstations OU with NMG-WS01 in it |
| 1 | Day1-WS01-Domain-Joined.png | NMG-WS01 joined to NMG.com |
| 1 | Day1-Logon-Banner-Settings.png | Logon banner title and text in the GPO |
| 1 | Day1-Logon-Banner-Login-Screen.png | Banner at sign in |
| 1 | Day1-gpresult-Computer.png | Banner GPO applied to WS01 |
| 2 | Day2-Password-Policy.png | Password policy settings |
| 2 | Day2-Account-Lockout-Policy.png | Lockout policy settings |
| 2 | Day2-Password-Rejected.png | Weak password rejected by policy |
| 2 | Day2-Audit-Baseline-Settings.png | Advanced audit policy in NMG-Audit-Baseline |
| 2 | Day2-auditpol-Verify.png | auditpol confirming audit settings on the DC |
| 2 | Day2-kmills-Locked-Out.png | kmills locked out after 5 bad attempts |
| 2 | Day2-Event-4740-Lockout.png | Event 4740 with Caller Computer Name NMG-WS01 |
| 2 | Day2-Event-4767-Unlock.png | Event 4767 showing the unlock |
| 3 | Day3-Finance-NTFS-Permissions.png | Finance-Users with Modify on the Finance folder |
| 3 | Day3-Finance-Wallpaper-Setting.png | Finance wallpaper setting |
| 3 | Day3-Finance-DriveMap.png | S: drive map to \\NMG-DC1\Finance |
| 3 | Day3-GPMC-All-GPOs-Linked.png | All department GPOs linked to their OUs |
| 3 | Day3-USB-Block-Setting.png | Removable storage deny setting |
| 3 | Day3-dchen-Finance-Desktop.png | dchen with Finance wallpaper and S: drive |
| 3 | Day3-dchen-HR-Access-Denied.png | dchen denied on \\NMG-DC1\HR |
| 3 | Day3-dchen-gpresult.png | Finance GPO applied to dchen |
| 3 | Day3-bfoster-Control-Panel-Blocked.png | Control Panel blocked for Operations |
| 4 | Day4-mgrant-Broken-Desktop.png | mgrant with default wallpaper and no S: drive |
| 4 | Day4-mgrant-HR-Share-Still-Accessible.png | mgrant can still open \\NMG-DC1\HR, so group access is fine |
| 4 | Day4-dchen-Phase1-Control-Check.png | dchen still working, so Group Policy itself is fine |
| 4 | Day4-storres-gpresult-No-HR-GPO.png | storres in OU=HR but no HR GPO applied |
| 4 | Day4-mgrant-gpresult-Wrong-Container.png | mgrant in CN=Users instead of the HR OU |
| 4 | Day4-GP-Results-RPC-Error.png | Remote Group Policy Results blocked by the firewall |
| 4 | Day4-Firewall-WMI-Fix.png | netsh enabling the WMI and Remote Event Log rules |
| 4 | Day4-GPMC-Results-storres.png | Group Policy Results for storres from the DC |
| 4 | Day4-HR-GPO-Delegation-Broken.png | HR GPO with no Authenticated Users Read |
| 4 | Day4-HR-GPO-Delegation-Fixed.png | Authenticated Users Read restored |
| 4 | Day4-storres-Fixed-Desktop.png | storres with HR wallpaper and S: drive |
| 4 | Day4-mgrant-Moved-To-HR-OU.png | mgrant moved into the HR OU |
| 4 | Day4-mgrant-Fixed-Desktop.png | mgrant with HR wallpaper and S: drive |

## Repository Structure
| Folder | Contents |
| :--- | :--- |
| `/Documentation` | Day 1 to Day 3 configuration notes and answers |
| `/GPO-Reports` | HTML export of every GPO in NMG.com (Get-GPOReport), showing each setting, link, and delegation in the final state |
| `/Incident-Reports` | NMG-0052 resolution plus the gpresult and Group Policy Results HTML reports used to diagnose it |
| `/Screenshots` | Proof of work from Days 1 through 4, named by day and step |
