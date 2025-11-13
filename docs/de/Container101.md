# Container101

## Warum diese Seite existiert

Sie sind ein OT-Fachmann und arbeiten mit SPSen, HMIs, Industrienetzwerken und Protokollen wie Modbus oder PROFINET. Vielleicht haben Sie von Docker und Containern gehört, aber sie nie wirklich gebraucht. Jetzt verwenden wir für diesen Workshop Containerlab, das auf Container-Technologie basiert.

Diese Seite erklärt, was Container sind, warum sie perfekt für unseren OT-Sicherheits- und Netzwerk-Workshop sind, und warum es nichts zu befürchten gibt.

## Die einfache Wahrheit über Container

**Container sind eine Möglichkeit, Software zu verpacken und auszuführen, die es leicht macht, dieselbe Einrichtung jedes Mal zu starten, zu stoppen und zu wiederholen.**

Das ist alles. Keine Magie. Keine versteckte Komplexität. Alles, was die Software benötigt, ist zusammen verpackt und läuft auf jedem System gleich. Ein einsatzbereites Gerät, statt jedes Mal Komponenten zusammenzubauen.

## Warum Container für diesen Workshop wichtig sind

Wir bauen Netzwerksicherheitslabore mit SPSen, HMIs, Switches und Sicherheitsüberwachungswerkzeugen auf. Das geben uns Container:

**Schnell und wiederholbar**  
Statt Software manuell auf jeder Maschine zu installieren, starten wir ein komplettes Labor in Sekunden. Neustart dauert Sekunden, nicht Minuten oder Stunden.

**Sauber und isoliert**  
Jede Komponente (SPS-Simulator, HMI, Netzwerkwerkzeug) läuft separat. Bei Problemen wird nur dieses eine Teil neu gestartet.

**Einfach zu teilen**  
Alle erhalten genau dasselbe Labor-Setup. Keine "funktioniert auf meiner Maschine"-Probleme.

**Leichtgewichtig**  
Mehrere SPSen, Switches und Werkzeuge laufen auf Ihrem Laptop. Container verwenden viel weniger Speicher und CPU als virtuelle Maschinen.

## GitHub Free Plan: Was Sie bekommen

Dieser Workshop nutzt den kostenlosen GitHub-Plan, der leistungsstarke Werkzeuge zum Lernen und Entwickeln kostenlos bereitstellt.

### GitHub Codespaces

**Was sind Codespaces?**  
Cloudbasierte Entwicklungsumgebungen, die in Ihrem Browser laufen. Denken Sie daran als vollständiger Computer mit VS Code, Docker und allen Workshop-Werkzeugen vorinstalliert, von überall zugänglich.

**Kostenloser Plan beinhaltet:**

- 120 Core-Stunden pro Monat
- 15 GB Speicher
- 4-Core-Maschinen verfügbar

**Für diesen Workshop:**  
Ein 4-Core-Codespace führt diese gesamte Laborumgebung aus. 120 Stunden bedeuten, Sie können das Labor jeden Monat mehrere volle Tage kostenlos ausführen.

### Private Repositories

**Was Sie kostenlos bekommen:**

- Unbegrenzte private Repositories
- Unbegrenzte Mitarbeiter
- Volle Versionskontrolle und Zusammenarbeitsfunktionen

### GitHub Copilot

**Was es ist:**  
KI-gestützter Coding-Assistent, der Ihnen hilft, Code zu schreiben, Konfigurationen zu verstehen und Probleme zu beheben.

**Kostenlos für:**

- Studenten (verifiziert über GitHub Student Developer Pack)
- Maintainer beliebter Open-Source-Projekte
- Kostenlose Testversion für alle verfügbar

**Für diesen Workshop:**  
Copilot kann Ihnen helfen, Topologie-Dateien zu verstehen, Automatisierungsskripte zu schreiben und Netzwerkkonfigurationen zu debuggen. Besonders nützlich beim Erweitern von Laboren nach dem Workshop.

### Warum das für OT-Fachleute wichtig ist

Entwicklungswerkzeuge auf Unternehmensniveau ohne Kosten. Keine Budgetgenehmigung nötig, um mit OT-Sicherheitslaboren zu lernen und zu experimentieren.

Komplette industrielle Netzwerksimulationen in der Cloud, zugänglich von jedem Gerät. Perfekt zum Lernen in Ihrem eigenen Tempo.

### Sicherheitshinweis: Produktionsdaten

**Verwenden Sie niemals Produktions- oder Firmendaten mit Codespaces oder Copilot.**

GitHub Codespaces läuft in der Cloud. GitHub Copilot verarbeitet Code, um Vorschläge zu liefern. Während GitHub Sicherheitsmassnahmen hat, verbieten die Sicherheitsrichtlinien Ihrer Organisation wahrscheinlich das Hochladen sensibler Daten auf externe Dienste.

**Verwenden Sie nicht:**

- Echte SPS-Konfigurationen aus Ihrer Anlage
- Tatsächliche Netzwerkdiagramme mit echten IP-Adressen
- Produktionspasswörter oder Anmeldedaten
- Firmenspezifischen Code oder Konfigurationen
- Alle Daten, die von NDAs oder Vertraulichkeitsvereinbarungen abgedeckt sind

**Dieser Workshop verwendet nur simulierte, generische Labordaten.**  
Perfekt zum Lernen ohne Sicherheitsbedenken.

Für echte Firmendaten verwenden Sie lokale Entwicklungsumgebungen auf firmengeführter Infrastruktur, nicht Cloud-Dienste.

## Container vs. Virtuelle Maschinen: Was Sie wissen müssen

Sie kennen vielleicht virtuelle Maschinen (VMs). Container sind ähnlich, aber viel leichter.

### Virtuelle Maschinen

- Vollständiges Betriebssystem für jede VM
- Braucht Minuten zum Starten
- Verwendet viel Speicher und Festplattenspeicher
- Gute Isolation, aber schwer

### Container

- Teilen das zugrunde liegende Betriebssystem
- Starten in Sekunden
- Verwenden viel weniger Speicher und Festplatte
- Leicht, aber trotzdem ausreichend isoliert für Labore

**Für diesen Workshop:** Container sind leichtgewichtige VMs, die sofort starten.

### Einfache Vergleichstabelle

| Was Sie brauchen | Virtuelle Maschine | Container |
|---------------|-----------------|-----------|
| Einen SPS-Simulator starten | Warten Sie 2-3 Minuten | Bereit in 2-3 Sekunden |
| 10 verschiedene Geräte ausführen | Schwer auf Ihrem Laptop | Komfortabel auf Ihrem Laptop |
| Auf sauberen Zustand zurücksetzen | Langsames Herunterfahren und Neustart | Sofort |
| Exakte Einrichtung mit Kollegen teilen | Grosse VM-Images zu übertragen | Kleine Container-Images |

## Was in diesem Workshop passiert

Sie werden Containerlab über eine VS Code-Erweiterung verwenden. Keine komplexen Befehle zum Tippen. Kein tiefes technisches Wissen erforderlich.

### Was Sie tatsächlich tun werden

1. **Topologie-Datei öffnen** - Textdatei, die Ihr Labornetzwerk beschreibt (SPSen, Switches, HMIs)
2. **"Deploy" in VS Code klicken** - Die Erweiterung startet Ihr Labor
3. **Labor verwenden** - Geräte verbinden, Sicherheitstests durchführen, Datenverkehr erfassen
4. **"Destroy" klicken** - Alles wird sauber entfernt

Vier Schritte. Einfach.

### Was hinter den Kulissen passiert (aber Sie müssen sich nicht darum kümmern)

Wenn Sie "Deploy" klicken, macht Containerlab:

- Liest Ihre Topologie-Datei
- Lädt vorgefertigte Container-Images für Ihre Geräte herunter oder verwendet sie
- Startet jedes Gerät als Container
- Verbindet sie mit virtuellen Netzwerkkabeln
- Gibt Ihnen Zugriff auf jedes Gerät

Wenn Sie fertig sind und "Destroy" klicken, wird alles sauber entfernt. Keine übrig gebliebenen Dateien oder Konfigurationen.

## Häufige Fragen von OT-Fachleuten

**F: Ist das sicher genug für unser Labor?**  
Ja. Container bieten gute Isolation für Training und Tests. Sie werden von Millionen von Entwicklern und Unternehmen weltweit verwendet. Für unseren Workshop sind sie mehr als ausreichend.

**F: Muss ich Docker-Befehle lernen?**  
Nein. Die Containerlab VS Code-Erweiterung bietet Buttons statt Befehle.

**F: Können Container industrielle Protokolle wie Modbus, S7 oder PROFINET handhaben?**  
Ja. Container führen jede Software aus, einschliesslich industrieller Protokollstacks und Simulatoren. Der Netzwerkverkehr funktioniert wie erwartet.

**F: Was passiert bei einem Absturz?**  
Komponente neu bereitstellen. Schnell und sauber.

**F: Ist das die "Cloud"?**  
Nein. Alles läuft lokal auf Ihrem Laptop. Kein Internet erforderlich (nach initialer Einrichtung). Volle Kontrolle.

**F: Kann ich das nach dem Workshop für eigene Tests verwenden?**  
Ja. Genau dafür ist es gedacht - für eigene OT-Sicherheits- und Netzwerk-Testlabore.

**F: Wird das meinen Computer durcheinanderbringen?**  
Nein. Container sind isoliert. Beim Entfernen verschwinden sie vollständig. Nichts bleibt zurück.

## Warum speziell Containerlab?

Containerlab wurde von Netzwerkingenieuren für Netzwerkingenieure entwickelt. Es löst das Problem, Netzwerkkonfigurationen und -topologien schnell zu testen.

**Warum es perfekt für OT-Arbeit ist:**

- Gesamte Netzwerktopologie in einer Textdatei definieren
- Unterstützt Netzwerkgeräte (Switches, Router) und Endgeräte (SPSen, HMIs)
- Bereitstellung in Sekunden, genauso schnell wieder abbauen
- Nahtlose VS Code Integration
- Einfaches Teilen von Laboren - Topologie-Datei an Kollegen senden
- Kostenlos und Open Source

Netzwerkingenieure bei grossen Herstellern wie Nokia, Cisco und Juniper verwenden Containerlab täglich. Es ist kampferprobt und zuverlässig.

## Reale Perspektive: Alter Weg vs. Neuer Weg

**Traditioneller Ansatz:**  
Physisches Labor aufbauen oder mehrere VMs konfigurieren. Dauert Stunden oder Tage. Bricht leicht. Schwer zu teilen. Schmerzhaftes Zurücksetzen.

**Container-Ansatz:**  
Textdatei schreiben, die Ihr Netzwerk beschreibt. Deploy klicken. Labor bereit in 30 Sekunden. Etwas kaputt? Erneut bereitstellen in 30 Sekunden. Datei mit Kollegen teilen, die sofort die identische Einrichtung erhalten.

**Der Unterschied:**  
Zeit für OT-Sicherheit und -Netzwerke lernen, statt mit Infrastruktur kämpfen.

## Die Wahrheit über Lernkurven

**Sie müssen kein Container-Experte werden.**

Was Sie wissen müssen:

- Wie man ein Labor bereitstellt (einen Button klicken)
- Wie man sich mit Geräten im Labor verbindet (wie immer)
- Wie man das Labor zerstört, wenn fertig (einen anderen Button klicken)

Das ist alles. Die Containerlab VS Code-Erweiterung übernimmt alles andere.

**Nach diesem Workshop**, wenn Sie tiefer eintauchen möchten, gibt es ausgezeichnete Ressourcen. Aber fürs Erste konzentrieren Sie sich auf OT-Sicherheit und -Netzwerke. Container-Technologie ist nur das Werkzeug, das die Labore zum Laufen bringt.

## Was Container für OT-Labore funktionieren lässt

Container zeichnen sich durch das Simulieren von OT-Umgebungen aus:

**Netzwerkverhalten ist realistisch**  
Virtuelle Netzwerkverbindungen verhalten sich wie echte Ethernet-Kabel. Industrielle Protokolle funktionieren identisch. Netzwerkverkehr sieht für Überwachungswerkzeuge gleich aus.

**Leistung ist ausreichend**  
Ein moderner Laptop führt bequem 10-20 containerisierte Geräte aus. Ausreichend für realistische OT-Netzwerktopologien mit mehreren Zonen.

**Zustand kann bewahrt werden**  
SPS-Konfiguration zwischen Sitzungen beibehalten oder jedes Mal frisch starten. Ihre Wahl.

**Timing ist vorhersehbar**  
Nicht für harte Echtzeit-Steuerung geeignet, aber perfekt zum Testen von Netzwerkprotokollen, Sicherheitsüberwachung und Angriffsszenarien.

## Wichtige Einschränkungen (ehrlich sein)

Seien wir klar darüber, was dies ist und was nicht:

**Nicht für Produktion**  
Dies sind Simulationen und Testumgebungen. Niemals in tatsächliche Anlagennetzwerke einsetzen.

**Keine Echtzeit-Steuerung**  
Perfekt zum Testen von Protokollen und Sicherheit. Nicht für sicherheitskritische Steuerungssysteme mit harten Echtzeit-Garantien geeignet.

**Nicht identisch mit echten Geräten**  
Simulatoren verhalten sich wie SPSen und HMIs, führen aber nicht Ihre genaue Produktions-Firmware aus.

**Fazit:**  
Für Training und Tests - perfekt.  
Für Produktionssteuerung - echte, zertifizierte Hardware verwenden.

## Ihr Workshop-Lernpfad

**Vor dem Workshop:**

- Stressen Sie nicht wegen Containern. Sie sind nur ein Werkzeug.
- Stellen Sie sicher, dass Docker Desktop und VS Code installiert sind (wir führen Sie).
- Vertrauen Sie darauf, dass Millionen von Menschen dies täglich verwenden.

**Während des Workshops:**

- Konzentrieren Sie sich auf OT-Sicherheit und -Netzwerke, nicht auf Container-Technologie.
- Verwenden Sie die VS Code-Erweiterung - Point-and-Click-Interface.
- Stellen Sie jederzeit Fragen, wenn etwas unklar ist.

**Nach dem Workshop:**

- Sie werden selbstbewusst Ihre eigenen Testlabore aufbauen.
- Erkunden Sie erweiterte Funktionen, wenn interessiert.
- Verstehen Sie, warum Container in OT-Sicherheits- und Netzwerktests beliebt sind.

## Möchten Sie mehr lernen? (Optional)

Wenn Sie nach dem Workshop tiefer eintauchen möchten, haben diese Ressourcen Millionen von Menschen geholfen, Container zu verstehen. Alle sind anfängerfreundlich:

### Sehr empfohlene Videos (hier starten)

**Docker in 100 Seconds** von Fireship  
[https://www.youtube.com/watch?v=Gjnup-PuquQ](https://www.youtube.com/watch?v=Gjnup-PuquQ)  
Schnell, visuell und perfekt, um die Grundlagen zu verstehen.

**you need to learn Docker RIGHT NOW** von NetworkChuck  
[https://www.youtube.com/watch?v=eGz9DS-aIeY](https://www.youtube.com/watch?v=eGz9DS-aIeY)  
Spassig, praktisch und anfängerfreundliche Einführung.

### Offizielle Docker-Ressourcen

**Docker Get Started Guide**  
[https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)  
Offizielle Docker-Dokumentation, gut strukturiert und klar.

**Docker 101 Interactive Tutorial**  
[https://www.docker.com/101-tutorial](https://www.docker.com/101-tutorial)  
Praktisches Tutorial, das Sie in Ihrem Browser oder auf Ihrer Maschine machen können.

**What is a Container? (Docker's explanation)**  
[https://www.docker.com/resources/what-container/](https://www.docker.com/resources/what-container/)  
Dockers eigene anfängerfreundliche Erklärung.

### Containerlab-spezifisch

**Containerlab Quick Start**  
[https://containerlab.dev/quickstart/](https://containerlab.dev/quickstart/)  
Die offizielle Schnellstartanleitung - sehr zugänglich.

**Containerlab VS Code Extension**  
[https://containerlab.dev/manual/vsc-extension/](https://containerlab.dev/manual/vsc-extension/)  
Dokumentation für die Erweiterung, die wir im Workshop verwenden werden.

**Containerlab YouTube Channel**  
[https://www.youtube.com/@containerlab](https://www.youtube.com/@containerlab)  
Video-Tutorials und Demonstrationen.

### Für visuelle Lerner

**Play with Docker**  
[https://labs.play-with-docker.com/](https://labs.play-with-docker.com/)  
Kostenlose Online-Umgebung, um Docker-Befehle auszuprobieren, ohne etwas zu installieren.

**Docker Curriculum by Prakhar Srivastav**  
[https://docker-curriculum.com/](https://docker-curriculum.com/)  
Anfängerfreundliches Tutorial mit klaren Erklärungen und Beispielen.

### Wenn Sie mit VMs vergleichen möchten

**Containers vs Virtual Machines (IBM Technology)**  
[https://www.youtube.com/watch?v=cjXI-yxqGTI](https://www.youtube.com/watch?v=cjXI-yxqGTI)  
Klare Videoerklärung der Unterschiede.

## Fazit für diesen Workshop

Sie müssen kein Container-Experte sein. Sie müssen nur wissen, dass Container ausgereifte, bewährte Technologie sind, die von Millionen von Entwicklern und Unternehmen weltweit verwendet wird.

Die VS Code-Erweiterung übernimmt die schwere Arbeit. Sie konzentrieren sich auf OT-Sicherheit und -Netzwerke.

Container sind einfach das Werkzeug, das uns erlaubt, schnelle, wiederholbare, teilbare OT-Sicherheits- und Netzwerklabore zu bauen. Das ist ihr Zweck hier. Sie machen es gut.

Bereit, in OT-Sicherheit und -Netzwerke einzutauchen? Los geht's.
