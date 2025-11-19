# Übung 4: Link-Aggregation und Lastverteilung

**Zeitschätzung**: 20-30 Minuten  
**Lab-Topologie**: Nur `ot-sec-segmented`  
**Schwierigkeit**: :material-signal-cellular-2: Fortgeschritten

!!! warning "Übung noch nicht verfügbar"
    Link-Aggregation ist derzeit in keiner Lab-Topologie konfiguriert. Diese Übung ist für eine zukünftige Lab-Version geplant.

## Lernziele

- LACP-Link-Aggregation-Betrieb analysieren
- Verkehrsverteilung über gebondete Links beobachten
- Link-Failover-Verhalten testen
- Lastverteilungsalgorithmen verstehen

## Voraussetzungen

- Übung 3 abgeschlossen (STP-Analyse)
- Verständnis von LACP-Konzepten
- Wireshark-Erfahrung
- Grundlegende MikroTik-Befehle

## Szenario

Link-Aggregation kombiniert mehrere physische Links zu einem logischen Link für erhöhte Bandbreite und Redundanz. LACP (802.3ad) verhandelt Bonding dynamisch. Der Datenverkehr wird über Mitglied-Links verteilt, wobei ein Hash-Algorithmus basierend auf Quell-/Ziel-MAC, IP oder Ports verwendet wird. Das Verständnis der Funktionsweise ist wichtig für Kapazitätsplanung und Troubleshooting in OT-Netzwerken.

## Aufgaben

### Aufgabe 1: LACP-Verhandlung überprüfen

1. SSH zu `sw-dist`:

```bash
ssh admin@192.168.100.12
```

!!! info "Standard-Anmeldedaten"
    Passwort: `admin`

2. LACP-Protokollbetrieb überwachen:

```routeros
/interface bonding monitor bond-sw-acc1 once
```

**Erwartete Ausgabe enthält**:

- `lacp-partner-system-id` - MAC des Partner-Switches
- `active-ports` - Anzahl aktiver Links
- `lacp-negotiated` - True/False

3. Detaillierte Bonding-Konfiguration überprüfen:

```routeros
/interface bonding print detail where name=bond-sw-acc1
```

**Fragen**:

- Was ist der Bonding-Modus?
- Welche physischen Interfaces sind Slaves?
- Was ist die Transmit-Hash-Policy?

### Aufgabe 2: Verkehrsverteilung beobachten

1. Wireshark-Captures auf BEIDEN Bond-Mitglied-Links starten:
   - Capture-Link `sw-dist` ether9 zu `sw-acc1`
   - Zweites Wireshark für ether10 öffnen

2. SSH zu `wago-plc2a-vlan10`:

```bash
ssh admin@192.168.100.53
```

3. Kontinuierlichen Verkehr mit variierenden Quellen generieren:

```bash
ping -f 1.1.1.1
```

Verwenden Sie `-f` (flood) für hohe Paketrate, um Verteilung zu sehen.

4. Beide Wireshark-Captures beobachten

**Fragen**:

- Ist der Verkehr gleichmässig über beide Links verteilt?
- Sehen Sie Verkehr auf beiden Links oder nur auf einem?
- Was bestimmt, welcher Link für bestimmte Flows verwendet wird?

### Aufgabe 3: Hash-Algorithmus testen

1. Flood-Ping stoppen
2. Verkehr von mehreren Quellen generieren:

Terminal 1 - `wago-plc2a-vlan10`:

```bash
ping -i 0.2 1.1.1.1
```

Terminal 2 - `wago-plc2b-vlan10`:

```bash
ssh admin@192.168.100.54
ping -i 0.2 1.1.1.1
```

3. Paketzähler auf Bond-Mitgliedern überprüfen:

```routeros
/interface monitor-traffic ether9,ether10
```

**Frage**: Werden verschiedene Quell-IPs auf verschiedene Links verteilt?

### Aufgabe 4: Link-Failover testen

1. Pings aus vorherigem Schritt weiter laufen lassen
2. Auf `sw-dist` ein Bond-Mitglied deaktivieren:

```routeros
/interface ethernet disable ether9
```

3. Beobachten:
   - Setzen sich Pings ohne Unterbrechung fort?
   - Wie schnell erfolgt das Failover?
   - Wireshark auf verbleibendem aktivem Link überprüfen

4. Interface wieder aktivieren:

```routeros
/interface ethernet enable ether9
```

5. Bond-Status überprüfen:

```routeros
/interface bonding monitor bond-sw-acc1 once
```

**Frage**: Kehren beide Links automatisch in den aktiven Zustand zurück?

## Überprüfung

Erfolgsindikatoren:

- LACP zeigt verhandelten Status
- Beide Bond-Mitglieder aktiv
- Verkehr auf Mitglied-Links sichtbar
- Failover erfolgt in unter 1 Sekunde
- Deaktivierter Link tritt automatisch wieder bei, wenn aktiviert
- Paketzahl steigt auf beiden Interfaces

## Zu überlegende Fragen

1. Warum könnte Verkehr nur einen Link für einen bestimmten Flow verwenden?
2. Was sind Vor- und Nachteile verschiedener Hash-Algorithmen?
3. Wie vergleicht sich LACP mit statischem Bonding?
4. Was passiert, wenn LACP-Hello-Pakete verloren gehen?

## Häufige Probleme

- **Problem**: Verkehr verwendet nur einen Link
  - **Lösung**: Hash-Algorithmus trifft Flow-basierte Entscheidungen. Einzelne Quelle/Ziel verwendet einen Link. Mehrere Quellen verwenden, um Verteilung zu sehen.

- **Problem**: Verkehr auf Interface kann nicht überwacht werden
  - **Lösung**: Verwenden Sie Befehlsformat `/interface monitor-traffic ether9,ether10`

- **Problem**: Failover dauert mehrere Sekunden
  - **Lösung**: Dies ist normal für LACP. RSTP muss ebenfalls neu konvergieren.

---

## Lösung

??? success "Zum Anzeigen der Lösung klicken"
    ### Verkehrsverteilungsbeobachtung

    Mit einzelner Quell-Ping verwendet Verkehr typischerweise konsistent einen Link aufgrund des Hash-Algorithmus, der Quell-/Ziel-IPs verwendet.

    Mit mehreren Quellen (verschiedene Quell-IPs) verteilt sich Verkehr über Links.

    Hash-Policy `layer-2-and-3` verwendet Quell-/Ziel-MAC- und IP-Adressen.

    ### Paketzahl-Überprüfung

    ```routeros
    /interface monitor-traffic ether9,ether10
    ```

    Zeigt rx-packets-per-second und tx-packets-per-second für jedes Interface.

    Mit diversem Verkehr zeigen beide Interfaces steigende Paketzahlen.

    ### Failover-Test

    Ein Mitglied deaktivieren:

    ```
    /interface ethernet disable ether9
    ```

    Pings setzen sich ohne Unterbrechung fort. Aller Verkehr wechselt zu ether10.

    Wieder aktivieren:

    ```
    /interface ethernet enable ether9
    ```

    LACP verhandelt neu (dauert einige Sekunden), dann sind beide Links wieder aktiv.

    Failover erfolgt in Subsekunden für bestehende Flows.
