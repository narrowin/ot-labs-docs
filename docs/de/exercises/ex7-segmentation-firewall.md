# Übung 7: Ein geroutetes VLAN aufbauen und prüfen

**Zeit:** 90–120 Minuten einschliesslich Zonenplanung. **Lab:** dein studentisches `ot-sec-flat`-Lab. **Niveau:** Geführte Praxis.

Erstelle eine neue Sicherheitszone, transportiere sie durch die Switches, route sie am Gateway und setze eine kleine Policy durch. Weise nach, dass die Anwendung funktioniert und eine unerlaubte Verbindung blockiert wird. Stelle danach den flachen Ausgangszustand wieder her.

**Du lernst:** Access-VLAN und getaggten Weg zu konfigurieren, die Bridge-CPU des Gateways einzubeziehen, ein nicht überlappendes Subnetz zu wählen, zustandsbehaftete Firewall-Regeln anzuordnen und beide Richtungen mit Paketen und Anwendungsergebnissen zu prüfen.

## Dein Änderungsauftrag

Eine temporäre Diagnosestation soll eine SPS-Visualisierung erreichen, ohne allgemeinen Zugriff auf die Zelle zu erhalten. Verwende den Linux-Endpunkt mit Hirschmann-Bezeichnung als **Testclient** und WAGO A als Visualisierungsserver. Das sind Übungsrollen; ein industrieller Switch ist nicht üblicherweise ein Bedienarbeitsplatz.

Der Verantwortliche erlaubt den WebVisu-Dienst und eine Gateway-Diagnose. SSH, andere neue zonenübergreifende Dienste und neue Verbindungen zurück zur Diagnosestation gehören nicht zum Auftrag. Formuliere diese Absicht zuerst als Kommunikationsmatrix. Setze sie danach um und belege sowohl die funktionierende Anwendung als auch die Einschränkung.

Eine **Zone** gruppiert Anlagenbestandteile mit gemeinsamen Sicherheitsanforderungen. Ein **Conduit** ist ein genehmigter Kommunikationspfad zwischen Zonen. Ein VLAN trennt Ethernet-Verkehr; Routing und Firewall-Regeln kontrollieren hier den Pfad zwischen Zonen. Eine **zustandsbehaftete** Firewall verfolgt bestehende Verbindungen: Antworten brauchen keine Freigabe als unabhängige neue Verbindungsaufnahme.

**Wichtige Grenze:** TCP 8080 erlaubt die WebVisu-Anwendung einschliesslich der dort angebotenen Bedienelemente. Das erzwingt **keinen reinen Lesezugriff**. Studierende bedienen die Elemente nicht; eine reale Anforderung «nur ansehen» braucht zusätzlich Anwendungsrollen oder einen geeigneten anwendungsbezogenen Zugriffsschutz. Dieses Praktikum prüft nur den Netzwerkzugriff.

## Vorbereitung

Verwende dein eigenes studentisches Lab nach Abschluss und Bereinigung der Übungen 1–6. Der Ausgangszustand muss zu Aufgabe 1 passen. Ein Checkpoint des flachen Ausgangszustands muss für die Wiederherstellung verfügbar sein; die [Vorbereitungsanleitung](../lab-practical.md) enthält die Befehle für die Konsole der Kursleitung.

Öffne Terminals für [Hirschmann](/network/terminal?node=hirschmann-edge1-vlan50), [WAGO A](/network/terminal?node=wago-plc2a-vlan10), [sw-acc1](/network/terminal?node=sw-acc1), [sw-dist](/network/terminal?node=sw-dist) und [gw](/network/terminal?node=gw-firewall). Lasse das Hirschmann-Terminal offen: Die Management-Verbindung verwendet `eth0`, geändert wird nur Dateninterface `eth1`.

Bearbeite die praktische Übung in einer laufenden Sitzung. Die Linux-Adress-/Routenbefehle ändern den aktuellen Knoten; seine Startkonfiguration bleibt flach. Wird ein Knoten während des Aufbaus neu gestartet, lasse vor dem Fortsetzen den Checkpoint wiederherstellen.

**Eigenständige Dokumentation:** Öffne das zugewiesene Übungs-LNE, wähle dessen Netz und Live-Snapshot und dann Explorer → Netmap. Suche die stabilen Namen oben und nutze ihre konfigurierten SSH-Aktionen. Wähle für Mitschnitte Switch und Interface und für VLAN-Ansichten VLAN 10 oder 70. Reader-Links übernehmen Netz und Snapshot. Nutze ausserhalb von LNE die Übungsnavigation der Website.

Der Name `hirschmann-edge1-vlan50` bleibt bestehen. Die Übung erstellt **ein neues VLAN 70**; Hostnamen erzeugen keine VLANs 20–50. Toolbox ist ein Entwurfswerkzeug. Das Speichern einer Matrix führt die folgenden Befehle nicht aus.

| Zustand | Hirschmann-Datenadresse | VLAN | Weg zu WAGO A | Erwarteter Zugriff |
| --- | --- | --- | --- | --- |
| Anfang | `10.50.0.12/8` | 10 | Direkt, geswitcht | HTTP und SSH offen |
| Nach Aufbau | `172.30.70.12/24` | 70 | Über `172.30.70.1` auf gw | HTTP erlaubt; neues SSH blockiert |
| Nach Bereinigung | `10.50.0.12/8` | 10 | Wieder direkt | Ursprünglicher Zugriff wiederhergestellt |

## Anlagenzonen vor der Konfiguration planen

**Entwurfsaufgabe, 20–30 Minuten.** Nutze Wasseraufbereitungsdiagramm und SPS-Zuordnung aus dem Kurshandout (PDF-Seiten 33–34 und 130–145). Gemeint sind gezählte PDF-Seiten; die aufgedruckten Foliennummern weichen später ab. Arbeite selbstständig oder besprich deinen Entwurf mit einer Kollegin oder einem Kollegen, bevor du dein eigenes Lab konfigurierst.

1. Gruppiere Anlagenbestandteile nach betrieblichem Zweck und Folgen von Ausfall oder Manipulation. Beginne mit Aufbereitung/Hygiene, Pumpen/Prozess, Entsorgung, Leitwarte und Engineering/Fernwartung. Das sind mögliche Gruppen, keine vorgeschriebene Lösung oder zugewiesenen Security Levels.
2. Prüfe die **Steuerungsgrenze**. WAGO PLC2 steuert UV/Chlor, Mischen und pH-Funktionen. Schneider PLC4 steuert den Mischer der ersten Stufe sowie Hilfs-/Entsorgungsanlagen. Ein VLAN an der einen Datenschnittstelle einer SPS verschiebt den gesamten Endpunkt; es trennt nicht dessen physische Ausgänge. Eine Firewall zwischen zwei intern von derselben SPS gesteuerten Aktoren im Diagramm erzwingt deren Trennung nicht.
3. Erfasse notwendige Kommunikation. Verwende beobachtete Clients/Server/Funktionen aus Übung 6 als Beleg und ergänze Zweck, Initiator, Empfänger, Protokoll, Verantwortlichen und Ausfallfolge. Kennzeichne eine Beziehung aus dem Prozessbild als **zu prüfen**, bis sie beobachtet oder vom Verantwortlichen bestätigt wurde. Vorhandener Verkehr ist nicht automatisch notwendiger Verkehr.
4. Finde eine Abhängigkeit über deine vorgesehenen Zonengrenzen. Entscheide, ob du die SPS mit ihren Funktionen gemeinsam gruppierst oder einen später zu qualifizierenden Engineering-Umbau vorschlägst. Eine TCP-Port-Regel löst weder eine interne Steuerungsabhängigkeit noch macht sie Modbus zu einem reinen Leseprotokoll.
5. Prüfe deinen Entwurf: Welcher Kompromiss begrenzt Störungen und erhält notwendige Steuerung? Vergleiche bei Gelegenheit mit einer Kollegin oder einem Kollegen. Notiere ein verbleibendes Risiko und den zuständigen Anlagenverantwortlichen. Diese Übung weist keine IEC-62443-Security-Levels zu und zertifiziert sie nicht.

**Ergebnis:** Zonenskizze und mindestens vier gerichtete Kommunikationszeilen, darunter Leitsystemkommunikation, Engineering-Zugriff, Lieferantenzugriff und lokale Geräteerkennung. Nutze eigene Notizen oder den Toolbox-Zonen-/Kommunikationsplaner in deinem Lab. In LNE: Produktmenü → **Toolbox** → **Policies**. Speichern eines Designs setzt keine Gerätekonfiguration um.

??? success "Begründung prüfen"
    Eine gute Antwort berücksichtigt die reale SPS-Zuordnung, erhält begründete Steuerungsbeziehungen und begrenzt Wartungszugriffe. Sie trennt UV/Chlor, Mischer oder pH nicht als unabhängig durchgesetzte VLANs, solange diese im selben WAGO-Endpunkt gesteuert werden. Sie erkennt auch Schneiders gemischte Prozess-/Entsorgungsaufgabe. Mehrere Zonendesigns sind vertretbar, wenn Abhängigkeiten, Grenzen und Restrisiken ausdrücklich benannt sind.

Die Umsetzung unten realisiert **einen kleinen, prüfbaren Conduit** für einen temporären Diagnoseclient. Sie setzt nicht das ganze Anlagendesign um. Die Prozesssteuerungen bleiben an ihrem Platz; so übst du Routing und Regeldurchsetzung, ohne eine laufende Steuerungsabhängigkeit zu verschieben.

| Initiator | Empfänger | Dienst | Gewünschtes Ergebnis und Grund |
| --- | --- | --- | --- |
| Diagnoseclient | WAGO A | TCP 8080 | Genehmigte WebVisu-Anwendung erlauben |
| Diagnoseclient | WAGO A | TCP 22 | Nicht genehmigten SSH-Aufbau sperren |
| WAGO A | Diagnoseclient | Neues TCP 22 | Nicht genehmigten umgekehrten Verbindungsaufbau sperren |
| Diagnoseclient | Sein Gateway | ICMP | Vorgesehene Erreichbarkeitsdiagnose erlauben |
| Diagnoseclient | Sein Gateway | TCP 22 | Gateway-Administration sperren |
| Teilnehmer einer erlaubten Verbindung | Gegenseite | Established/related | Zugeordnete Antworten erlauben |

Notiere Verantwortlichen und Ablauf zum Wartungsende für den vorgeschlagenen WebVisu-Zugriff. Die Bereinigung setzt dieses Ablaufdatum im Praktikum um. Sage vor Aufgabe 2 für jede Zeile die erste passende Firewall-Regel voraus.

## Aufgabe 1: Ausgangszustand festhalten

Auf **Hirschmann**:

```bash
ip -br address
ip route
ip route get 10.10.0.11
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Weise auf **WAGO A** vor der Policy nach, dass Hirschmanns SSH-Dienst offen ist:

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

**Kontrollpunkt:** Hirschmann hat `10.50.0.12/8` auf `eth1`, Management `192.168.100.59/24` auf `eth0` sowie Standardroute und explizite Routen für `1.0.0.0/8` und `2.2.2.0/24` über `10.10.0.1`. WAGO wird direkt erreicht; 22/8080 sind offen und HTTP liefert 200. WAGO erreicht Hirschmanns SSH und routet `172.30.70.12` über `10.10.0.1`.

VLAN/Interface 70, Subnetz `172.30.70.0/24` und Kommentare mit `ex7-` müssen unbenutzt sein. Die beiden letzten Gateway-Abfragen müssen jeweils `1` ausgeben: Sie identifizieren die Einfügepunkte für Aufgabe 2. sw-acc1 ether5 hat PVID 10 und gehört dynamisch zu VLAN 10. Die statische VLAN-10-Zeile enthält `bond-sw-dist,ether10`; sw-dist führt VLAN 10 über `bond-sw-acc1,ether12,ether8`. Weichen diese Anfangswerte ab, stoppe und lasse den zugewiesenen flachen Checkpoint wiederherstellen.

**Umzusetzende Policy:** Hirschmann → WAGO A TCP 8080 und Hirschmann → neues Gateway ICMP erlauben; Established/Related-Antworten erhalten; andere neue weitergeleitete Verbindungen von/zu VLAN 70 und anderen Gateway-Zugriff aus VLAN 70 verweigern. Die vorhandene Private-Source-Liste enthält `172.16.0.0/12`. Sie würde das neue Subnetz breit erlauben, wenn deine begrenzten Regeln nicht vorher greifen.

## Aufgabe 2: Gateway und Policy vorbereiten

Erstelle auf **gw** zuerst das Interface, damit Firewall-Regeln es referenzieren können:

```routeros
/interface vlan add name=vlan70 interface=bridge vlan-id=70 comment=ex7-vlan70
```

Füge die Regeln in dieser Reihenfolge hinzu. Die `place-before`-Ausdrücke identifizieren anhand ihrer Ausgangskommentare die ersten Forward-/Input-Regeln nach der Zustandsfreigabe. Vorhandene Established/Related-Accepts bleiben vor deinen Regeln; breitere ICMP-, Management- und Private-Source-Accepts dahinter.

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

**Kontrollpunkt:** Forward-Reihenfolge: Established/Related → `ex7-http` → `ex7-deny-from` → `ex7-deny-to` → bisherige Policies. Input-Reihenfolge: Established/Related → `ex7-gateway-ping` → `ex7-deny-input` → bisherige Policies. Die neue VLAN-Tabellenzeile enthält **Bridge-CPU und ether12**. Nur ether12 würde den ausdrücklich konfigurierten CPU-Weg fürs Routing auslassen.

Neue Regelzeilen können beim Start des Interfaces kurz `I` zeigen. Warte einige Sekunden und wiederhole den letzten Print-Befehl. Keine `ex7-`-Zeile darf vor dem Fortfahren noch das Flag `I` tragen. Die allgemeine Flag-Legende erklärt `I` auch weiterhin.

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

**Kontrollpunkt:** Der neue Weg lautet Hirschmann → ungetaggtes ether5 → getaggter `bond-sw-dist` → sw-dist `bond-sw-acc1` → getaggtes ether12 → gw CPU/vlan70. Die PVID-Änderung entfernt ether5 aus der dynamisch abgeleiteten VLAN-10-Mitgliedschaft. Vorhandene getaggte VLAN-10-Zeilen bleiben erhalten. Der Bond bleibt ein logischer Bridge-Port.

## Aufgabe 4: Endpunkt ins neue Subnetz verschieben

Entferne auf **Hirschmann** die Routen zum alten Gateway vor dem Ersetzen seiner Datenadresse. Die neue Standardroute deckt auch die Ziele der beiden bisher expliziten Routen ab.

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

Filtere den Mitschnitt mit `ip.addr == 172.30.70.12 && (tcp.port == 22 || tcp.port == 8080)`. Ordne Quelle/Ziel und Ports deinen Regellogs zu. Logs und Zähler können auch frühere Versuche oder Hintergrundverkehr in deinem Lab enthalten.

Verwende die neuesten Einträge aus deinem Testzeitraum. Frühere `EX7-`-Einträge können nach einer vorherigen Bereinigung erhalten bleiben.

**Kontrollpunkt:** TCP 8080 ist offen und HTTP liefert 200. TCP 22 ist **filtered**, obwohl der Server vorher nachweislich offen war. Dein SSH-SYN ist in VLAN 70 sichtbar und `ex7-deny-from` protokolliert die passende Verweigerung. Erfolgreiches HTTP und Gateway-Ping belegen einen funktionierenden Weg. Ein Timeout allein identifiziert keine Firewall-Regel.

Öffne **Data Collector → Devices → Collect new data**. Warte auf das Ende der Erfassung und aktualisierte **Last poll**-Werte. [Untersuche dann VLAN 70 in Netmap](/network/?viewMode=vlan&vlanIds=70). Hirschmann und der Weg über sw-acc1, sw-dist und gw sollen nun VLAN 70 führen. Vergleiche gemeldete Mitgliedschaft, Konfiguration und Pakete. Ein Neuladen von Netmap allein fragt die Switches nicht neu ab.

Die aktuelle VLAN-Ansicht kann die zusammengefasste Bond-Kante zwischen sw-dist und sw-acc1 grau lassen, obwohl beide Switches VLAN 70 melden. Prüfe diesen Abschnitt mit der VLAN-Tabelle des Bonds und getaggten Paketen. Die Kantenfarbe beweist nicht, ob Verkehr fliesst.

**Hauptergebnis erreicht.** Lasse VLAN 70 für die optionalen Zusatzteile konfiguriert. Führe danach in jedem Fall die **Bereinigung** durch. Stoppe den Mitschnitt vor einem Zielwechsel.

Wenn du vor den Extras pausierst, führe zuerst die Bereinigung durch. Prüfe beim Fortsetzen den flachen Ausgangszustand mit Aufgabe 1 und wiederhole Aufgaben 2–5 vor den Extras. Bewahre deine bisherigen Ergebnisse zum Vergleichen auf.

## Dein Abnahmeprotokoll

Notiere genehmigte Quelle, Ziel und Dienst, HTTP-Ergebnis, passende abgewiesene SSH-Anfrage mit Firewall-Log sowie die nach der Bereinigung wiederhergestellte Ausgangslage. Ergänze Gegenrichtungs- und Gateway-Prüfungen aus Extra 2, falls durchgeführt. Behaupte weder eine Prüfung aller denkbaren Datenflüsse noch einen durch HTTP-Freigabe erzwungenen HMI-Lesezugriff.

## Lösung

??? success "Vergleiche deine Umsetzung"
    VLAN 70 benötigt Access-PVID, ungetaggten Client-Port, getaggte Mitgliedschaft auf beiden Switch-Uplinks und Gateway ether12/CPU, VLAN-Interface, nicht überlappendes Subnetz und funktionierende Rückroute. WAGO A sendet Antworten zum neuen Subnetz über sein bestehendes Gateway `10.10.0.1`.

    HTTP-Initiierung trifft `ex7-http`. Antworten treffen den früheren Established/Related-Accept. Neues SSH trifft `ex7-deny-from` vor der breiten Private-Source-Freigabe. Input erlaubt die gewählte Gateway-Diagnose und blockiert andere Anfragen aus VLAN 70 an gw. Bestehender OT-zu-OT-Verkehr bleibt in VLAN 10.

## Extra 1: Eine Anfrage vor und nach dem Router verfolgen

**Optional, 10 Minuten.** Lasse den Gateway-Uplink-Mitschnitt laufen und sende eine weitere HTTP-Anfrage von Hirschmann. Filtere `ip.src == 172.30.70.12 && tcp.dstport == 8080`.

Finde dieselbe Anfrage vor und nach dem Routing: Vergleiche TCP-Sequenznummer/Nutzlast, VLAN-ID, Ethernet-Adressen, IP-Adressen und TTL.

**Kontrollpunkt:** Auf diesem einen Uplink kommt die Anfrage mit VLAN 70 an und geht mit VLAN 10 hinaus. IP-Quelle/-Ziel bleiben gleich, Ethernet-Adressen ändern sich und die TTL sinkt um eins. Das ist Routing ohne Source NAT. Ein erfolgreicher Ping allein würde diese Änderungen nicht zeigen.

Beim Mitschnitt beider Seiten kann Wireshark die zweite Kopie als Duplicate/Out-of-Order markieren. Vergleiche Sequenznummer, VLAN und TTL, bevor du diese Markierung als Netzwerkfehler deutest.

## Extra 2: Antworten und neue Initiierungen unterscheiden

**Optional, 10–15 Minuten.** Prüfe auf **WAGO A** eine neue SSH-Verbindung in Gegenrichtung zum in Aufgabe 1 nachgewiesenen Listener:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.12
```

Prüfe auf **Hirschmann** den Gateway-Schutz und wiederhole danach erlaubtes HTTP:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.1
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Lies nochmals die Statistiken/Logs der `ex7-`-Regeln. Die Gegenrichtung muss `ex7-deny-to` treffen, Gateway-SSH `ex7-deny-input`, und HTTP mit zustandsbehafteten Antworten weiter 200 liefern. Beide SSH-Ergebnisse sind filtered. Bestehende Sitzungen oder globale Zähler müssen nicht geleert werden.

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

Wiederhole die Adress-, Routen-, Dienst- und Switch-Prüfungen aus Aufgabe 1. Hirschmann muss wieder `10.50.0.12/8`, PVID 10 und einen direkten WAGO-Weg verwenden; HTTP und SSH müssen wieder offen sein. WAGO muss Hirschmanns SSH wieder unter `10.50.0.12` erreichen. VLAN/Interface 70 und die `ex7-`-Regeln müssen fehlen. Ursprüngliche VLAN-10-/60-Mitgliedschaft und Management-Verbindung müssen deinen Notizen entsprechen.

Führe **Collect new data** erneut aus, um gemeldete Port-Mitgliedschaften zu aktualisieren. Prüfe das Entfernen in den tatsächlichen RouterOS-Tabellen. Eine VLAN-Nummer im Katalog allein beweist nicht, dass ein Port sie weiterhin führt.

Weicht die Bereinigung vom protokollierten Ausgangszustand ab, verwende mit der Kursleitung das gespeicherte Checkpoint-Verfahren und wiederhole Aufgabe 1. Schliesse nach der Prüfung alle Terminal-Tabs.

## Weiterforschen

- [MikroTik Bridge VLAN Table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): Verfolge PVID, Tagged Ports und CPU-Zugang.
- [RouterOS Firewall Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/): Vergleiche Input und Forward.
- [RouterOS Connection Tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/): Erkläre zustandsbehaftete Antworten über die frühere Regel.
- [NIST SP 800-82r3](https://csrc.nist.gov/pubs/sp/800/82/r3/final): Verbinde durchgesetzte Grenzen mit OT-Verfügbarkeit.
