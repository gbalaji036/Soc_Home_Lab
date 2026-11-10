# Screenshot Manifest (internal — not required in the public repo)

Mapping from your original export (`cyber_screen_shorts.zip`, 46 files) to what's
used in this repo, so you can go back for more detail or extra evidence if needed.

## Used in the repo (safe, no credentials visible)

| Original filename | Used as |
|---|---|
| `Screenshot 2026-11-10 192352.png` | `screenshots/01-network-config.png` |
| `Screenshot 2026-11-10 204927.png` | `screenshots/02-wazuh-overview-menu.png` |
| `Screenshot 2026-11-10 212817.png` | `screenshots/03-disk-full-boot-failure.png` |
| `Screenshot 2026-11-10 211756.png` | `screenshots/03b-disk-full-df-output.png` |
| `Screenshot 2026-11-10 211103.png` | `screenshots/04-raw-log-evidence-and-hydra-missing.png` |
| `Screenshot 2026-11-10 214700.png` | `screenshots/05-bait-account-login-screen.png` |
| `Screenshot 2026-11-10 215219.png` | `screenshots/06-recovery-commands.png` |
| `Screenshot 2026-11-10 215933.png` | `screenshots/07-logs-flowing-after-recovery.png` |
| `Screenshot 2026-11-11 000256.png` | `screenshots/08-discover-28-hits-overview.png` |
| `Screenshot 2026-11-11 000700.png` | `screenshots/09-discover-rule-levels.png` |
| `Screenshot 2026-11-11 000727.png` | `screenshots/10-discover-rule-level-12-compromise.png` |

## ⚠️ DO NOT publish these two — real admin password visible in plaintext

| Original filename | Why excluded |
|---|---|
| `Screenshot 2026-11-10 201034.png` | Wazuh install summary prints the real admin password |
| `Screenshot 2026-11-10 215605.png` | A `curl` command includes the real admin password |

**Change this Wazuh admin password** since it's now been shared in this upload —
don't just avoid publishing the screenshot, rotate the credential itself.

## Remaining originals (not used — reviewed selectively; check before using any)

The other ~33 screenshots in your export are additional moments from the same
session (install steps, intermediate dashboard views, repeated troubleshooting
attempts, duplicate/near-duplicate shots). None were confirmed to contain
credentials beyond the two flagged above, but **before adding any of them to the
public repo, glance at each one yourself** — several are terminal screenshots and
it only takes one more `curl -u admin:<password>` to leak the same secret again.
If you want a second incident report (e.g. the nmap port scan), the timestamps
between `201305`–`204026` and `210320`–`210627` are good places to look for that
evidence.
