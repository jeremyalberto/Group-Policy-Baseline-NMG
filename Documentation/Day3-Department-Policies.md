# Day 3: Department Policies

Groups decide access, OUs decide policy.

## Shares
| Share | Path on DC | Share permission | NTFS permission |
| :--- | :--- | :--- | :--- |
| Finance | C:\Shares\Finance | Authenticated Users: Change | Finance-Users: Modify |
| HR | C:\Shares\HR | Authenticated Users: Change | HR-Users: Modify |
| IT | C:\Shares\IT | Authenticated Users: Change | IT-Users: Modify |
| Operations | C:\Shares\Operations | Authenticated Users: Change | Operations-Users: Modify |
| Wallpapers | C:\Shares\Wallpapers | Domain Users: Read | Default (Users: Read and execute) |

On each department folder, inheritance was disabled and the Users group removed. SYSTEM and Administrators keep Full control. CREATOR OWNER was removed so users can't change permissions on files they create, which keeps access controlled only through the department security group. Modify lets department members open, add, edit, rename, and delete files, but not change permissions or take ownership. The share permission is left open on purpose because Windows uses whichever is more restrictive, so NTFS does the real control.

Each folder is shared on its own, so the network path is \\NMG-DC1\Finance, not \\NMG-DC1\Shares\Finance.

## Department GPOs (User Configuration)
| GPO | Linked to | Drive | Wallpaper | Extra settings |
| :--- | :--- | :--- | :--- | :--- |
| NMG-Finance-User-Policy | Finance OU | S: \\NMG-DC1\Finance | finance.jpg | USB storage blocked |
| NMG-HR-User-Policy | HR OU | S: \\NMG-DC1\HR | hr.jpg | USB storage blocked |
| NMG-IT-User-Policy | IT OU | S: \\NMG-DC1\IT | it.jpg | USB allowed |
| NMG-Operations-User-Policy | Operations OU | S: \\NMG-DC1\Operations | operations.jpg | USB storage blocked, Control Panel blocked |

Drive maps use Group Policy Preferences with the Update action and Reconnect checked. Wallpapers point to \\NMG-DC1\Wallpapers.

USB storage is blocked for Operations, Finance, and HR because all three deal with patient or sensitive information. IT keeps USB access since they use USB for imaging and troubleshooting tools.

A user can have the S: drive mapped from their OU's GPO and still be denied if they are not in the department security group. The drive map is just a shortcut, the security group is what grants access.

## Test results
- dchen (Finance): Finance wallpaper, S: drive mapped and accessible, NMG-Finance-User-Policy listed in gpresult
- dchen opening \\NMG-DC1\HR: access denied, as configured
- bfoster (Operations): Control Panel blocked by GPO

## Understand what you built

**1. dchen got her drive and wallpaper from the OU, but her access to the Finance folder came from what? Why are those two separate?**
Access comes from the security group. The NTFS permissions are assigned to Finance-Users. OUs are for applying policies, and security groups are for granting access to resources.

**2. Why are these User Configuration settings instead of Computer Configuration?**
These settings should follow the user. The wallpaper and drive are tailored to the department. If they were computer settings, they would follow the computer's OU (Workstations), and everyone signing in to that PC would get the same wallpaper and drives no matter their department.

**3. What happens to a Finance user's drive and wallpaper if they're moved to the HR OU but nobody updates their groups?**
They get the HR wallpaper and the HR S: drive, but they cannot open the drive because access comes from groups. They also still have access to the Finance share, since they are still in Finance-Users. That is access creep (privilege creep). An HR employee should only have HR access, and if their account is compromised the attacker now has financial data on top of employee records.

**4. Why is blocking USB storage for Finance and HR a HIPAA control?**
These groups handle PHI and other sensitive data. Blocking USB storage prevents accidental or intentional copying of patient information onto a personal drive that can be plugged into any device. It supports the HIPAA Security Rule's device and media controls. Managed network drives and cloud storage are the standard instead.

Date completed: October 5, 2026
