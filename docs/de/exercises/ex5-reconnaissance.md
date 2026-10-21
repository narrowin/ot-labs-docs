# Übung 5: Zugriff aus der DMZ beurteilen

**Zeitbedarf:** 35–45 Minuten. **Lab:** dein eigenes `ot-sec-flat`-Lab. **Niveau:** angeleitete praktische Übung.

Prüfe, welche Ziele aus der DMZ erreichbar sind, und dokumentiere die Ergebnisse mit Belegen. Begrenze die Erkennung auf die vier genannten Lab-IPs und die Dienstprüfung auf WAGO A. Breite Subnetz-Scans, Passwortversuche und Softwareinstallation sind nicht nötig.

**Lernziele:** Den Ausgangspunkt des Fernzugriffs bestimmen, begrenzte Lab-Ziele erkennen, erreichbare Dienste identifizieren, Beobachtungen von Schwachstellen unterscheiden und einen reproduzierbaren Befund mit Empfehlungen schreiben.

## Dein Auftrag zum Fernzugriff

Ein Lieferant braucht für eine vereinbarte Diagnose Zugang zur WebVisu von WAGO A. Der Anlagenverantwortliche möchte wissen, ob der bestehende DMZ-Jumphost auch Engineering- oder Steuerungsdienste erreicht, die dafür nicht nötig sind. Prüfe die tatsächliche Erreichbarkeit und formuliere eine Empfehlung für den benötigten Zugriff.

Eine **DMZ** ist eine Netzwerkzone zwischen Vertrauensbereichen. Ein **Jumphost** ist ein zwischengeschalteter Rechner für den Zugriff auf andere Systeme. Seine Bezeichnung allein sagt nichts darüber aus, welche Verbindungen erlaubt sind. Ein TCP-**Port** bezeichnet einen Dienst-Endpunkt. Eine erfolgreiche Verbindung belegt die Erreichbarkeit des Dienstes. Sie sagt nichts darüber aus, ob jemand sich anmelden oder den Prozess verändern darf.

Dieses Szenario umfasst ausschliesslich die genannten Simulatorziele. Stimme bei realen OT-Geräten Erkennungsmethoden und Zeitpunkt zuerst mit Betrieb und Geräteverantwortlichen ab; selbst Verbindungsprüfungen können empfindliche Dienste beeinträchtigen. Beginne, wo verfügbar, mit Inventar und passiven Beobachtungen.

## Vorbereitung

Öffne das konfigurierte [DMZ-Jumphost-Terminal](/network/terminal?node=proxmox-jumphost). Alle Hauptaufgaben laufen dort. Finde das Ziel und seinen Gateway-Weg mit der [WAGO-Suche in Netmap](/network/?search=wago-plc2a-vlan10).

| Ziel | Rolle |
| --- | --- |
| `10.10.0.1` | OT-Gateway |
| `10.10.0.11` | WAGO A, Ziel der Dienstprüfung |
| `10.10.0.12` | WAGO B |
| `10.20.0.11` | CtrlX |

Das sind Lab-Adressen. Historische Namen wie `vlan20` beweisen keine getrennten VLANs im aktuellen flachen OT-Netz.

??? info "Du verwendest die eigenständige Dokumentation?"
    Öffne **Netmap** deines zugewiesenen Live-Labs, suche **proxmox-jumphost** und wähle im Aktionsmenü **Open terminal**. Wähle für Extra 1 **wago-plc2b-vlan10**. Wähle für Extra 2 **wago-plc2a-vlan10**, **Capture packets**, Interface **eth1**. Links in der Übungsansicht behalten Netzwerk und Snapshot bei und wählen diese Ziele automatisch.

## Aufgabe 1: Den eigenen Ausgangspunkt bestimmen

```bash
date -Iseconds
hostname
ip -br addr show eth1
ip route get 10.10.0.11
command -v nmap curl
```

**Kontrollpunkt:** Der Jumphost hat Datenadresse `2.2.2.2/24`. Die Route zu WAGO A verwendet Gateway `2.2.2.1`, Interface `eth1` und Quelladresse `2.2.2.2`. Das Management-Interface ist ein separater Weg. Notiere die tatsächlich gewählte Route zum Ziel.

## Aufgabe 2: Nur die genannten Hosts erkennen

```bash
nmap -n -sn --max-retries 1 --host-timeout 10s 10.10.0.1 10.10.0.11 10.10.0.12 10.20.0.11
```

`-n` vermeidet DNS-Abfragen; `-sn` erkennt Hosts ohne Port-Scan. Notiere die antwortenden IPs.

**Kontrollpunkt:** Beim Durchgang im unveränderten Lab antworteten alle vier. Dein Ergebnis gilt für diese Quelle und diesen Zeitpunkt. Eine fehlende Antwort beweist weder einen fehlenden Host noch ein blockiertes VLAN.

??? info "Hinweis: Erkennung und Diensterreichbarkeit unterscheiden"
    Kontrolliere zuerst die Route. Die nächste Aufgabe verwendet `-Pn` für ein bekanntes Lab-Ziel und benötigt deshalb keine erfolgreiche Antwort auf die vorherige Host-Erkennung. Erweitere bei der Fehlersuche nicht den Zielbereich.

## Aufgabe 3: Erreichbare TCP-Ports messen

```bash
nmap -n -sT -Pn -p22,502,4840,8080 --max-retries 1 --host-timeout 20s --scan-delay 100ms 10.10.0.11
```

Der Befehl führt eine begrenzte Anzahl von TCP-Verbindungsversuchen aus. Er meldet sich nicht an und sendet keine SPS-Schreibbefehle. Notiere **STATE** getrennt von **SERVICE**.

| Port | Übliche Verwendung; noch zu verifizieren |
| --- | --- |
| 22/tcp | SSH |
| 502/tcp | Modbus TCP |
| 4840/tcp | OPC UA |
| 8080/tcp | WebVisu-HTTP im Lab |

**Kontrollpunkt:** Im Ausgangszustand waren alle vier offen. Ohne Versionserkennung stammt Nmaps SERVICE-Spalte hauptsächlich aus einer Port-Datenbank. `http-proxy` bei 8080 beweist keinen Proxy. Ein offener TCP-Port beweist weder eine Schwachstelle noch fehlende Anwendungsauthentifizierung.

## Aufgabe 4: Eine Anwendung bestätigen

```bash
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

Die Anfrage lädt die Seite, zeigt Header und verwirft den Inhalt. Lies Status und Inhaltstyp ab. Gib keine Zugangsdaten ein und bediene keine Prozesssteuerung.

**Kontrollpunkt:** Der Durchgang lieferte HTTP `200 OK`, `Content-type: text/html` und das CODESYS-Banner `3S_WebServer`. Das belegt HTTP besser als die Port-Bezeichnung. Ein Banner allein beweist weder Softwareversion noch Patchstand oder Ausnutzbarkeit.

??? info "Hinweis: Verbindung möglich, Seite fehlerhaft"
    Halte den HTTP-Status fest. 404 bedeutet: HTTP antwortet, aber der Pfad fehlt. Ein Timeout ist ein anderes Ergebnis. Prüfe `/webvisu.htm`, Ziel und Port. Startet die Runtime gerade neu, warte und wiederhole diese eine Anfrage.

## Aufgabe 5: Einen reproduzierbaren Befund schreiben

Ergänze in deinen eigenen Notizen:

| Feld | Dein Beleg |
| --- | --- |
| Zeit und Quelle | Zeitstempel, Hostname, gewählte Quell-IP |
| Ziel und Umfang | Genau geprüfte IPs und Ports |
| Beobachtung | Portzustände, HTTP-Status und Banner |
| Auswirkung | Welcher DMZ-zu-OT-Zugriff wurde nachgewiesen? |
| Grenze | Nicht geprüft: Login, Berechtigungen, Verschlüsselung, Schwachstellen |
| Empfehlung | Eine für den Diagnoseauftrag benötigte Verbindung und eine unnötige Freigabe, die geprüft werden soll |
| Nachprüfung | Genaue Angaben zu Quelle, Ziel, Befehl und erwartetem Ergebnis nach einer späteren Regeländerung |

**Kontrollpunkt:** Eine andere Person kann die Beobachtung ohne Raten reproduzieren. Schreibe «aus der DMZ erreichbar», wenn du genau das gemessen hast. Behaupte keine erfolgreichen Standardpasswörter; diese Übung probiert keine aus. Schliesse anschliessend das Terminal.

## Deine Zugriffsentscheidung

Unterscheide für diesen Diagnoseauftrag WebVisu TCP 8080 von SSH 22, Modbus 502 und OPC UA 4840. Notiere, was erreichbar ist und was der Auftrag tatsächlich benötigt. Empfiehl Einschränkungen nach Quelle, Ziel und Dienst. Halte fest, wer den temporären Zugriff verantwortet und wann er abläuft. Anwendungsberechtigungen, Identitätsprüfung und MFA des Fernzugangs brauchen eigene Belege; dieser Scan prüft sie nicht.

## Lösung

??? success "Vergleiche deinen Befund"
    Der DMZ-Jumphost routet über `2.2.2.1` ins flache OT-Netz. Die vier geprüften Ziele antworten. WAGO A akzeptiert im Ausgangszustand TCP auf 22, 502, 4840 und 8080. Die HTTP-Antwort bestätigt eine Anwendung auf 8080.

    Ein tragfähiger Befund lautet: «Die DMZ-Quelle kann diese OT-Dienste erreichen; notwendige Verbindungen bestätigen und unnötige Verbindungen unterbinden.» Er lautet nicht «alle SPS sind verwundbar». `open`, `closed` und `filtered` beschreiben die Sicht dieses Scans, keine Sicherheitsbewertung.

## Extra 1: Zwei Ausgangspunkte vergleichen

**Optional, 10 Minuten.** Wiederhole zuerst auf dem **Jumphost**, danach im [WAGO-B-Terminal](/network/terminal?node=wago-plc2b-vlan10):

```bash
ip route get 10.10.0.11
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
```

Erstelle zwei Tabellenzeilen mit Quelle, nächstem Hop und Portzuständen.

**Kontrollpunkt:** Beide Ports sind im Ausgangszustand von beiden Quellen erreichbar. WAGO B erreicht A direkt über `eth1`, der Jumphost über ein Gateway. Gleiche Portzustände bedeuten weder gleiche Wege noch, dass eine Gateway-Firewall den direkten VLAN-Verkehr filtern könnte. Schliesse das Terminal von B.

## Extra 2: Offen und geschlossen mit Paketen beweisen

**Optional, 10 Minuten.** [Zeichne den Datenport von WAGO A auf](/network/capture?node=wago-plc2a-vlan10&interface=eth1), wähle **Start Capture** und führe auf dem **Jumphost** aus:

```bash
nmap -n -sT -Pn -p8080,65000 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
```

Warte einige Sekunden und filtere in Wireshark:

```wireshark
ip.addr == 2.2.2.2 && (tcp.port == 8080 || tcp.port == 65000)
```

Finde für jeden der beiden Ports das erste SYN-Paket und die Antwort des Ziels. Prüfe **TCP → Flags**.

**Kontrollpunkt:** Im Ausgangszustand antwortet 8080 mit SYN/ACK und 65000 mit RST/ACK, passend zu `open` und `closed`. Wird 65000 später genutzt, notiere das tatsächliche Ergebnis. Für den Zustand `filtered` sind weitere Belege nötig: Eine ausbleibende Antwort lässt sich keiner bestimmten Firewall-Regel zuordnen. Schliesse den Mitschnitt und die Terminals. Weiter mit [Übung 6](ex6-protocol-discovery.md).

## Weiterforschen

- [Nmaps Portzustände](https://nmap.org/book/man-port-scanning-basics.html): Vergleiche `closed` und `filtered`.
- [Nmap Host Discovery](https://nmap.org/book/host-discovery-controls.html): Suche `-sn` und `-Pn`.
- [Nmap TCP Connect Scan](https://nmap.org/book/man-port-scanning-techniques.html): Suche **TCP connect scan**.
