# Übung 3: Spanning-Tree-Protokoll-Analyse

**Zeitschätzung**: 25-35 Minuten  
**Lab-Topologie**: `ot-sec-flat` oder `ot-sec-segmented`  
**Schwierigkeit**: :material-signal-cellular-2: Fortgeschritten

## Lernziele

- Spanning-Tree-Protokoll-Betrieb verstehen
- Root-Bridge und Port-Status identifizieren
- STP-BPDU-Austausch beobachten
- (Nur segmentiert) LACP-Bonding mit STP analysieren

## Voraussetzungen

- Verständnis von Layer-2-Switching
- Kenntnisse über STP/RSTP-Konzepte
- Grundlegende MikroTik-RouterOS-Befehle
- SSH-Zugang zu Switches

## Szenario

Das Spanning Tree Protocol (STP) verhindert Schleifen in geswitchten Netzwerken durch Blockieren redundanter Pfade. Industrielle Netzwerke verwenden RSTP (Rapid Spanning Tree Protocol) für schnellere Konvergenz. Beide Lab-Topologien führen STP auf allen Switches aus. Das segmentierte Lab verwendet zusätzlich LACP (Link Aggregation Control Protocol) Bonding zwischen Switches, das zur Bereitstellung von Redundanz und erhöhter Bandbreite mit STP zusammenarbeitet.

## Aufgaben

### Aufgabe 1: Switch-Topologie identifizieren

1. SSH-Verbindung zum Distribution-Switch:

```bash
ssh admin@192.168.100.12
```

!!! info "Standard-Anmeldedaten"
    Passwort: `admin`

2. Bridge-Konfiguration anzeigen:

```routeros
/interface bridge print
```

3. Alle Bridge-Ports auflisten:

```routeros
/interface bridge port print
```

**Fragen**:

- Wie lautet der Bridge-Name?
- Welche Interfaces sind Bridge-Mitglieder?
- Welche Interfaces haben VLAN-Filterung aktiviert?

### Aufgabe 2: Link-Topologie untersuchen

1. Überprüfen, welche Interfaces mit anderen Switches verbunden sind:

```routeros
/interface bridge port print
```

**Erwartete Ausgabe**:

- **Flache Topologie**: ether10, ether12, ether8, ether2 als Bridge-Mitglieder
- **Segmentierte Topologie**: ether12, ether10, ether8, ether2 als Bridge-Mitglieder

**Fragen**:

- Wie viele Links verbinden sw-dist mit sw-acc1?
- (Segmentiert) Sind beide Bond-Mitglieder aktiv?

### Aufgabe 3: BPDU-Frames erfassen

1. In TopoViewer Wireshark-Erfassung auf der Verbindung zwischen `sw-dist` und `sw-acc1` starten
2. Filter anwenden:

```wireshark
stp
```

Oder alternativ:

```wireshark
eth.dst == 01:80:c2:00:00:00
```

**Beobachtungen**:

- BPDUs werden an Multicast-Adresse `01:80:c2:00:00:00` gesendet
- Root-Bridge-Identifikator
- Port-Rollen (designated, root)
- Bridge-Prioritätswerte

3. Bridge-Einstellungen auf beiden Switches überprüfen:

Auf `sw-dist`:

```routeros
/interface bridge print detail
```

Auf `sw-acc1` (SSH zu 192.168.100.13):

```routeros
/interface bridge print detail
```

**Frage**: Welcher Switch hat niedrigere Priorität und wird Root-Bridge?

### Aufgabe 4: Redundanzansatz verstehen

**In flacher Topologie**:

- Einzelne Verbindung zwischen Switches
- STP verhindert Schleifen zwischen anderen Switch-Verbindungen
- Keine Link-Level-Redundanz zwischen sw-dist und sw-acc1

**In segmentierter Topologie**:

- LACP-Bond bietet Link-Redundanz
- STP läuft weiterhin auf anderen Ports
- Zwei komplementäre Redundanzmechanismen

Vom Distribution-Switch Konfiguration überprüfen:

```routeros
/interface ethernet print
/interface bridge port print
```

**Frage**: Was passiert, wenn der einzelne Uplink in flacher Topologie ausfällt?

## Überprüfung

Erfolgsindikatoren:

- Bridge-Konfiguration zeigt aktivierte VLAN-Filterung
- BPDU-Frames in Wireshark-Captures sichtbar
- Root-Bridge aus Bridge-Einstellungen identifiziert
- (Segmentiert) Bond-Interface existiert mit aktivem LACP
- (Flat) Einzelne Link-Topologie dokumentiert
- Verständnis, wie STP Schleifen verhindert

## Zu überlegende Fragen

1. Warum läuft STP auch dann, wenn kein redundanter Pfad vorhanden ist (flat)?
2. (Segmentiert) Was passiert, wenn ein Bond-Mitglied ausfällt?
3. Wie schnell konvergiert STP nach einer Topologieänderung?
4. Welche Sicherheitsrisiken birgt STP (BPDU-Angriffe)?

## Häufige Probleme

- **Problem**: Kein Bonding-Interface in flat sichtbar
  - **Lösung**: Korrekt - flache Topologie hat kein Bonding, nur einzelne Links

- **Problem**: SSH zu Switches nicht möglich
  - **Lösung**: Verwenden Sie Management-IPs 192.168.100.11-14, nicht Datenebenen-IPs

- **Problem**: Keine BPDUs im Capture
  - **Lösung**: BPDUs werden periodisch gesendet (alle 2 Sekunden). Warten oder Topologieänderung erzeugen.

- **Problem**: Übung sagt "nur segmentiert", aber ich bin in flat
  - **Lösung**: Übung 3 funktioniert in beiden Topologien - Bonding-Aufgaben sind innerhalb der Übung als nur-segmentiert markiert

---

## Lösung

??? success "Zum Anzeigen der Lösung klicken"


    ### Bridge-Konfiguration

    [OT-LAB RSTP Topologie](https://demo.narrowin.ch/network/?network=34&snapshot=1&viewMode=spanning-tree&dataType=All&showDetailsId=edge-01J7Y2D1W00BWT5MZHJGJR5RH9-01J7Y2D1W0244K1AK4HRJAAPDX&stpInstanceId=0)

    SSH zu `sw-dist`:

    ```bash
    ssh admin@192.168.100.12
    ```

    Bridge anzeigen:

    ```
    /interface bridge print
    ```

    Ausgabe:

    ```text
    Flags: X - disabled, R - running
     0 R name="bridge" mtu=auto actual-mtu=1500 l2mtu=65535 arp=enabled 
         arp-timeout=auto mac-address=XX:XX:XX:XX:XX:XX auto-mac=no 
         protocol-mode=rstp vlan-filtering=yes
    ```

    Bridge-Ports:

    ```
    /interface bridge port print
    ```

    **Flache Topologie** zeigt ether10, ether12, ether8, ether2 als Bridge-Mitglieder mit VLAN-Einstellungen.
    
    **Segmentierte Topologie** zeigt ether12, ether10, ether8, ether2 als Bridge-Mitglieder mit VLAN-Einstellungen.

    !!! info "Bonding nicht konfiguriert"
        Link-Aggregation (Bonding) ist derzeit in keiner Lab-Topologie konfiguriert. In beiden Topologien verbindet sich jeder Switch über einzelne physische Interfaces.

    ### Root-Bridge-Identifikation

    Priorität auf beiden Switches prüfen. Niedrigere Priorität gewinnt. Standard-Priorität ist 32768.

    BPDU-Captures zeigen Root-Bridge-ID in den Protokollfeldern.
