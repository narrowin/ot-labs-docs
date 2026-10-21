# Übung 7: Ein geroutetes VLAN aufbauen und prüfen

**Zeitbedarf:** 90–120 Minuten einschliesslich Zonenplanung. **Lab:** dein eigenes `ot-sec-flat`-Lab. **Niveau:** angeleitete praktische Übung.

Erstelle eine neue Sicherheitszone mit einem eigenen VLAN. Konfiguriere den VLAN-Pfad über die Switches sowie Routing und gezielte Firewall-Regeln am Gateway. Weise nach, dass die Anwendung funktioniert und eine unerlaubte Verbindung blockiert wird. Stelle danach den flachen Ausgangszustand wieder her.

**Du lernst:** Access-VLAN und getaggten Weg zu konfigurieren, den CPU-Port der Gateway-Bridge einzubeziehen, ein nicht überlappendes Subnetz zu wählen, zustandsbehaftete Firewall-Regeln anzuordnen und beide Richtungen mit Paketen und Anwendungsergebnissen zu prüfen.

## Dein Änderungsauftrag

Eine temporäre Diagnosestation soll eine SPS-Visualisierung erreichen, ohne allgemeinen Zugriff auf die Zelle zu erhalten. Verwende `hirschmann-edge1` als **Testclient** und `wago-plc2a` als Visualisierungsserver. Der vorhandene Lab-Knoten `hirschmann-edge1` läuft unter Linux und stellt für diese Aufgabe den Diagnoseclient bereit.

Der Verantwortliche erlaubt den Zugriff auf WebVisu und die Erreichbarkeitsprüfung des Gateways. SSH und andere neue Verbindungen zwischen den Zonen sind nicht erlaubt. Das gilt auch für neue Verbindungen zur Diagnosestation. Halte diese Vorgaben zuerst in einer Kommunikationsmatrix fest. Setze sie danach um und belege sowohl die funktionierende Anwendung als auch die Einschränkung.

Eine **Zone** gruppiert Anlagenbestandteile mit gemeinsamen Sicherheitsanforderungen. Ein **Conduit** ist ein genehmigter Kommunikationspfad zwischen Zonen. Ein VLAN trennt Ethernet-Verkehr; Routing und Firewall-Regeln kontrollieren hier den Pfad zwischen Zonen. Eine **zustandsbehaftete** Firewall verfolgt bestehende Verbindungen: Antwortpakete einer erlaubten Verbindung benötigen deshalb keine eigene Freigabe für eine neue Verbindung in Gegenrichtung.

**Wichtige Grenze:** Eine Freigabe von TCP-Port 8080 erlaubt den Zugriff auf die WebVisu-Anwendung einschliesslich ihrer Bedienelemente. Das erzwingt **keinen reinen Lesezugriff**. Betätige die Bedienelemente in dieser Übung nicht. Für die Anforderung «nur ansehen» wären zusätzlich passende Anwendungsrollen oder ein anderer Zugriffsschutz auf Anwendungsebene nötig. Dieses Praktikum prüft nur den Netzwerkzugriff.

## Vorbereitung

Verwende dein eigenes Lab nach Abschluss und Bereinigung der Übungen 1–6. Der Ausgangszustand muss zu Aufgabe 1 passen. Ein Checkpoint des flachen Ausgangszustands muss für die Wiederherstellung verfügbar sein; die [Vorbereitungsanleitung](../lab-practical.md) enthält die Befehle für die Konsole der Kursleitung.

Öffne Terminals für [`hirschmann-edge1`](/network/terminal?node=hirschmann-edge1-vlan50), [`wago-plc2a`](/network/terminal?node=wago-plc2a-vlan10), [sw-acc1](/network/terminal?node=sw-acc1), [sw-dist](/network/terminal?node=sw-dist) und [`gw-firewall`](/network/terminal?node=gw-firewall). Lasse das `hirschmann-edge1`-Terminal offen: Die Management-Verbindung verwendet `eth0`, geändert wird nur das Dateninterface `eth1`.

Bearbeite die praktische Übung in einer laufenden Sitzung. Die Linux-Befehle für Adressen und Routen ändern den laufenden Zustand des Knotens. Seine Startkonfiguration enthält weiterhin das flache Ausgangsnetz. Wird ein Knoten während des Aufbaus neu gestartet, lasse vor dem Fortsetzen den Checkpoint wiederherstellen.

**Eigenständige Dokumentation:** Öffne das zugewiesene Übungs-LNE, wähle dessen Netz und Live-Snapshot und dann Explorer → Netmap. Suche die oben genannten Knotennamen und nutze ihre konfigurierten SSH-Aktionen. Wähle für Mitschnitte Switch und Interface und für VLAN-Ansichten VLAN 10 oder 70. Links in der Übungsansicht übernehmen Netz und Snapshot. Nutze ausserhalb von LNE die Übungsnavigation der Website.

Der Name `hirschmann-edge1` bleibt bestehen. Du verschiebst diesen Testclient von VLAN 10 in **ein neues VLAN 70**. Toolbox ist ein Entwurfswerkzeug. Das Speichern einer Matrix führt die folgenden Befehle nicht aus.

| Zustand | `hirschmann-edge1`-Datenadresse | VLAN | Weg zu `wago-plc2a` | Erwarteter Zugriff |
| --- | --- | --- | --- | --- |
| Anfang | `10.50.0.12/8` | 10 | Direkt, geswitcht | HTTP und SSH offen |
| Nach Aufbau | `172.30.70.12/24` | 70 | Über `172.30.70.1` auf `gw-firewall` | HTTP erlaubt; neues SSH blockiert |
| Nach Bereinigung | `10.50.0.12/8` | 10 | Wieder direkt | Ursprünglicher Zugriff wiederhergestellt |

## Anlagenzonen vor der Konfiguration planen

**Entwurfsaufgabe, 20–30 Minuten.** Nutze das Wasseraufbereitungsdiagramm und die SPS-Zuordnung aus dem Kurshandout (PDF-Seiten 33–34 und 130–145). Gemeint sind gezählte PDF-Seiten; die aufgedruckten Foliennummern weichen später ab. Arbeite selbstständig oder besprich deinen Entwurf mit einer Kollegin oder einem Kollegen, bevor du dein eigenes Lab konfigurierst.

1. Gruppiere Anlagenbestandteile nach betrieblichem Zweck und Folgen von Ausfall oder Manipulation. Beginne mit Aufbereitung und Hygiene, Pumpen und Prozess, Entsorgung, Leitwarte sowie Engineering und Fernwartung. Das sind mögliche Gruppen; sie schreiben weder eine Lösung noch bestimmte Security Levels vor.
2. Prüfe die **Steuerungsgrenze**. Führe `wago-plc2a` (PLC2A) und `wago-plc2b` (PLC2B) als getrennte Endpunkte in Anlagenliste und Kommunikationsmatrix. Prüfe ihre Aufbereitungsfunktionen anhand des Handouts. `schneider-plc4` (PLC4) steuert den Mischer der ersten Stufe sowie Hilfs-/Entsorgungsanlagen. Die VLAN-Zuordnung der Datenschnittstelle gilt für die gesamte SPS; sie trennt deren physische Ausgänge nicht voneinander. Zeichnest du eine Firewall zwischen zwei Aktoren ein, die dieselbe SPS intern steuert, bewirkt das keine tatsächliche Trennung.
3. Erfasse notwendige Kommunikation. Verwende beobachtete Clients/Server/Funktionen aus Übung 6 als Beleg und ergänze Zweck, Initiator, Empfänger, Protokoll, Verantwortlichen und Ausfallfolge. Kennzeichne eine Beziehung aus dem Prozessbild als **zu prüfen**, bis sie beobachtet oder vom Verantwortlichen bestätigt wurde. Vorhandener Verkehr ist nicht automatisch notwendiger Verkehr.
4. Finde eine Abhängigkeit über deine vorgesehenen Zonengrenzen. Entscheide, ob du die SPS mit ihren Funktionen gemeinsam gruppierst oder einen Umbau der Steuerung vorschlägst. Dessen Eignung müsste später gesondert geprüft werden. Eine Regel für einen TCP-Port hebt keine interne Steuerungsabhängigkeit auf und kann Modbus-Lesezugriffe nicht von Schreibzugriffen unterscheiden.
5. Prüfe deinen Entwurf: Wie begrenzt du Störungen und erhältst zugleich die notwendige Steuerungskommunikation? Vergleiche bei Gelegenheit mit einer Kollegin oder einem Kollegen. Notiere ein verbleibendes Risiko und den zuständigen Anlagenverantwortlichen. Diese Übung weist keine IEC-62443-Security-Levels zu und zertifiziert sie nicht.

**Ergebnis:** Eine Zonenskizze und mindestens vier Einträge in der Kommunikationsmatrix mit jeweils angegebener Richtung, darunter Leitsystemkommunikation, Engineering-Zugriff, Lieferantenzugriff und lokale Geräteerkennung. Nutze eigene Notizen oder den Toolbox-Zonen-/Kommunikationsplaner in deinem Lab. In LNE: Produktmenü → **Toolbox** → **Policies**. Das Speichern eines Entwurfs ändert keine Gerätekonfiguration.

??? success "Begründung prüfen"
    Eine gute Antwort berücksichtigt die reale SPS-Zuordnung, erhält begründete Steuerungsbeziehungen und begrenzt Wartungszugriffe. Sie stellt UV-/Chlorbehandlung, Mischer oder pH-Regelung nicht als getrennte VLAN-Zonen dar, solange eine einzelne SPS diese Funktionen steuert. Sie berücksichtigt auch, dass die `schneider-plc4`-SPS sowohl Prozess- als auch Entsorgungsaufgaben übernimmt. Mehrere Zonenentwürfe sind vertretbar, wenn Abhängigkeiten, Grenzen und Restrisiken ausdrücklich benannt sind.

Im praktischen Teil setzt du **einen einzelnen, überprüfbaren Conduit** für eine temporäre Diagnosestation um. Der vollständige Zonenentwurf bleibt eine Planungsaufgabe. Die Prozesssteuerungen bleiben an ihrem Platz; so übst du Routing und Regeldurchsetzung, ohne eine laufende Steuerungsabhängigkeit zu verschieben.

| Initiator | Empfänger | Dienst | Gewünschtes Ergebnis und Grund |
| --- | --- | --- | --- |
| `hirschmann-edge1` | `wago-plc2a` | TCP 8080 | Genehmigte WebVisu-Anwendung erlauben |
| `hirschmann-edge1` | `wago-plc2a` | TCP 22 | Nicht genehmigten SSH-Aufbau sperren |
| `wago-plc2a` | `hirschmann-edge1` | Neue Verbindung zu TCP 22 | Nicht genehmigten umgekehrten Verbindungsaufbau sperren |
| `hirschmann-edge1` | `gw-firewall` | ICMP | Vorgesehene Erreichbarkeitsdiagnose erlauben |
| `hirschmann-edge1` | `gw-firewall` | TCP 22 | Gateway-Administration sperren |
| Teilnehmer einer erlaubten Verbindung | Gegenseite | Established/related | Zugeordnete Antworten erlauben |

Halte fest, wer den vorgeschlagenen WebVisu-Zugriff verantwortet und dass er mit dem Ende der Wartung ablaufen soll. Im Praktikum entfernst du ihn bei der Bereinigung. Sage vor Aufgabe 2 für jede Zeile die erste passende Firewall-Regel voraus.

## Aufgabe 1: Ausgangszustand festhalten

Auf **`hirschmann-edge1`**:

```bash
ip -br address
ip route
ip route get 10.10.0.11
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Weise auf **`wago-plc2a`** vor dem Einrichten der Firewall-Regeln nach, dass der SSH-Dienst von `hirschmann-edge1` offen ist:

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

Auf **`gw-firewall`**:

```routeros
/interface vlan print
/interface bridge vlan print detail
/ip address print
/ip firewall filter print detail
/ip firewall address-list print where list=mgmt
:put [:len [/ip firewall filter find where chain=forward and comment~"^allow HMI"]]
:put [:len [/ip firewall filter find where chain=input and comment~"^allow DHCP"]]
```

**Kontrollpunkt:** `hirschmann-edge1` hat die Datenadresse `10.50.0.12/8` auf `eth1`, die Management-Adresse `192.168.100.59/24` auf `eth0` sowie eine Standardroute und explizite Routen für `1.0.0.0/8` und `2.2.2.0/24` über `10.10.0.1`. `wago-plc2a` wird direkt erreicht; 22/8080 sind offen und HTTP liefert 200. `wago-plc2a` erreicht den SSH-Dienst von `hirschmann-edge1` und routet `172.30.70.12` über `10.10.0.1`.

VLAN 70, das Interface `vlan70`, das Subnetz `172.30.70.0/24` und Kommentare mit `ex7-` dürfen noch nicht verwendet werden. Die beiden letzten Gateway-Abfragen müssen jeweils `1` ausgeben: Sie identifizieren die Einfügepunkte für Aufgabe 2. sw-acc1 ether5 hat PVID 10 und gehört dynamisch zu VLAN 10. Die statische VLAN-10-Zeile enthält `bond-sw-dist,ether10`; sw-dist führt VLAN 10 über `bond-sw-acc1,ether12,ether8`. Weichen diese Anfangswerte ab, stoppe und lasse den zugewiesenen flachen Checkpoint wiederherstellen.

**Umzusetzende Zugriffsregeln:** `hirschmann-edge1` → `wago-plc2a` TCP 8080 und `hirschmann-edge1` → neues Gateway ICMP erlauben; die vorhandene Freigabe für bestehende und zugehörige Verbindungen (Established/Related) beibehalten; andere neue weitergeleitete Verbindungen von/zu VLAN 70 und anderen Gateway-Zugriff aus VLAN 70 verweigern. Die vorhandene Private-Source-Liste enthält `172.16.0.0/12`. Die zugehörige Freigaberegel würde dem neuen Subnetz umfassenden Zugriff gewähren, wenn deine enger gefassten Regeln nicht vorher greifen.

## Aufgabe 2: Gateway und Firewall-Regeln vorbereiten

Erstelle auf **`gw-firewall`** zuerst das Interface, damit du es in den Firewall-Regeln angeben kannst:

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

**Kontrollpunkt:** Der neue Weg führt von `hirschmann-edge1` ungetaggt zu sw-acc1 ether5, mit VLAN-Tag über `bond-sw-dist` und sw-dist `bond-sw-acc1` zu ether12 und schliesslich zum CPU-Port und zum Interface `vlan70` auf `gw-firewall`. Die PVID-Änderung entfernt ether5 aus der dynamisch abgeleiteten VLAN-10-Mitgliedschaft. Vorhandene getaggte VLAN-10-Zeilen bleiben erhalten. Der Bond bleibt ein logischer Bridge-Port.

## Aufgabe 4: Endpunkt ins neue Subnetz verschieben

Entferne auf **`hirschmann-edge1`** die Routen zum alten Gateway, bevor du die Datenadresse des Endpunkts ersetzt. Die neue Standardroute deckt auch die Ziele der beiden bisher expliziten Routen ab.

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

**Kontrollpunkt:** `eth0` behält die Management-Adresse und dein Terminal bleibt verbunden. `eth1` hat nur die neue Datenadresse, seine Routen für `224.0.0.0/4` und `255.255.255.255` bleiben erhalten, die `wago-plc2a`-Route verwendet `172.30.70.1` und das Gateway antwortet. Die neue Adresse wird zuerst ergänzt, damit das Entfernen der alten keine anderen Interface-Routen löscht. `10.70.0.0/24` würde mit den bisherigen OT-`/8`-Rückrouten überlappen; `172.30.70.0/24` vermeidet dies.

## Aufgabe 5: Erlaubtes HTTP und blockiertes SSH beweisen

[Öffne den Gateway-Uplink-Mitschnitt](/network/capture?node=sw-dist&interface=ether12) und wähle **Start Capture**.

Zeigt der Viewer ERF/LEGACY statt Ethernet, wähle vor dem Testverkehr einmal **Restart Capture**. Melde einen wiederkehrenden Fehler an die Kursleitung.

Führe auf **`hirschmann-edge1`** aus:

```bash
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Auf **`gw-firewall`**:

```routeros
/ip firewall filter print stats where comment~"^ex7-"
/log print where message~"EX7-"
```

Filtere den Mitschnitt mit `ip.addr == 172.30.70.12 && (tcp.port == 22 || tcp.port == 8080)`. Vergleiche Quell- und Zieladresse sowie die Ports mit den Log-Einträgen deiner Firewall-Regeln. Logs und Zähler können auch frühere Versuche oder Hintergrundverkehr in deinem Lab enthalten.

Verwende die neuesten Einträge aus deinem Testzeitraum. Frühere `EX7-`-Einträge können nach einer vorherigen Bereinigung erhalten bleiben.

**Kontrollpunkt:** TCP 8080 ist offen und HTTP liefert 200. TCP 22 ist **filtered**, obwohl der SSH-Port vorher nachweislich offen war. Dein SSH-SYN ist in VLAN 70 sichtbar und `ex7-deny-from` protokolliert das Verwerfen des Pakets. Erfolgreiches HTTP und Gateway-Ping belegen einen funktionierenden Weg. Ein Timeout allein identifiziert keine Firewall-Regel.

Öffne **Data Collector → Devices → Collect new data**. Warte auf das Ende der Erfassung und aktualisierte **Last poll**-Werte. [Untersuche dann VLAN 70 in Netmap](/network/?viewMode=vlan&vlanIds=70). `hirschmann-edge1` und der Weg über sw-acc1, sw-dist und `gw-firewall` sollen nun VLAN 70 führen. Vergleiche gemeldete Mitgliedschaft, Konfiguration und Pakete. Ein Neuladen von Netmap allein fragt die Switches nicht neu ab.

Die aktuelle VLAN-Ansicht kann die zusammengefasste Bond-Kante zwischen sw-dist und sw-acc1 grau lassen, obwohl beide Switches VLAN 70 melden. Prüfe diesen Abschnitt mit der VLAN-Tabelle des Bonds und getaggten Paketen. Die Kantenfarbe beweist nicht, ob Verkehr fliesst.

**Die Hauptaufgabe ist abgeschlossen.** Lasse VLAN 70 für die optionalen Zusatzaufgaben konfiguriert. Führe danach in jedem Fall die **Bereinigung** durch. Stoppe den Mitschnitt vor einem Zielwechsel.

Wenn du vor den Extras pausierst, führe zuerst die Bereinigung durch. Prüfe beim Fortsetzen den flachen Ausgangszustand mit Aufgabe 1 und wiederhole Aufgaben 2–5 vor den Extras. Bewahre deine bisherigen Ergebnisse zum Vergleichen auf.

## Dein Abnahmeprotokoll

Notiere die genehmigte Quelle, das Ziel und den Dienst. Halte das HTTP-Ergebnis und eine abgewiesene SSH-Anfrage mit passendem Firewall-Log fest. Dokumentiere nach der Bereinigung auch den wiederhergestellten Ausgangszustand. Ergänze Gegenrichtungs- und Gateway-Prüfungen aus Extra 2, falls durchgeführt. Beschränke deine Aussage auf die tatsächlich geprüften Datenflüsse. Die HTTP-Freigabe allein erzwingt keinen reinen Lesezugriff auf die Visualisierung.

## Lösung

??? success "Vergleiche deine Umsetzung"
    Für VLAN 70 brauchst du PVID 70 am ungetaggten Client-Port sowie eine getaggte VLAN-Mitgliedschaft auf beiden Switch-Uplinks, am Gateway-Port ether12 und am CPU-Port der Gateway-Bridge. Hinzu kommen das VLAN-Interface, ein nicht überlappendes Subnetz und eine funktionierende Rückroute. `wago-plc2a` sendet Antworten zum neuen Subnetz über sein bestehendes Gateway `10.10.0.1`.

    Der HTTP-Verbindungsaufbau wird durch `ex7-http` erlaubt. Antworten passieren die davorstehende Freigaberegel für Established/Related. Neue SSH-Verbindungen werden durch `ex7-deny-from` verworfen, bevor die umfassendere Freigabe für private Quelladressen greifen kann. Die Input-Regeln erlauben die vorgesehene Gateway-Diagnose und blockieren andere Anfragen aus VLAN 70 an `gw-firewall`. Bestehender OT-zu-OT-Verkehr bleibt in VLAN 10.

## Extra 1: Eine Anfrage vor und nach dem Router verfolgen

**Optional, 10 Minuten.** Lasse den Gateway-Uplink-Mitschnitt laufen und sende eine weitere HTTP-Anfrage von `hirschmann-edge1`. Filtere `ip.src == 172.30.70.12 && tcp.dstport == 8080`.

Finde dieselbe Anfrage vor und nach dem Routing: Vergleiche TCP-Sequenznummer/Nutzlast, VLAN-ID, Ethernet-Adressen, IP-Adressen und TTL.

**Kontrollpunkt:** Auf diesem einen Uplink kommt die Anfrage mit VLAN 70 an und geht mit VLAN 10 hinaus. Quell- und Ziel-IP-Adresse bleiben gleich, Ethernet-Adressen ändern sich und die TTL sinkt um eins. Das ist Routing ohne Source NAT. Ein erfolgreicher Ping allein würde diese Änderungen nicht zeigen.

Beim Mitschnitt beider Seiten kann Wireshark die zweite Kopie als Duplicate/Out-of-Order markieren. Vergleiche Sequenznummer, VLAN und TTL, bevor du diese Markierung als Netzwerkfehler deutest.

## Extra 2: Antworten und neue Verbindungsaufnahmen unterscheiden

**Optional, 10–15 Minuten.** Prüfe auf **`wago-plc2a`** eine neue SSH-Verbindung in Gegenrichtung zum in Aufgabe 1 nachgewiesenen Listener:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.12
```

Prüfe auf **`hirschmann-edge1`** den Gateway-Schutz und wiederhole danach die erlaubte HTTP-Anfrage:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.1
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Lies nochmals die Zähler und Log-Einträge der `ex7-`-Regeln. Die neue SSH-Verbindung von `wago-plc2a` zur Diagnosestation muss durch `ex7-deny-to` blockiert werden, die SSH-Verbindung zum Gateway durch `ex7-deny-input`. Die erlaubte HTTP-Verbindung muss weiterhin Status 200 liefern. Bei beiden SSH-Prüfungen muss Nmap den Zustand `filtered` melden. Bestehende Sitzungen oder globale Zähler müssen nicht geleert werden.

## Bereinigung: Flachen Ausgangszustand wiederherstellen

Stoppe und schliesse den Mitschnitt. Auf **`hirschmann-edge1`**:

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

Auf **`gw-firewall`**:

```routeros
/ip address remove [find where comment=ex7-vlan70]
/interface bridge vlan remove [find where comment=ex7-vlan70]
/interface vlan remove [find where name=vlan70]
/ip firewall filter remove [find where comment~"^ex7-"]
```

Wiederhole die Adress-, Routen-, Dienst- und Switch-Prüfungen aus Aufgabe 1. `hirschmann-edge1` muss wieder `10.50.0.12/8`, PVID 10 und einen direkten `wago-plc2a`-Weg verwenden; HTTP und SSH müssen wieder offen sein. `wago-plc2a` muss den SSH-Dienst von `hirschmann-edge1` wieder unter `10.50.0.12` erreichen. VLAN 70, das Interface `vlan70` und die `ex7-`-Regeln müssen entfernt sein. Die ursprünglichen Mitgliedschaften in VLAN 10 und 60 sowie die Management-Verbindung müssen deinen Notizen entsprechen.

Führe **Collect new data** erneut aus, um gemeldete Port-Mitgliedschaften zu aktualisieren. Prüfe das Entfernen in den tatsächlichen RouterOS-Tabellen. Eine VLAN-Nummer im Katalog allein beweist nicht, dass ein Port sie weiterhin führt.

Weicht die Bereinigung vom protokollierten Ausgangszustand ab, verwende mit der Kursleitung das gespeicherte Checkpoint-Verfahren und wiederhole Aufgabe 1. Schliesse nach der Prüfung alle Terminal-Tabs.

Weiter mit [Übung 8: Ein Altsystem mit der Diode mikrosegmentieren](ex8-diode-microsegmentation.md).

## Weiterforschen

- [MikroTik Bridge VLAN Table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): Prüfe den Zusammenhang zwischen PVID, getaggten Ports und CPU-Zugang.
- [RouterOS Firewall Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/): Vergleiche Input und Forward.
- [RouterOS Connection Tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/): Erkläre, warum Antworten die davorstehende Freigaberegel für bestehende Verbindungen passieren.
- [NIST SP 800-82r3](https://csrc.nist.gov/pubs/sp/800/82/r3/final): Prüfe, wie sich Netzwerkgrenzen auf die Verfügbarkeit einer OT-Anlage auswirken.
