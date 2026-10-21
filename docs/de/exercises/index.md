# Übungen für dein Lab

## Anlagenübersicht und SPS-Namen

- **PLC1**: `abb-800xa`
- **PLC2A**: `wago-plc2a`
- **PLC2B**: `wago-plc2b`
- **PLC3**: `ctrlx-plc3`
- **PLC4**: `schneider-plc4`

![Übersicht der Wasseraufbereitung aus der Lab-Visualisierung](../img/water-treatment-overview.png)

Bearbeite die sieben Übungen in deinem eigenen `ot-sec-flat`-Lab der Reihe nach. Übungen 1–6 beginnen im flachen Ausgangszustand; Übung 7 baut ein geroutetes VLAN auf und stellt danach diesen Zustand wieder her. Plane für Hauptaufgaben, Diskussion und Übergaben etwa 5–6½ Stunden. Jede Übung enthält eine ausklappbare Lösung und danach **zwei optionale Zusatzaufgaben**. Alle Extras zusammen benötigen weitere 2–2½ Stunden.

Die Zeiten sind Richtwerte. Bearbeite die Hauptaufgaben in deinem Tempo, nutze bei Bedarf die Hinweise und hebe Extras für später auf. Bewahre Befehle, Filter und Ergebnisse auf, damit du nach dem Kurs in deinem Lab weiterarbeiten kannst. Beende vor einer Pause die Bereinigung der Übung und wiederhole beim Fortsetzen ihre Ausgangsprüfungen.

## Hier beginnen

1. Öffne dein zugewiesenes Network-Explorer-Lab und wähle das **Live**-Netzwerk.
2. Wähle im Produktmenü **Exercises**, danach Übung 1. Mit **English / Deutsch** wechselst du die Sprache.
3. Die Links öffnen konfigurierte Terminals, eine gefilterte Netmap-Ansicht, Zieldetails oder den ausgewählten Port für den Paketmitschnitt. In Terminal- und Mitschnitt-Tabs öffnet der aktive Übungstitel die Anleitung. Lass das Terminal beim Wechsel zum Mitschnitt offen.
4. Bearbeite die Aufgabe zuerst selbst, bevor du den Hinweis oder die Lösung öffnest. Notiere das Ausgangsgerät, das Interface, den Befehl oder Filter und die tatsächlichen Ergebnisse.

Du benötigst das von der Kursleitung gestartete Lab und eingerichtete Zugangsdaten für die Lab-Geräte. Verwende das bereitgestellte Lab und seine eingerichteten Werkzeug-Aktionen. Bei **unavailable** prüfst du das gewählte Live-Netzwerk und lässt die Kursleitung Berechtigungen und die Zuordnung des Werkzeugs zum Lab-Gerät prüfen.

Werkzeug-Links sind für die Übungsansicht in Network Explorer gedacht. Öffne bei Nutzung dieser veröffentlichten Dokumentation das genannte Werkzeug im zugewiesenen Lab und wähle Knoten/Interface manuell. Die simulierten Daten-IPs sind keine direkt vom Laptop erreichbaren Ziele.

## Übungen

| # | Übung | Hauptzeit | Zusatzaufgaben |
| --- | --- | --- | --- |
| 1 | [Getaggten und ungetaggten Verkehr vergleichen](ex1-vlan-discovery.md) | 30–40 Min. | Ersatz-SPS prüfen; Next-Hop-MAC |
| 2 | [Eine ARP-Umleitung nachweisen](ex2-arp-spoofing.md) | 40–50 Min. | Vorfall-Zeitleiste; ARP-Schutz bei Wartungsarbeiten |
| 3 | [Den aktiven Spanning Tree erklären](ex3-spanning-tree.md) | 30–40 Min. | Switch-Tausch; Uplink-Umschaltung prüfen |
| 4 | [Datenströme durch LACP verfolgen](ex4-link-aggregation.md) | 30–40 Min. | Blinder Fleck im Mitschnitt; gemeinsame Ausfallpunkte |
| 5 | [Zugriff aus der DMZ beurteilen](ex5-reconnaissance.md) | 35–45 Min. | Ausgangspunkte vergleichen; TCP-Portzustände beweisen |
| 6 | [OT-Protokolle erkennen](ex6-protocol-discovery.md) | 40–50 Min. | HMI-Registerzuordnung; Grenzen der Fernerkennung |
| 7 | [Ein geroutetes VLAN aufbauen und prüfen](ex7-segmentation-firewall.md) | 90–120 Min. | Eine geroutete Anfrage verfolgen; Antworten und neue Verbindungen |

**Optionale Diode-Zusatzübung, 35–45 Minuten:** Fahre in [Übung 7](ex7-segmentation-firewall.md) mit **Zusatzübung: Ein Altsystem mit der Diode mikrosegmentieren** fort. Verwende die neu ergänzte geroutete Diode, um den Engineering-Zugriff zu erhalten, einen anderen Client zu sperren und neue Verbindungen des Endpunkts zu begrenzen. Die Aufgabe beginnt im flachen Ausgangszustand und enthält eine eigene Bereinigung; der Aufbau von VLAN 70 ist keine Voraussetzung.

## Verbindung zum Kurs

Die Kursanlage ist ein Beispiel zur Wasseraufbereitung. `abb-800xa` übernimmt die Leitfunktion, `wago-plc2a` und `wago-plc2b` die Aufbereitungsfunktionen und `ctrlx-plc3` die Pumpen. `schneider-plc4` umfasst auch Prozess- und Entsorgungsfunktionen. Dies sind die Lehrrollen des Handouts. Prüfe einen Netzwerkfluss, bevor du ihn als belegte Steuerungsabhängigkeit bezeichnest.

| Thema im Kurshandout | Lab-Ergebnis |
| --- | --- |
| Ethernet, VLANs und Grenzen (PDF-S. 38–52) | Übung 1: Den tatsächlichen SPS-Anschluss erklären |
| ARP-Angriffe und Gegenmassnahmen (S. 61–64) | Übung 2: Belege, Schutz und Wiederherstellung unterscheiden |
| Schleifenvermeidung und Verfügbarkeit (S. 87–97) | Übung 3: Einen intakten Reservepfad begründen |
| Link Aggregation und Erfassungsumfang von Mitschnitten (S. 85–86, 109, 154–156) | Übung 4: Fehlende Pakete im Mitschnitt und gemeinsame Ausfallpunkte erklären |
| Fernzugriff (S. 100–104) | Übung 5: Den benötigten Lieferantenzugriff begrenzen |
| Protokollschichten und lokale Erkennung (S. 36–41, 50) | Übung 6: Belege für Netzwerk, Dienst und Prozessfunktion unterscheiden |
| Anlagenzonen, Steuerungsabhängigkeiten und Logs (S. 128–148, 162–164) | Übung 7: Zonen begründen, einen Conduit umsetzen und den Ausgangszustand wiederherstellen |
| Mikrosegmentierung eines ungepatchten Entsorgungssystems (S. 149–152) | Diode-Zusatzübung: Zugriff an der Grenze eines Endpunkts begrenzen und beide Richtungen prüfen |

Die Angaben beziehen sich auf die PDF-Seiten in **OT Network Security – Design and Defend handout**; aufgedruckte Foliennummern weichen später ab. Nutze beim Selbststudium die nummerierte Reihenfolge. Im Präsenzkurs kann die Kursleitung LACP vor RSTP einführen und Übungen 3–4 zusammen bearbeiten, sobald beide Begriffe bekannt sind.

In diesem Praktikum konfigurierst du weder MRP, VPN/MFA, 802.1X noch industrielle DPI. Auch Echtzeiteigenschaften werden nicht geprüft. Diese Themen aus dem Handout behandelst du in Diskussionen und Entwürfen, sofern die Kursleitung keine gesondert geprüfte praktische Umgebung bereitstellt.

## Werkzeuge ohne Netzwerk-CLI-Vorkenntnisse nutzen

- Ein **Terminal** führt Befehle auf dem benannten Lab-Gerät aus. Prüfe den Namen vor jedem Block. Linux-Befehle wie `ip` und RouterOS-Befehle mit `/interface` gehören in unterschiedliche Terminals.
- Gib Befehle zeilenweise ein und lies das Ergebnis. Die Markdown-Begrenzer mit drei Backticks werden nicht eingegeben. Stoppe bei unerwarteten Fehlern, bevor du weitere Konfigurationszeilen ausführst.
- Ein Wireshark-**Anzeigefilter** gehört in die Filterleiste, nicht ins Terminal. Er begrenzt sichtbare Pakete; er erzeugt keinen Verkehr und ändert kein Netzwerk.
- Gliedere jede Übergabe in **Vorhersage → Beobachtung → Erklärung → nächste Handlung**. Notiere Gerät, Interface, Zeitpunkt und Frame-Nummer oder Befehlsausgabe. «Nicht beobachtet» ist mit angegebenem Zeitfenster ein gültiges Ergebnis.

## Lab-Fakten gegen falsche Schlüsse

- Die SPS und Zellenrechner starten in **VLAN 10 und `10.0.0.0/8`**. Ihre Knotennamen bleiben bestehen, wenn du in Übung 7 die Konfiguration änderst.
- Die **DMZ verwendet VLAN 60, `2.2.2.0/24`**. Adressen wie `1.1.1.1` und `2.2.2.2` sind innerhalb des Labs simuliert.
- sw-dist und sw-acc1 haben einen **aktiven LACP-Bond mit zwei Mitgliedern**. Ein stabiler Datenstrom nutzt normalerweise ein Mitglied.
- Betriebszustand, aufgezeichnete Pakete und erfolgreiche Anwendungsantworten belegen unterschiedliche Sachverhalte. Eine Port-Bezeichnung, ein Diagramm oder ein Hostname ersetzt diese Prüfungen nicht.
- Toolbox speichert deinen LNE-Entwurf und konfiguriert keine Geräte. Übung 7 baut VLAN 70 in **deinem eigenen Lab** auf und stellt danach den flachen Ausgangszustand wieder her. Sie ändert einen Endpunkt, zwei Switches und das Gateway. Die [Vorbereitung des praktischen Labs](../lab-practical.md) richtet den Checkpoint für die Wiederherstellung ein.

## Jede Übung abschliessen

Schliesse deine Capture- und Terminal-Tabs. Nach Übung 3 Extra 2 muss sw-acc2 ether12 wieder aktiv sein; prüfe ursprüngliche Portrollen und Kommunikation. Stoppe in Übung 2 zusätzlich Hilfsprogramm und Listener. Prüfe vor dem Weitergehen die ursprüngliche Gateway-MAC und den Forwarding-Wert. Die Bereinigung in Übung 7 ist auch ohne Extras zwingend.

Der vorhandene Assistent kennt die ausgewählte Übung. Nenne die Aufgabe, den tatsächlich ausgeführten Befehl und dessen Ausgabe. Ergänze bei einem Mitschnitt den Anzeigefilter und die Frame-Nummer. Warte auf die Antwort, bevor du aus einem anderen Tab fragst. Vergleiche den Rat mit deinen Belegen und der lokalen Anleitung.

Siehe die [Filter- und Fehlerreferenz](appendix.md).
