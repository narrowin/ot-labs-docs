# Workshop-Spickzettel

## Zugangsdaten

```text
Benutzername: admin
Passwort: admin
```

## Labor starten und stoppen

Verwenden Sie die VS Code Containerlab-Erweiterung mit dem Graph TopoViewer. Rechtsklick auf `ot-sec-segmented.clab.yml` (oder `ot-sec-flat.clab.yml`) und `Graph Topoviewer` wählen.

Labor starten: Play-Button in der TopoViewer-Navigation drücken.

## Mit Geräten verbinden

Schneller SSH-Zugriff (Gerätenamen verwenden):

```bash
ssh clab-ot-sec-flat-proxmox-jumphost
ssh clab-ot-sec-flat-wago-plc2a-vlan10
ssh clab-ot-sec-flat-wago-plc2b-vlan10
ssh clab-ot-sec-flat-abb-800xa-vlan40
```

Oder SSH über IP-Adresse:

```bash
ssh admin@192.168.100.52   # jumphost
ssh admin@192.168.100.53   # WAGO SPS 2A
ssh admin@192.168.100.54   # WAGO SPS 2B
ssh admin@192.168.100.57   # ABB 800XA
```

## Web-Zugriff

HMI-Desktop:

<http://192.168.100.60:5800>

ABB-System:

<http://192.168.100.57:8080>

## Befehle auf der SPS

Prüfen, ob die SPS-Runtime läuft:

```bash
sudo systemctl status wago-plclinux-rt
```

Ping-Test:

```bash
ping 10.10.0.1
```

Netzwerkverkehr mitschneiden:

```bash
sudo tcpdump -lnei eth1
```

## Netzwerk vom Jumphost scannen

Geräte finden:

```bash
nmap 10.10.0.0/24
```

Prüfen, ob SPS-Port offen ist:

```bash
nc -zv 10.10.0.11 1217
```
