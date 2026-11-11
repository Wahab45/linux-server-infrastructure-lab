# Evidence Register / Nachweisregister

## Purpose
This register separates **reported results**, **reproducible commands**, and **stored artifacts**. Missing screenshots should not be recreated as if they were taken during the original test.

| Exercise | Evidence currently in GitHub | Status | Next verification |
|---|---|---|---|
| Phases 1–7 | README narrative; no screenshots | Documented, but not independently verified | Capture configurations and relevant redacted command output on next run |
| Phase 8: Cron/monitoring | `infrastructure screenshots/phase8-crontab.png`, `phase8-cron-log-vergleich.png` | Two stored screenshots | Check current crontab and latest logs |
| Phase 9: DNS forwarder | `phase9-forwarder-vorher.png`, `phase9-dns-forwarder.png`, `phase9-dns-query.png` | Three stored screenshots | Repeat DNS lookup and check current resolver |
| Later Linux domain-client check | Only reported: `id wtest` displayed `hr@lab.local` | Reported, original output missing | Run `id wtest`, `getent passwd wtest`; verify login separately |
| Later Fail2ban test | Only reported: `Currently banned: 1`, banned IP `192.168.122.10`, `unbanip` returned `1` | Reported, original output missing | Verify jail/status, source logs, rules, and a controlled ban/unban |
| Original exercises called Parts 10–13 | No full exercise text or results provided | **Mapping not verified** | Reconstruct from saved notes / command history before assigning phase numbers |

## How to gather evidence later (German / English)

1. **Ausgangszustand / Baseline:** Write down the VM role, hostname, Ubuntu version, timestamp, test account, relevant IP addresses and intended outcome.
2. **Symptom / Symptom:** Preserve redacted command output and the failure.
3. **Diagnose / Diagnosis:** Record the exact commands and their results, including journal logs when useful.
4. **Lösung / Fix:** Document the applied change, with configuration diffs if possible.
5. **Nachtest / Verification:** Repeat the test and preserve the new result. Include rollback instructions.
6. **Datenschutz / Privacy:** Do not commit passwords, Kerberos tickets, private keys, tokens, confidential host details, or users' personal data. Use placeholders where necessary.

A text transcript in Markdown is acceptable evidence for learning; a screenshot is optional. Do not claim a result was reproduced unless the rerun actually happened.

## Suggested evidence filenames for future retests
- `evidence/client-id-wtest-redacted.txt`
- `evidence/fail2ban-status-redacted.txt`
- `evidence/fail2ban-journal-redacted.txt`
- `evidence/fail2ban-firewall-redacted.txt`

These filenames are **suggestions**, not files already collected.
