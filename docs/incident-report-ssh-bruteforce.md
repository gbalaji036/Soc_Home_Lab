# Incident Report — SSH Brute-Force Against Bait Account

## Summary

| Field | Detail |
|---|---|
| Date detected | November 10, 2026 |
| Source | `192.168.50.3` (Kali Linux, attacker VM) |
| Destination | `192.168.50.4` (Ubuntu, SSH target + Wazuh SIEM host) |
| Target account | `victim` (intentionally weak bait credential) |
| Detection tool | Wazuh SIEM (manager + indexer + dashboard) |
| Attack tool | Hydra (dictionary attack against SSH) |
| Outcome | Account compromised as designed; SIEM correctly flagged both the
brute-force attempt and the eventual successful login |

## Timeline

1. **Reconnaissance** — `nmap` run from Kali against `192.168.50.4` to confirm
   SSH (port 22) was open and identify the service.
2. **Attack** — Hydra launched from Kali against the `victim` account over SSH,
   using a password list that included the bait password (`password123`).
3. **Detection** — Wazuh, ingesting `/var/log/auth.log` via a `localfile`
   (syslog) entry in `ossec.conf`, generated a sequence of alerts as the
   attack progressed:
   - Repeated `sshd: authentication failed` events (rule level 5) as each
     guess failed.
   - `PAM: Multiple failed logins in a small period of time` (rule level 10).
   - `sshd: brute force trying to get access to the system. Authentication
     failed.` (rule level 10) — Wazuh's frequency-based brute-force rule
     correlating the repeated failures.
   - `Multiple authentication failures followed by a success` (**rule level
     12**) — the highest-severity alert in this chain, fired once the correct
     password was found, confirming an actual compromise rather than just
     noisy failed attempts.
4. **Confirmation** — Verified directly against raw logs via `journalctl -u
   ssh | grep 192.168.50.3`, cross-referencing `Failed password for victim`
   and `Accepted password for victim` lines against the Wazuh alerts to
   confirm the SIEM's interpretation matched ground truth.
5. **Evidence captured** — Wazuh Discover, index pattern `wazuh-alerts-*`,
   query `data.srcip:192.168.50.3`, time range covering the attack window:
   **28 total matching events.**

## MITRE ATT&CK Mapping

| Technique | ID | Notes |
|---|---|---|
| Brute Force | [T1110](https://attack.mitre.org/techniques/T1110/) | Hydra dictionary attack against SSH credentials |
| Valid Accounts | [T1078](https://attack.mitre.org/techniques/T1078/) | Weak/bait credential successfully used post-compromise |
| Active Scanning | [T1595](https://attack.mitre.org/techniques/T1595/) | Preceding `nmap` reconnaissance |

## Detection Logic (why this worked)

Wazuh's default `sshd` and PAM rule groups use frequency-based correlation:
individual failed logins are low severity (level 5) on their own, but repeated
failures within a short window escalate to a brute-force rule (level 10), and
a failure-then-success pattern on the *same* account within the *same* window
escalates further to a level-12 compromise rule. This tiered escalation is
what let a single log source (`auth.log`) produce an alert that's actually
actionable — "this account is now compromised" — rather than just a wall of
undifferentiated failed-login noise.

## Response (what a SOC would do next)

1. Disable or force-reset the `victim` account immediately.
2. Block `192.168.50.3` at the firewall / add to a blocklist.
3. Check for lateral movement or further activity from the compromised
   account's session.
4. Review whether SSH should be restricted to key-based auth only, or placed
   behind a bastion/VPN, to prevent recurrence.

## Operational note

Mid-lab, the Ubuntu host's disk filled to 100% (see main README's
"Lessons learned"), which pushed the Wazuh indexer into read-only mode and
interrupted ingestion. This was diagnosed and resolved (cleared
`/var/ossec/tmp` and the vulnerability-detection queue, disabled that module,
resized the disk) before re-running the attack to capture the evidence above.
