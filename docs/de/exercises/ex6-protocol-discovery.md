# Übung 6: OT-Protokoll-Erkennung

**Zeitschätzung**: 45-50 Minuten  
**Lab-Topologie**: `ot-sec-flat` (empfohlen) oder `ot-sec-segmented`  
**Schwierigkeit**: :material-signal-cellular-2: Fortgeschritten

## Lernziele

- OT-Protokolle im Live-Verkehr entdecken
- Modbus-TCP-Kommunikation identifizieren
- CODESYS-Runtime-Protokoll analysieren
- OPC-UA-Sicherheitskonfigurationen bewerten
- PROFINET-Eigenschaften erkennen
- Sicherheitsimplikationen von Klartext-Protokollen verstehen

## Voraussetzungen

- Übung 1 (Wireshark-Grundlagen)
- Grundverständnis von TCP/IP

## Szenario

In einer OT-Umgebung ist Sichtbarkeit der Schlüssel. Im Gegensatz zu IT-Netzwerken, wo HTTPS und SSH dominieren, verlassen sich OT-Netzwerke oft auf Klartext-Protokolle wie Modbus TCP und proprietäre Protokolle wie CODESYS. In dieser Übung generieren Sie Verkehr zwischen einem HMI und einer SPS, erfassen ihn und analysieren die Protokolldetails, um zu verstehen, was ein Angreifer sehen könnte.

## Aufgaben

### Aufgabe 1: OT-Verkehr generieren

Um Verkehr zu analysieren, müssen wir ihn zuerst generieren. Wir verwenden das HMI zur Interaktion mit der Web-Visualisierung einer SPS.

1.  Öffnen Sie den **HMI Desktop** in Ihrem Browser:
    *   URL: `http://localhost:5800`
    *   Dies verbindet Sie über VNC mit dem `pilz-hmi01-vlan40`-Desktop.

2.  Öffnen Sie auf dem HMI-Desktop den **Chromium**-Webbrowser.

3.  Navigieren Sie zur WebVisu von `wago-plc2a-vlan10`:
    *   URL: `http://10.10.0.11:8080/webvisu.htm`
    *   (Falls das nicht lädt, versuchen Sie `http://10.10.0.11:8080`)

4.  Interagieren Sie mit der Visualisierung (klicken Sie Buttons, schalten Sie Schalter), um Verkehr zu generieren.

### Aufgabe 2: Verkehr mit Wireshark erfassen

Jetzt erfassen wir den Verkehr, der zur SPS fliesst, mit den integrierten Lab-Tools.

1. Gehen Sie in VS Code zur **Containerlab**-Erweiterung (TopoViewer).

2. Lokalisieren Sie die Verbindung zwischen `wago-plc2a-vlan10` und `sw-acc1`.

3. Klicken Sie mit der **rechten Maustaste** auf die Verbindung (oder den Interface-Punkt auf der SPS-Seite) und wählen Sie **Wireshark Capture**.
    - Dies startet eine Wireshark-Instanz direkt in Ihrem Browser bzw. Editor über Edgeshark.

4. Gehen Sie zurück zum HMI (VNC) und interagieren Sie 10-15 Sekunden lang erneut mit der WebVisu (klicken Sie Buttons, schalten Sie Schalter).

5. Kehren Sie zum Wireshark-Fenster zurück. Sie sollten Pakete in Echtzeit erscheinen sehen.

6. Stoppen Sie die Erfassung (roter quadratischer Button in Wireshark-Toolbar), um die Daten zu analysieren.

### Aufgabe 3: Modbus TCP analysieren

1. In Wireshark für Modbus-TCP-Verkehr filtern:

    ```wireshark
    mbtcp
    ```

    *Hinweis: Falls Sie `mbtcp` nicht sehen, versuchen Sie Filter für Port `tcp.port == 502`.*

2. Wählen Sie ein Paket. Erweitern Sie den **Modbus**-Abschnitt in den Paketdetails.

3. **Fragen**:
    - Welche Function Codes sehen Sie? (z.B. Read Holding Registers, Write Single Coil)
    - Können Sie die Werte sehen, die gelesen oder geschrieben werden?
    - Gibt es Verschlüsselung oder Authentifizierung?

### Aufgabe 4: CODESYS-Protokoll analysieren

Die WebVisu kommuniziert mit der Runtime, oft über proprietäre Protokolle oder HTTP/XML. Die Engineering-Workstation würde jedoch das CODESYS-Protokoll verwenden. Suchen wir danach oder nach anderem Management-Verkehr.

1. Filter löschen und nach Verkehr auf Port **2455** (UDP) oder **1217** (TCP) suchen.

    ```wireshark
    udp.port == 2455 || tcp.port == 1217
    ```

2. Falls Sie diesen Verkehr nicht sehen, liegt es möglicherweise daran, dass wir nur WebVisu verwenden. Um CODESYS-Engineering-Verkehr zu generieren, würden wir die CODESYS IDE benötigen.

    *Alternative*: Schauen Sie nach HTTP-Verkehr für die WebVisu.

    ```wireshark
    http && ip.addr == 10.10.0.11
    ```

3. HTTP-Streams analysieren.
    - Rechtsklicken Sie auf ein Paket -> Follow -> HTTP Stream.
    - Beobachten Sie, dass die Visualisierungsdaten im Klartext übertragen werden (es sei denn, HTTPS ist konfiguriert, was in älteren Setups oft nicht der Standard ist).

### Aufgabe 5: OPC-UA-Sicherheitskonfiguration analysieren

OPC UA ist der moderne Industrie-4.0-Protokollstandard. Im Gegensatz zu Modbus wurde es mit Sicherheitsfunktionen entwickelt, diese werden jedoch oft in Produktionsumgebungen deaktiviert. Ihre Aufgabe als OT-Sicherheitsexperte ist es, zu bewerten, ob die Sicherheit korrekt konfiguriert ist.

#### Hintergrund

OPC UA unterstützt:

- **Verschlüsselung** (SignAndEncrypt)
- **Authentifizierung** (Benutzername/Passwort oder zertifikatbasiert)
- **Autorisierung** (rollenbasierte Zugriffskontrolle)

Die meisten Implementierungen laufen jedoch mit `SecurityPolicy=None` und `Anonymous`-Authentifizierung aus Gründen der Einfachheit oder Legacy-Kompatibilität.

#### OPC-UA-Verkehr generieren

Die Lab-SPSen haben OPC-UA-Server auf Port 4840 aktiviert:

- `ctrlx-plc3-vlan20` unter `10.20.0.11:4840`
- `schneider-plc4-vlan30` unter `10.30.0.11:4840`

Wir müssen OPC-UA-Verkehr generieren, um ihn zu analysieren. Der einfachste Ansatz ist die Nutzung des HMI-Webbrowsers.

**Verkehr vom HMI generieren**

1. Öffnen Sie den HMI-Desktop in Ihrem Browser: `http://localhost:5800`

2. Öffnen Sie im HMI-Chromium-Browser einen neuen Tab

3. Navigieren Sie zu: `http://10.20.0.11:8080/webvisu.htm`
   - Die WebVisu kann im Hintergrund über OPC UA mit der SPS kommunizieren
   - Interagieren Sie mit der Visualisierung, um Verkehr zu generieren

**Alternative: Verbindung vom Jumphost testen**

Falls Sie Terminal-Zugriff auf den Jumphost haben, prüfen Sie, ob der OPC-UA-Port erreichbar ist:

```bash
# Konnektivität testen
nc -zv 10.20.0.11 4840

# Einfache Verbindungsprobe senden
timeout 2 telnet 10.20.0.11 4840
```

Sie sollten eine etablierte Verbindung sehen, die bestätigt, dass der OPC-UA-Server läuft.

#### OPC-UA-Verkehr erfassen

1. Starten Sie die Wireshark-Erfassung auf der Verbindung zwischen `ctrlx-plc3-vlan20` und `sw-acc1`
2. Führen Sie die OPC-UA-Client-Verbindung aus (Option oben)
3. Stoppen Sie die Erfassung nach Abschluss der Verbindung

#### Sicherheitskonfiguration analysieren

1. Filtern Sie nach OPC-UA-Verkehr:

    ```wireshark
    opcua
    ```

    Oder nach Port:

    ```wireshark
    tcp.port == 4840
    ```

2. Finden Sie das **CreateSessionRequest**-Paket:
    - Erweitern: `OpcUa Binary Protocol -> Message -> CreateSessionRequest`
    - Finden: `SecurityPolicyUri`
    - Wert prüfen: `http://opcfoundation.org/UA/SecurityPolicy#None` bedeutet keine Verschlüsselung

3. Finden Sie das **ActivateSessionRequest**-Paket:
    - Erweitern: `OpcUa Binary Protocol -> Message -> ActivateSessionRequest`
    - Prüfen: `UserIdentityToken`
    - Suchen nach: `AnonymousIdentityToken` bedeutet keine Authentifizierung

4. Analysieren Sie eine **BrowseRequest/Response**:
    - Dies zeigt die komplette Tag-Struktur der SPS
    - Alle Tag-Namen sind im Klartext sichtbar
    - Ein Angreifer kann das komplette Prozessdatenmodell abbilden

5. Schauen Sie sich **ReadRequest/Response** an:
    - Variablenwerte werden im Klartext übertragen
    - Kein Integritätsschutz
    - Ein Angreifer kann Echtzeit-Prozesswerte sehen

#### Bewertungsfragen

1. **Welche SecurityPolicy ist konfiguriert?** (Erwartet: None)
2. **Ist die Kommunikation verschlüsselt?** (Erwartet: Nein)
3. **Welche Authentifizierung wird benötigt?** (Erwartet: Anonymous - keine)
4. **Können Sie Tag-Namen in der Browse-Response sehen?** (Erwartet: Ja)
5. **Können Sie Variablenwerte in Read-Responses sehen?** (Erwartet: Ja)
6. **Welche IEC-62443-Anforderungen werden verletzt?**
    - SR 1.1: Identifikation und Authentifizierung (FAIL - Anonymous)
    - SR 3.1: Kommunikationsintegrität (FAIL - keine Signierung)
    - SR 4.1: Informationsvertraulichkeit (FAIL - keine Verschlüsselung)

#### Praxisbezug

Dies ist einer der häufigsten Befunde in OT-Sicherheitsaudits. OPC UA hat ausgezeichnete Sicherheitsfunktionen, aber Organisationen deaktivieren sie, weil:

- Konfigurationskomplexität (wahrgenommen, nicht tatsächlich)
- Legacy-Kompatibilitätsbedenken (manchmal berechtigt)
- Performance-Mythen (Verschlüsselungs-Overhead ist auf moderner Hardware vernachlässigbar)
- Mangelndes Bewusstsein (häufigster Grund)

**Audit-Empfehlung:** Aktivieren Sie OPC-UA-Sicherheit mit `SecurityPolicy=Basic256Sha256`, `SecurityMode=SignAndEncrypt` und zertifikatbasierter Authentifizierung. Testen Sie mit bestehenden Clients, um Kompatibilität zu prüfen.

### Aufgabe 6: PROFINET-Protokoll-Erkennung

PROFINET ist das dominierende Echtzeit-Industrie-Ethernet-Protokoll in der europäischen Fertigung, insbesondere in der Automobil- und Prozessindustrie. Im Gegensatz zu den vorherigen Protokollen, die auf TCP/IP-Schichten operieren, läuft PROFINET-Echtzeit-Verkehr direkt auf Layer 2 (Ethernet-Frames) für Mikrosekunden-genaue Steuerung.

#### Wesentliche Unterschiede

| Aspekt | Modbus/OPC UA | PROFINET RT |
|--------|---------------|-------------|
| OSI-Schicht | Layer 4 (TCP) | Layer 2 (Ethernet) |
| Routbar? | Ja | Nein (benötigt L2-Nachbarschaft) |
| Verschlüsselung | Möglich | Nein (Echtzeit-Einschränkungen) |
| Überwachung | Gerouteter TAP | SPAN-Port erforderlich |

#### Warum dies wichtig ist

Das Verständnis von Layer-2-Protokollen ändert Ihren Sicherheitsbewertungsansatz:

- **Keine Segmentierung über Subnetze** - PROFINET benötigt L2-Nachbarschaft
- **Keine Verschlüsselung** - Echtzeit-Performance-Anforderungen (Sub-Millisekunden-Latenz)
- **Andere IDS-Platzierung** - SPAN-Ports verwenden, keine gerouteten TAPs
- **Sicherheit ist architektonisch** - Physische Zugriffskontrolle, VLAN-Isolation, Anomalieerkennung

#### PROFINET-Verkehr erfassen

Das ABB-800xA-System (`abb-800xa-vlan40` unter `10.40.0.11`) hat aktive PROFINET-Echtzeit-Kommunikation.

1. Gehen Sie in VS Code zur **Containerlab**-Erweiterung (TopoViewer)

2. Lokalisieren Sie die Verbindung zwischen `abb-800xa-vlan40` und `sw-acc2`

3. Klicken Sie mit der **rechten Maustaste** auf die Verbindung und wählen Sie **Wireshark Capture**

4. Lassen Sie es 10-15 Sekunden erfassen, um PROFINET-Frames zu sammeln

5. Stoppen Sie die Erfassung

#### PROFINET-Echtzeit-Verkehr analysieren

**Filter für PROFINET:**

```wireshark
# PROFINET-Echtzeit-Verkehr
eth.type == 0x8892
```

Oder im Wireshark-Display-Filter:

```wireshark
pn_io
```

**Was Sie sehen sollten:**

1. **Ethertype 0x8892** - Echtzeit-Zyklus-I/O-Daten
    - Keine TCP/IP-Header (dies ist Layer 2, nicht TCP/IP)
    - Direkte Ethernet-Frames
    - Quell-/Ziel-MAC-Adressen sichtbar

2. **FrameID in hex (typischerweise 0x8000-0x8001)**
    - 0x8000-Bereich = RT-Zyklus-Daten
    - Schauen Sie sich die Paketdetails für das FrameID-Feld an

3. **Zyklisches Muster**
    - PROFINET RT läuft in festen Intervallen (typischerweise 1-10ms)
    - In diesem Lab: ca. 4ms Zykluszeit
    - Paket-Timing ist extrem regelmässig
    - Wählen Sie mehrere Pakete und prüfen Sie das Zeitdelta zwischen ihnen

4. **Nutzdaten**
    - Rohe I/O-Daten (digital/analoge Werte)
    - Keine Verschlüsselung möglich (Echtzeit-Performance-Anforderung)
    - Prozesswerte werden im Klartext übertragen

#### Bewertungsfragen

1. **Was ist der Ethertype?** (Erwartet: 0x8892)
2. **Gibt es TCP/IP-Header?** (Erwartet: Nein - dies ist Layer 2)
3. **Was ist die ungefähre Zykluszeit?** (Zeit zwischen Paketen messen - sollte ~4ms sein)
4. **Können Sie MAC-Adressen sehen?** (Erwartet: Ja - 00:03:2c:40:00:11 und 00:0b:0f:20:00:11)
5. **Sind die Daten verschlüsselt?** (Erwartet: Nein - kann Echtzeit-Verkehr nicht verschlüsseln)
6. **Was passiert, wenn Sie nach `tcp` oder `udp` filtern?** (Erwartet: PROFINET RT erscheint nicht - es ist Layer 2)

#### Sicherheitsimplikationen

**Angriffsszenarien:**

- **Aufklärung**: DCP-Broadcasts offenbaren alle Geräte, Hersteller, Firmware-Versionen
- **Topologie-Mapping**: LLDP (oft mit PROFINET verwendet) exponiert Netzwerkarchitektur
- **Paket-Injektion**: Angreifer im selben L2-Segment kann RT-Frames fälschen
- **Timing-Angriffe**: Störung des zyklischen Timings kann Notabschaltungen verursachen

**Verteidigungsanforderungen:**

- **Physische Sicherheit**: L2-Zugriff schützen (Switch-Ports)
- **VLAN-Isolation**: PROFINET-Domänen trennen
- **Port-Security**: MAC-Adressen-Filterung auf Switches
- **Anomalieerkennung**: Auf Timing-Unregelmässigkeiten, unerwartete Geräte überwachen
- **Keine Anwendungsschicht-Sicherheit**: Kann keine Verschlüsselung, TLS usw. verwenden

**IEC-62443-Kontext:**

PROFINET entstand vor IEC 62443 und wurde für Performance, nicht für Sicherheit entwickelt. Sicherheit muss durch Folgendes implementiert werden:

- Netzwerkarchitektur (Zonen und Leitungen)
- Physische Zugriffskontrolle
- Überwachung und Erkennung (nicht Prävention auf Protokollebene)

### Aufgabe 7: Protokollvergleich und Sicherheitsbewertung

Nachdem Sie mehrere OT-Protokolle analysiert haben, vergleichen Sie deren Sicherheitsmerkmale und verstehen Sie, wann Sie jedes in realen Umgebungen antreffen werden.

#### Protokollvergleichs-Matrix

| Protokoll | Schicht | Port/Typ | Verschlüsselung | Auth | Hauptanwendungsfall | Sicherheitsansatz |
|----------|-------|-----------|------------|------|------------------|-------------------|
| **Modbus TCP** | L4 TCP | 502 | Nein | Nein | Legacy-SCADA, einfaches SPS-Polling | Segmentierung, Überwachung, Modbus/TLS erwägen |
| **CODESYS** | L4 TCP/UDP | 1217, 2455 | Optional | Optional | SPS-Engineering, Diagnose | In Produktion deaktivieren, nur während Wartung verwenden |
| **OPC UA** | L4 TCP | 4840 | Ja (oft deaktiviert) | Ja (oft deaktiviert) | Vertikale Integration, MES/SCADA, Industrie 4.0 | **Integrierte Sicherheitsfunktionen aktivieren** |
| **PROFINET RT** | L2 Ethernet | 0x8892 | Nein (unmöglich) | Nein | Echtzeit-I/O-Steuerung, Automotive, diskrete Fertigung | Physische + architektonische Kontrollen |

#### Kritische Bewertungsfragen

Für jedes in einer OT-Umgebung entdeckte Protokoll fragen:

1. **Ist dieses Protokoll notwendig?**
    - CODESYS-Engineering-Ports sollten in der Produktion deaktiviert sein
    - Modbus kann Legacy sein und durch OPC UA ersetzt werden

2. **Ist die Sicherheit optimal konfiguriert?**
    - OPC UA: SecurityPolicy und Authentifizierung sollten aktiviert sein
    - Modbus: Upgrade auf Modbus/TLS oder Einbindung in VPN erwägen

3. **Ist die Netzwerkarchitektur angemessen?**
    - Alle Protokolle: Ordentliche VLAN-Segmentierung
    - PROFINET: Muss L2-Isolation haben, kann nicht routen
    - OPC UA: Kann routen, Firewall-Regeln zwischen Zonen berücksichtigen

4. **Ist Überwachung vorhanden?**
    - Modbus/OPC UA: Kann auf L3/L4 überwachen (gerouteter TAP oder Firewall-Logs)
    - PROFINET: Benötigt L2-SPAN-Port zum Switch
    - Alle: Normales Verhalten als Baseline, bei Anomalien alarmieren

#### Audit-Checkliste

Bei der Bewertung einer OT-Umgebung:

- [ ] Alle verwendeten Protokolle inventarisieren (Wireshark auf SPAN-Ports)
- [ ] Für OPC UA: Prüfen, dass SecurityPolicy NICHT None ist
- [ ] Für OPC UA: Prüfen, dass Authentifizierung erforderlich ist (nicht Anonymous)
- [ ] Für Modbus: Netzwerksegmentierung verhindert unbefugten Zugriff prüfen
- [ ] Für CODESYS: Engineering-Ports sind deaktiviert oder zugriffskontrolliert prüfen
- [ ] Für PROFINET: VLAN-Isolation und Port-Security prüfen
- [ ] Für alle: Überwachung/IDS ist vorhanden und auf OT-Protokolle abgestimmt prüfen
- [ ] Befunde mit spezifischen IEC-62443-SR-Verletzungen dokumentieren

#### IEC-62443-Sicherheitsanforderungs-Mapping

| SR | Anforderung | Modbus TCP | OPC UA (Fehlkonfiguriert) | OPC UA (Korrekt) | PROFINET |
|----|-------------|------------|------------------------|-----------------|----------|
| SR 1.1 | Benutzeridentifikation | Nicht anwendbar | FAIL (Anonymous) | PASS (Zertifikat) | Nicht anwendbar |
| SR 3.1 | Kommunikationsintegrität | FAIL (kein Schutz) | FAIL (keine Signierung) | PASS (Signiert) | FAIL (kein Schutz) |
| SR 4.1 | Informationsvertraulichkeit | FAIL (Klartext) | FAIL (keine Verschlüsselung) | PASS (Verschlüsselt) | FAIL (unmöglich) |
| SR 5.1 | Netzwerksegmentierung | Erforderlich | Erforderlich | Empfohlen | **Kritisch** |

#### Fazit: Verteidigung in der Tiefe

Keine einzelne Protokollsicherheitsfunktion bietet vollständigen Schutz. OT-Sicherheit erfordert:

1. **Protokollebenen-Sicherheit** (wo verfügbar) - OPC-UA-Sicherheitsfunktionen aktivieren
2. **Netzwerkarchitektur** - Ordentliche Segmentierung, VLANs, Zugriffskontrolle
3. **Überwachung und Erkennung** - Normales Verhalten als Baseline, bei Anomalien alarmieren
4. **Physische Sicherheit** - Besonders für L2-Protokolle wie PROFINET
5. **Zugriffsverwaltung** - Unnötige Dienste deaktivieren (CODESYS in Produktion)

**Wichtigste Erkenntnis:** Legacy-Protokolle (Modbus) erfordern kompensierende Kontrollen. Moderne Protokolle (OPC UA) haben Sicherheitsfunktionen, die aktiviert werden müssen. Echtzeit-Protokolle (PROFINET) erfordern architektonische Sicherheit, da Protokollebenen-Sicherheit nicht möglich ist.

## Fazit

Diese Übung demonstrierte die Sicherheitsrealität von OT-Protokollen:

- **Legacy-Protokolle** (Modbus) haben keine Sicherheitsfunktionen - verlassen sich auf Segmentierung und Überwachung
- **Moderne Protokolle** (OPC UA) haben ausgezeichnete Sicherheitsfunktionen, die oft deaktiviert sind - auditieren und aktivieren
- **Echtzeit-Protokolle** (PROFINET) können keine Verschlüsselung verwenden - erfordern architektonische Kontrollen
- **Alle Protokolle** profitieren von Netzwerksegmentierung, Überwachung und ordentlicher Zugriffskontrolle

Als OT-Sicherheitsexperte ist Ihre Rolle:

1. Inventarisieren, welche Protokolle verwendet werden
2. Ihre Sicherheitskonfiguration bewerten
3. Lücken gegenüber IEC-62443-Anforderungen identifizieren
4. Praktische, betriebsfreundliche Massnahmen empfehlen
5. Überwachung implementieren, um unbefugte Protokollnutzung zu erkennen

Die heute analysierten Protokolle repräsentieren 80% dessen, was Sie in realen Industrieumgebungen antreffen werden. Das Verständnis ihrer Sicherheitsmerkmale ist grundlegend für eine effektive OT-Sicherheitsbewertung.

---

## Lösung

??? success "Zum Anzeigen der Lösung klicken"
    ### Aufgabe 1: OT-Verkehr generieren

    HMI-Desktop öffnen im Browser:
    ```
    http://localhost:5800
    ```

    Im HMI-Desktop Chromium öffnen und zur WebVisu navigieren:
    ```
    http://10.10.0.11:8080/webvisu.htm
    ```

    Interagieren Sie mit Buttons und Schaltern in der Visualisierung, um HTTP- und möglicherweise Modbus-Verkehr zu generieren.

    ### Aufgabe 2: Verkehr erfassen

    1. In VS Code Containerlab TopoViewer öffnen
    2. Rechtsklick auf Verbindung zwischen `wago-plc2a-vlan10` und `sw-acc1`
    3. "Wireshark Capture" wählen
    4. Im HMI erneut mit WebVisu interagieren
    5. Verkehr in Wireshark beobachten

    ### Aufgabe 3: Modbus TCP analysieren

    Wireshark-Filter anwenden:
    ```wireshark
    mbtcp
    ```

    Oder alternativ:
    ```wireshark
    tcp.port == 502
    ```

    **Erwartete Beobachtungen:**

    - **Function Codes**: Typischerweise Read Holding Registers (0x03), Write Single Coil (0x05), Write Multiple Registers (0x10)
    - **Werte sichtbar**: Ja - alle Registerwerte sind im Klartext sichtbar
    - **Verschlüsselung**: Nein - Modbus TCP hat keine Verschlüsselung
    - **Authentifizierung**: Nein - keine Authentifizierung im Protokoll

    **Sicherheitsimplikation**: Ein Angreifer im selben Netzwerksegment kann:
    - Alle Prozesswerte lesen
    - Befehle an die SPS senden
    - Verkehr manipulieren (Man-in-the-Middle)

    Vom Jumphost können Sie die Modbus-Konnektivität testen:
    ```bash
    ssh admin@192.168.100.52
    nc -zv 10.10.0.11 502
    ```

    Erwartete Ausgabe:
    ```text
    Connection to 10.10.0.11 502 port [tcp/*] succeeded!
    ```

    ### Aufgabe 4: CODESYS-Protokoll analysieren

    Filter für HTTP-Verkehr zur WebVisu:
    ```wireshark
    http && ip.addr == 10.10.0.11
    ```

    HTTP-Stream folgen:
    - Rechtsklick auf HTTP-Paket
    - Follow → HTTP Stream
    - Beobachten Sie Visualisierungsdaten im Klartext

    **Erwartete Beobachtungen:**
    - WebVisu verwendet HTTP (Port 8080)
    - Keine HTTPS-Verschlüsselung
    - Variablenwerte und Status im Klartext
    - Session-Informationen exponiert

    Für CODESYS-Runtime-Protokoll (normalerweise von Engineering-Workstation):
    ```wireshark
    udp.port == 2455 || tcp.port == 1217
    ```

    Im Lab sehen Sie möglicherweise wenig CODESYS-Runtime-Verkehr, da keine Engineering-Workstation aktiv programmiert.

    **Port-Übersicht auf der SPS:**

    Sie können die offenen Ports vom Jumphost oder von der SPS selbst prüfen:
    ```bash
    ssh admin@192.168.100.53  # wago-plc2a-vlan10
    ss -tuln | grep LISTEN
    ```

    Erwartete Ausgabe:
    ```text
    tcp   LISTEN 0      5              0.0.0.0:8080       0.0.0.0:*    # WebVisu
    tcp   LISTEN 0      128            0.0.0.0:22         0.0.0.0:*    # SSH
    tcp   LISTEN 0      63          10.10.0.11:502        0.0.0.0:*    # Modbus
    tcp   LISTEN 0      10          10.10.0.11:4840       0.0.0.0:*    # OPC UA
    ```

    ### Aufgabe 5: OPC-UA-Sicherheitskonfiguration analysieren

    **OPC-UA-Verkehr generieren:**

    Option 1 - Vom HMI-Browser:
    ```
    http://10.20.0.11:8080/webvisu.htm  # ctrlx-plc3-vlan20
    ```

    Option 2 - Konnektivität vom Jumphost testen:
    ```bash
    ssh admin@192.168.100.52
    nc -zv 10.20.0.11 4840
    ```

    Erwartete Ausgabe:
    ```text
    Connection to 10.20.0.11 4840 port [tcp/*] succeeded!
    ```

    **Wireshark-Analyse:**

    Erfassen Sie Verkehr zwischen `ctrlx-plc3-vlan20` und `sw-acc1`.

    Filter für OPC UA:
    ```wireshark
    opcua
    ```

    Oder:
    ```wireshark
    tcp.port == 4840
    ```

    **CreateSessionRequest analysieren:**

    Erweitern: `OpcUa Binary Protocol → Message → CreateSessionRequest`

    Suchen: `SecurityPolicyUri`

    **Erwarteter Wert:**
    ```
    http://opcfoundation.org/UA/SecurityPolicy#None
    ```

    **Bedeutung**: Keine Verschlüsselung aktiviert.

    **ActivateSessionRequest analysieren:**

    Erweitern: `OpcUa Binary Protocol → Message → ActivateSessionRequest`

    Suchen: `UserIdentityToken`

    **Erwarteter Typ:**
    ```
    AnonymousIdentityToken
    ```

    **Bedeutung**: Keine Authentifizierung erforderlich.

    **BrowseRequest/Response analysieren:**

    - Komplette OPC-UA-Node-Struktur ist sichtbar
    - Alle Tag-Namen im Klartext
    - Organisationsstruktur des Prozessdatenmodells exponiert
    - Ein Angreifer kann vollständiges Asset-Inventar erstellen

    **ReadRequest/Response analysieren:**

    - Variablenwerte im Klartext übertragen
    - Keine Integritätsprüfung
    - Keine Vertraulichkeit
    - Echtzeit-Prozessdaten für Angreifer sichtbar

    **Bewertungsergebnisse:**

    1. **SecurityPolicy**: None (FAIL)
    2. **Verschlüsselung**: Nein (FAIL)
    3. **Authentifizierung**: Anonymous (FAIL)
    4. **Tag-Namen sichtbar**: Ja (FAIL)
    5. **Variablenwerte sichtbar**: Ja (FAIL)

    **IEC-62443-Verletzungen:**

    - SR 1.1 (Identifikation & Authentifizierung): FAIL
    - SR 3.1 (Kommunikationsintegrität): FAIL
    - SR 4.1 (Informationsvertraulichkeit): FAIL

    **Empfehlung:**

    OPC-UA-Server-Konfiguration ändern:
    ```
    SecurityPolicy: Basic256Sha256
    SecurityMode: SignAndEncrypt
    Authentication: Certificate-based
    ```

    ### Aufgabe 6: PROFINET-Protokoll-Erkennung

    **PROFINET-Verkehr erfassen:**

    Erfassen Sie Verkehr zwischen `abb-800xa-vlan40` und `sw-acc2`.

    **Wireshark-Filter:**

    ```wireshark
    eth.type == 0x8892
    ```

    Oder:
    ```wireshark
    pn_io
    ```

    **Erwartete Beobachtungen:**

    1. **Ethertype**: 0x8892 (PROFINET Echtzeit)
    2. **Keine TCP/IP-Header**: Dies ist Layer 2, nicht Layer 4
    3. **FrameID**: Typischerweise 0x8000-0x8001 (RT-Zyklus-Daten)
    4. **Zyklisches Muster**: Pakete alle ~4ms (sehr regelmässig)
    5. **MAC-Adressen sichtbar**:
       - Quelle: 00:03:2c:40:00:11 (ABB-System)
       - Ziel: 00:0b:0f:20:00:11 (PROFINET-Gerät)

    **Timing-Analyse:**

    Wählen Sie mehrere aufeinanderfolgende PROFINET-Pakete und prüfen Sie das Zeitdelta:

    In Wireshark:
    - View → Time Display Format → Seconds Since Previous Displayed Packet
    - Erwartung: ~0.004 Sekunden (4ms Zykluszeit)

    **Bewertungsergebnisse:**

    1. **Ethertype**: 0x8892 (Layer 2 Echtzeit)
    2. **TCP/IP-Header**: Nein - direkter Ethernet-Frame
    3. **Zykluszeit**: ~4ms (sehr deterministisch)
    4. **MAC-Adressen**: Ja, sichtbar
    5. **Verschlüsselung**: Nein - unmöglich bei Echtzeit
    6. **Nach tcp/udp filtern**: PROFINET RT erscheint nicht

    **Sicherheitsimplikationen:**

    - Kein Routing möglich (Layer 2)
    - Keine Verschlüsselung möglich
    - Physischer Zugang = volle Kontrolle
    - VLAN-Isolation ist kritisch
    - Port Security erforderlich
    - Anomalieerkennung basierend auf Timing

    **PROFINET DCP (Discovery):**

    Für PROFINET-Geräteerkennung:
    ```wireshark
    pn_dcp
    ```

    DCP-Broadcasts offenbaren:
    - Gerätename
    - Hersteller
    - IP-Konfiguration
    - Rolle im Netzwerk

    ### Aufgabe 7: Protokollvergleich

    **Zusammenfassung der Analyse:**

    | Protokoll | Verschlüsselung | Auth | Hauptrisiko | Verteidigung |
    |-----------|-----------------|------|-------------|--------------|
    | **Modbus TCP** | Nein | Nein | Klartext-Zugriff | Segmentierung + Überwachung |
    | **CODESYS** | Optional, meist Nein | Optional | Engineering-Zugriff | In Produktion deaktivieren |
    | **OPC UA** | Ja (oft deaktiviert) | Ja (oft deaktiviert) | Fehlkonfiguration | Sicherheitsfunktionen aktivieren |
    | **PROFINET RT** | Unmöglich | Nein | Layer-2-Angriffe | Physische + VLAN-Isolation |

    **Audit-Checkliste erfüllt:**

    - [x] Protokolle inventarisiert (Modbus, OPC UA, PROFINET, HTTP)
    - [x] OPC-UA-SecurityPolicy geprüft (None = FAIL)
    - [x] OPC-UA-Authentifizierung geprüft (Anonymous = FAIL)
    - [x] Modbus-Segmentierung geprüft
    - [x] CODESYS-Ports identifiziert (sollten deaktiviert sein)
    - [x] PROFINET-VLAN-Isolation geprüft
    - [x] IEC-62443-Verletzungen dokumentiert

    **Wichtigste Erkenntnisse:**

    1. **Legacy-Protokolle** (Modbus): Keine Sicherheitsfunktionen → Kompensierende Kontrollen erforderlich
    2. **Moderne Protokolle** (OPC UA): Sicherheit vorhanden, aber oft deaktiviert → Audit und Aktivierung erforderlich
    3. **Echtzeit-Protokolle** (PROFINET): Verschlüsselung unmöglich → Architektonische Sicherheit kritisch
    4. **Alle Protokolle**: Profitieren von Segmentierung, Überwachung und Zugriffskontrolle

    **Nächste Schritte:**

    1. OPC-UA-Sicherheitsfunktionen aktivieren (höchste Priorität)
    2. Netzwerksegmentierung durchsetzen (siehe Übung 7)
    3. CODESYS-Engineering-Ports für Produktion deaktivieren
    4. IDS/IPS für OT-Protokoll-Anomalien implementieren
    5. Regelmässige Protokoll-Audits durchführen

```
