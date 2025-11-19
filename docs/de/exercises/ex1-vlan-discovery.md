# Übung 1: VLAN-Erkennung und Verkehrsanalyse

**Zeitschätzung**: 20-30 Minuten  
**Lab-Topologie**: `ot-sec-flat` oder `ot-sec-segmented`  
**Schwierigkeit**: :material-signal: Anfänger

## Lernziele

- 802.1Q-VLAN-Tagging auf Trunk-Ports verstehen
- Unterschied zwischen Trunk- und Access-Port-Verhalten erkennen
- VLAN-Isolation in Paket-Captures identifizieren
- Sicherheitsimplikationen von VLAN-Segmentierung verstehen

## Voraussetzungen

- Bereitgestelltes und laufendes Lab
- Grundlegende Wireshark-Kenntnisse
- Verständnis des ping-Befehls

## Szenario

Industrielle Netzwerke verwenden VLANs zur logischen Segmentierung verschiedener Betriebszonen. In diesem Lab trennen VLANs Hygienesysteme (VLAN 10), Prozesssteuerung (VLAN 20), Entsorgungssysteme (VLAN 30), Leitwarte (VLAN 40) und Engineering-Workstations (VLAN 50). Das Verständnis, wie VLANs funktionieren und wie man sie im Datenverkehr identifiziert, ist für die OT-Sicherheitsüberwachung unerlässlich.

## Aufgaben

### Aufgabe 1: PLC-Konnektivität innerhalb desselben VLANs überprüfen

1. SSH-Verbindung zu `wago-plc2a-vlan10`:

```bash
ssh admin@192.168.100.53
```

!!! info "Standard-Anmeldedaten"
    Passwort: `admin`

2. Konnektivität zu einem anderen Gerät in VLAN 10 testen:

```bash
ping -c 4 10.10.0.12
```

3. Konnektivität zu Gerät in anderem VLAN testen:

```bash
ping -c 4 10.20.0.11
```

**Frage**: Sind beide Pings erfolgreich? Warum oder warum nicht?

### Aufgabe 2: Datenverkehr auf Trunk-Port zwischen Switches erfassen

1. Klicken Sie in der Containerlab-Erweiterung TopoViewer mit der rechten Maustaste auf die Verbindung zwischen `sw-dist` und `sw-acc1`
2. Wählen Sie "Capture" von der `sw-dist`-Seite
3. Wireshark öffnet sich über Edgeshark

4. Wireshark-Filter anwenden:

```wireshark
vlan
```

5. Datenverkehr generieren durch Ping von einer SPS (verwenden Sie bestehende SSH-Sitzung):

```bash
ping -c 10 10.10.0.1
```

**Fragen**:

- Welche VLANs sehen Sie in den Captures?
- Schauen Sie sich die Ethernet-Frame-Details an - wo befindet sich das VLAN-Tag?
- Was ist der Wert des VLAN-ID-Feldes?

### Aufgabe 3: Datenverkehr auf Access-Port erfassen

1. Stoppen Sie die vorherige Erfassung
2. Klicken Sie mit der rechten Maustaste auf die Verbindung zwischen `sw-acc1` und `wago-plc2a-vlan10`
3. Wählen Sie "Capture" von der Switch-Seite
4. Generieren Sie erneut Datenverkehr:

```bash
ping -c 10 10.10.0.1
```

5. Wenden Sie denselben Filter an:

```wireshark
vlan
```

**Fragen**:

- Sehen Sie jetzt VLAN-Tags?
- Was ist der Unterschied zwischen Trunk- und Access-Ports bezüglich VLAN-Tags?
- Warum entfernt der Switch VLAN-Tags auf Access-Ports?

## Überprüfung

Erfolgsindikatoren:

- Pings innerhalb desselben VLANs sind erfolgreich
- Pings über VLANs hinweg sind erfolgreich (flat) oder schlagen fehl (segmentiert mit Firewall)
- Trunk-Port-Captures zeigen 802.1Q-Header
- Access-Port-Captures zeigen ungetaggte Frames
- Mehrere VLAN-IDs auf Trunk sichtbar (10, 20, 30, 40, 50)

## Zu überlegende Fragen

1. Wenn ein Angreifer Zugang zu einem Trunk-Port erhält, welche zusätzliche Angriffsfläche entsteht dadurch?
2. Wie würden Sie unbefugten VLAN-Zugriff erkennen?
3. Was ist VLAN-Hopping und wie könnte es verhindert werden?

## Häufige Probleme

- **Problem**: VLAN-Tags in Capture nicht sichtbar
  - **Lösung**: Stellen Sie sicher, dass Sie auf dem Trunk-Port zwischen Switches erfassen, nicht auf dem Access-Port zum Gerät

- **Problem**: Wireshark öffnet sich nicht
  - **Lösung**: Beim ersten Mal muss möglicherweise die Edgeshark-Eingabeaufforderung akzeptiert werden. Schliessen und erneut versuchen.

- **Problem**: Pings schlagen innerhalb desselben VLANs fehl
  - **Lösung**: Überprüfen Sie, ob das Lab vollständig bereitgestellt ist. Führen Sie `scripts/lab-test.py ot-sec-flat` zur Überprüfung aus.

---

## Lösung

??? success "Zum Anzeigen der Lösung klicken"
    ### Aufgabe 1: Konnektivitätstest

    SSH zu wago-plc2a-vlan10:

    ```bash
    ssh admin@192.168.100.53
    ```

    Ping innerhalb von VLAN 10 - ERFOLGREICH:

    ```bash
    ping -c 4 10.10.0.12
    ```

    Erwartete Ausgabe:

    ```text
    PING 10.10.0.12 (10.10.0.12) 56(84) bytes of data.
    64 bytes from 10.10.0.12: icmp_seq=1 ttl=64 time=0.521 ms
    64 bytes from 10.10.0.12: icmp_seq=2 ttl=64 time=0.389 ms
    64 bytes from 10.10.0.12: icmp_seq=3 ttl=64 time=0.412 ms
    64 bytes from 10.10.0.12: icmp_seq=4 ttl=64 time=0.398 ms

    --- 10.10.0.12 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss
    ```

    Ping zu VLAN 20 - ERFOLG (flat) oder FEHLSCHLAG (segmentiert mit Firewall):

    ```bash
    ping -c 4 10.20.0.11
    ```

    In `ot-sec-flat`: Erfolgreich, da keine Firewall den Inter-VLAN-Verkehr blockiert.
    In `ot-sec-segmented`: Kann fehlschlagen, wenn Firewall-Regeln diese Kommunikation blockieren.

    ### Aufgabe 2: Trunk-Port-Capture

    In Wireshark auf Trunk-Link zwischen Switches, mit Filter `vlan`:

    **Beobachtet**: 802.1Q-Header sichtbar mit VLAN-ID 10 für Datenverkehr von VLAN 10-Geräten.

    Frame-Struktur:

    ```text
    Ethernet II
        Destination: [MAC]
        Source: [MAC]
        Type: 802.1Q (0x8100)
    802.1Q Virtual LAN
        Priority: 0
        CFI: 0
        ID: 10
        Type: IPv4 (0x0800)
    ```

    ### Aufgabe 3: Access-Port-Capture

    Auf Access-Port zu `wago-plc2a-vlan10` zeigt Filter `vlan` KEINE Ergebnisse.

    Frames sind ungetaggt. Switch fügt VLAN-Tag beim Weiterleiten an Trunk hinzu, entfernt es beim Weiterleiten an Access-Port.

    **Antworten auf die Fragen**:

    - Trunk-Ports zeigen mehrere VLANs: 10, 20, 30, 40, 50
    - VLAN-Tag wird nach Quell-MAC, vor EtherType eingefügt
    - Access-Ports haben keine VLAN-Tags - Switch handhabt das Tagging
    - Dies ermöglicht es Endgeräten, VLAN-unabhängig zu sein
