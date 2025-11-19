# Übung 5: Netzwerk-Reconnaissance und Sicherheitsbewertung

**Zeitschätzung**: 35-45 Minuten  
**Lab-Topologie**: `ot-sec-flat` (einfacher) oder `ot-sec-segmented` (realistisch)  
**Schwierigkeit**: :material-signal-cellular-2: Fortgeschritten

## Lernziele

- Netzwerk-Reconnaissance vom DMZ-Jumphost durchführen
- Exponierte Dienste mittels Port-Scanning identifizieren
- Potenzielle Sicherheitsschwachstellen entdecken
- Professionelle Sicherheitsbewertungsmethodik praktizieren

## Voraussetzungen

- Verständnis von TCP/IP-Grundlagen
- Grundlegende nmap-Kenntnisse
- SSH- und Service-Konzepte
- Bewusstsein für Sicherheitsbewertung

## Szenario

Ein Sicherheitsbewerter hat Remote-Zugriff auf den Jumphost in der DMZ. Das Ziel ist es, das OT-Netzwerk zu kartieren, laufende Dienste zu identifizieren und potenzielle Sicherheitsprobleme zu dokumentieren. Dies simuliert reale Remote-Zugriffs-Szenarien, in denen Ingenieure oder Angreifer anfänglichen Zugang erhalten und versuchen, den Zugriff auf OT-Zonen zu erweitern.

## Aufgaben

### Aufgabe 1: Erste Reconnaissance vom Jumphost

1. SSH zum Jumphost:

```bash
ssh admin@192.168.100.52
```

!!! info "Standard-Anmeldedaten"
    Passwort: `admin`

2. Überprüfen Sie Ihre Netzwerkposition:

```bash
ip addr show
ip route show
```

**Fragen**:

- Welche IP-Adresse hat der Jumphost?
- Welche Netzwerke sind gemäss Routing-Tabelle erreichbar?

3. Grundlegende Konnektivität testen:

```bash
ping -c 3 10.10.0.1
ping -c 3 10.10.0.11
```

**Hinweis**: In segmentierter Topologie kann die Firewall einigen Verkehr blockieren.

### Aufgabe 2: Host-Erkennung

1. VLAN 10 (Hygiene) nach aktiven Hosts scannen:

```bash
nmap -sn 10.10.0.0/24
```

`-sn` führt Ping-Scan durch (kein Port-Scan)

**Erwartete Ausgabe**:

```text
Nmap scan report for 10.10.0.1
Host is up.

Nmap scan report for 10.10.0.11
Host is up.

Nmap scan report for 10.10.0.12
Host is up.
```

2. Andere VLANs versuchen:

```bash
nmap -sn 10.20.0.0/24
nmap -sn 10.40.0.0/24
```

**Frage**: Welche VLANs sind vom Jumphost aus erreichbar?

### Aufgabe 3: Service-Identifikation

1. Gängige OT-Ports auf entdeckter SPS scannen:

```bash
nmap -sV -p22,80,502,8080,2455 10.10.0.11
```

Port-Referenz:

- 22: SSH
- 80: HTTP
- 502: Modbus TCP
- 8080: CODESYS WebVisu
- 2455: CODESYS Runtime

**Erwartete Ausgabe**:

```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.x
8080/tcp open  http    nginx
```

2. Alle SPSen in VLAN 10 scannen:

```bash
nmap -p8080 10.10.0.11-12
```

**Fragen**:

- Welche Dienste sind exponiert?
- Laufen Dienste, die nicht laufen sollten?
- Was können Sie aus den Service-Bannern über die Geräte ableiten?

### Aufgabe 4: Zugriffstest

1. SSH-Zugriff auf SPS testen:

```bash
ssh admin@10.10.0.11
```

Passwort versuchen: `admin`

**Frage**: Ist dies erfolgreich? Sollten SPSen direkt per SSH vom Jumphost erreichbar sein?

2. Web-Interface-Zugriff testen:

```bash
curl -I http://10.10.0.11:8080
```

**Beobachtung**: HTTP-Response-Header auf Versionsinformationen prüfen.

3. Falls `nc` verfügbar, Modbus-Port testen:

```bash
nc -zv 10.10.0.11 502
```

### Aufgabe 5: Ergebnisse dokumentieren

Einfachen Bericht der Ergebnisse erstellen:

```bash
cat > assessment_findings.txt << 'EOF'
OT-Netzwerk-Sicherheitsbewertung

Datum: $(date)
Bewerter: $(whoami)

ERGEBNISSE:

1. Erreichbare Netzwerke:
   - VLAN 10: [Liste entdeckter Hosts]
   - VLAN 20: [erreichbar? ja/nein]

2. Exponierte Dienste:
   - Host 10.10.0.11:
     * Port 22 (SSH): OFFEN
     * Port 8080 (HTTP): OFFEN
     * Port 502 (Modbus): [Status]

3. Sicherheitsprobleme:
   - [Bedenken auflisten]

4. Empfehlungen:
   - [Vorgeschlagene Abhilfemassnahmen]
EOF

cat assessment_findings.txt
```

## Überprüfung

Erfolgsindikatoren:

- Host-Erkennung identifiziert aktive SPSen
- Service-Scans zeigen offene Ports
- SSH-Zugriff getestet (kann je nach Firewall erfolgreich sein oder fehlschlagen)
- HTTP-Interfaces entdeckt
- Ergebnisse dokumentiert

## Zu überlegende Fragen

1. Welche Dienste sollten niemals direkt von der DMZ aus erreichbar sein?
2. Wie würden Sie den Jumphost-Zugriff in der Produktion einschränken?
3. Welche zusätzlichen Sicherheitskontrollen würden diese Architektur verbessern?
4. Wie könnten diese Reconnaissance-Aktivitäten erkannt werden?

## Häufige Probleme

- **Problem**: nmap-Befehl nicht gefunden
  - **Lösung**: Verwenden Sie `apt-get update && apt-get install nmap` oder Tool ist vorinstalliert

- **Problem**: Alle Hosts erscheinen als down
  - **Lösung**: In segmentierter Topologie kann Firewall ICMP blockieren. Verwenden Sie `-Pn`-Flag, um Ping zu überspringen.

- **Problem**: Keine 10.x-Netzwerke erreichbar
  - **Lösung**: Routing mit `ip route` prüfen. Überprüfen, ob Lab korrekt bereitgestellt wurde.

## Erweiterte Herausforderung

1. Shell-Skript schreiben zur Automatisierung der Erkennung aller VLANs
2. CSV-Bericht aller entdeckten Dienste generieren
3. Auf Standard-Anmeldedaten auf entdeckten Diensten testen
4. Netzwerkdiagramm basierend auf entdeckter Topologie erstellen

---

## Lösung

??? success "Zum Anzeigen der Lösung klicken"
    ### Host-Erkennungsergebnisse

    Vom Jumphost:

    ```bash
    nmap -sn 10.10.0.0/24
    ```

    Erwartete Entdeckungen in flacher Topologie:

    ```text
    10.10.0.1 (Gateway)
    10.10.0.11 (wago-plc2a-vlan10)
    10.10.0.12 (wago-plc2b-vlan10)
    ```

    Segmentierte Topologie kann weniger Hosts aufgrund von Firewall-Regeln zeigen.

    ### Service-Scan-Ergebnisse

    ```bash
    nmap -sV -p22,80,502,8080,2455 10.10.0.11
    ```

    Erwartete Ausgabe:

    ```text
    PORT     STATE SERVICE  VERSION
    22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu
    8080/tcp open  http     nginx 1.18.0
    ```

    Andere Ports können je nach SPS-Konfiguration gefiltert oder geschlossen sein.

    ### Zugriffstest

    SSH zur SPS:

    ```bash
    ssh admin@10.10.0.11
    ```

    In flacher Topologie: Wahrscheinlich erfolgreich mit Passwort `admin`.
    In segmentierter Topologie: Kann durch Firewall-Regeln blockiert werden.

    Web-Interface:

    ```bash
    curl -I http://10.10.0.11:8080
    ```

    Gibt HTTP-Header zurück, die CODESYS-WebVisu-Interface zeigen.

    ### Sicherheitsergebnisse

    Typische entdeckte Probleme:

    1. SPSen per SSH mit Standard-Anmeldedaten erreichbar
    2. Web-Interfaces ohne Authentifizierung exponiert
    3. Modbus TCP ohne Authentifizierung erreichbar
    4. Keine Netzwerksegmentierung (flache Topologie)
    5. Standard-Passwörter im Einsatz
    6. Unnötige Dienste laufen

    ### Empfohlene Abhilfemassnahmen

    1. Zero-Trust-Netzwerksegmentierung implementieren
    2. Multi-Faktor-Authentifizierung für Remote-Zugriff verlangen
    3. Ungenutzte Dienste auf SPSen deaktivieren
    4. Standard-Anmeldedaten ändern
    5. Anwendungsbewusste Firewall-Regeln implementieren
    6. IDS/IPS für Anomalieerkennung einsetzen
    7. VPN für Remote-Zugriff verlangen
    8. Jumphost-Härtung implementieren
