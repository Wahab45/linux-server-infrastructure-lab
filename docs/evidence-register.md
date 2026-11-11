# Evidence Register / Nachweisregister

## Purpose
The lab exercises were completed and tested by the project owner. This register tracks which evidence artifacts have been saved to GitHub. Missing screenshots do not mean an exercise failed or was not performed.

| Exercise | Evidence currently in GitHub | Status | Next verification |
|---|---|---|---|
| Phases 1–7 | README narrative; no screenshots | Completed in the lab; documentation is mainly textual | Capture configurations and relevant redacted command output on next run |
| Phase 8: Cron/monitoring | `Screenshots/phase08-crontab.png`, `phase08-cron-log-comparison.png` | Two stored screenshots | Check current crontab and latest logs |
| Phase 9: DNS forwarder | `phase09-dns-forwarder-before.png`, `phase09-dns-forwarder-fixed.png`, `phase09-dns-zone-query.png` | Three stored screenshots | Repeat DNS lookup and check current resolver |
| Later Linux domain-client check | Only reported: `id wtest` displayed `hr@lab.local` | Test completed according to project owner; full original output not archived | Run `id wtest`, `getent passwd wtest`; verify login separately |
| Later Fail2ban test | Only reported: `Currently banned: 1`, banned IP `192.168.122.10`, `unbanip` returned `1` | Reported, original output missing | Verify jail/status, source logs, rules, and a controlled ban/unban |
| Phase 10 Kea/libvirt DHCP | Seven Phase-10 screenshots organized in `Screenshots/`. Kea active, libvirt XML without DHCP, client IP/lease screenshots | Lab test completed; screenshots stored; no DHCP packet capture archived | Capture DHCP option 54 and active libvirt net info |
| Samba configuration (`testparm -s`) | `Screenshots/phase11-samba-configuration.png` | Screenshot stored; output shows AD DC role, realm and shares | Functional access controls not independently proved by configuration screenshot |
| Bash monitoring script | `Screenshots/phase07-monitoring-script.png` | Screenshot stored | Script text not yet committed as a source file |
| Kea configuration | `Screenshots/phase10-kea-configuration.png` | Screenshot stored | Config text not yet committed as a source file |

## How to gather evidence later (German / English)

1. **Ausgangszustand / Baseline:** Write down the VM role, hostname, Ubuntu version, timestamp, test account, relevant IP addresses and intended outcome.
2. **Symptom / Symptom:** Preserve redacted command output and the failure.
3. **Diagnose / Diagnosis:** Record the exact commands and their results, including journal logs when useful.
4. **Lösung / Fix:** Document the applied change, with configuration diffs if possible.
5. **Nachtest / Verification:** Repeat the test and preserve the new result. Include rollback instructions.
6. **Datenschutz / Privacy:** Do not commit passwords, Kerberos tickets, private keys, tokens, confidential host details, or users' personal data. Use placeholders where necessary.

A text transcript in Markdown is acceptable evidence for learning; a screenshot is optional. Describe the user's completed lab work accurately. Distinguish missing archived evidence from a failed or unperformed test.

## Optional evidence filenames for future documentation
- `evidence/client-id-wtest-redacted.txt`
- `evidence/fail2ban-status-redacted.txt`
- `evidence/fail2ban-journal-redacted.txt`
- `evidence/fail2ban-firewall-redacted.txt`

These filenames are **suggestions**, not files already collected.
