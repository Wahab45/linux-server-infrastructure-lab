# Runbook — SSH Brute-force Simulation & Fail2ban (Ubuntu-only)

**Category:** Linux system administration / defensive monitoring  
**Environment:** Ubuntu VMs, Samba DC and a separate Ubuntu test target  
**Evidence status:** Observations below are reported from an earlier lab; the original logs and jail configuration have not been uploaded.  
**Safety:** Use only in an isolated lab with authorization. Ensure console access to the target before testing; never risk locking yourself out of a remote production server.

## 1. Symptom / Beobachtung
The training summary recorded:
```text
Currently banned: 1
Banned IP list: 192.168.122.10
unbanip -> 1
```
It also reported rejected connections after failed authentication attempts while a separate SSH session from another source IP remained usable. These are **reported observations**, not complete preserved command outputs.

## 2. Assessment / Einordnung
Fail2ban reads configured log sources and can trigger firewall actions after matching failures. Banning an IP is evidence that a jail action triggered; it **does not prove** an actual intrusion or correct attribution of every failed login.

**Important:** The source `192.168.122.10` was reportedly the `samba-dc1` VM. Blocking a domain controller can have unintended effects in a real environment. The exact source of `Connection refused` cannot be established from this summary alone.

## 3. Diagnosis / Diagnose (run only when retesting)
First, find which jail applies:
```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```
Inspect the Fail2ban service and its journal:
```bash
sudo systemctl status fail2ban --no-pager
sudo journalctl -u fail2ban --since "30 minutes ago" --no-pager
```
Inspect recent SSH authentication events on a system using systemd-journald:
```bash
sudo journalctl -u ssh --since "30 minutes ago" --no-pager
```
Note: unit name and journal source can differ by Ubuntu release and local configuration. Confirm the real service and the jail's `backend`, `filter` and `logpath` before concluding.

Confirm which action has actually been configured:
```bash
sudo fail2ban-client get sshd actions
```
Firewall checks are backend-dependent (for example nftables or iptables). Use the configured action to decide what to inspect; don't assume a particular firewall tool.

## 4. Controlled fix / Entsperren
Only after recording the status and confirming an administrative reason to lift the ban:
```bash
sudo fail2ban-client set sshd unbanip 192.168.122.10
sudo fail2ban-client status sshd
```
A return value of `1` typically means one ban was removed. Confirm independently that the IP is no longer listed.

## 5. Verification / Nachtest
- Confirm SSH works for the intended permitted account.
- Confirm incorrect passwords fail without granting access.
- Confirm the offending lab source IP appears in the jail when its configured threshold is reached.
- Confirm a different source IP is not blocked merely because the first one was.
- Check the jail status, source authentication logs and actual firewall action together.

**Do not infer from an SSH error string alone** that Fail2ban caused it. Check logs, IP/port, listener and firewall state.

## 6. Troubleshooting table

| Symptom | Possible cause | Diagnostic direction |
|---|---|---|
| Jail is missing | Jail not enabled / config invalid | `fail2ban-client status`, service journal |
| Failures do not lead to ban | Filter mismatch, backend/source mismatch, threshold/timing | Compare actual auth logs to jail filter and settings |
| Wrong machine is blocked | All test traffic originated behind same NAT, or source IP misidentified | Inspect actual source address on target |
| Ban appears but SSH still works | Action/chain not effective, IPv4/IPv6 difference, existing connection | Inspect actual rules and test a **new** connection |
| SSH gives 'Connection refused' | No listener or active reject action, among other causes | Inspect `ss -lntp`, journal and firewall; do not assume cause |

## 7. Rollback / Wiederherstellung
Remove only the test ban; restore original jail settings if changed, and verify the service configuration. Avoid blanket firewall flushes or disabling SSH protection on real infrastructure.

## 8. Evidence to capture next time
Store redacted timestamps, jail configuration, source IP, ban counts, matched logs, firewall action and the successful post-unban connection. See [evidence register](../evidence-register.md).

## 9. German vocabulary
- **Sperrung** — Ban / حظر
- **Entsperren** — Unban / إلغاء الحظر
- **Fehlgeschlagene Anmeldung** — Failed login / تسجيل دخول فاشل
- **Protokolldatei** — Log file / ملف السجلات
- **Fehleranalyse** — Troubleshooting / تحليل الخطأ
- **Nachprüfung** — Verification / إعادة التحقق
