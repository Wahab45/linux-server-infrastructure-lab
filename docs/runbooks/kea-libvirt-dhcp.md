# Runbook — DHCP troubleshooting: Kea vs libvirt (Ubuntu-only Lab)

**Status:** Screenshots supplied by lab owner; snapshots from different points in time. Not independently reproduced.

## Ausgangslage / Baseline
- Host Ubuntu VM runs `libvirt`, default NAT network via bridge `virbr0`, gateway `192.168.122.1/24`.
- `samba-dc1` is an Ubuntu guest running `kea-dhcp4-server` on interface `enp1s0`.
- Another Ubuntu VM `dhcp-test` has IP `192.168.122.50/24` with default route `192.168.122.1`.

## Symptom / Beobachtung
One screenshot of `virsh net-dumpxml default` includes an internal libvirt DHCP range `192.168.122.2–192.168.122.254`; a later/other snapshot does not. The same subnet is served by Kea in the training environment, creating a **risk of competing DHCP servers** if both operate simultaneously.

## Diagnose / Diagnosis
Use these commands on the **Ubuntu libvirt host**:
```bash
sudo virsh --connect qemu:///system net-info default
sudo virsh --connect qemu:///system net-dumpxml default
sudo virsh --connect qemu:///system net-dumpxml --inactive default
sudo ss -lunp | grep ':67 '
```
Confirm which network is active and whether its DHCP service is actually listening. A static XML snapshot alone does not show which process responded to the last DHCP Discover.

On `samba-dc1`:
```bash
sudo systemctl status kea-dhcp4-server --no-pager
sudo journalctl -u kea-dhcp4-server --since "30 minutes ago" --no-pager
sudo tail -n 12 /var/lib/kea/kea-leases4.csv
```
On Ubuntu `dhcp-test`:
```bash
ip -4 address
ip route
resolvectl status
```
Use an isolated packet capture during a controlled renew to identify the responder by DHCP **Server Identifier (option 54)**. For example on an authorized VM or host interface:
```bash
sudo tcpdump -ni enp1s0 -vvv 'udp port 67 or udp port 68'
```
Interface names differ; replace `enp1s0` with the correct interface. Capture only your own lab traffic.

## Lösung / Fix strategy
Choose **one** DHCP authority for this subnet. In this lab, the intended DHCP server appears to be Kea, while libvirt supplies the NAT gateway. Do not blindly remove libvirt DHCP on a shared/production network: confirm affected guests and preserve a rollback copy of the network configuration. A full procedure must match the active/inactive libvirt network definition and maintenance window.

DNS and DHCP are separate: `nameserver 127.0.0.53` in Ubuntu's `/etc/resolv.conf` means a local **systemd-resolved stub**. It does **not** establish the upstream DNS server. Verify using `resolvectl status`.

## Nachtest / Verification
1. Confirm only the intended DHCP authority responds to a fresh DHCP Discover.
2. Confirm fresh lease data includes the expected client and lifetime.
3. Confirm client IP/netmask, route and DNS upstream independently.
4. Validate domain DNS, not merely external reachability (for example the domain SRV records).
5. Document the complete before/after network XML, Kea configuration (without secrets), and capture.

## What the screenshots prove / don't prove
**Supported:** Kea service active; CSV includes `192.168.122.50` for `dhcp-test`; client holds that address and route; snapshots demonstrate existence and absence of libvirt XML DHCP block.

**Not established conclusively:** Which server supplied every earlier lease; whether the runtime active libvirt network matched the displayed XML at the instant of lease acquisition; whether the Ubuntu client used the Samba DNS server as upstream.

## Deutsches Glossar
- **Adressvergabe**: IP address assignment / توزيع العناوين
- **Adressbereich**: DHCP address range / مجال العناوين
- **Netzwerkbrücke**: bridge / جسر الشبكة
- **Standardroute**: default route / المسار الافتراضي
- **Namensauflösung**: DNS resolution / حل الأسماء
- **Paketmitschnitt**: packet capture / التقاط الحزم
- **Fehlersuche**: troubleshooting / تشخيص الأعطال
