# Runbook — Ubuntu Domain Client & HR Share Troubleshooting

**Source:** Completed lab exercises documented in the Arabic training journal (Part 12) and German learning notes (Teil 12). These are historical records of actual lab exercises, not instructions to rerun them.

**Environment:** Ubuntu Samba AD DC (`samba-dc1`) and joined Ubuntu Linux client (`dhcp-test`), domain `lab.local`. Samba administrative changes were performed on the DC; client checks on the Ubuntu client.

## Scenario A — Share access denied

**Symptom:** CIFS mount/access to `hr-share` returns `mount error(13): Permission denied` for `wtest`.

**Diagnosis:** `samba-tool user show wtest` and `memberOf` showed that the user was not in the `HR` group.

**Fix actually performed:** On `samba-dc1`, add the user to the group:

```bash
sudo samba-tool group addmembers HR wtest
```

The journal records successful access after the change.

**Important limitation:** The share was deliberately configured with `chmod 0777` at the filesystem level during the lab, while a Samba `valid users` setting restricted access. This is **not** a production-ready permissions design; filesystem ACLs and share permissions should both be constrained.

## Scenario B — Domain account disabled

**Symptom:** Interactive user login failed.

**Diagnosis:** `kinit wtest@LAB.LOCAL` returned `Client's credentials have been revoked`; on the DC, `userAccountControl` changed to `514` (disabled), compared with `512` (normal enabled account).

**Fix actually performed:** On the DC:

```bash
sudo samba-tool user enable wtest
```

The journal records return to `512`. `sss_cache -E` was considered if stale client cached identity data persisted; do not represent that command as performed unless separately evidenced.

## Scenario C — Kerberos time difference (negative test)

**Attempt:** Deliberately moved the client's time approximately ten minutes.

**Observed result:** `kinit` **still succeeded**; the expected clock-skew authentication error was **not reproduced**.

**Recovery:** Restored time synchronization with `timedatectl set-ntp true` and the time-sync service as described in the notes.

**Learning:** A hypothesis about time-offset behavior is not proof of a reproduced outage. Compensation by the Kerberos client/KDC was suggested as one possible explanation but was not established conclusively.

## Scenario D — DNS name resolution failure

**Symptom:** `host samba-dc1.lab.local` reported `SERVFAIL`, while access by IP continued.

**Cause deliberately introduced:** DNS for `enp1s0` had been set to `192.168.122.99`.

**Diagnosis:** `resolvectl status` exposed the incorrect DNS server.

**Fix actually performed:**

```bash
sudo resolvectl revert enp1s0
```

The original DHCP-provided DNS server (`192.168.122.10`) returned, and resolution worked again.

## Scenario E — Client left the domain

**Symptom:** After `realm leave`, `realm list` had no member entry and `id wtest@lab.local` returned `no such user`.

**Diagnosis:** Domain membership was absent.

**Fix actually performed:**

```bash
sudo realm join -U Administrator lab.local
id wtest@lab.local
```

The journal records restored identity and group resolution.

## Separate troubleshooting finding — Samba command executed on the wrong VM

On the Ubuntu client, administrative `samba-tool` commands requiring the DC's local SAM database failed with an error that `/var/lib/samba/private/sam.ldb` could not be opened. Running the appropriate DC administration commands on `samba-dc1` resolved the workstation/server role confusion.

## Evidence and scope

The source journals record the exercises as completed; they are not all accompanied by separate screenshots. This does **not** mean they failed. The documentation describes specific observed outcomes and distinguishes the one time-difference test that did not trigger the anticipated failure.

No production security or domain behavior is claimed.
