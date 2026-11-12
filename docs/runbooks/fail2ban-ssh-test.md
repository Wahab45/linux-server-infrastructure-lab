# Runbook — Completed SSH / Fail2ban Lab Test (Ubuntu)

**Source:** Arabic full training journal (Part 13) and German notes (Teil 13). The exercise was completed and tested on Ubuntu VMs. Not every original terminal output was separately archived.

## Setup / Ausgangslage

Fail2ban was installed on the Ubuntu test client (`dhcp-test`, `192.168.122.50`). The journal records the following `/etc/fail2ban/jail.local` values:

```ini
[sshd]
enabled = true
backend = systemd
maxretry = 3
findtime = 10m
bantime = 120
```

These are the documented **lab settings**, not a recommended production baseline.

## Symptom and experiment

From the separate `samba-dc1` VM (`192.168.122.10`), failed SSH authentication attempts were simulated against the Ubuntu client. The original notes record the test loop using `sshpass`, `baduser`, and a wrong password; this was an isolated authorized lab.

After two visible `Permission denied` responses, a `Connection refused` was observed. The Fail2ban status subsequently reported:

```text
Currently banned: 1
Banned IP list: 192.168.122.10
```

The login test from `Ubuntu-Server-Base` (`192.168.122.1`) continued to work. The test ban was manually cleared:

```bash
sudo fail2ban-client set sshd unbanip 192.168.122.10
```

The command returned `1`, indicating removal of one ban.

## Diagnosis / Fehleranalyse

The **ban status** is the strongest recorded evidence that the configured jail banned the source. `Connection refused` alone does not uniquely diagnose Fail2ban; it can also occur due to a rejecting firewall rule or a service/network condition. Similarly, two visible SSH failures are not necessarily only two internal authentication events.

The notes also document one unrelated failed SSH login from the management host before the test (`Currently failed: 1`). This demonstrated that accidental password mistakes can consume the configured retry threshold and risk locking out administrators.

## Resolution / Entsperrung

The controlled test ended with `unbanip` as recorded above. The journal also includes a routine resource check using:

```bash
uptime
free -h
df -h
sudo ss -tulnp
```

At the time, the notes recorded roughly 709 MiB total RAM, 393 MiB available, and 60% use of the root filesystem.

## Safety / Produktion

- Do not conduct failure simulations outside authorized test environments.
- Secure console or alternative administrative access before testing bans.
- Review `ignoreip` carefully for management addresses, based on an explicit access policy.
- Inspect logs and the active firewall action when diagnosing a real production SSH outage.
- Use appropriate SSH authentication policy and a properly reviewed Fail2ban configuration in production.

## German vocabulary

- **Sperrung** — ban / حظر
- **Entsperren** — unban / إلغاء الحظر
- **Fehlgeschlagene Anmeldung** — failed login / تسجيل دخول فاشل
- **Protokolldatei** — log file / ملف سجلات
- **Fehleranalyse** — troubleshooting / تشخيص المشكلة

The test is documented as completed. Missing historical full logs are a limitation of archived materials, not a failed exercise.
