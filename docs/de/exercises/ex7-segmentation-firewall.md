# Übung 7: Eine Policy entwerfen, die den Verkehr erreicht

**Zeit:** 35–45 Minuten. **Lab:** aktuelles studentisches `ot-sec-flat`. **Niveau:** Fortgeschritten.

Weise zuerst den Weiterleitungsweg nach und entscheide dann über den Filterort. Erstelle aus deinen Belegen eine kleine, begründete Kommunikationsmatrix. Diese Übung untersucht das laufende Lab und entwirft eine Policy; sie verändert keine Firewall-Regeln der gemeinsam genutzten Appliance.

**Du lernst:** geswitchte und geroutete Wege zu unterscheiden, eine gerichtete Dienst-Policy zu erstellen, zustandsbehaftete Antworten und Regelreihenfolge zu erklären und Belege für einen späteren Durchsetzungstest festzulegen.

## Vorbereitung

Verwende deine Beobachtungen aus [Übung 5](ex5-reconnaissance.md) und [Übung 6](ex6-protocol-discovery.md). Öffne Terminals für [ABB](/network/terminal?node=abb-800xa-vlan40), den [DMZ-Jumphost](/network/terminal?node=proxmox-jumphost) und [gw](/network/terminal?node=gw-firewall). Öffne [VLAN 10 in Netmap](/network/?viewMode=vlan&vlanIds=10).

**Eigenständige Dokumentation:** Öffne dein zugewiesenes LNE, wähle studentisches Netz und Live-Snapshot und dann Explorer → Netmap → VLAN-Ansicht → VLAN 10. Suche die obigen stabilen Knotennamen und verwende deren konfigurierte SSH-Aktionen. Öffne Toolbox im Produktmenü. Reader-Links übernehmen Netz und Snapshot automatisch.

Das OT-Ausgangsnetz ist `10.0.0.0/8` in VLAN 10, die DMZ `2.2.2.0/24` in VLAN 60. Hostnamen erzeugen keine VLANs 20–50. Halte deinen Entwurf in eigenen Notizen fest: Der gespeicherte LNE-Workspace in Toolbox ist mit anderen Studierenden geteilt.

## Aufgabe 1: Den tatsächlichen Kontrollpunkt finden

Auf **ABB**:

```bash
ip route get 10.10.0.11
```

Auf dem **Jumphost**:

```bash
ip route get 10.10.0.11
```

Auf **gw**:

```routeros
/interface vlan print
/ip address print
```

**Kontrollpunkt:** ABB erreicht WAGO A direkt über `eth1`, der Jumphost über `2.2.2.1`. ABB `10.40.0.11/8` und WAGO A `10.10.0.11/8` sind im selben OT-VLAN. Eine Forward-Regel am Gateway kann den rein geswitchten Verkehr nicht einschränken. An das Gateway selbst gerichteter Verkehr verwendet `input`, gerouteter Transitverkehr `forward`.

## Aufgabe 2: Ausgangsmatrix und Zielmatrix trennen

Öffne **Toolbox** im Produktmenü. Wähle in der Karte **Network segmentation / Netzwerksegmentierung** den Schritt **Policies**, um die Kommunikationsmatrix zu öffnen. Untersuche die vorhandenen Zonen des gewählten Netzes und die Richtung der Policies. Kehre im ursprünglichen Tab zu dieser Anleitung zurück.

Der ursprüngliche **Student Lab Flat Starting Project** enthält OT Network und DMZ mit Allow All in beiden Richtungen. Ein gespeicherter Entwurf kann bereits verändert sein. Notiere den tatsächlichen Zustand. Leere, importiere, ändere oder speichere den geteilten Entwurf bei dieser Beobachtungsaufgabe nicht.

Entwirf diese **Ziel-Policy** in deinen eigenen Notizen und passe die Begründungen an deine Beobachtungen an:

| Initiator | Ziel | Dienst | Zielaktion | Begründung / Beleg |
| --- | --- | --- | --- | --- |
| DMZ-Jumphost | WAGO A | TCP 8080 | Erlauben | WebVisu-Bedienerzugriff, falls genehmigt |
| DMZ-Jumphost | WAGO A | TCP 22 | Neue Initiierung verweigern | Kein Wartungsbedarf belegt |
| DMZ-Jumphost | WAGO A | TCP 502 | Neue Initiierung verweigern | Kein Modbus-Client in der DMZ erforderlich |
| ABB/HMI | WAGO A | TCP 502 | Nach Schaffung eines Kontrollpunkts erlauben | Beobachteter Anwendungsfluss; benötigte Lese-/Schreibfunktionen bestätigen |
| PLCs | Simuliertes externes Netz | Beliebig | Neue Initiierung ohne Begründung verweigern | Ausnahme benötigt Verantwortliche und Zweck |

**Kontrollpunkt:** Jede Freigabe hat einen Geschäftsgrund, Initiator und Ziel. Die Gegenrichtung beschreibt eine neue Verbindung der anderen Seite, nicht die Antwort einer erlaubten zustandsbehafteten Verbindung. DNS und Zeitsynchronisation benötigen eigene begründete Abhängigkeiten.

## Aufgabe 3: Regeln in tatsächlicher Reihenfolge lesen

Auf **gw**:

```routeros
/ip firewall filter print detail where chain=forward
/ip firewall filter print stats where chain=forward
/ip firewall address-list print where list=mgmt
```

Finde Established/Related-Accept, die breite DMZ-zu-VLAN10-Freigabe, Private-Source-Accept und den abschliessenden Drop. Identifiziere Regeln anhand von Kommentar und Bedingungen; Zeilennummern können wechseln.

**Kontrollpunkt:** Der abschliessende Drop ist aktiv, frühere Accepts erlauben aber den gemessenen Zugriff. Die Liste `mgmt` enthält vollständig `10.0.0.0/8`, `172.16.0.0/12` und `192.168.0.0/16`. Ihr Name bedeutet nicht, dass sie nur Management-Hosts enthält. Frühere FIN-/RST-Regeln mit `action=log` beobachten Flags und setzen die Verarbeitung fort; sie sind keine Accept-Entscheide.

??? info "Hinweis: Ein Deny am Ende hilft nicht"
    Verfolge das erste Paket einer neuen Jumphost-zu-WAGO-TCP-Verbindung. Finde den ersten passenden abschliessenden Entscheid. Ein späteres Deny kann einen früheren Accept nicht aufheben. Bestehende Sitzungen können über die Zustandsregel weiterlaufen; spätere Nachprüfungen benötigen neue Verbindungen.

## Aufgabe 4: Ausgangszustand prüfen und Nachtest festlegen

Wiederhole auf dem **Jumphost** nur diese begrenzte Prüfung:

```bash
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

Schreibe je eine Vorher-/Soll-Nachher-Zeile. **Aktuell** werden offenes SSH und funktionierendes HTTP erwartet. Unter deiner **Ziel-Policy** soll neues HTTP weiter funktionieren, neue DMZ-SSH-Initiierung jedoch verweigert werden. Hier wurde keine Policy angewendet. Berichte den Sollzustand nicht als gemessen.

**Kontrollpunkt:** Dein Nachtest nennt Quelle, Ziel, Protokoll, erwartetes Anwendungsergebnis, Kontrollgerät und passende Regel-/Logbelege. Geteilte Zähler enthalten Verkehr anderer Personen; eine Differenz allein ordnet kein Paket deinem Test zu. Toolbox ist ein Entwurfswerkzeug; das Speichern einer Matrix konfiguriert das Gateway nicht.

Schliesse Toolbox und Terminals.

## Lösung

??? success "Vergleiche deine Begründung"
    Der DMZ-zu-OT-Test durchquert gw und ist momentan breit erlaubt. Der ABB-zu-WAGO-Anwendungsfluss bleibt in VLAN 10. Eine tatsächliche Kontrollgrenze ist nötig, bevor eine Gateway-Policy ihn steuern kann. Verschiedene IP-Bereiche im aktuellen `/8` schaffen diese Grenze nicht.

    Ein brauchbarer Entwurf erlaubt dokumentierte Initiierungen und passende Established/Related-Antworten und verweigert unbegründete neue Flüsse dort, wo sie tatsächlich durchlaufen. Er schützt auch das Gateway selbst. Weder ein abschliessender Drop noch eine schöne Matrix beweist Durchsetzung ohne Weg-, Reihenfolge- und Neuverbindungsbelege.

## Extra 1: Bestehende Antwort und neue Verbindung unterscheiden

**Optional, 10 Minuten.** Zeichne in deinen Notizen zwei Paketgespräche für die geplante DMZ-zu-WAGO-HTTP-Freigabe:

1. Jumphost mit kurzlebigem Quellport → WAGO TCP 8080, danach Antwort von 8080 → diesen Quellport.
2. WAGO beginnt eine separate TCP-Verbindung zu Jumphost-Port 8080.

Ordne anhand der aktuellen Forward-Regeln die Regelart für jedes Paket zu. Warum muss die Antwort aus Gespräch 1 nicht auch die neue Initiierung aus Gespräch 2 erlauben? Vergleiche dies mit den beiden Toolbox-Richtungen, ohne Änderungen zu speichern.

**Kontrollpunkt:** Du unterscheidest Verbindungsrichtung, Antwortverkehr und unabhängige Initiatoren. Du erklärst auch, warum eine bereits offene Sitzung ein schlechter Test einer verschärften Policy ist. Weder Connection-Tracking-Tabelle noch Live-Regeln müssen geleert werden.

## Extra 2: Einen vollständigen neuen VLAN-Weg entwerfen

**Optional, 15–20 Minuten.** Erstelle ein Arbeitsblatt für ein späteres isoliertes Konfigurationslab. Wende es nicht auf die geteilte Appliance an.

Verwende **VLAN 70**, Client **hirschmann-edge1-vlan50**, geplante Datenadresse **172.22.70.12/24**, Gateway **172.22.70.1**. Der Hostname darf als Bezeichnung bestehen bleiben. Vergleiche auf [WAGO A](/network/terminal?node=wago-plc2a-vlan10):

```bash
ip route get 10.70.0.12
ip route get 172.22.70.12
```

**Kontrollpunkt:** `10.70.0.12` ist innerhalb WAGOs `/8` direkt, `172.22.70.12` wird über das Gateway geroutet. Deshalb benötigt das neue VLAN ein nicht überlappendes Subnetz.

Vervollständige jede Zeile, bevor du den Entwurf als durchgängig bezeichnest:

| Teil | Nötige Änderung im isolierten Lab |
| --- | --- |
| Endpunkt | Neue `/24`-Datenadresse und Gateway; explizite Routen zum alten Gateway ersetzen; Management-`eth0` erhalten |
| sw-acc1 Access | `ether5` ordnet ungetaggten Eingang PVID 70 zu; VLAN 70 enthält diesen Access-Port |
| sw-acc1 Uplink | VLAN 70 getaggt auf `bond-sw-dist`, VLAN 10 erhalten |
| sw-dist | VLAN 70 getaggt zwischen `bond-sw-acc1` und Gateway-Port `ether12` |
| Gateway | Getaggte VLAN-70-Mitgliedschaft für `ether12` **und Bridge-CPU**; VLAN-Interface auf `bridge` mit `172.22.70.1/24` |
| Firewall und Rückweg | Client → WAGO A TCP 8080 erlauben; neues TCP 22 zum selben nachweislich offenen Server verweigern. Regeln an das neue Interface binden, bestehende Antworten erhalten und Gateway-Input vor breiten Accepts schützen |
| Belege und Rückbau | Ungetaggter Access- und getaggter VLAN-70-Uplink-Mitschnitt, erlaubter Fluss, verweigerte neue Verbindung, Ausgangswerte und umgekehrte Wiederherstellungsreihenfolge |

Die aktuelle Freigabe für `172.16.0.0/12` erfasst auch das geplante Subnetz. Ein VLAN allein würde deshalb keine Isolation schaffen. WAGO A routet Antworten an `172.22.70.12` bereits über `10.10.0.1`. Mit dem DMZ-Jumphost als Server wäre zusätzlich dessen Rückroute über `2.2.2.1` nötig; seine Standardroute verwendet Management. Füge VLAN 70 nicht jedem Port hinzu und mache einzelne LACP-Mitglieder nicht zu Bridge-Ports.

**Ergebnis:** Beschreibe den Weg und das Fehlersymptom beim Weglassen jeder Zeile. Ein späterer Aufbau soll zuerst das lokale Gateway, dann einen ausdrücklich erlaubten TCP-Austausch über die Zonengrenze und anschliessend die verweigerte unerlaubte Neuverbindung beweisen und schliesslich zurückgebaut werden. Dieser Zusatz ist Entwurfsanalyse; VLAN-70-Konfiguration und -Verbindung wurden auf dem geteilten Lab nicht erprobt.

Schliesse die Werkzeug-Tabs. Du hast alle sieben Übungen abgeschlossen.

## Weiterforschen

- [MikroTik Bridge VLAN Table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): Suche PVID, Tagged Ports und CPU-Zugang.
- [RouterOS Firewall Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/): Vergleiche Input und Forward.
- [RouterOS Connection Tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/): Suche Established/Related.
- [NIST SP 800-82r3](https://csrc.nist.gov/pubs/sp/800/82/r3/final): Lies Segmentierung zusammen mit OT-Verfügbarkeitsanforderungen.
