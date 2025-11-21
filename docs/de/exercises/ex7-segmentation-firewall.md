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

---

## Lösung

??? success "Zum Anzeigen der Lösung klicken"
    ### Aufgabe 1: Kommunikationsflüsse kartieren

    **Legitime Kommunikationspfade:**

    1. **HMI (VLAN 40) → SPSen (VLAN 10, 20, 30)**
        - Modbus TCP (Port 502): JA - für Prozesssteuerung
        - HTTP/WebVisu (Port 8080): JA - für Visualisierung
        - OPC UA (Port 4840): JA - moderne Alternative
        - SSH (Port 22): NEIN - nur für Engineering, nicht HMI

    2. **Engineering-Station (VLAN 50) → SPSen**
        - CODESYS Runtime (Port 2455, 1217): JA - für Programmierung
        - SSH (Port 22): JA - für Wartung
        - HTTP (Port 8080): JA - für WebVisu-Zugriff

    3. **SPSen → Internet**
        - NEIN - SPSen sollten keinen direkten Internet-Zugang haben
        - Ausnahme: NTP für Zeitsynchronisation (kontrolliert über Firewall)

    4. **SPSen untereinander**
        - JA - Inter-Controller-Kommunikation möglich
        - Protokolle: Modbus TCP, OPC UA, proprietäre Protokolle

    **Antwort auf die Frage:**

    - **Muss HMI per SSH in die SPS?** NEIN - SSH ist für Engineering/Wartung, nicht für normale HMI-Funktionen
    - **Muss HMI SPS anpingen?** JA - für Verfügbarkeitsprüfungen ist ICMP sinnvoll
    - **Muss HMI auf Webserver zugreifen?** JA - WebVisu läuft auf HTTP (Port 8080)

    ### Aufgabe 2: Kommunikationsmatrix erstellen

    **Erweiterte Kommunikationsmatrix:**

    | Quelle | Ziel | Protokoll | Port | Aktion | Begründung |
    |--------|------|-----------|------|--------|------------|
    | HMI (10.40.0.12) | SPS (10.10.0.11) | Modbus TCP | 502 | ERLAUBEN | Prozessdaten lesen/schreiben |
    | HMI (10.40.0.12) | SPS (10.10.0.11) | HTTP | 8080 | ERLAUBEN | WebVisu-Zugriff |
    | HMI (10.40.0.12) | SPS (10.10.0.11) | OPC UA | 4840 | ERLAUBEN | Alternative zu Modbus |
    | HMI (10.40.0.12) | SPS (10.10.0.11) | ICMP | - | ERLAUBEN | Verfügbarkeitsprüfung |
    | HMI (10.40.0.12) | SPS (10.10.0.11) | SSH | 22 | BLOCKIEREN | Nicht für HMI-Betrieb nötig |
    | EWS (10.50.0.11) | SPS (10.10.0.11) | SSH | 22 | ERLAUBEN | Engineering/Wartung |
    | EWS (10.50.0.11) | SPS (10.10.0.11) | CODESYS | 2455, 1217 | ERLAUBEN | Programmierung |
    | EWS (10.50.0.11) | SPS (10.10.0.11) | HTTP | 8080 | ERLAUBEN | WebVisu-Konfiguration |
    | SPS (10.10.0.11) | Internet | Any | Any | BLOCKIEREN | Keine direkte Internet-Verbindung |
    | Any | Any | Any | Any | BLOCKIEREN | Default-Deny-Policy |

    ### Aufgabe 3: Aktuelle Firewall-Regeln analysieren

    **SSH zur Firewall:**

    ```bash
    ssh admin@192.168.100.11
    ```

    Passwort: `admin`

    Alternativ mit sshpass (für Automatisierung):
    ```bash
    sshpass -p admin ssh admin@192.168.100.11
    ```

    **Firewall-Regeln anzeigen:**

    ```bash
    /ip firewall filter print where chain=forward
    ```

    **Erwartete Ausgabe (ot-sec-flat):**

    ```text
    Flags: X - disabled, I - invalid; D - dynamic 
     0    ;;; allow established,related connections
          chain=forward action=accept connection-state=established,related 

     1    ;;; allow forwarding ICMP
          chain=forward action=accept protocol=icmp 

     2    ;;; allow SSH to the jumphost from internet
          chain=forward action=accept protocol=tcp dst-address=2.2.2.2 
          in-interface-list=ext dst-port=22 

     3    ;;; allow all traffic from DMZ to VLAN10
          chain=forward action=accept in-interface=dmz-ext out-interface=vlan10 

     4    ;;; default forward drop
          chain=forward action=accept log=yes log-prefix="default forward drop"
    ```

    **Erwartete Ausgabe (ot-sec-segmented):**

    ```text
    Flags: X - disabled, I - invalid; D - dynamic 
     0    ;;; allow established,related connections
          chain=forward action=accept connection-state=established,related

     1    ;;; allow forwarding ICMP
          chain=forward action=accept protocol=icmp

     2    ;;; allow SSH to the jumphost from internet
          chain=forward action=accept protocol=tcp dst-address=2.2.2.2
          in-interface-list=ext dst-port=22

     3    ;;; allow Control Room to Hygiene
          chain=forward action=accept in-interface=vlan40 out-interface=vlan10

     4    ;;; allow Control Room to Process
          chain=forward action=accept in-interface=vlan40 out-interface=vlan20

     5    ;;; allow Control Room to Disposal
          chain=forward action=accept in-interface=vlan40 out-interface=vlan30

     6    ;;; allow Parameterization to Hygiene
          chain=forward action=accept in-interface=vlan50 out-interface=vlan10

     ... weitere Regeln ...
    ```

    **Analyse:**

    - **Flat-Topologie**: Regel 3 erlaubt ALL traffic von DMZ zu VLAN 10 - zu permissiv
    - **Segmented-Topologie**: Regel 3 erlaubt ALL traffic von VLAN 40 zu VLAN 10 - zu permissiv
    - **Problem**: Keine Port-Einschränkungen, keine Protokoll-Spezifizierung
    - **Risiko**: SSH, unnötige Dienste sind erreichbar

    ### Aufgabe 4: Restriktive Regeln implementieren

    **WARNUNG**: Diese Änderungen können die Netzwerkverbindung unterbrechen. In Produktion immer in Wartungsfenster durchführen und Rollback-Plan haben.

    **Schritt 1: Spezifische Regel für Modbus TCP hinzufügen**

    ```bash
    /ip firewall filter add chain=forward action=accept \
        src-address=10.40.0.12 dst-address=10.10.0.11 \
        protocol=tcp dst-port=502 \
        comment="Allow HMI to PLC Modbus" place-before=0
    ```

    **Schritt 2: Spezifische Regel für WebVisu HTTP hinzufügen**

    ```bash
    /ip firewall filter add chain=forward action=accept \
        src-address=10.40.0.12 dst-address=10.10.0.11 \
        protocol=tcp dst-port=8080 \
        comment="Allow HMI to PLC WebVisu" place-before=0
    ```

    **Schritt 3: Spezifische Regel für OPC UA hinzufügen**

    ```bash
    /ip firewall filter add chain=forward action=accept \
        src-address=10.40.0.12 dst-address=10.10.0.11 \
        protocol=tcp dst-port=4840 \
        comment="Allow HMI to PLC OPC-UA" place-before=0
    ```

    **Schritt 4: Regeln überprüfen**

    ```bash
    /ip firewall filter print where chain=forward
    ```

    Sie sollten jetzt die neuen spezifischen Regeln ganz oben sehen.

    **Schritt 5: Breite "Allow any"-Regel identifizieren**

    Finden Sie die Regel-Nummer der breiten Regel:

    ```bash
    /ip firewall filter print where chain=forward
    ```

    In `ot-sec-segmented`: Suchen Sie nach "allow Control Room to Hygiene" (z.B. Regel #3)
    In `ot-sec-flat`: Suchen Sie nach "allow all traffic from DMZ to VLAN10" (z.B. Regel #3)

    **Schritt 6: Breite Regel deaktivieren**

    ```bash
    /ip firewall filter disable [find comment="allow Control Room to Hygiene"]
    ```

    Oder für flat:
    ```bash
    /ip firewall filter disable [find comment="allow all traffic from DMZ to VLAN10"]
    ```

    **Schritt 7: Default-Drop aktivieren (Optional)**

    In `ot-sec-flat` ist die Default-Drop-Regel tatsächlich auf "accept" gesetzt. Um echte Sicherheit zu haben:

    ```bash
    /ip firewall filter set [find comment="default forward drop"] action=drop
    ```

    **ACHTUNG**: Dies blockiert allen Verkehr, der nicht explizit erlaubt ist. Stellen Sie sicher, dass alle notwendigen Regeln vorhanden sind!

    **Schritt 8: Testen**

    Von einem anderen Terminal (nicht über SSH zur Firewall!):

    ```bash
    # Test 1: HMI kann WebVisu erreichen (sollte funktionieren)
    docker exec clab-ot-sec-flat-pilz-hmi01-vlan40 curl -I --connect-timeout 3 http://10.10.0.11:8080

    # Test 2: HMI kann Modbus erreichen (sollte funktionieren)
    docker exec clab-ot-sec-flat-pilz-hmi01-vlan40 nc -zv 10.10.0.11 502

    # Test 3: HMI kann SSH NICHT erreichen (sollte blockiert sein, falls Default-Drop aktiv)
    docker exec clab-ot-sec-flat-pilz-hmi01-vlan40 nc -zv -w 2 10.10.0.11 22
    ```

    **Erwartete Ergebnisse:**

    Test 1 & 2: Erfolgreich
    Test 3: Connection timed out oder refused (wenn Default-Drop aktiv)

    **Rollback (falls nötig):**

    ```bash
    # Breite Regel wieder aktivieren
    /ip firewall filter enable [find comment="allow Control Room to Hygiene"]

    # Default-Drop wieder auf accept setzen
    /ip firewall filter set [find comment="default forward drop"] action=accept
    ```

    ### Aufgabe 5: Flat vs. Segmentiert vergleichen

    **Vergleichstabelle:**

    | Aspekt | Flat Network | Segmented Network (mit Firewall) |
    |--------|--------------|----------------------------------|
    | **Broadcast-Domäne** | Eine grosse Domäne | Mehrere kleine VLANs |
    | **ARP-Spoofing-Reichweite** | Alle Geräte betroffen | Auf VLAN beschränkt |
    | **Lateral Movement** | Einfach - direkter Zugang | Schwierig - Firewall blockiert |
    | **Angriffsfläche** | Alle Ports auf allen Geräten | Nur erlaubte Ports/Protokolle |
    | **Verkehrsfilterung** | Keine | Zentral an Firewall |
    | **Überwachung** | Verteilt, komplex | Zentral an Firewall |
    | **Incident Response** | Schwierig zu isolieren | Kann Zonen isolieren |
    | **Compliance** | IEC-62443 SR 5.1 FAIL | IEC-62443 SR 5.1 PASS |

    **Angriffsszenario - Flat Network:**

    1. Angreifer kompromittiert HMI (10.40.0.12)
    2. Führt ARP-Spoofing durch → Man-in-the-Middle auf gesamtem Netzwerk
    3. Direkter Zugriff auf SPS über SSH (Port 22)
    4. Direkter Zugriff auf alle Engineering-Ports
    5. Kann sich lateral zu allen anderen Geräten bewegen
    6. Keine zentrale Überwachung des Verkehrs

    **Angriffsszenario - Segmented Network:**

    1. Angreifer kompromittiert HMI (10.40.0.12 in VLAN 40)
    2. ARP-Spoofing nur in VLAN 40 möglich
    3. Versuch, auf SPS (10.10.0.11 in VLAN 10) zuzugreifen
    4. Firewall blockiert SSH (Port 22) → Zugriff verweigert
    5. Firewall erlaubt nur Modbus, HTTP, OPC UA
    6. Firewall loggt alle Zugriffsversuche
    7. IDS/IPS kann Anomalien erkennen
    8. Security-Team wird alarmiert
    9. VLAN 10 kann isoliert werden ohne VLAN 40 zu beeinflussen

    **Fazit:**

    - **Flat Network**: Einfach zu verwalten, aber katastrophal bei Kompromittierung
    - **Segmented Network**: Komplexer, aber deutlich sicherer und IEC-62443-konform
    - **Best Practice**: Segmentierung + spezifische Firewall-Regeln + Monitoring

    ### Zusätzliche Best Practices

    **1. Logging aktivieren:**

    ```bash
    /ip firewall filter set [find comment="default forward drop"] log=yes log-prefix="fw-drop"
    ```

    **2. Address Lists für bessere Verwaltung:**

    ```bash
    # HMI-Geräte definieren
    /ip firewall address-list add list=hmi-devices address=10.40.0.12
    /ip firewall address-list add list=hmi-devices address=10.40.0.13

    # SPS-Geräte definieren
    /ip firewall address-list add list=plc-devices address=10.10.0.11
    /ip firewall address-list add list=plc-devices address=10.10.0.12
    /ip firewall address-list add list=plc-devices address=10.20.0.11

    # Regel mit Address Lists
    /ip firewall filter add chain=forward action=accept \
        src-address-list=hmi-devices dst-address-list=plc-devices \
        protocol=tcp dst-port=502 \
        comment="Allow HMI to PLCs Modbus"
    ```

    **3. Port-Objekte (Service Groups):**

    ```bash
    # In RouterOS können Sie mit Address Lists arbeiten, aber keine Port-Groups
    # Alternative: Mehrere spezifische Regeln oder Kommentare zur Gruppierung
    ```

    **4. Regelmässige Audits:**

    ```bash
    # Alle Regeln mit Zählern anzeigen
    /ip firewall filter print stats

    # Nicht genutzte Regeln identifizieren (bytes=0)
    /ip firewall filter print stats where bytes=0
    ```

    **5. Change Management:**

    - Alle Firewall-Änderungen dokumentieren
    - Backup vor Änderungen erstellen:
      ```bash
      /system backup save name=before-firewall-change
      ```
    - In Testumgebung validieren
    - Schrittweise in Produktion ausrollen

    ### Zusammenfassung: Lessons Learned

    1. **Segmentierung allein reicht nicht** - Firewall-Regeln sind essentiell
    2. **Default-Deny ist kritisch** - "Allow any" macht Segmentierung wertlos
    3. **Least Privilege durchsetzen** - Nur absolut notwendige Ports erlauben
    4. **Monitoring ist Pflicht** - Logging und Alerting aktivieren
    5. **Dokumentation ist entscheidend** - Wissen, warum jede Regel existiert
    6. **Testen, testen, testen** - Vor Produktivsetzung validieren
    7. **IEC 62443 SR 5.1** - Netzwerksegmentierung ist Grundanforderung
