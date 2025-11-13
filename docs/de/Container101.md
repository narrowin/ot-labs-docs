# Container101

## Warum diese Seite existiert

Sie sind ein OT-Fachmann und arbeiten mit SPSen, HMIs, Industrienetzwerken und Protokollen wie Modbus oder PROFINET. Vielleicht haben Sie von Docker und Containern gehört, aber sie nie wirklich gebraucht. Jetzt verwenden wir für diesen Workshop Containerlab, das auf Container-Technologie basiert.

Diese Seite erklärt, was Container sind, warum sie perfekt für unseren OT-Sicherheits- und Netzwerk-Workshop sind, und warum es nichts zu befürchten gibt.

## Die einfache Wahrheit über Container

**Container sind einfach eine Möglichkeit, Software zu verpacken und auszuführen, die es leicht macht, dieselbe Einrichtung jedes Mal zu starten, zu stoppen und zu wiederholen.**

Das ist alles. Keine Magie. Keine versteckte Komplexität. Alles, was die Software benötigt, ist zusammen verpackt, und sie läuft auf jedem System gleich. Als hätte man ein einsatzbereites Gerät, anstatt jedes Mal Komponenten zusammenzubauen.

## Warum Container für diesen Workshop wichtig sind

Wir bauen Netzwerksicherheitslabore mit SPSen, HMIs, Switches und Sicherheitsüberwachungswerkzeugen auf. Das geben uns Container:

**Schnell und wiederholbar**  
Anstatt Software manuell auf jeder Maschine zu installieren, können wir ein komplettes Labor in Sekunden starten. Neustart nötig? Es dauert Sekunden, nicht Minuten oder Stunden.

**Sauber und isoliert**  
Jede Komponente (SPS-Simulator, HMI, Netzwerkwerkzeug) läuft separat. Wenn etwas kaputt geht, starten Sie einfach dieses eine Teil neu.

**Einfach zu teilen**  
Wir können Ihnen genau dasselbe Labor-Setup wie allen anderen geben. Keine "funktioniert auf meiner Maschine"-Probleme.

**Leichtgewichtig**  
Sie können mehrere SPSen, Switches und Werkzeuge auf Ihrem Laptop ausführen, ohne dass er schmilzt. Container verwenden viel weniger Speicher und CPU als traditionelle virtuelle Maschinen.

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

Sie erhalten Entwicklungswerkzeuge auf Unternehmensniveau ohne Kosten. Keine Budgetgenehmigung nötig, um mit OT-Sicherheitslaboren zu lernen und zu experimentieren.

Führen Sie komplette industrielle Netzwerksimulationen in der Cloud aus, zugänglich von jedem Gerät. Perfekt zum Lernen in Ihrem eigenen Tempo, wo auch immer Sie sind.

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

Wenn Sie mit echten Firmendaten arbeiten müssen, verwenden Sie lokale Entwicklungsumgebungen auf firmengeführter Infrastruktur, nicht Cloud-Dienste.

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

**Für diesen Workshop:** Denken Sie an Container als "leichtgewichtige VMs, die sofort starten." Das ist nahe genug dran.

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

1. **Eine Topologie-Datei öffnen** - Eine einfache Textdatei, die Ihr Labornetzwerk beschreibt (welche SPSen, Switches, HMIs Sie möchten)
2. **"Deploy" in VS Code klicken** - Die Erweiterung startet Ihr gesamtes Labor
3. **Ihr Labor verwenden** - Mit Geräten verbinden, Sicherheitstests durchführen, Datenverkehr erfassen
4. **"Destroy" klicken, wenn fertig** - Alles wird sauber entfernt

Das ist der komplette Arbeitsablauf. Vier Schritte. Einfach.

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
Nein. Wir verwenden die Containerlab VS Code-Erweiterung, die Ihnen Buttons zum Klicken statt Befehle zum Tippen gibt.

**F: Können Container industrielle Protokolle wie Modbus, S7 oder PROFINET handhaben?**  
Ja. Container können jede Software ausführen, einschliesslich industrieller Protokollstacks und Simulatoren. Der Netzwerkverkehr funktioniert genau wie Sie es erwarten.

**F: Was ist, wenn etwas abstürzt?**  
Stellen Sie einfach diese Komponente neu bereit. Es ist schnell und bringt Sie sofort zurück in einen sauberen Zustand.

**F: Ist das wie die "Cloud"?**  
Nein. Alles läuft lokal auf Ihrem Laptop. Kein Internet erforderlich (nach der anfänglichen Einrichtung). Sie haben volle Kontrolle.

**F: Kann ich das nach dem Workshop für meine eigenen Tests verwenden?**  
Absolut. Das ist eines der Ziele - Ihnen ein Werkzeug zu geben, das Sie verwenden können, um Ihre eigenen OT-Sicherheits- und Netzwerk-Testlabore aufzubauen.

**F: Wird das meinen Computer durcheinanderbringen?**  
Nein. Container sind isoliert. Wenn Sie sie entfernen, sind sie vollständig weg. Nichts bleibt auf Ihrem System zurück.

## Warum speziell Containerlab?

Containerlab wurde von Netzwerkingenieuren für Netzwerkingenieure entwickelt. Es löst das Problem, Netzwerkkonfigurationen und -topologien schnell zu testen.

**Warum es perfekt für OT-Arbeit ist:**

- Definieren Sie Ihre gesamte Netzwerktopologie in einer einfachen Textdatei
- Unterstützt sowohl Netzwerkgeräte (Switches, Router) als auch Endgeräte (SPSen, HMIs)
- Bereitstellung in Sekunden, genauso schnell wieder abbauen
- Funktioniert nahtlos mit VS Code
- Labore einfach teilen - senden Sie einfach die Topologie-Datei an Kollegen
- Kostenlos und Open Source

Netzwerkingenieure bei grossen Herstellern wie Nokia, Cisco und Juniper verwenden Containerlab täglich. Es ist kampferprobt und zuverlässig.

## Reale Perspektive: Alter Weg vs. Neuer Weg

**Traditioneller Ansatz:**  
Physisches Labor aufbauen oder mehrere VMs konfigurieren. Dauert Stunden oder Tage. Bricht leicht. Schwer zu teilen. Schmerzhaft zurückzusetzen.

**Container-Ansatz:**  
Schreiben Sie eine Textdatei, die Ihr Netzwerk beschreibt. Klicken Sie auf Deploy. Labor bereit in 30 Sekunden. Etwas kaputt? Erneut bereitstellen in 30 Sekunden. Teilen Sie die Datei mit Kollegen, die sofort die identische Einrichtung erhalten.

**Der Unterschied:**  
Sie verbringen Zeit mit dem Lernen von OT-Sicherheit und -Netzwerken, anstatt mit der Infrastruktur zu kämpfen.

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
Virtuelle Netzwerkverbindungen verhalten sich genau wie echte Ethernet-Kabel. Industrielle Protokolle funktionieren identisch. Netzwerkverkehr sieht für Ihre Überwachungswerkzeuge gleich aus.

**Leistung ist ausreichend**  
Ein moderner Laptop führt bequem 10-20 containerisierte Geräte aus. Das reicht für realistische OT-Netzwerktopologien mit mehreren Zonen.

**Zustand kann bewahrt werden**  
Behalten Sie Ihre SPS-Konfiguration zwischen Sitzungen bei oder starten Sie jedes Mal frisch. Ihre Wahl.

**Timing ist vorhersehbar**  
Nicht geeignet für harte Echtzeit-Steuerung, aber perfekt zum Testen von Netzwerkprotokollen, Sicherheitsüberwachung und Angriffsszenarien.

## Wichtige Einschränkungen (ehrlich sein)

Seien wir klar darüber, was dies ist und was nicht:

**Nicht für Produktion**  
Dies sind Simulationen und Testumgebungen. Setzen Sie dies niemals in Ihr tatsächliches Anlagennetzwerk ein.

**Keine Echtzeit-Steuerung**  
Perfekt zum Testen von Protokollen und Sicherheit. Nicht geeignet für sicherheitskritische Steuerungssysteme, die harte Echtzeit-Garantien erfordern.

**Nicht identisch mit echten Geräten**  
Wir verwenden Simulatoren, die sich wie Ihre SPSen und HMIs verhalten, aber sie führen nicht Ihre genaue Produktions-Firmware aus.

**Fazit:**  
Für Training und Tests - perfekt.  
Für Produktionssteuerung - verwenden Sie echte, zertifizierte Hardware.

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
