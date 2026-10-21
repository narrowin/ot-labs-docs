# Anleitung zum studentischen Lab

Jede Person hat ihre eigene Lab-Appliance. Öffne deine im Browser. Sie enthält das laufende Netz `ot-sec-flat`, konfigurierte Terminals, Paketmitschnitt, PLC-Visualisierungen und die sieben Übungen. Du benötigst deine zugewiesene LNE-URL und Anmeldung; eine lokale Werkzeuginstallation ist nicht nötig.

## Erste Schritte

1. Melde dich am zugewiesenen LNE an. Wähle das studentische Netz und den **Live**-Snapshot.
2. Öffne **Übungen / Exercises** im Produktmenü und wähle Übung 1. Wähle **English / Deutsch** für die Anleitung.
3. Öffne Explorer aus der Übung. Links übernehmen Netz und Snapshot. In Terminal- und Mitschnitt-Tabs öffnet der Titel **Aktive Übung** die Anleitung neben deiner Arbeit.
4. Führe die folgenden Kontrollen durch und bearbeite danach die [sieben Übungen](exercises/index.md) der Reihe nach.

Die Appliance ist bereits bereitgestellt. Verwende konfigurierte Terminal-Aktionen, statt ein Gerätepasswort anzunehmen.

## Die ersten fünf Minuten

| Aktion im Reader | Beobachtbarer Kontrollpunkt |
| --- | --- |
| [`wago-plc2a-vlan10` finden](/network/?search=wago-plc2a-vlan10) | Netmap hebt den Endpunkt hervor; verfolge den Weg über sw-acc1 und sw-dist zu `gw-firewall` |
| [Terminal von `wago-plc2a-vlan10` öffnen](/network/terminal?node=wago-plc2a-vlan10) | Status Connected; die Befehle unten zeigen Datenadresse und vier Gateway-Antworten |
| [VLAN 10 untersuchen](/network/?viewMode=vlan&vlanIds=10) | Netmap zeigt die VLAN-Ansicht mit VLAN 10 |
| [`wago-plc2a-vlan10` WebVisu öffnen](/network/webvisu?node=wago-plc2a-vlan10) | Ein separater Tab zeigt die native CODESYS-Visualisierung; konstante Werte sind möglich |
| [Gateway-Uplink mitschneiden](/network/capture?node=sw-dist&interface=ether12) | sw-dist ether12 ist gewählt; Start Capture starten, Ping wiederholen und `icmp` filtern |

Auf **`wago-plc2a-vlan10`** ausführen:

```bash
ip -br addr show eth1
ping -c 4 10.10.0.1
```

Im unveränderten Lab werden `10.10.0.11/8` und vier Antworten erwartet. Wähle im Uplink-Mitschnitt ein Ping-Paket und klappe **802.1Q Virtual LAN** auf: Die VLAN-ID ist 10. Schliesse danach den Mitschnitt-Tab. [Übung 1](exercises/ex1-vlan-discovery.md) vergleicht dies mit dem ungetaggten Access-Port.

**Eigenständige Dokumentation:** Diese relativen App-Links werden im LNE-Übungsreader aufgelöst. Öffne in einer separat gehosteten Kopie dein zugewiesenes LNE manuell, wähle Netz/Live-Snapshot und Explorer → Netmap. Suche den Knotennamen und verwende sein Aktionsmenü → **Open terminal** oder **Capture packets** mit dem genannten Interface. Öffne für WebVisu die Details einer antwortenden PLC und deren WebVisu-Browseraktion. Wähle in Netmap **VLAN**, dann **10**; für [Spanning Tree](/network/?viewMode=spanning-tree&stpInstanceId=0&dataType=All) wähle **Spanning tree**, Instanz **0**, Datentyp **All**. Simulierte Datenadressen sind innerhalb des Labs erreichbar, nicht direkt von deinem Laptop.

## Die beiden Netzebenen verstehen

Die Management-Ebene verbindet die Appliance für SSH und Erfassung mit den Knoten. Linux-Knoten verwenden dafür den unabhängigen Weg über `eth0`. Das **Dateninterface** ist `eth1`; dort untersuchen Mitschnitte und Testverkehr das eigentliche Labornetz. Ein funktionierendes Management-Terminal beweist keine Datenverbindung.

Der Ausgangszustand hat zwei OT-/DMZ-Broadcast-Domänen:

| Segment | VLAN | Adressierung | Gateway |
| --- | --- | --- | --- |
| Flaches OT-Netz | 10 | `10.0.0.0/8` | `10.10.0.1` |
| DMZ | 60 | `2.2.2.0/24` | `2.2.2.1` |

Namen mit `vlan20`, `vlan30`, `vlan40` oder `vlan50` beschreiben vorgesehene Rollen. Diese Endpunkte sind aktuell mit `/8`-Adressen in VLAN 10 und nicht bereits in `/24`-Sicherheitszonen getrennt. Das simulierte externe Paar `1.1.1.1` / `1.1.1.2` befindet sich innerhalb der Appliance.

| Stabiler Knotenname | Datenadresse / Rolle |
| --- | --- |
| `wago-plc2a-vlan10` | `10.10.0.11/8`, PLC2A |
| `wago-plc2b-vlan10` | `10.10.0.12/8`, PLC2B |
| `ctrlx-plc3-vlan20` | `10.20.0.11/8`, Prozess-PLC |
| `schneider-plc4-vlan30` | `10.30.0.11/8`, Entsorgungs-PLC |
| `abb-800xa-vlan40` | `10.40.0.11/8`, Leitstand-Anwendung |
| `pilz-hmi01-vlan40` | `10.40.0.12/8`, HMI-Browser |
| `dell-ews1-vlan50` | `10.50.0.11/8`, Engineering-Workstation |
| `hirschmann-edge1-vlan50` | `10.50.0.12/8`, Edge-Endpunkt |
| `proxmox-jumphost` | `2.2.2.2/24`, DMZ-Testquelle |
| `panasonic-toughbook-internet` | `1.1.1.1/24`, externe Simulation |

Die RouterOS-Geräte heissen `gw-firewall`, `sw-dist`, `sw-acc1` und `sw-acc2`. sw-dist und sw-acc1 verwenden einen LACP-Bond mit zwei Mitgliedern. RSTP verhindert Schleifen im redundanten Switch-Weg. Die Übungen nennen genaue Ports und untersuchen den Betriebszustand vor Schlussfolgerungen.

## Den simulierten Prozess beobachten

Die CODESYS-Anwendungen modellieren Wasseraufbereitungsfunktionen. Beobachte den Prozess in WebVisu und korreliere in [Übung 6](exercises/ex6-protocol-discovery.md) die GUI mit HTTP und Modbus. Ein Herstellername ist eine Rollenbezeichnung in diesem Containerlab und kein Beleg für dessen physische Controller-Firmware. Nicht jeder Endpunkt hat eine PLC-Runtime oder jedes OT-Protokoll aktiviert.

Beobachte die Bedienelemente, ohne sie während der Netzwerkaufgaben zu betätigen. Ein konstanter Wert kann korrekt sein. Erfolgreiche laufende Anwendungskommunikation ist ein besserer Verfügbarkeitsbeleg als eine erzwungene Prozessänderung.

## Das passende Werkzeug verwenden

| Werkzeug | Verwendung |
| --- | --- |
| Netmap | Wege, Nachbarn, Ports, VLAN- und Spanning-Tree-Ansichten |
| Konfiguriertes Terminal | Befehle auf dem genannten Knoten mit vorhandener Zugangsdatenwahl |
| Paketmitschnitt | Am genannten Port vor begrenztem Testverkehr starten; Wireshark-Anzeigefilter nutzen |
| WebVisu | Native Visualisierung einer freigegebenen, antwortenden Runtime |
| Toolbox | Zonen und gerichtete Policies entwerfen; Speichern konfiguriert keine Geräte |
| Assistent | Hilfe mit aktiver Übung, Befehlsausgabe und Paketbelegen |

Toolbox wird im Produktmenü geöffnet. Der gespeicherte Netzwerk-Workspace enthält deinen Entwurf; Speichern setzt keine Regeln durch. [Übung 7](exercises/ex7-segmentation-firewall.md) baut VLAN 70 über Geräteterminals in deinem eigenen Lab auf. Die [praktische Vorbereitung](lab-practical.md) richtet einen Checkpoint für die Wiederherstellung ein. Übungen 1–6 nutzen den flachen Ausgangszustand. Übung 7 verschiebt vorübergehend nur `hirschmann-edge1-vlan50` und stellt ihn vor Abschluss wieder her.

## Wenn ein Kontrollpunkt fehlschlägt

Prüfe zuerst Live-Netz, Knoten und Interface. Historische Snapshots zeigen Topologie, erlauben aber keine Live-Terminals, Mitschnitte oder WebVisu. Ist eine Aktion nicht verfügbar, lasse Bindings, Zugangsdaten, Runtime-Status und Berechtigungen durch die Dozierenden prüfen. Notiere Knoten/Aktion/Fehler, statt sachfremde Konfiguration zu verändern.

Bei Mitschnittproblemen den Anzeigefilter leeren, gestartete Erfassung prüfen, den Vier-Paket-Ping wiederholen und einige Sekunden auf die Anzeige warten. Siehe die [Filter- und Fehlerreferenz](exercises/appendix.md).

Schliesse Mitschnitte und Terminals nach jeder Übung. Übung 2 stellt zusätzlich die Gateway-Zuordnung des Opfers wieder her und prüft das Ende des Helpers. Fahre mit [Übung 1](exercises/ex1-vlan-discovery.md) fort.
