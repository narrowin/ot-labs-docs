# Übungen für das studentische Lab

Bearbeite die sieben Übungen der Reihe nach auf der aktuellen `ot-sec-flat`-Appliance. Plane für die Hauptaufgaben etwa 3½–4½ Stunden. Jede Übung enthält eine ausklappbare Lösung und danach **zwei optionale Zusatzteile**. Alle Extras zusammen benötigen weitere 2–2½ Stunden.

## Hier beginnen

1. Öffne dein zugewiesenes Network-Explorer-Lab und wähle das **Live**-Netzwerk.
2. Wähle im Produktmenü **Exercises**, danach Übung 1. Mit **English / Deutsch** wechselst du die Sprache.
3. Die Links öffnen konfigurierte Terminals, gefilterte Netmap, Zieldetails oder den ausgewählten Capture-Port. In Terminal- und Capture-Tabs öffnet der aktive Übungstitel die Anleitung. Lass das Terminal beim Wechsel zum Mitschnitt offen.
4. Versuche zuerst die Beobachtung, bevor du Hinweis oder Lösung öffnest. Notiere Quellknoten, Interface, Befehl/Filter und tatsächliche Belege.

Du benötigst das von der Kursleitung gestartete Lab und konfigurierte Knotenzugangsdaten. Starte keine andere Topologie, installiere keine Erweiterungen und verwende keine alten Standardpasswörter. Bei **unavailable** prüfst du das gewählte Live-Netzwerk und lässt Zugriff/Bindings von der Kursleitung prüfen.

Werkzeug-Links sind für den Network-Explorer-Reader gedacht. Öffne bei Nutzung dieser veröffentlichten Dokumentation das genannte Werkzeug im zugewiesenen Lab und wähle Knoten/Interface manuell. Die simulierten Daten-IPs sind keine direkt vom Laptop erreichbaren Ziele.

## Übungen

| # | Übung | Hauptzeit | Zusatzaufgaben |
| --- | --- | --- | --- |
| 1 | [Getaggten und ungetaggten Verkehr vergleichen](ex1-vlan-discovery.md) | 20–30 Min. | Vier-Byte-Rätsel; Next-Hop-MAC |
| 2 | [Eine ARP-Umleitung nachweisen](ex2-arp-spoofing.md) | 30–40 Min. | Vorfall-Zeitleiste; direkte und geroutete Umleitung |
| 3 | [Den aktiven Spanning Tree erklären](ex3-spanning-tree.md) | 25–35 Min. | Root-Wahl; gesunder nicht weiterleitender Link |
| 4 | [Datenströme durch LACP verfolgen](ex4-link-aggregation.md) | 25–35 Min. | Paketgrösse und Hash; Actor-/Partner-Prüfung |
| 5 | [Zugriff aus der DMZ beurteilen](ex5-reconnaissance.md) | 30–40 Min. | Ausgangspunkte vergleichen; TCP-Portzustände beweisen |
| 6 | [OT-Protokolle erkennen](ex6-protocol-discovery.md) | 35–45 Min. | Registernummerierung; Discovery ohne TCP-Port |
| 7 | [Eine durchsetzbare Policy entwerfen](ex7-segmentation-firewall.md) | 35–45 Min. | Zustandsbehaftete Antworten; vollständiges VLAN-Arbeitsblatt |

## Lab-Fakten gegen falsche Schlüsse

- Das OT-Ausgangsnetz verwendet **VLAN 10 und `10.0.0.0/8`**. Historische Namen wie `vlan20` beweisen keine Segmentierung.
- Die **DMZ verwendet VLAN 60, `2.2.2.0/24`**. Adressen wie `1.1.1.1` und `2.2.2.2` sind innerhalb des Labs simuliert.
- sw-dist und sw-acc1 haben einen **aktiven LACP-Bond mit zwei Mitgliedern**. Ein stabiler Datenstrom nutzt normalerweise ein Mitglied.
- Betriebszustand, Pakete und erfolgreiche Anwendungsantworten sind unterschiedliche Belege. Port-Bezeichnung, Diagramm oder Hostname ersetzen sie nicht.
- Der LNE-Entwurf in Toolbox ist geteilt und konfiguriert keine Geräte. In Übung 7 entwirfst du Änderungen in eigenen Notizen. Der neue VLAN-Zusatz ist bis zu einem isolierten Konfigurationsdurchgang eine Entwurfsaufgabe.

## Jede Übung abschliessen

Schliesse deine Capture- und Terminal-Tabs. Stoppe in Übung 2 zusätzlich Hilfsprogramm und Listener. Prüfe vor dem Weitergehen die ursprüngliche Gateway-MAC und den Forwarding-Wert. Lass geteilte Gerätekonfigurationen und Toolbox-Policies unverändert.

Der vorhandene Assistent kennt die ausgewählte Übung. Nenne Aufgabe, tatsächlichen Befehl/Ausgabe sowie Capture-Filter/Frame-Nummer. Warte auf die Antwort, bevor du aus einem anderen Tab fragst. Vergleiche den Rat mit deinen Belegen und der lokalen Anleitung.

Siehe die [Filter- und Fehlerreferenz](appendix.md).
