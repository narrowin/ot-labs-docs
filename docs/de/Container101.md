# Container101

Die Workshop-Appliance betreibt simulierte SPSen, Switches und Überwachungswerkzeuge. Container verpacken diese Komponenten, damit die Kursleitung ein wiederholbares Lab bereitstellen kann. Du verwendest das laufende Lab im Browser.

## Dein Workshop-Ablauf

1. Öffne die zugewiesene LNE-URL und melde dich an.
2. Wähle gemäss [Laboranleitung](lab.md) das studentische Live-Netz.
3. Öffne [Übung 1](exercises/ex1-vlan-discovery.md). Ihre Links wählen das benannte Terminal, den Mitschnitt oder die Visualisierung aus.
4. Führe nach jeder Übung das Aufräumen aus und schliesse deren Werkzeug-Tabs.

Die Kursleitung hat Appliance und Lab-Netz bereits bereitgestellt. Dein Laptop benötigt einen Browser und Zugriff auf die zugewiesene URL; eine lokale Container-Installation oder ein GitHub-Konto sind nicht nötig. Bereitstellung und Appliance-Resets übernimmt die Kursleitung.

## Images, Container und persistente Daten

| Objekt | Bedeutung | Lab-Beispiel |
| --- | --- | --- |
| Image | Verpackte Anwendungsdateien und Vorgaben zum Erstellen eines Containers | Software für einen SPS-Simulator |
| Container | Laufende Instanz mit eigenen Prozessen, Dateisystemansicht und Netzwerkschnittstellen | Der Lab-Knoten WAGO A |
| Persistente Daten | Dateien in einem Volume oder eingebundenen Host-Verzeichnis ausserhalb der beschreibbaren Container-Schicht | Gespeicherter Geräte- oder Anwendungszustand |

Ein Image ist ein Ausgangspunkt. Ein laufender Container besitzt zusätzlich aktuelle Prozesse, Verbindungen und Konfiguration. Ein Neustart stellt den Ausgangszustand der Übung nicht automatisch wieder her. Persistente Daten können auch das Neuerstellen eines Containers überleben. [Docker erklärt die Volume-Persistenz](https://docs.docker.com/engine/storage/volumes/).

## Container und virtuelle Maschinen

Eine virtuelle Maschine führt ein Gastbetriebssystem mit eigenem Kernel aus. Linux-Container teilen normalerweise den Host-Kernel und isolieren Prozesse und Ressourcen. Der Workshop kann beides verwenden: Die Appliance kann eine VM sein, in der Lab-Komponenten laufen. Manche Netzwerkgeräte-Container starten intern ein virtualisiertes Netzwerkbetriebssystem.

| Frage | Container | Virtuelle Maschine |
| --- | --- | --- |
| Was startet? | Isolierte Anwendungsprozesse | Ein Gastbetriebssystem |
| Was wird geteilt? | Host-Kernel | Physische Hardware über einen Hypervisor |
| Bedeutet Neustart ein sauberes Lab? | Abhängig von gespeichertem Zustand und Startkonfiguration | Abhängig von Festplatten, Snapshots und Startkonfiguration |

Startzeit und Ressourcenbedarf hängen von der Komponente ab. Ein laufender Container allein beweist weder Anwendungsbereitschaft noch einen vollständigen simulierten Protokollumfang. Die Übungen prüfen Antworten, dekodierte Pakete und Anwendungsverhalten.

## Was Containerlab beiträgt

Containerlab liest eine Topologiedefinition mit Knoten, Software-Images und Verbindungen. Es erstellt die Verbindungen des Labs. Damit lässt sich eine Topologie leichter reproduzieren als durch manuelles Verkabeln jedes virtuellen Geräts.

Ein Knotenname identifiziert eine Komponente; er erzwingt keine VLAN- oder Firewall-Regel. Beispielsweise behält `ctrlx-plc3-vlan20` seinen historischen Namen im aktuellen flachen Ausgangszustand. Verwende die [Netzwerktabelle der Laboranleitung](lab.md#die-beiden-netzebenen-verstehen) und die Kontrollpunkte der Übungen, um die tatsächliche Konfiguration festzustellen.

## Welches Terminal hast du geöffnet?

| Terminal | Ausführungskontext |
| --- | --- |
| **Open terminal** bei einem Knoten | Im benannten Gerät: Linux auf einem WAGO-Endpunkt, RouterOS auf Switch oder Gateway |
| Appliance-Konsole der Kursleitung | Auf dem Host, der Docker, Lab-Knoten und Plattformdienste betreibt |

Die Übungen nennen das Zielgerät über jedem Befehlsblock. Behalte dessen Namen beim Arbeiten im Blick. Wird ein Terminal getrennt, notiere Knoten und Fehler vor einem Neustart: Dieser kann Diagnosebelege und temporäre Konfiguration in deinem Lab löschen.

## Optionale Demonstration durch die Kursleitung: Laufendes Lab ansehen

Bitte die Kursleitung, diese lesenden Befehle auf der **Konsole der zugewiesenen Appliance** zu zeigen. Sie verwenden deren bestehende Docker-Installation. Ein SPS- oder Switch-Terminal ist ein anderer Kontext.

Laufende Lab-Container auflisten:

```bash
docker ps --filter label=containerlab=ot-sec-flat --format 'table {{.Names}}\t{{.Status}}'
```

Finde `clab-ot-sec-flat-wago-plc2a-vlan10` und ordne ihn WAGO A in Netmap zu. Prüfe dann ausschliesslich dessen Namen und Image:

```bash
docker inspect --format '{{.Name}} {{.Config.Image}}' clab-ot-sec-flat-wago-plc2a-vlan10
```

**Kontrollpunkt:** Du unterscheidest den Laufstatus des Containers, sein verpacktes Image und das Anwendungsverhalten. Ein erfolgreicher WAGO-Ping oder eine WebVisu-Antwort liefert andere Belege als Dockers Status `Up`.

## Was passiert mit dem Zustand?

Das Schliessen eines Browser-Tabs beendet die Werkzeugsitzung; es setzt die Appliance nicht zurück. Gerätekonfiguration und Anwendungsdaten können zwischen Sitzungen erhalten bleiben. Räume gemäss jeder Übung auf und prüfe deren Kontrollpunkt.

Übungen 1–6 verwenden den flachen Ausgangszustand deines eigenen Labs. Übung 7 baut darin vorübergehend ein geroutetes VLAN auf und stellt danach den Ausgangszustand wieder her. Die Kursleitung bereitet dafür einen [privaten Checkpoint mit Wiederherstellung](lab-practical.md) vor. Ein historischer Snapshot im Explorer zeigt gesammelte Netzwerkdaten; er stellt keinen Appliance-Zustand wieder her.

## Was dieses Lab vermittelt

Virtuelle Verbindungen transportieren echte Ethernet-Frames und Protokollnachrichten, die deine Capture-Werkzeuge untersuchen können. Simulatoren ermöglichen wiederholbare Netzwerk- und Sicherheitsbeobachtungen ohne Eingriff in Produktionsgeräte. Timing, Protokollumfang und Firmware-Verhalten hängen weiterhin von der installierten Komponente ab; sie belegen kein hartes Echtzeit- oder Sicherheitsverhalten einer physischen Anlage.

## Die Konzepte vertiefen

- [Docker: Was ist ein Container?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/): Vergleiche einen Prozess-Container mit einer VM.
- [Docker-Volumes](https://docs.docker.com/engine/storage/volumes/): Erkläre, warum persistente Dateien einen Container überleben können.
- [Containerlab-Topologiedefinition](https://containerlab.dev/manual/topo-def-file/): Finde die Abschnitte `nodes` und `links`, die Geräte und Verbindungen eines Labs beschreiben.

Weiter mit der [Laboranleitung](lab.md) oder [Übung 1](exercises/ex1-vlan-discovery.md).
