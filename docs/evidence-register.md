# Evidence Register / Nachweisregister

## Purpose
All 13 training parts were carried out as described by the project owner. This register differentiates **completed practical work**, the **level of retained documentation**, and **optional future exercises**. A missing screenshot does not mean a test was not performed or did not work.

**Sources:** The supplied Arabic full lab journal (Parts 1–13), German learning notes (Teile 1–13), README lab history and stored GitHub screenshots. The journals are not uploaded unchanged because their page branding must be removed before public distribution.

| Part | Completed lab content | Archived evidence |
| --- | --- | --- |
| 1–3 | Hyper-V lab, Ubuntu Server, QEMU/libvirt, first test VMs | Written chronological journals |
| 4–5 | Samba AD DC, Kerberos, domain users and groups | Journals; Samba configuration screenshot captured later (`Screenshots/phase11-samba-configuration.png`) |
| 6–7 | Authentication log analysis and Bash detection script | Journals; `Screenshots/phase07-monitoring-script.png` |
| 8 | Cron and incremental log processing | `Screenshots/phase08-crontab.png`, `Screenshots/phase08-cron-log-comparison.png` |
| 9 | Samba DNS query and external forwarder repair | `Screenshots/phase09-dns-forwarder-before.png`, `Screenshots/phase09-dns-forwarder-fixed.png`, `Screenshots/phase09-dns-zone-query.png` |
| 10 | Kea DHCP, libvirt DHCP adjustment, Ubuntu client lease test | Eight `phase10-` screenshots in `Screenshots/`, including service, configuration, network and lease results; see README |
| 11 | Ubuntu domain join, SSSD, login, home creation and group-based sudo | Detailed Arabic and German journals; original complete terminal transcript not archived |
| 12 | HR share and five troubleshooting scenarios; wrong-host `samba-tool` issue | Detailed Arabic and German journals; [scenario runbooks](runbooks/domain-client-and-hr-share.md) |
| 13 | Fail2ban SSH test, recorded ban/unban, different-source connectivity and resource checks | Detailed Arabic and German journals; [Fail2ban runbook](runbooks/fail2ban-ssh-test.md) |

## Technical qualifications

- The Part 12 **clock-skew** experiment did not produce the anticipated Kerberos error. The clock was restored. The experiment was completed, but the predicted fault was not reproduced.
- The Part 12 `chmod 0777` permissions were deliberately permissive for a controlled lab and should not be copied into a production configuration.
- The Part 13 Fail2ban ban and unban were observed. The number of visible SSH messages must not be treated as a precise count of internal authentication failures.
- The Part 10 screenshots and journal document a functional DHCP exercise. A packet capture was not saved; this is a limit on forensic detail, not evidence of a failed configuration.

## Documentation hygiene

Only redacted, non-sensitive configs and source code should be committed. Avoid credentials, SSH keys, Kerberos tickets, customer records or confidential company material. Original screenshots and history should not be fabricated. Any future verification steps are optional expansions, **not requirements to repeat completed exercises**.
