# Mock Security Incident Report

**Incident ID:** INC-2026-001
**Classification:** Account Lockout — Repeated Failed Authentication
**Environment:** HOMELAB.LOCAL (isolated Active Directory lab)
**Reported by:** Ahmed Mahdy, IT Support Engineer (Lab Simulation)
**Date of Report:** September 12, 2026

---

## 1. Incident Summary

On September 12, 2026, at approximately 15:31 EDT, the domain user account `jsmith` (IT Department, HOMELAB\IT OU) was automatically locked out following five consecutive failed logon attempts from workstation `WKS01`. The lockout was triggered by the domain's configured Account Lockout Policy, which enforces a threshold of 5 invalid attempts before automatic lockout. This event was simulated to test the effectiveness of the lab's account lockout controls and to practice a standard SOC detection-and-response workflow.

## 2. Timeline

| Time (approx.) | Event |
|---|---|
| 15:31:xx | Five consecutive failed logon attempts recorded against `jsmith` from `WKS01` |
| 15:31:19 | Account lockout triggered; Event ID 4740 logged on Domain Controller (DC01) |
| Shortly after | Lockout confirmed via `Get-ADUser jsmith -Properties LockedOut` (returned `True`) |
| Shortly after | Event Viewer reviewed on DC01; Security log filtered for Event ID 4740 to confirm source and account |
| Shortly after | Account unlocked via Active Directory Users and Computers (Unlock Account) and verified via PowerShell (`LockedOut: False`) |

## 3. Technical Details

- **Affected account:** `jsmith` (HOMELAB\jsmith), member of IT organizational unit
- **Source computer:** WKS01 (192.168.38.20)
- **Domain Controller:** DC01 (192.168.38.10)
- **Triggering event:** Windows Security Event ID **4740** (A user account was locked out)
- **Policy enforced:** Account Lockout Policy — threshold: 5 invalid attempts, lockout duration: 15 minutes, reset counter after: 15 minutes (configured via Default Domain Policy, GPO)

## 4. Root Cause

Repeated invalid password attempts against the `jsmith` account from its assigned workstation (WKS01) triggered the domain's account lockout threshold. This scenario was simulated to represent either (a) a legitimate user mistyping credentials repeatedly, or (b) a potential brute-force/credential-guessing attempt by an unauthorized party or insider — both of which the lockout policy is designed to contain by limiting the number of guesses an attacker (internal or external) can make against a single account.

## 5. Impact Assessment

- **Confidentiality/Integrity:** No evidence of unauthorized access; the account was locked before any successful authentication occurred.
- **Availability:** Temporary — `jsmith` was unable to log on to `WKS01` for the duration of the lockout, until manually remediated.
- **Scope:** Contained to a single user account and single workstation; no indication of lateral movement or broader compromise.

## 6. Remediation Actions Taken

1. Confirmed lockout status via `Get-ADUser jsmith -Properties LockedOut`.
2. Reviewed Security Event Log (Event ID 4740) on DC01 to confirm the source computer and timestamp, establishing an audit trail.
3. Unlocked the account via Active Directory Users and Computers (Account tab → Unlock account).
4. Verified successful unlock via `Get-ADUser jsmith -Properties LockedOut` (returned `False`).
5. Advised (in a real environment) that the affected user reset their password if the cause is confirmed to be a forgotten/mistyped credential rather than malicious activity.

## 7. Recommendations

- **Centralized alerting:** Forward Security Event ID 4740 to a SIEM (e.g., Splunk, Microsoft Sentinel — see related lab projects) to enable real-time alerting on lockout events rather than manual log review.
- **Pattern monitoring:** Establish a detection rule for repeated 4740 events across multiple accounts within a short window, which would indicate a broader brute-force campaign rather than an isolated user error.
- **User awareness:** Reinforce password hygiene training to reduce accidental lockouts from mistyped credentials.
- **Review lockout thresholds periodically:** Balance security (low threshold deters brute-force) against user experience (avoid excessive helpdesk tickets from legitimate mistakes).
- **Consider Just-In-Time (JIT) or MFA controls** for privileged accounts (e.g., IT-Admins group) to reduce reliance on password-only authentication.

---

*This report was produced as part of a self-directed home lab exercise simulating SOC/IAM incident response workflows. All systems, accounts, and data referenced are contained within an isolated, non-production lab environment (HOMELAB.LOCAL).*
