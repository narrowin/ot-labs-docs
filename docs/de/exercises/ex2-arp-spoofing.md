# Übung 2: ARP-Spoofing-Angriff

**Zeitschätzung**: 30-40 Minuten  
**Lab-Topologie**: `ot-sec-flat` (empfohlen) oder `ot-sec-segmented`  
**Schwierigkeit**: :material-signal-cellular-2: Fortgeschritten

## Lernziele

- Einen Layer-2-Man-in-the-Middle-Angriff ausführen
- ARP-Protokoll-Schwachstellen verstehen
- Klartext-Datenverkehr während eines MITM-Angriffs erfassen
- Erkennungs- und Abwehrstrategien lernen

## Voraussetzungen

- Verständnis des ARP-Protokolls
- Grundkenntnisse über Netzwerkangriffe
- Wireshark-Erfahrung
- Root/sudo-Konzepte

## Szenario

ARP (Address Resolution Protocol) ordnet IP-Adressen MAC-Adressen in lokalen Netzwerken zu. Da ARP keine Authentifizierung hat, kann ein Angreifer falsche ARP-Antworten senden, um den ARP-Cache von Zielgeräten zu vergiften. Dies leitet den Datenverkehr über das System des Angreifers um und ermöglicht Man-in-the-Middle-Angriffe. In OT-Umgebungen kann dies SPS-Kommunikation, Engineering-Workstation-Datenverkehr oder HMI-Verbindungen abfangen.

## Aufgaben

### Aufgabe 1: Normales ARP-Verhalten beobachten

1. SSH-Verbindung zur Opfer-SPS `wago-plc2b-vlan10`:

```bash
ssh admin@192.168.100.54
```

!!! info "Standard-Anmeldedaten"
    Passwort: `admin`

2. Aktuelle ARP-Tabelle anzeigen:

```bash
ip neighbor
```

3. Notieren Sie die MAC-Adresse für Gateway `10.10.0.1`

4. Kontinuierlichen Ping zum externen Netzwerk starten:

```bash
ping 1.1.1.1
```

Lassen Sie dies laufen (drücken Sie nicht Ctrl+C).

### Aufgabe 2: Verkehrserfassung einrichten

1. Öffnen Sie ein neues Terminal bzw. eine neue SSH-Sitzung
2. Stellen Sie eine SSH-Verbindung zur Angreifer-SPS `wago-plc2a-vlan10` her:

```bash
ssh admin@192.168.100.53
```

3. Wireshark-Erfassung auf der Verbindung zwischen `wago-plc2a-vlan10` und `sw-acc1` starten
4. Filter anwenden:

```wireshark
icmp
```

**Beobachtung**: Derzeit sollten Sie noch KEINEN Datenverkehr sehen, da der Ping nicht über den Angreifer läuft.

### Aufgabe 3: ARP-Spoofing-Angriff ausführen

1. Auf dem Angreifer (`wago-plc2a-vlan10`) das ARP-Spoofing-Skript ausführen:

```bash
sudo /home/admin/arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
```

Parameter:

- `eth1` - mit Netzwerk verbundenes Interface
- `10.10.0.12` - Opfer-IP (wago-plc2b-vlan10)
- `10.10.0.1` - Gateway-IP

**Erwartete Ausgabe**:

```text
[*] Enabling IP forwarding...
[*] Poisoning 10.10.0.12 (telling it we are 10.10.0.1)...
[*] Poisoning 10.10.0.1 (telling it we are 10.10.0.12)...
```

2. Überprüfen Sie Wireshark - Sie sollten jetzt ICMP-Echo-Anfragen und -Antworten sehen, die durch den Angreifer fliessen

3. Beim Opfer ARP-Tabelle erneut überprüfen:

```bash
ip neighbor
```

**Frage**: Hat sich die MAC-Adresse für `10.10.0.1` geändert? Vergleichen Sie mit dem Original.

### Aufgabe 4: Klartext-Anmeldedaten erfassen

1. ARP-Spoofing-Skript stoppen (Ctrl+C auf Angreifer)
2. Neues Terminal öffnen und SSH-Verbindung zum Internet-Knoten `panasonic-toughbook-internet` herstellen:

    ```bash
    ssh admin@192.168.100.51
    ```

3. Auf dem Internet-Knoten netcat-Listener auf Port 2323 starten:

    ```bash
    nc -l -k 2323
    ```

    Lassen Sie dies laufen, um eingehende Verbindungen zu empfangen.

4. Beim Opfer (`wago-plc2b-vlan10`) Test-Anmeldedaten senden:

    ```bash
    echo "username:admin password:secretpass123" | nc 1.1.1.1 2323
    ```

5. Auf dem Angreifer ARP-Spoofing neu starten:

    ```bash
    sudo /home/admin/arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
    ```

6. Vom Opfer Anmeldedaten erneut senden:

    ```bash
    echo "username:admin password:secretpass123" | nc 1.1.1.1 2323
    ```

7. In Wireshark ICMP-Filter entfernen und nach TCP-Datenverkehr auf Port 2323 suchen
8. TCP-Stream folgen, um Klartext-Daten zu sehen
9. Netcat-Listener auf Internet-Knoten prüfen - er sollte die empfangenen Anmeldedaten anzeigen

## Überprüfung

Erfolgsindikatoren:

- Angreifer sieht Ping-Datenverkehr des Opfers in Wireshark
- ARP-Tabelle des Opfers zeigt MAC des Angreifers für Gateway
- Skript-Ausgabe zeigt Poisoning-Meldungen
- TCP-Stream enthüllt Klartext-Anmeldedaten

## Zu überlegende Fragen

1. Wie können Sie ARP-Spoofing in einem Produktionsnetzwerk erkennen?
2. Welche Abwehrstrategien existieren (DAI, Port-Security, statisches ARP)?
3. Warum ist dieser Angriff in OT-Umgebungen besonders gefährlich?
4. Welche Protokolle sind anfällig für MITM-Anmeldedaten-Diebstahl?

## Häufige Probleme

- **Problem**: Skript schlägt fehl mit "must be run as root"
  - **Lösung**: Verwenden Sie `sudo` vor dem Skript-Befehl

- **Problem**: Kein Datenverkehr in Wireshark während des Angriffs sichtbar
  - **Lösung**: Überprüfen Sie, ob IP-Forwarding aktiviert ist (Skript macht dies automatisch)

## Erweiterte Herausforderung

Verwenden Sie das `dsniff`-Tool zur automatischen Erfassung von Anmeldedaten:

```bash
sudo dsniff -i eth1
```

Während der Ausführung HTTP- oder FTP-Datenverkehr generieren und automatische Anmeldedaten-Extraktion beobachten.

---

## Lösung

??? success "Zum Anzeigen der Lösung klicken"
    ### Zustand vor dem Angriff

    ARP-Tabelle des Opfers vor dem Angriff:

    ```bash
    ip neighbor
    ```

    Ausgabe zeigt:

    ```text
    10.10.0.1 dev eth1 lladdr 02:42:ac:14:00:0b REACHABLE
    ```

    Notieren Sie diese MAC-Adresse.

    ### Angriffsdurchführung

    Vom Angreifer `wago-plc2a-vlan10`:

    ```bash
    sudo /home/admin/arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
    ```

    Ausgabe:

    ```text
    [*] Enabling IP forwarding...
    [*] Poisoning 10.10.0.12 (telling it we are 10.10.0.1)...
    [*] Poisoning 10.10.0.1 (telling it we are 10.10.0.12)...
    ```

    Skript läuft weiter.

    ### Überprüfung

    Beim Opfer ARP erneut prüfen:

    ```bash
    ip neighbor
    ```

    MAC für `10.10.0.1` zeigt jetzt die MAC des Angreifers anstelle der echten Gateway-MAC.

    In Wireshark auf dem Interface des Angreifers ist nun ICMP-Datenverkehr vom Opfer sichtbar.

    ### Erkennungsmethoden

    1. ARP-Tabellen auf unerwartete Änderungen überwachen
    2. Gratuitous ARP-Pakete erkennen
    3. ARP-Überwachungstools verwenden (arpwatch)
    4. MAC-zu-IP-Zuordnungen mit bekanntem guten Zustand vergleichen

    ### Abwehrstrategien

    1. Dynamic ARP Inspection (DAI) auf Switches
    2. Port-Security zur Begrenzung der MAC-Adressen pro Port
    3. Statische ARP-Einträge für kritische Infrastruktur
    4. Netzwerksegmentierung zur Reduzierung des Angriffsbereichs
    5. 802.1X-Authentifizierung vor Netzwerkzugang
