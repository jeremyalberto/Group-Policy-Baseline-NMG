# Day 2: Security Baseline

## Password policy (Default Domain Policy)
| Setting | Value |
| :--- | :--- |
| Enforce password history | 24 passwords |
| Minimum password length | 14 characters |
| Password must meet complexity requirements | Enabled |
| Maximum password age | 0 (never expires, per NIST guidance) |

It may seem like passwords that never expire are less secure, but NIST SP 800-63B recommends removing forced expiration. Forced changes make users pick simpler passwords with predictable changes (Summer2025! becomes Fall2025!), so a threat actor with one old password can guess the next one. Length, lockout, and MFA do more. In an environment where an auditor requires rotation, I would follow that requirement.

To prove the policy works, I tried to reset a user's password to one that did not meet the requirements and it was rejected.

## Account lockout policy (Default Domain Policy)
| Setting | Value |
| :--- | :--- |
| Account lockout threshold | 5 invalid attempts |
| Account lockout duration | 15 minutes |
| Reset account lockout counter after | 15 minutes |

This slows down brute force password guessing.

## Audit policy (NMG-Audit-Baseline, linked at NMG.com)
Success and Failure enabled for Credential Validation, Logon, Account Lockout, User Account Management, Security Group Management, and Audit Policy Change. "Force audit policy subcategory settings to override audit policy category settings" is enabled. Verified on the DC with `auditpol /get /category:*`.

The GPO is linked at the domain so it applies to every computer. The DC records password checks, lockouts, and account and group changes for the whole domain, so if someone gets locked out or added to a group, I can see who, when, and from which computer. Workstations log their own sign ins locally. The next step would be forwarding those logs to one place (Windows Event Forwarding or a SIEM) so everything is searchable together.

## Lockout test
- User: kmills (Karen Mills), signed in on NMG-WS01 with the wrong password 5 times
- Event 4740 on NMG-DC1: account locked out, Caller Computer Name NMG-WS01
- Unlocked kmills in ADUC as NMG\Administrator
- Event 4767 on NMG-DC1: account unlocked, showing who unlocked it

## Understand what you built

**1. Why do password settings only work when linked at the domain level?**
Domain accounts are managed by the domain controller, not the local device. The password and lockout rules are stored on the domain object itself, and DCs only read them from GPOs linked at the domain. If they were linked to a workstation OU, they would only apply to local accounts on those computers.

**2. If IT admins needed a stricter 20 character password, how would you do it without changing everyone's?**
Use a Fine Grained Password Policy, called a Password Settings Object (PSO), created in Active Directory Administrative Center. PSOs apply to users or global security groups, not OUs, and override the domain password policy for those members. If a user is covered by more than one PSO, the one with the lowest precedence number wins.

**3. Why is 4740's Caller Computer Name useful when a user keeps getting locked out with no idea why?**
It shows where the failed attempts are coming from. If it is a workstation or site they don't use, at a time they weren't signing in, someone may be guessing their password. It can also confirm the user is failing the attempts themselves. The most common help desk cause is a device saving an old password after a password change, like email syncing on a phone, a scheduled task, a mapped drive, or a session left open on another PC. The Caller Computer Name tells you exactly which device to fix.

**4. Why is "Audit Policy Change" worth logging?**
It alerts you when auditing gets turned off. Threat actors often disable auditing so their next moves are not recorded. This shows up as event 4719, which records who changed the audit policy and when, and is something to investigate right away.

Date completed: October 4, 2026
