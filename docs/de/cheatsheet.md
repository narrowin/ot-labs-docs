# Workshop-Kurzreferenz

Verwende dein zugewiesenes, bereits laufendes studentisches LNE und wähle das Live-Netz. Die Werkzeuglinks funktionieren im Übungsreader. Nutze in eigenständiger Dokumentation die [manuelle Navigation](lab.md#die-ersten-funf-minuten). Die [Übungsreferenz](exercises/appendix.md) enthält Paketfilter und Fehlersuche.

## Werkzeug öffnen

| Aktion | Ziel |
| --- | --- |
| [WAGO-A-Terminal](/network/terminal?node=wago-plc2a-vlan10) | Hauptbeobachtungsquelle, `10.10.0.11/8` |
| [WAGO-B-Terminal](/network/terminal?node=wago-plc2b-vlan10) | Zweite Quelle, `10.10.0.12/8` |
| [DMZ-Jumphost-Terminal](/network/terminal?node=proxmox-jumphost) | Geroutete Quelle, `2.2.2.2/24` |
| [Gateway-Uplink-Mitschnitt](/network/capture?node=sw-dist&interface=ether12) | Getaggter VLAN-10-Verkehr |
| [WAGO-Access-Mitschnitt](/network/capture?node=sw-acc1&interface=ether2) | Ungetaggter Endpunktverkehr |
| [VLAN-Ansicht](/network/?viewMode=vlan&vlanIds=10) | VLAN 10 in gewähltem Netz/Snapshot |
| [Spanning-Tree-Ansicht](/network/?viewMode=spanning-tree&stpInstanceId=0&dataType=All) | Instanz 0, alle Daten |

Warte im Terminal auf **Connected**. Starte den Mitschnitt vor dem Testverkehr und stoppe ihn vor einem Zielwechsel. Verwende `exit` und schliesse danach Terminal-/Mitschnitt-Tabs. Konfigurierte Aktionen verwenden vorhandene Zugangsdaten.

## SPS-Weboberflächen

| Runtime | Reader-Link |
| --- | --- |
| WAGO A | [WebVisu](/network/webvisu?node=wago-plc2a-vlan10) |
| WAGO B | Aktuell nicht in den freigegebenen Browseraktionen; für die Visualisierungsaufgabe WAGO A verwenden |
| CtrlX | [WebVisu](/network/webvisu?node=ctrlx-plc3-vlan20) |
| Schneider | [WebVisu](/network/webvisu?node=schneider-plc4-vlan30) |
| ABB | [WebVisu](/network/webvisu?node=abb-800xa-vlan40) |

Angeboten werden nur freigegebene, antwortende Runtimes im Live-Netz. Öffne in eigenständiger Dokumentation die WebVisu-Aktion der PLC im LNE. `http://<Daten-IP>:8080/webvisu.htm` ist eine Adresse innerhalb des Labs und keine Portweiterleitungsanleitung für den Laptop.

## Begrenzte Linux-Prüfungen

Auf dem in der Übung genannten Knoten ausführen:

```bash
ip -br addr show eth1
ip route get 10.10.0.11
ip neigh show dev eth1
ping -c 4 10.10.0.1
```

Auf dem **DMZ-Jumphost** lautet die begrenzte Dienstprüfung:

```bash
nmap -n -sT -Pn -p22,502,4840,8080 --max-retries 1 --host-timeout 20s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

Diese Beobachtungen ändern kein PLC-Programm. Verwende den genauen lesenden Modbus-Befehl aus [Übung 6](exercises/ex6-protocol-discovery.md). Die begrenzte ARP-Aufgabe und ihre zwingende Bereinigung stehen in [Übung 2](exercises/ex2-arp-spoofing.md).

## RouterOS-Beobachtungen

```routeros
/interface bridge monitor bridge once
/interface bridge port print
/interface bridge vlan print
/ip firewall filter print detail where chain=forward
```

Der Ausgangszustand ist OT-VLAN 10 mit `/8`-Adressen und DMZ-VLAN 60. Ein Hostname, offener Port oder abschliessender Drop beweist allein keine Segmentierung. Arbeite in deinem eigenen Lab. [Übung 7](exercises/ex7-segmentation-firewall.md) erstellt dort VLAN 70 und enthält die Bereinigung zum flachen Ausgangszustand. Fahre in [Übungsreihenfolge](exercises/index.md) fort.
