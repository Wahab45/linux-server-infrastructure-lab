# Server Infrastructure Training

Trainingsprojekt: Linux Server Administration & Security (HybridShield Security)

---

### Phase 1 — Hypervisor & Lab-Umgebung einrichten

**Durchgeführte Schritte:**
- Windows-Rechner auf Kompatibilität geprüft (CPU, RAM, Hyper-V-Status)
- Isolierten virtuellen Switch erstellt (`LabSwitch-Internal`), um das Firmennetzwerk nicht zu berühren
- Ubuntu Server 24.04.4 LTS heruntergeladen und die Datei mit SHA256 überprüft:
```
Get-FileHash "ubuntu-24.04.4-live-server-amd64.iso" -Algorithm SHA256
```
- Erste virtuelle Maschine `Ubuntu-Server-Base` in Hyper-V erstellt und installiert

**Probleme und Lösungen:**
- PowerShell-Befehle für Hyper-V waren durch eine zentrale Firmenrichtlinie blockiert → Lösung: alles über die grafische Oberfläche (Hyper-V Manager) gemacht
- Secure Boot hat den Ubuntu-Start blockiert (Vorlage war "Microsoft Windows") → Vorlage auf "Microsoft UEFI-Zertifizierungsstelle" geändert

**Was ich gelernt habe:**
- Eine Prüfsumme (Checksum) zeigt, ob eine heruntergeladene Datei unverändert ist
- Manche Sicherheitsrichtlinien blockieren nur PowerShell, nicht die grafische Oberfläche

---

### Phase 2 — Ubuntu Server Installation & libvirt Aktivierung

**Durchgeführte Schritte:**
- Ubuntu Server installiert, SSH-Server aktiviert
- Zweite Netzwerkkarte für Internetzugang hinzugefügt (Default Switch)
- KVM/QEMU/libvirt installiert:
```
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
```

**Probleme und Lösungen:**
- Verschachtelte Virtualisierung (Nested Virtualization) war durch eine Firmenrichtlinie blockiert, auch für Lese-Befehle wie `Get-VMProcessor` → bis heute ungelöst, IT-Anfrage läuft; Training läuft trotzdem weiter, nur ohne Hardware-Beschleunigung
- `libvirtd` stürzte beim ersten Start ab, weil `/dev/kvm` fehlte → nach `sudo reboot` lief der Dienst normal im Software-Modus (TCG) weiter

**Was ich gelernt habe:**
- Ein Server kann auch ohne Hardware-Beschleunigung virtuelle Maschinen betreiben — nur langsamer
- Nach einem großen Update (`apt upgrade`) sollte man den Server neu starten

---

### Phase 3 — Aufbau von samba-dc1

**Durchgeführte Schritte:**
- Cloud-Image heruntergeladen und als "goldene Kopie" gesichert
- Erste Test-VM (`test-vm`) über `virt-install` erfolgreich gestartet
- Echten Server `samba-dc1` mit fester IP-Adresse erstellt:
```
sudo virt-install --name samba-dc1 --memory 1024 --vcpus 2 \
  --disk /var/lib/libvirt/images/samba-dc1.qcow2,format=qcow2 \
  --disk /var/lib/libvirt/images/samba-dc1-seed.iso,device=cdrom \
  --os-variant ubuntu24.04 --network network=default \
  --graphics none --console pty,target_type=serial --import
```

**Probleme und Lösungen:**
- Die Netzwerk-Konfiguration wurde nicht übernommen, weil die Festplatte von der schon benutzten `test-vm` kopiert wurde → neue, saubere Kopie vom Original-Image erstellt
- `virsh` fand die VM nicht, weil es zwei getrennte Bereiche gibt (`session` und `system`) → ab jetzt immer `sudo virsh --connect qemu:///system` verwendet

**Was ich gelernt habe:**
- Das Original-Cloud-Image darf man nie direkt starten — immer nur Kopien davon benutzen ("Goldenes Abbild")
- `virsh` braucht immer denselben Verbindungsbereich, sonst findet man seine eigenen VMs nicht wieder

---

### Phase 4 — Samba AD DC Installation

**Durchgeführte Schritte:**
- Voraussetzungen geprüft: FQDN (`hostname -f`) und Zeit-Synchronisation (`timedatectl`)
- Samba, Kerberos und Winbind installiert
- Domäne `lab.local` erstellt:
```
sudo samba-tool domain provision --use-rfc2307 --realm=LAB.LOCAL \
  --domain=LAB --server-role=dc --dns-backend=SAMBA_INTERNAL
```

**Probleme und Lösungen:**
- `provision` schlug fehl, weil schon eine alte `smb.conf` existierte → alte Datei umbenannt, `provision` erneut ausgeführt
- Der Dienst `samba-ad-dc` konnte den DNS-Port 53 nicht belegen, weil `systemd-resolved` ihn schon benutzte → `DNSStubListener=no` gesetzt und `systemd-resolved` neu gestartet

**Was ich gelernt habe:**
- Ubuntu hat standardmäßig einen eigenen DNS-Dienst, der mit neuen DNS-Servern in Konflikt geraten kann
- Eine Fehlermeldung nennt oft direkt die Lösung — genau lesen, bevor man weitersucht

---

### Phase 5 — Benutzer, Gruppen & Kerberos-Test

**Durchgeführte Schritte:**
- Ersten Domänen-Benutzer und eine Gruppe erstellt:
```
sudo samba-tool user create wtest '<Passwort>'
sudo samba-tool group add ITAdmins
sudo samba-tool group addmembers ITAdmins wtest
```
- Kerberos-Anmeldung getestet:
```
kinit wtest
klist
```

**Probleme und Lösungen:**
- Das Sonderzeichen `!` im Passwort verursachte einen Bash-Fehler → Passwort in einfache Anführungszeichen gesetzt
- Passwort von `wahab` vergessen → über die direkte Konsole (`virsh console`) neu gesetzt

**Was ich gelernt habe:**
- Ein erfolgreiches Kerberos-Ticket ist der beste Beweis, dass die ganze Kette (Benutzer, LDAP, Kerberos) funktioniert
- Wer Konsolen-Zugriff auf einen Server hat, kann Passwörter zurücksetzen — Festplattenverschlüsselung schützt davor

---

### Phase 6 — Monitoring: Authentifizierungs-Logs

**Durchgeführte Schritte:**
- Log-Level von Samba erhöht, um Anmeldeversuche sichtbar zu machen (in `smb.conf`, Abschnitt `[global]`):
```
log level = 3
```
- Erfolgreiche und fehlgeschlagene Anmeldung im Log verglichen

**Probleme und Lösungen:**
- Erfolgreiche Anmeldungen erschienen gar nicht im Standard-Log → Log-Level erhöht und `samba-ad-dc` neu gestartet

**Was ich gelernt habe:**
- Die Event-IDs `4624` (erfolgreiche Anmeldung) und `4625` (fehlgeschlagene Anmeldung) stammen aus dem **Windows Security Event Log**. Samba-Logs auf Ubuntu verwenden nicht automatisch diese Windows-Event-IDs. In diesem Lab wurden Anmeldeversuche anhand der Samba-/Linux-Logs ausgewertet; für eine Zuordnung zu Windows-Event-IDs braucht man passende Windows-Ereignisquellen oder eine nachvollziehbare Normalisierung.
- Ohne eine "Baseline" (normales Verhalten) kann man keinen Angriff erkennen

---

### Phase 7 — Erstes eigenes Überwachungsskript

**Durchgeführte Schritte:**
- Erstes Bash-Skript geschrieben, das fehlgeschlagene Anmeldungen zählt und warnt
- Mit einer Schleife 5 Fehlversuche simuliert:
```
for i in 1 2 3 4 5; do echo "wrongpass$i" | kinit wtest; done
```

**Probleme und Lösungen:**
- Die genaue Anzahl war höher als erwartet (12 statt 7), weil Kerberos pro Versuch mehrere interne Ereignisse loggt → kein Fehler, der Erkennungsmechanismus selbst hat trotzdem richtig gewarnt

**Was ich gelernt habe:**
- Ein einfaches Skript mit `grep -c` und `if/else` ist das Grundprinzip jedes Erkennungssystems (auch bei großen Tools wie Fail2ban)

---

### Phase 8 — Cron Job & intelligentes Log-Tracking

**Durchgeführte Schritte:**
- Skript als geplante Aufgabe (Cron Job) alle 5 Minuten eingerichtet:
```
*/5 * * * * /home/wahab/check_failed_logins.sh >> /home/wahab/monitoring.log 2>&1
```
- Skript verbessert: merkt sich jetzt die letzte Log-Position, statt immer die ganze Datei neu zu prüfen

**Probleme und Lösungen:**
- `sudo wc -l < datei` gab "Permission denied", weil die Weiterleitung (`<`) mit den Rechten des normalen Benutzers läuft, nicht mit `sudo` → durch `sudo cat datei | wc -l` ersetzt

**Was ich gelernt habe:**
- `sudo befehl < datei` ist nicht dasselbe wie `sudo cat datei | befehl` — ein wichtiger, oft übersehener Unterschied
- Ein gutes Überwachungsskript merkt sich seinen letzten Stand, sonst wiederholt es alte Warnungen für immer

![Cron Job in der crontab](infrastructure%20screenshots/phase8-crontab.png)

![monitoring.log: alte Läufe (12) und neue Läufe (0)](infrastructure%20screenshots/phase8-cron-log-vergleich.png)

---

### Phase 9 — DNS prüfen und Forwarder reparieren

**Durchgeführte Schritte:**
- DNS-Zonen von Samba angezeigt:
```
sudo samba-tool dns zonelist samba-dc1.lab.local -U Administrator
```
- Inhalt der Zone `lab.local` abgefragt:
```
sudo samba-tool dns query samba-dc1.lab.local lab.local @ ALL -U Administrator
```
- Externe Namensauflösung getestet:
```
host google.com 127.0.0.1
```

**Probleme und Lösungen:**
- `samba-tool dns` ohne `-U` fragte nach dem Passwort von `LAB\root` → mit `sudo` läuft der Befehl als `root`, und diesen Benutzer gibt es in der Domäne nicht. Lösung: Benutzer mit `-U Administrator` angeben
- Externe Adressen wie `google.com` wurden nicht aufgelöst (`timed out`) → in `smb.conf` stand `dns forwarder = 127.0.0.53`. Das war die Adresse von `systemd-resolved`, den ich in Phase 4 abgeschaltet hatte. Lösung: neuen Forwarder setzen und den Dienst neu starten:
```
sudo sed -i 's/dns forwarder = 127.0.0.53/dns forwarder = 192.168.122.1/' /etc/samba/smb.conf
sudo systemctl restart samba-ad-dc
```
- Ein PowerShell-Befehl wurde in die Bash-Sitzung eingefügt, und es erschien nur noch `>` → Bash wartete auf das Ende eines unvollständigen Befehls. Lösung: `Ctrl+C`

**Was ich gelernt habe:**
- Samba speichert den Forwarder beim `provision` fest in `smb.conf`. Ändert man später das System, muss man ihn selbst anpassen
- Vorher prüfen, ob der neue Forwarder antwortet (`host google.com 192.168.122.1`), und erst dann ändern
- Ein einzelnes `>` als Prompt bedeutet: Bash wartet auf das Ende eines Befehls

![DNS-Forwarder: Test vorher (timed out)](infrastructure%20screenshots/phase9-forwarder-vorher.png)

![DNS-Forwarder: Änderung und erfolgreicher Test](infrastructure%20screenshots/phase9-dns-forwarder.png)

![DNS-Abfrage der Zone lab.local](infrastructure%20screenshots/phase9-dns-query.png)
