# Übung 7: Ein geroutetes VLAN aufbauen und prüfen

**Zeitbedarf:** 90–120 Minuten einschliesslich Zonenplanung. **Lab:** dein eigenes `ot-sec-flat`-Lab. **Niveau:** angeleitete praktische Übung.

Erstelle eine neue Sicherheitszone mit einem eigenen VLAN. Konfiguriere den VLAN-Pfad über die Switches sowie Routing und gezielte Firewall-Regeln am Gateway. Weise nach, dass die Anwendung funktioniert und eine unerlaubte Verbindung blockiert wird. Stelle danach den flachen Ausgangszustand wieder her.

**Du lernst:** Access-VLAN und getaggten Weg zu konfigurieren, den CPU-Port der Gateway-Bridge einzubeziehen, ein nicht überlappendes Subnetz zu wählen, zustandsbehaftete Firewall-Regeln anzuordnen und beide Richtungen mit Paketen und Anwendungsergebnissen zu prüfen.

## Dein Änderungsauftrag

Eine temporäre Diagnosestation soll eine SPS-Visualisierung erreichen, ohne allgemeinen Zugriff auf die Zelle zu erhalten. Verwende den Linux-Endpunkt mit Hirschmann-Bezeichnung als **Testclient** und WAGO A als Visualisierungsserver. Das sind Übungsrollen; ein industrieller Switch ist nicht üblicherweise ein Bedienarbeitsplatz.

Der Verantwortliche erlaubt den Zugriff auf WebVisu und die Erreichbarkeitsprüfung des Gateways. SSH und andere neue Verbindungen zwischen den Zonen sind nicht erlaubt. Das gilt auch für neue Verbindungen zur Diagnosestation. Halte diese Vorgaben zuerst in einer Kommunikationsmatrix fest. Setze sie danach um und belege sowohl die funktionierende Anwendung als auch die Einschränkung.

Eine **Zone** gruppiert Anlagenbestandteile mit gemeinsamen Sicherheitsanforderungen. Ein **Conduit** ist ein genehmigter Kommunikationspfad zwischen Zonen. Ein VLAN trennt Ethernet-Verkehr; Routing und Firewall-Regeln kontrollieren hier den Pfad zwischen Zonen. Eine **zustandsbehaftete** Firewall verfolgt bestehende Verbindungen: Antwortpakete einer erlaubten Verbindung benötigen deshalb keine eigene Freigabe für eine neue Verbindung in Gegenrichtung.

**Wichtige Grenze:** Eine Freigabe von TCP-Port 8080 erlaubt den Zugriff auf die WebVisu-Anwendung einschliesslich ihrer Bedienelemente. Das erzwingt **keinen reinen Lesezugriff**. Betätige die Bedienelemente in dieser Übung nicht. Für die Anforderung «nur ansehen» wären zusätzlich passende Anwendungsrollen oder ein anderer Zugriffsschutz auf Anwendungsebene nötig. Dieses Praktikum prüft nur den Netzwerkzugriff.

## Vorbereitung

Verwende dein eigenes Lab nach Abschluss und Bereinigung der Übungen 1–6. Der Ausgangszustand muss zu Aufgabe 1 passen. Ein Checkpoint des flachen Ausgangszustands muss für die Wiederherstellung verfügbar sein; die [Vorbereitungsanleitung](../lab-practical.md) enthält die Befehle für die Konsole der Kursleitung.

Öffne Terminals für [Hirschmann](/network/terminal?node=hirschmann-edge1-vlan50), [WAGO A](/network/terminal?node=wago-plc2a-vlan10), [sw-acc1](/network/terminal?node=sw-acc1), [sw-dist](/network/terminal?node=sw-dist) und [gw](/network/terminal?node=gw-firewall). Lasse das Hirschmann-Terminal offen: Die Management-Verbindung verwendet `eth0`, geändert wird nur das Dateninterface `eth1`.

Bearbeite die praktische Übung in einer laufenden Sitzung. Die Linux-Befehle für Adressen und Routen ändern den laufenden Zustand des Knotens. Seine Startkonfiguration enthält weiterhin das flache Ausgangsnetz. Wird ein Knoten während des Aufbaus neu gestartet, lasse vor dem Fortsetzen den Checkpoint wiederherstellen.

**Eigenständige Dokumentation:** Öffne das zugewiesene Übungs-LNE, wähle dessen Netz und Live-Snapshot und dann Explorer → Netmap. Suche die oben genannten Knotennamen und nutze ihre konfigurierten SSH-Aktionen. Wähle für Mitschnitte Switch und Interface und für VLAN-Ansichten VLAN 10 oder 70. Links in der Übungsansicht übernehmen Netz und Snapshot. Nutze ausserhalb von LNE die Übungsnavigation der Website.

Der Name `hirschmann-edge1-vlan50` bleibt bestehen. Du verschiebst diesen Testclient von VLAN 10 in **ein neues VLAN 70**. Toolbox ist ein Entwurfswerkzeug. Das Speichern einer Matrix führt die folgenden Befehle nicht aus.

| Zustand | Hirschmann-Datenadresse | VLAN | Weg zu WAGO A | Erwarteter Zugriff |
| --- | --- | --- | --- | --- |
| Anfang | `10.50.0.12/8` | 10 | Direkt, geswitcht | HTTP und SSH offen |
| Nach Aufbau | `172.30.70.12/24` | 70 | Über `172.30.70.1` auf gw | HTTP erlaubt; neues SSH blockiert |
| Nach Bereinigung | `10.50.0.12/8` | 10 | Wieder direkt | Ursprünglicher Zugriff wiederhergestellt |

## Anlagenzonen vor der Konfiguration planen

**Entwurfsaufgabe, 20–30 Minuten.** Nutze das Wasseraufbereitungsdiagramm und die SPS-Zuordnung aus dem Kurshandout (PDF-Seiten 33–34 und 130–145). Gemeint sind gezählte PDF-Seiten; die aufgedruckten Foliennummern weichen später ab. Arbeite selbstständig oder besprich deinen Entwurf mit einer Kollegin oder einem Kollegen, bevor du dein eigenes Lab konfigurierst.

1. Gruppiere Anlagenbestandteile nach betrieblichem Zweck und Folgen von Ausfall oder Manipulation. Beginne mit Aufbereitung und Hygiene, Pumpen und Prozess, Entsorgung, Leitwarte sowie Engineering und Fernwartung. Das sind mögliche Gruppen; sie schreiben weder eine Lösung noch bestimmte Security Levels vor.
2. Prüfe die **Steuerungsgrenze**. Führe **PLC2A** und **PLC2B** als getrennte WAGO-Endpunkte in Anlagenliste und Kommunikationsmatrix. Prüfe ihre Aufbereitungsfunktionen anhand des Handouts. Schneider **PLC4** steuert den Mischer der ersten Stufe sowie Hilfs-/Entsorgungsanlagen. Die VLAN-Zuordnung der Datenschnittstelle gilt für die gesamte SPS; sie trennt deren physische Ausgänge nicht voneinander. Zeichnest du eine Firewall zwischen zwei Aktoren ein, die dieselbe SPS intern steuert, bewirkt das keine tatsächliche Trennung.
3. Erfasse notwendige Kommunikation. Verwende beobachtete Clients/Server/Funktionen aus Übung 6 als Beleg und ergänze Zweck, Initiator, Empfänger, Protokoll, Verantwortlichen und Ausfallfolge. Kennzeichne eine Beziehung aus dem Prozessbild als **zu prüfen**, bis sie beobachtet oder vom Verantwortlichen bestätigt wurde. Vorhandener Verkehr ist nicht automatisch notwendiger Verkehr.
4. Finde eine Abhängigkeit über deine vorgesehenen Zonengrenzen. Entscheide, ob du die SPS mit ihren Funktionen gemeinsam gruppierst oder einen Umbau der Steuerung vorschlägst. Dessen Eignung müsste später gesondert geprüft werden. Eine Regel für einen TCP-Port hebt keine interne Steuerungsabhängigkeit auf und kann Modbus-Lesezugriffe nicht von Schreibzugriffen unterscheiden.
5. Prüfe deinen Entwurf: Wie begrenzt du Störungen und erhältst zugleich die notwendige Steuerungskommunikation? Vergleiche bei Gelegenheit mit einer Kollegin oder einem Kollegen. Notiere ein verbleibendes Risiko und den zuständigen Anlagenverantwortlichen. Diese Übung weist keine IEC-62443-Security-Levels zu und zertifiziert sie nicht.

**Ergebnis:** Eine Zonenskizze und mindestens vier Einträge in der Kommunikationsmatrix mit jeweils angegebener Richtung, darunter Leitsystemkommunikation, Engineering-Zugriff, Lieferantenzugriff und lokale Geräteerkennung. Nutze eigene Notizen oder den Toolbox-Zonen-/Kommunikationsplaner in deinem Lab. In LNE: Produktmenü → **Toolbox** → **Policies**. Das Speichern eines Entwurfs ändert keine Gerätekonfiguration.

??? success "Begründung prüfen"
    Eine gute Antwort berücksichtigt die reale SPS-Zuordnung, erhält begründete Steuerungsbeziehungen und begrenzt Wartungszugriffe. Sie stellt UV-/Chlorbehandlung, Mischer oder pH-Regelung nicht als getrennte VLAN-Zonen dar, solange dieselbe WAGO-SPS diese Funktionen steuert. Sie berücksichtigt auch, dass die Schneider-SPS sowohl Prozess- als auch Entsorgungsaufgaben übernimmt. Mehrere Zonenentwürfe sind vertretbar, wenn Abhängigkeiten, Grenzen und Restrisiken ausdrücklich benannt sind.

Im praktischen Teil setzt du **einen einzelnen, überprüfbaren Conduit** für eine temporäre Diagnosestation um. Der vollständige Zonenentwurf bleibt eine Planungsaufgabe. Die Prozesssteuerungen bleiben an ihrem Platz; so übst du Routing und Regeldurchsetzung, ohne eine laufende Steuerungsabhängigkeit zu verschieben.

| Initiator | Empfänger | Dienst | Gewünschtes Ergebnis und Grund |
| --- | --- | --- | --- |
| Diagnoseclient | WAGO A | TCP 8080 | Genehmigte WebVisu-Anwendung erlauben |
| Diagnoseclient | WAGO A | TCP 22 | Nicht genehmigten SSH-Aufbau sperren |
| WAGO A | Diagnoseclient | Neue Verbindung zu TCP 22 | Nicht genehmigten umgekehrten Verbindungsaufbau sperren |
| Diagnoseclient | Sein Gateway | ICMP | Vorgesehene Erreichbarkeitsdiagnose erlauben |
| Diagnoseclient | Sein Gateway | TCP 22 | Gateway-Administration sperren |
| Teilnehmer einer erlaubten Verbindung | Gegenseite | Established/related | Zugeordnete Antworten erlauben |

Halte fest, wer den vorgeschlagenen WebVisu-Zugriff verantwortet und dass er mit dem Ende der Wartung ablaufen soll. Im Praktikum entfernst du ihn bei der Bereinigung. Sage vor Aufgabe 2 für jede Zeile die erste passende Firewall-Regel voraus.

## Aufgabe 1: Ausgangszustand festhalten

Auf **Hirschmann**:

```bash
ip -br address
ip route
ip route get 10.10.0.11
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Weise auf **WAGO A** vor dem Einrichten der Firewall-Regeln nach, dass Hirschmanns SSH-Dienst offen ist:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 10.50.0.12
ip route get 172.30.70.12
```

Auf **sw-acc1**:

```routeros
/interface bridge port print detail where interface=ether5
/interface bridge vlan print detail
```

Auf **sw-dist**:

```routeros
/interface bridge vlan print detail
```

Auf **gw**:

```routeros
/interface vlan print
/interface bridge vlan print detail
/ip address print
/ip firewall filter print detail
/ip firewall address-list print where list=mgmt
:put [:len [/ip firewall filter find where chain=forward and comment~"^allow HMI"]]
:put [:len [/ip firewall filter find where chain=input and comment~"^allow DHCP"]]
```

**Kontrollpunkt:** Hirschmann hat die Datenadresse `10.50.0.12/8` auf `eth1`, die Management-Adresse `192.168.100.59/24` auf `eth0` sowie eine Standardroute und explizite Routen für `1.0.0.0/8` und `2.2.2.0/24` über `10.10.0.1`. WAGO wird direkt erreicht; 22/8080 sind offen und HTTP liefert 200. WAGO erreicht Hirschmanns SSH und routet `172.30.70.12` über `10.10.0.1`.

VLAN 70, das Interface `vlan70`, das Subnetz `172.30.70.0/24` und Kommentare mit `ex7-` dürfen noch nicht verwendet werden. Die beiden letzten Gateway-Abfragen müssen jeweils `1` ausgeben: Sie identifizieren die Einfügepunkte für Aufgabe 2. sw-acc1 ether5 hat PVID 10 und gehört dynamisch zu VLAN 10. Die statische VLAN-10-Zeile enthält `bond-sw-dist,ether10`; sw-dist führt VLAN 10 über `bond-sw-acc1,ether12,ether8`. Weichen diese Anfangswerte ab, stoppe und lasse den zugewiesenen flachen Checkpoint wiederherstellen.

**Umzusetzende Zugriffsregeln:** Hirschmann → WAGO A TCP 8080 und Hirschmann → neues Gateway ICMP erlauben; die vorhandene Freigabe für bestehende und zugehörige Verbindungen (Established/Related) beibehalten; andere neue weitergeleitete Verbindungen von/zu VLAN 70 und anderen Gateway-Zugriff aus VLAN 70 verweigern. Die vorhandene Private-Source-Liste enthält `172.16.0.0/12`. Die zugehörige Freigaberegel würde dem neuen Subnetz umfassenden Zugriff gewähren, wenn deine enger gefassten Regeln nicht vorher greifen.

## Aufgabe 2: Gateway und Firewall-Regeln vorbereiten

Erstelle auf **gw** zuerst das Interface, damit du es in den Firewall-Regeln angeben kannst:

```routeros
/interface vlan add name=vlan70 interface=bridge vlan-id=70 comment=ex7-vlan70
```

Füge die Regeln in dieser Reihenfolge hinzu. Die `place-before`-Ausdrücke identifizieren anhand ihrer Ausgangskommentare die ersten Forward-/Input-Regeln nach der Zustandsfreigabe. Die vorhandenen Freigaberegeln für Established/Related bleiben vor deinen Regeln. Die umfassenderen Freigaben für ICMP, Management und private Quelladressen stehen dahinter.

```routeros
/ip firewall filter add chain=forward action=accept in-interface=vlan70 out-interface=vlan10 src-address=172.30.70.12 dst-address=10.10.0.11 protocol=tcp dst-port=8080 log=yes log-prefix="EX7-HTTP " comment=ex7-http place-before=[find where chain=forward and comment~"^allow HMI"]
/ip firewall filter add chain=forward action=drop in-interface=vlan70 log=yes log-prefix="EX7-FROM " comment=ex7-deny-from place-before=[find where chain=forward and comment~"^allow HMI"]
/ip firewall filter add chain=forward action=drop out-interface=vlan70 log=yes log-prefix="EX7-TO " comment=ex7-deny-to place-before=[find where chain=forward and comment~"^allow HMI"]
/ip firewall filter add chain=input action=accept in-interface=vlan70 src-address=172.30.70.12 dst-address=172.30.70.1 protocol=icmp comment=ex7-gateway-ping place-before=[find where chain=input and comment~"^allow DHCP"]
/ip firewall filter add chain=input action=drop in-interface=vlan70 log=yes log-prefix="EX7-INPUT " comment=ex7-deny-input place-before=[find where chain=input and comment~"^allow DHCP"]
/interface bridge vlan add bridge=bridge vlan-ids=70 tagged=bridge,ether12 comment=ex7-vlan70
/ip address add address=172.30.70.1/24 interface=vlan70 comment=ex7-vlan70
/ip firewall filter print detail
```

**Kontrollpunkt:** Forward-Reihenfolge: Established/Related → `ex7-http` → `ex7-deny-from` → `ex7-deny-to` → bisherige Regeln. Input-Reihenfolge: Established/Related → `ex7-gateway-ping` → `ex7-deny-input` → bisherige Regeln. Die neue VLAN-Tabellenzeile enthält **den CPU-Port der Bridge (`bridge`) und ether12**. Mit ether12 allein fehlt in dieser Tabellenzeile der CPU-Zugang für das Routing.

Neue Regelzeilen können beim Start des Interfaces kurz das Flag `I` (invalid, ungültig) zeigen. Warte einige Sekunden und wiederhole den letzten Print-Befehl. Keine `ex7-`-Zeile darf vor dem Fortfahren noch das Flag `I` tragen. Das `I` in der allgemeinen Flag-Legende darf weiterhin sichtbar sein; entscheidend sind die einzelnen Regelzeilen.

## Aufgabe 3: VLAN 70 über beide Switches transportieren

Auf **sw-dist**:

```routeros
/interface bridge vlan add bridge=bridge vlan-ids=70 tagged=bond-sw-acc1,ether12 comment=ex7-vlan70
/interface bridge vlan print detail where vlan-ids=70
```

Auf **sw-acc1**:

```routeros
/interface bridge vlan add bridge=bridge vlan-ids=70 tagged=bond-sw-dist untagged=ether5 comment=ex7-vlan70
/interface bridge port set [find where interface=ether5] pvid=70
/interface bridge vlan print detail
```

**Kontrollpunkt:** Der neue Weg führt von Hirschmann ungetaggt zu sw-acc1 ether5, mit VLAN-Tag über `bond-sw-dist` und sw-dist `bond-sw-acc1` zu ether12 und schliesslich zum CPU-Port und zum Interface `vlan70` auf gw. Die PVID-Änderung entfernt ether5 aus der dynamisch abgeleiteten VLAN-10-Mitgliedschaft. Vorhandene getaggte VLAN-10-Zeilen bleiben erhalten. Der Bond bleibt ein logischer Bridge-Port.

## Aufgabe 4: Endpunkt ins neue Subnetz verschieben

Entferne auf **Hirschmann** die Routen zum alten Gateway, bevor du die Datenadresse des Endpunkts ersetzt. Die neue Standardroute deckt auch die Ziele der beiden bisher expliziten Routen ab.

```bash
ip route del 1.0.0.0/8 via 10.10.0.1 dev eth1
ip route del 2.2.2.0/24 via 10.10.0.1 dev eth1
ip route del default via 10.10.0.1 dev eth1
ip address add 172.30.70.12/24 dev eth1
ip address del 10.50.0.12/8 dev eth1
ip route add default via 172.30.70.1 dev eth1
ip -br address
ip route get 10.10.0.11
ping -c 4 172.30.70.1
```

**Kontrollpunkt:** `eth0` behält die Management-Adresse und dein Terminal bleibt verbunden. `eth1` hat nur die neue Datenadresse, seine Routen für `224.0.0.0/4` und `255.255.255.255` bleiben erhalten, die WAGO-Route verwendet `172.30.70.1` und das Gateway antwortet. Die neue Adresse wird zuerst ergänzt, damit das Entfernen der alten keine anderen Interface-Routen löscht. `10.70.0.0/24` würde mit den bisherigen OT-`/8`-Rückrouten überlappen; `172.30.70.0/24` vermeidet dies.

## Aufgabe 5: Erlaubtes HTTP und blockiertes SSH beweisen

[Öffne den Gateway-Uplink-Mitschnitt](/network/capture?node=sw-dist&interface=ether12) und wähle **Start Capture**.

Zeigt der Viewer ERF/LEGACY statt Ethernet, wähle vor dem Testverkehr einmal **Restart Capture**. Melde einen wiederkehrenden Fehler an die Kursleitung.

Führe auf **Hirschmann** aus:

```bash
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Auf **gw**:

```routeros
/ip firewall filter print stats where comment~"^ex7-"
/log print where message~"EX7-"
```

Filtere den Mitschnitt mit `ip.addr == 172.30.70.12 && (tcp.port == 22 || tcp.port == 8080)`. Vergleiche Quell- und Zieladresse sowie die Ports mit den Log-Einträgen deiner Firewall-Regeln. Logs und Zähler können auch frühere Versuche oder Hintergrundverkehr in deinem Lab enthalten.

Verwende die neuesten Einträge aus deinem Testzeitraum. Frühere `EX7-`-Einträge können nach einer vorherigen Bereinigung erhalten bleiben.

**Kontrollpunkt:** TCP 8080 ist offen und HTTP liefert 200. TCP 22 ist **filtered**, obwohl der SSH-Port vorher nachweislich offen war. Dein SSH-SYN ist in VLAN 70 sichtbar und `ex7-deny-from` protokolliert das Verwerfen des Pakets. Erfolgreiches HTTP und Gateway-Ping belegen einen funktionierenden Weg. Ein Timeout allein identifiziert keine Firewall-Regel.

Öffne **Data Collector → Devices → Collect new data**. Warte auf das Ende der Erfassung und aktualisierte **Last poll**-Werte. [Untersuche dann VLAN 70 in Netmap](/network/?viewMode=vlan&vlanIds=70). Hirschmann und der Weg über sw-acc1, sw-dist und gw sollen nun VLAN 70 führen. Vergleiche gemeldete Mitgliedschaft, Konfiguration und Pakete. Ein Neuladen von Netmap allein fragt die Switches nicht neu ab.

Die aktuelle VLAN-Ansicht kann die zusammengefasste Bond-Kante zwischen sw-dist und sw-acc1 grau lassen, obwohl beide Switches VLAN 70 melden. Prüfe diesen Abschnitt mit der VLAN-Tabelle des Bonds und getaggten Paketen. Die Kantenfarbe beweist nicht, ob Verkehr fliesst.

**Die Hauptaufgabe ist abgeschlossen.** Lasse VLAN 70 für die optionalen Zusatzaufgaben konfiguriert. Führe danach in jedem Fall die **Bereinigung** durch. Stoppe den Mitschnitt vor einem Zielwechsel.

Wenn du vor den Extras pausierst, führe zuerst die Bereinigung durch. Prüfe beim Fortsetzen den flachen Ausgangszustand mit Aufgabe 1 und wiederhole Aufgaben 2–5 vor den Extras. Bewahre deine bisherigen Ergebnisse zum Vergleichen auf.

## Dein Abnahmeprotokoll

Notiere die genehmigte Quelle, das Ziel und den Dienst. Halte das HTTP-Ergebnis und eine abgewiesene SSH-Anfrage mit passendem Firewall-Log fest. Dokumentiere nach der Bereinigung auch den wiederhergestellten Ausgangszustand. Ergänze Gegenrichtungs- und Gateway-Prüfungen aus Extra 2, falls durchgeführt. Beschränke deine Aussage auf die tatsächlich geprüften Datenflüsse. Die HTTP-Freigabe allein erzwingt keinen reinen Lesezugriff auf die Visualisierung.

## Lösung

??? success "Vergleiche deine Umsetzung"
    Für VLAN 70 brauchst du PVID 70 am ungetaggten Client-Port sowie eine getaggte VLAN-Mitgliedschaft auf beiden Switch-Uplinks, am Gateway-Port ether12 und am CPU-Port der Gateway-Bridge. Hinzu kommen das VLAN-Interface, ein nicht überlappendes Subnetz und eine funktionierende Rückroute. WAGO A sendet Antworten zum neuen Subnetz über sein bestehendes Gateway `10.10.0.1`.

    Der HTTP-Verbindungsaufbau wird durch `ex7-http` erlaubt. Antworten passieren die davorstehende Freigaberegel für Established/Related. Neue SSH-Verbindungen werden durch `ex7-deny-from` verworfen, bevor die umfassendere Freigabe für private Quelladressen greifen kann. Die Input-Regeln erlauben die vorgesehene Gateway-Diagnose und blockieren andere Anfragen aus VLAN 70 an gw. Bestehender OT-zu-OT-Verkehr bleibt in VLAN 10.

## Extra 1: Eine Anfrage vor und nach dem Router verfolgen

**Optional, 10 Minuten.** Lasse den Gateway-Uplink-Mitschnitt laufen und sende eine weitere HTTP-Anfrage von Hirschmann. Filtere `ip.src == 172.30.70.12 && tcp.dstport == 8080`.

Finde dieselbe Anfrage vor und nach dem Routing: Vergleiche TCP-Sequenznummer/Nutzlast, VLAN-ID, Ethernet-Adressen, IP-Adressen und TTL.

**Kontrollpunkt:** Auf diesem einen Uplink kommt die Anfrage mit VLAN 70 an und geht mit VLAN 10 hinaus. Quell- und Ziel-IP-Adresse bleiben gleich, Ethernet-Adressen ändern sich und die TTL sinkt um eins. Das ist Routing ohne Source NAT. Ein erfolgreicher Ping allein würde diese Änderungen nicht zeigen.

Beim Mitschnitt beider Seiten kann Wireshark die zweite Kopie als Duplicate/Out-of-Order markieren. Vergleiche Sequenznummer, VLAN und TTL, bevor du diese Markierung als Netzwerkfehler deutest.

## Extra 2: Antworten und neue Verbindungsaufnahmen unterscheiden

**Optional, 10–15 Minuten.** Prüfe auf **WAGO A** eine neue SSH-Verbindung in Gegenrichtung zum in Aufgabe 1 nachgewiesenen Listener:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.12
```

Prüfe auf **Hirschmann** den Gateway-Schutz und wiederhole danach die erlaubte HTTP-Anfrage:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.1
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Lies nochmals die Zähler und Log-Einträge der `ex7-`-Regeln. Die neue SSH-Verbindung von WAGO A zur Diagnosestation muss durch `ex7-deny-to` blockiert werden, die SSH-Verbindung zum Gateway durch `ex7-deny-input`. Die erlaubte HTTP-Verbindung muss weiterhin Status 200 liefern. Bei beiden SSH-Prüfungen muss Nmap den Zustand `filtered` melden. Bestehende Sitzungen oder globale Zähler müssen nicht geleert werden.

## Bereinigung: Flachen Ausgangszustand wiederherstellen

Stoppe und schliesse den Mitschnitt. Auf **Hirschmann**:

```bash
ip route del default via 172.30.70.1 dev eth1
ip address add 10.50.0.12/8 dev eth1
ip address del 172.30.70.12/24 dev eth1
ip route add default via 10.10.0.1 dev eth1
ip route add 1.0.0.0/8 via 10.10.0.1 dev eth1
ip route add 2.2.2.0/24 via 10.10.0.1 dev eth1
```

Auf **sw-acc1**:

```routeros
/interface bridge port set [find where interface=ether5] pvid=10
/interface bridge vlan remove [find where comment=ex7-vlan70]
```

Auf **sw-dist**:

```routeros
/interface bridge vlan remove [find where comment=ex7-vlan70]
```

Auf **gw**:

```routeros
/ip address remove [find where comment=ex7-vlan70]
/interface bridge vlan remove [find where comment=ex7-vlan70]
/interface vlan remove [find where name=vlan70]
/ip firewall filter remove [find where comment~"^ex7-"]
```

Wiederhole die Adress-, Routen-, Dienst- und Switch-Prüfungen aus Aufgabe 1. Hirschmann muss wieder `10.50.0.12/8`, PVID 10 und einen direkten WAGO-Weg verwenden; HTTP und SSH müssen wieder offen sein. WAGO muss Hirschmanns SSH wieder unter `10.50.0.12` erreichen. VLAN 70, das Interface `vlan70` und die `ex7-`-Regeln müssen entfernt sein. Die ursprünglichen Mitgliedschaften in VLAN 10 und 60 sowie die Management-Verbindung müssen deinen Notizen entsprechen.

Führe **Collect new data** erneut aus, um gemeldete Port-Mitgliedschaften zu aktualisieren. Prüfe das Entfernen in den tatsächlichen RouterOS-Tabellen. Eine VLAN-Nummer im Katalog allein beweist nicht, dass ein Port sie weiterhin führt.

Weicht die Bereinigung vom protokollierten Ausgangszustand ab, verwende mit der Kursleitung das gespeicherte Checkpoint-Verfahren und wiederhole Aufgabe 1. Schliesse nach der Prüfung alle Terminal-Tabs.

## Zusatzübung: Ein Altsystem mit der Diode mikrosegmentieren

**Optional, 35–45 Minuten.** Verwende dein eigenes Lab mit der gerouteten `vlan10-diode` und dem Endpunkt `protected-linux`. Beginne im flachen Ausgangszustand; schliesse die Bereinigung von VLAN 70 oben ab, falls du es aufgebaut hast. Die beiden anderen Zusatzaufgaben sind keine Voraussetzung.

**Dein Auftrag:** Im Entsorgungsbeispiel des Handouts gefährdet ein ungepatchtes System den Rest seiner Zone (PDF-Seiten 149–152, aufgedruckte Folien 158–161). Hier steht `protected-linux` für einen älteren Rechner in diesem Anlagenteil. Erhalte den Wartungszugriff vom Dell-Engineering-Arbeitsplatz, blockiere einen anderen Zellenrechner und begrenze neue Verbindungen, die der ältere Rechner selbst aufbaut.

**Du lernst:** eine Grenze um einen einzelnen Endpunkt durchzusetzen, eine Ausnahme für genau eine Adresse zu wählen, beide Richtungen zu prüfen und funktionierende Antworten einer erlaubten Anwendung nachzuweisen.

Das Lab verwendet eine **geroutete Diode** mit konfigurierbarer Firewall-Policy. Sie ist keine physische Einweg-Datendiode. Ihr Profil filtert Adressen und Subnetze statt einzelner Dienstports. Linux ist der Stellvertreter für diese Aufgabe; die Diode patcht ihn nicht und weist ihm kein IEC-62443-Security-Level zu.

### A1: Die Grenze in der GUI finden

1. Öffne Produktmenü → **Diodes**, danach `vlan10-diode`. Notiere das zugewiesene Profil **Microsegmentation - baseline**. Warte vor den Ausgangsmessungen auf **Production current** und einen Zeitpunkt bei **Last confirmed** unter **Configuration Apply Status**. Notiere **Config Revision** und vergleiche mit **ASSIGNED REV**.
2. Untersuche die Schnittstellen der Diode unter **Inventory**. Verfolge in **Netmap** den Weg von sw-acc2 über die Diode zu Protected Linux. Die gepunktete Switch-Verbindung ist eine gespeicherte Verkabelungsnotiz. Schnittstellen- und ARP-Meldungen liefern weitere Belege; eine Linie im Diagramm beweist allein keine Filterung.
3. Unterscheide die Seiten: WAN `10.10.0.21` zeigt zu VLAN 10; LAN `192.168.254.254/24` zu Protected Linux `192.168.254.1`. Die beiden folgenden Clients bleiben in VLAN 10. Sie verbinden sich mit **`10.10.0.21`**; die Diode leitet erlaubten Verkehr an Linux weiter.

| Rolle | Terminal | Datenadresse im Ausgangszustand |
| --- | --- | --- |
| Berechtigter Engineering-Arbeitsplatz | [Dell](/network/terminal?node=dell-ews1-vlan50) | `10.50.0.11/8` |
| Anderer Zellenrechner | [Hirschmann](/network/terminal?node=hirschmann-edge1-vlan50) | `10.50.0.12/8` |
| Stellvertreter für das Altsystem | [Protected Linux](/network/terminal?node=protected-linux) | `192.168.254.1/24` |

Verwende durchgehend diese eingerichteten Management-Terminals. Ein offenes Management-Terminal ermöglicht die Wiederherstellung; es beweist keine Freigabe der Datenverbindung durch die Diode. Teste die Policy nicht über Management-Adressen.

### A2: Die Anwendungskommunikation im Ausgangszustand prüfen

Notiere auf **Dell**, danach auf **Hirschmann**, die tatsächliche Adresse und die ausgewählte Route:

```bash
ip -4 -br address show dev eth1
ip route get 10.10.0.21
```

Führe auf **beiden Clients** aus:

```bash
curl --interface eth1 --connect-timeout 3 --max-time 5 -sS http://10.10.0.21/
nc -vz -w 3 10.10.0.21 22
```

Beide müssen die Seite **Diode protected Linux endpoint** erhalten und die TCP-Verbindung zu Port 22 aufbauen können. `nc` prüft die Verbindung; es meldet keinen SSH-Benutzer an.

Prüfe auf **Protected Linux** eine ausgehende Anwendungsverbindung zu WAGO A:

```bash
curl --interface eth1 --connect-timeout 3 --max-time 5 -sS -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

**Kontrollpunkt:** HTTP liefert `200`. Notiere unter **Diodes → Profiles → Microsegmentation - baseline** beide Richtungen: **From Network** und **From Endpoint** stehen auf **Default allow**, jeweils ohne Adressausnahmen. Erhalte die anderen Profileinstellungen. Weicht der Ausgangszustand ab, gleiche zuerst deine frühere Arbeit ab; ein fehlender Dienst kann keine neu wirksame Sperre belegen.

### A3: Nur den Engineering-Arbeitsplatz zulassen

1. Öffne das zugewiesene Profil und wähle **Edit**.
2. Wähle unter **From Network** die Einstellung **Default deny**. Wähle **Add allowed source**, trage Dells beobachtete Datenadresse ein und wähle **/32**. Im Ausgangszustand lautet der Eintrag `10.50.0.11/32`; `/32` bedeutet genau eine IPv4-Adresse. Lass **From Endpoint** zunächst auf **Default allow**.
3. Wähle **Save** und bestätige **Save & apply**. Kehre zu den Details der Diode zurück und warte auf die bestätigte beziehungsweise synchronisierte Konfiguration. Ein gespeichertes Profil beweist noch nicht, dass das Gerät es angewendet hat.
4. Sage die Ergebnisse beider Clients voraus und wiederhole danach die HTTP- und TCP-22-Befehle aus A2 mit neuen Verbindungen.

**Kontrollpunkt:** Dell erreicht HTTP und TCP 22 weiterhin. Hirschmann erreicht beide nicht mehr. Die Clients benötigten keine unterschiedlichen VLANs; ihre Zugriffe unterscheiden sich, weil die Diode die Policy an der Grenze des geschützten Endpunkts durchsetzt. Eine fehlgeschlagene Anfrage allein reicht nicht: Vergleiche mit dem erfolgreichen berechtigten Client und der bestätigten Konfiguration.

Warum nicht `10.50.0.0/24`? Dieses Subnetz enthält beide Clients und würde auch Hirschmann zulassen. Warum ist dies keine Benutzerfreigabe? Die IP-Adresse kennzeichnet eine Quelle für die Regel; sie authentifiziert keine Person und macht einen kompromittierten zugelassenen Arbeitsplatz nicht vertrauenswürdig. HTTP und SSH folgen derselben Quelladressregel; dieses Profil kann für dieselbe Quelle nicht «HTTP erlaubt, SSH gesperrt» ausdrücken.

### A4: Neue Verbindungen des Altsystems begrenzen

Das Risiko aus der Folie besteht auch in Gegenrichtung: Ein ungepatchter Rechner könnte selbst Verbindungen zum Rest der Anlage aufbauen.

1. Bearbeite das Profil erneut. Lass **From Network** auf **Default deny** mit Dells `/32`-Ausnahme. Setze **From Endpoint** auf **Default deny**, ohne erlaubte Ziele. Speichere, bestätige **Save & apply** und warte auf die Konfigurationsbestätigung.
2. Wiederhole auf **Protected Linux** den ausgehenden HTTP-Befehl zu WAGO. Wiederhole danach Dells HTTP- und TCP-22-Prüfungen.

**Kontrollpunkt:** Die neue Anfrage von Protected Linux zu WAGO scheitert. Neue Anfragen von Dell zu Protected Linux und deren Antworten funktionieren weiterhin. Damit unterscheidest du neue Verbindungen des Endpunkts von Antworten innerhalb einer erlaubten eingehenden Verbindung. Daraus folgt nicht, dass die Hardware Pakete nur in einer Richtung durchlässt.

Entferne für eine abschliessende Isolationsprüfung Dells **allowed source**. Lass beide Richtungen auf **Default deny**. Wende die Änderung an und warte auf die Bestätigung. Neue HTTP- und TCP-22-Prüfungen beider Clients müssen nun scheitern. Das verwaltete Linux-Terminal bleibt über den separaten Management-Weg erreichbar.

### A5: Kann der VLAN-10-Verkehr der Diode weiterhin abgefangen werden?

Vergleiche diese Grenze mit Übung 2. Prüfe auf **Protected Linux** die Route zurück zu Dell und die Zuordnung des lokalen Gateways:

```bash
ip route get 10.50.0.11
ip neigh show 192.168.254.254 dev eth1
```

Die Route führt über `192.168.254.254`. Dell erreicht dagegen die WAN-Adresse `10.10.0.21` der Diode direkt in VLAN 10, wie in A2 notiert. Eine leere Nachbartabelle bedeutet, dass gerade kein Eintrag gespeichert ist; sie beweist keinen Schutz.

Beantworte vor dem Öffnen der Lösung:

1. Warum können gefälschte ARP-Antworten aus VLAN 10 die lokale Gateway-Zuordnung von Protected Linux nicht direkt ersetzen?
2. Welche ARP-Einträge auf Dell und auf der VLAN-10-Seite der Diode könnte ein Angreifer während der Dell-Freigabe aus A3 manipulieren, um deren Kommunikation abzufangen?
3. Würde das Weiterleiten von Dells Paketen mit ihrer ursprünglichen Quell-IP die Freigaberegel verletzen? Würden abgefangene SSH-Pakete deren verschlüsselten Inhalt lesbar machen?

??? success "Lösung: ARP-Grenze und Anwendungsweg unterscheiden"
    **Direkte Manipulation aus VLAN 10: nein.** Protected Linux und VLAN 10 liegen in unterschiedlichen Ethernet-Segmenten. Diese geroutete Diode leitet ARP-Frames aus VLAN 10 nicht in ihr LAN weiter. Linux ermittelt die MAC-Adresse seines lokalen Gateways `192.168.254.254`, nicht die des externen Gateways aus Übung 2. Der Schutz entsteht durch die Routing-Grenze, nicht durch die `/32`-Regel.

    **Der VLAN-10-Verkehr der Diode kann weiterhin abgefangen werden.** Ihr WAN-Interface teilt VLAN 10 mit Dell und einem möglichen Angreifer. Ein manipulierter ARP-Eintrag für Dell (`10.50.0.11`) auf der Diode kann deren Antworten zum Angreifer umleiten. Ein manipulierter Eintrag für die Diode (`10.10.0.21`) auf Dell kann Anfragen umleiten. Leitet der Angreifer die Kommunikation weiter, kann er zwischen Dell und der Diode sitzen. Diese gefälschten ARP-Antworten bleiben in VLAN 10: Sie müssen weder den geschützten Endpunkt erreichen noch dessen ARP-Cache verändern.

    **Die Quell-IP-Regel ist kein ARP-Schutz.** Bleibt die Quell-IP beim Weiterleiten Dells erlaubte Adresse, trifft die Freigaberegel weiterhin zu. Der erlaubte HTTP-Verkehr lässt sich auf diesem Weg mitlesen. Abgefangene SSH-Pakete bleiben bei korrekter SSH-Authentifizierung und Host-Key-Prüfung verschlüsselt; siehe [SSH-Host-Key-Prüfung](https://www.rfc-editor.org/rfc/rfc4251.html#section-9.3.4). ARP-Spoofing allein macht aus einer gesperrten Quelle keine erlaubte Quelle. Das verbleibende Risiko ist das Abfangen oder Unterbrechen von Verkehr auf der VLAN-10-Seite der Diode.

    Wähle einen Schutz für das verbleibende Risiko: ARP-Prüfung am Switch mit gepflegten IP/MAC-Zuordnungen in VLAN 10 sowie authentifizierte Verschlüsselung, etwa HTTPS oder SSH mit Zertifikats- beziehungsweise Host-Key-Prüfung. Verschlüsselung schützt Anwendungsdaten; sie verhindert nicht, dass ein Angreifer die Verbindung unterbricht. Siehe [RFC 826](https://www.rfc-editor.org/rfc/rfc826.html) und [die Erklärung zur ARP-Prüfung](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9400/software/release/17-13/configuration_guide/sec/b_1713_sec_9400_cg/configuring_dynamic_arp_inspection.html).

Dies ist eine Aufgabe zur Topologie und zur Einschätzung des Angriffswegs. Die Routen- und Nachbarprüfungen belegen die beiden Seiten; sie weisen kein erfolgreiches Abfangen nach. Notiere, wessen ARP-Eintrag angegriffen würde und welcher Verkehr betroffen wäre.

### A6: Wiederherstellen und übergeben

Stelle das notierte Ausgangsprofil wieder her: **From Network = Default allow**, **From Endpoint = Default allow**, keine Ausnahmen in beiden Richtungen. Speichere, bestätige **Save & apply** und warte auf die Bestätigung. Wiederhole HTTP/TCP 22 von beiden Clients sowie die HTTP-Anfrage des geschützten Endpunkts an WAGO. Alles muss wieder funktionieren. Profilzuordnung, DNS, NTP und Logging müssen den in A2 notierten Einstellungen entsprechen.

Ergänze tatsächliche Ergebnisse und Zeitpunkte in dieser Tabelle:

| Policy-Zustand | Dell → Endpunkt HTTP / TCP 22 | Hirschmann → Endpunkt HTTP / TCP 22 | Endpunkt → WAGO HTTP |
| --- | --- | --- | --- |
| Ausgangszustand | Funktioniert / funktioniert | Funktioniert / funktioniert | HTTP 200 |
| Eingehend nur Dell | Funktioniert / funktioniert | Gesperrt / gesperrt | HTTP 200 |
| Eingehend nur Dell; keine neuen ausgehenden Verbindungen | Funktioniert / funktioniert | Gesperrt / gesperrt | Gesperrt |
| Keine eingehenden Ausnahmen; keine neuen ausgehenden Verbindungen | Gesperrt / gesperrt | Gesperrt / gesperrt | Gesperrt |
| Wiederhergestellter Ausgangszustand | Funktioniert / funktioniert | Funktioniert / funktioniert | HTTP 200 |

Jeder Befehl baut eine neue Verbindung auf. Eine bestehende SSH-Sitzung kann ihren Zustand in der Verbindungsverfolgung behalten und eignet sich nicht als Beleg für eine neu angewendete Sperre. Weicht ein Ergebnis ab, notiere Quelladresse, Ziel, Zeitpunkt, angewendete Revision und tatsächlichen Fehler, bevor du weitere Einstellungen änderst.

??? success "Lösung der Zusatzübung und betriebliche Schlussfolgerung"
    Die Diode setzt eine Grenze um den älteren Endpunkt durch, während der Rest der Zelle seine bestehenden VLANs behält. Die Ausnahme für Dells `/32`-Adresse erhält den gewählten Wartungsweg. Das Begrenzen neuer, vom Endpunkt ausgehender Verbindungen reduziert einen Weg zur seitlichen Ausbreitung. Zustandsbehaftete Antworten erhalten die erlaubte eingehende Anwendungskommunikation. Das Entfernen der Ausnahme zeigt die Isolation; das Wiederherstellen des notierten Profils belegt die Rückkehr zum Ausgangszustand.

    Der Schutz hängt davon ab, dass der Verkehr diese Grenze durchläuft. Ein weiterer ungefilterter Anschluss, eine kompromittierte zugelassene Quelle oder eine gefälschte Adressidentität können die Annahmen unterlaufen. Der Management-Weg im Lab ist eine beabsichtigte Ausnahme für die Plattform. Eine Produktionsplanung muss alle Wege, notwendige Prozessabhängigkeiten und tatsächliche Anwendungsberechtigungen berücksichtigen. Die Belege zeigen die Lab-Policy, keine Immunität gegen Angriffe und kein zertifiziertes Security Level.

## Weiterforschen

- [MikroTik Bridge VLAN Table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): Prüfe den Zusammenhang zwischen PVID, getaggten Ports und CPU-Zugang.
- [RouterOS Firewall Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/): Vergleiche Input und Forward.
- [RouterOS Connection Tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/): Erkläre, warum Antworten die davorstehende Freigaberegel für bestehende Verbindungen passieren.
- [NIST SP 800-82r3](https://csrc.nist.gov/pubs/sp/800/82/r3/final): Prüfe, wie sich Netzwerkgrenzen auf die Verfügbarkeit einer OT-Anlage auswirken.
