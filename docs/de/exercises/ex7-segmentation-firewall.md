# Übung 7: Netzwerksegmentierung und Firewall-Design

**Zeitschätzung**: 35-45 Minuten  
**Lab-Topologie**: `ot-sec-segmented` (primär), `ot-sec-flat` (Vergleich)  
**Schwierigkeit**: :material-signal-cellular-3: Fortgeschritten bis Experte

## Lernziele

- Zonenbasierte Segmentierung verstehen (Purdue-Modell-Konzepte)
- Kommunikationsmatrix für OT-Geräte erstellen
- Firewall-Regeln entwerfen, um Verkehr auf notwendige Protokolle zu beschränken
- Sicherheitslage von flachen vs. segmentierten Netzwerken vergleichen

## Voraussetzungen

- Übung 1 & 6
- Grundverständnis von Firewalls

## Szenario

In der `ot-sec-flat`-Topologie befanden sich alle Geräte in derselben Broadcast-Domäne. Wenn ein Gerät kompromittiert wurde, konnte der Angreifer auf alles zugreifen. In `ot-sec-segmented` haben wir VLANs, die Funktionen trennen:
- **VLAN 10 (Hygiene)**: SPSen
- **VLAN 20 (Prozess)**: SPSen
- **VLAN 40 (Leitwarte)**: HMI
- **VLAN 50 (Engineering)**: Workstations

Segmentierung allein reicht jedoch nicht aus, wenn die Firewall allen Verkehr zwischen Zonen erlaubt. Wir müssen "Least Privilege" implementieren.

## Aufgaben

### Aufgabe 1: Kommunikationsflüsse kartieren

1.  Identifizieren Sie die legitimen Kommunikationspfade, die für den Betrieb der Anlage notwendig sind:
    *   **HMI (VLAN 40)** muss Daten von **SPSen (VLAN 10, 20, 30)** lesen/schreiben.
    *   **Engineering-Station (VLAN 50)** muss **SPSen** programmieren.
    *   **SPSen** benötigen normalerweise *keinen* Zugang zum Internet.
    *   **SPSen** müssen möglicherweise miteinander kommunizieren (Inter-Controller-Kommunikation).

2.  **Frage**: Muss das HMI per SSH in die SPS? Muss es sie anpingen? Muss es auf den Webserver zugreifen?
    *   *Antwort*: Typischerweise benötigt HMI nur Modbus TCP (502) oder ein proprietäres Protokoll (z.B. 1217/2455 für CODESYS). WebVisu verwendet HTTP (80/8080).

### Aufgabe 2: Kommunikationsmatrix erstellen

Eine Kommunikationsmatrix definiert erlaubten Verkehr. Füllen Sie die unten stehende Tabelle aus (gedanklich oder auf Papier):

| Quelle | Ziel | Protokoll | Port | Aktion |
| :--- | :--- | :--- | :--- | :--- |
| HMI (10.40.0.12) | SPS (10.10.0.11) | Modbus TCP | 502 | Erlauben |
| HMI (10.40.0.12) | SPS (10.10.0.11) | HTTP (WebVisu) | 8080 | Erlauben |
| HMI (10.40.0.12) | SPS (10.10.0.11) | SSH | 22 | **Blockieren** |
| Any | Any | Any | Any | **Blockieren** |

### Aufgabe 3: Aktuelle Firewall-Regeln analysieren

1.  SSH zur `gw-firewall` (Mikrotik):
    ```bash
    ssh admin@192.168.100.11
    ```
    (Passwort: `admin`)

2.  Aktuelle Forwarding-Regeln anzeigen:
    ```bash
    /ip firewall filter print where chain=forward
    ```

3.  Suchen Sie nach der Regel, die Leitwarte (VLAN 40) zu Hygiene (VLAN 10) erlaubt.
    *   Sie sieht wahrscheinlich so aus: `action=accept chain=forward in-interface=vlan40 out-interface=vlan10`
    *   **Analyse**: Diese Regel erlaubt *alle* Protokolle. Dies ist zu permissiv.

### Aufgabe 4: Restriktive Regeln entwerfen und implementieren (Optional/Erweitert)

*Hinweis: Diese Aufgabe erfordert Vertrautheit mit der RouterOS-Syntax. Gehen Sie mit Vorsicht vor.*

Wir möchten die breite "Allow VLAN 40 to VLAN 10"-Regel durch spezifische Regeln für Modbus und HTTP ersetzen.

#### Expertenansicht: Komplexität vs. Wartbarkeit

**Warnung**: Bei Firewall-Regelsätzen kann die Granularität sehr schnell sehr komplex werden. Es ist verlockend, extrem fein granulare Regeln zu erstellen (z.B. separate Regeln für jedes einzelne Gerät, jeden Port, jede Protokollvariante), aber dies führt zu Regelsätzen mit hunderten von Einträgen, die niemand mehr versteht oder warten kann.

**Häufige Fallen**:

- Zu viele spezifische Host-zu-Host-Regeln statt zonenbasierter Regeln
- Mehrfache überlappende Regeln, bei denen unklar ist, welche greift
- Fehlende Dokumentation oder veraltete Kommentare
- "Notfall"-Regeln, die temporär sein sollten, aber permanent werden
- Regeln, die niemand mehr zu löschen wagt, weil unbekannt ist, was sie tun

**Empfehlung: Einfach und verwaltbar halten**:

1. **Zonenbasierte Policies** bevorzugen statt einzelne Hosts (z.B. "VLAN 40 zu VLAN 10" statt "Host A zu Host B")
2. **Serviceobjekte gruppieren** (z.B. "OT-Basisdienste" = Modbus + OPC-UA + HTTP)
3. **Klare Namenskonventionen** (z.B. "ALLOW_LEITWARTE_zu_HYGIENE_OT-Services")
4. **Regelmässig aufräumen** - Nicht verwendete Regeln entfernen
5. **Dokumentieren** - Geschäftsbegründung in Kommentaren festhalten
6. **Testen** - Vor Produktivsetzung in Testumgebung validieren

**Faustregel**: Wenn Ihr Regelsatz mehr als 50 Regeln hat und keine klare Struktur, ist es Zeit für eine Vereinfachung. Ein gut gestalteter Regelsatz mit 20 verständlichen Regeln ist sicherer als ein chaotischer mit 200 Regeln.

1.  **Spezifische Regeln hinzufügen** (oben oder vor der Drop-Regel platziert):

    ```bash
    # Modbus TCP vom HMI zur SPS erlauben
    /ip firewall filter add chain=forward action=accept src-address=10.40.0.12 dst-address=10.10.0.11 protocol=tcp dst-port=502 comment="Allow HMI to PLC Modbus" place-before=0
    
    # HTTP (WebVisu) vom HMI zur SPS erlauben
    /ip firewall filter add chain=forward action=accept src-address=10.40.0.12 dst-address=10.10.0.11 protocol=tcp dst-port=8080 comment="Allow HMI to PLC Web" place-before=0
    ```

2.  **Breite Regel deaktivieren**:
    Finden Sie die Nummer der Regel `allow Control Room to Hygiene` mit `/ip firewall filter print`. Nehmen wir an, sie ist Nummer `X`.
    ```bash
    /ip firewall filter disable X
    ```

3.  **Überprüfen**:
    *   Können Sie noch von HMI aus auf die WebVisu zugreifen? (Sollte JA sein)
    *   Können Sie per SSH vom HMI zur SPS? (Sollte NEIN sein, falls die Standard-Policy Drop ist oder keine andere Regel passt).
    *   *Hinweis*: Die aktuelle Lab-Konfiguration hat eine `default forward drop`-Regel, die `disabled=yes` ist. Um dies vollständig durchzusetzen, müssten Sie diese Drop-Regel am Ende aktivieren.

    ```bash
    # Standard-Drop-Regel am Ende aktivieren
    /ip firewall filter enable [find comment="default forward drop"]
    ```

### Aufgabe 5: Flat vs. Segmentiert vergleichen

1.  **Flaches Netzwerk**:
    *   Angreifer auf HMI kann ARP-Spoofing auf die SPS durchführen.
    *   Angreifer kann auf jeden Port auf der SPS zugreifen.
    *   Kein zentraler Punkt zum Filtern von Verkehr.

2.  **Segmentiertes Netzwerk (mit Firewall)**:
    *   ARP-Spoofing ist auf das VLAN beschränkt.
    *   Firewall setzt Policy durch (z.B. "Nur Modbus erlaubt").
    *   Zentrales Logging von verweigerten Versuchen (falls Logging aktiviert ist).

## Fazit

Segmentierung ist die Grundlage der OT-Sicherheit, muss aber von strikten Firewall-Regeln begleitet werden. Durch den Wechsel von "Allow Any" zu "Allow Specific Ports" reduzieren wir die Angriffsfläche erheblich.
