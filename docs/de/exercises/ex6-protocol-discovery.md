# Übung 6: OT-Protokoll-Erkennung

**Zeitschätzung**: 30-35 Minuten  
**Lab-Topologie**: `ot-sec-flat` (empfohlen) oder `ot-sec-segmented`  
**Schwierigkeit**: :material-signal-cellular-2: Fortgeschritten

## Lernziele

- OT-Protokolle im Live-Verkehr entdecken
- Modbus-TCP-Kommunikation identifizieren
- CODESYS-Runtime-Protokoll analysieren
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

### Aufgabe 5: Sicherheitsimplikationen

Basierend auf Ihren Beobachtungen:

1. **Vertraulichkeit**: Könnte ein Angreifer, der passiv im Netzwerk lauscht, den Zustand des Prozesses kennen (z.B. "Temperatur ist 80°C")?
2. **Integrität**: Könnte ein Angreifer Pakete einfügen, um Werte zu ändern (z.B. "Geschwindigkeit auf 10000 setzen")?
3. **Verfügbarkeit**: Was passiert, wenn ein Angreifer Port 502 flutet?

## Fazit

OT-Protokolle wie Modbus TCP wurden für Zuverlässigkeit, nicht für Sicherheit entwickelt. Ihnen fehlen Verschlüsselung und Authentifizierung. Diese Übung demonstrierte, wie einfach dieser Verkehr erfasst und analysiert werden kann, und hebt die Notwendigkeit für Netzwerksegmentierung (um einzuschränken, wer schnüffeln kann) und Überwachung (um unbefugte Befehle zu erkennen) hervor.
