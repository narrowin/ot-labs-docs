# OT-Netzwerke verstehen & den OT-Netzwerkdschungel lichten

**Workshop**

**Dozenten:**

- [Martin Scheu](mailto:martin.scheu@switch.ch) - OT Security Engineer, [SWITCH](https://switch.ch)
- [Mischa Diehm](mailto:mischa.diehm@narrowin.ch) - CTO und Network Engineer, [Narrowin](https://narrowin.ch)

---

## Workshop-Materialien

<a href="slides/CPH2025%20-%20Unfold%20the%20OT%20Network%20Jungle%20-%20Diehm%20-%20Scheu.pdf" class="md-button md-button--primary">Workshop-Folien herunterladen (PDF)</a>

---

## Überblick

### Über diesen Workshop

Operational-Technology-Netzwerke (OT) unterscheiden sich grundlegend von IT-Netzwerken. Sie verbinden industrielle Steuerungssysteme, SPSen, HMIs und Feldgeräte für kritische Infrastrukturen und Fertigungsprozesse. Viele Organisationen haben Schwierigkeiten, Transparenz über diese Umgebungen zu gewinnen, was sie anfällig für Sicherheitsbedrohungen macht und Compliance-Lücken hinterlässt.

Dieser Workshop vermittelt, wie Sie OT-Netzwerke mit praktischen Open-Source-Tools verstehen und absichern. Sie lernen, OT-Protokolle zu bewältigen, Geräte und Datenflüsse zu identifizieren, blinde Flecken aufzudecken und effektive Netzwerk-Sicherheitsüberwachung für industrielle Umgebungen zu implementieren.

Der Workshop fokussiert auf pragmatische Lösungen, die sofort in realen OT-Umgebungen einsetzbar sind.

### Was macht OT anders?

OT-Netzwerke stellen einzigartige Herausforderungen dar:

* Veraltete Protokolle ohne Sicherheitskonzept (Modbus, S7, CIP, BACnet)
* Lange Lebenszyklen, Geräte können nicht einfach gepatcht oder ersetzt werden
* Echtzeitanforderungen, Verfügbarkeit hat Vorrang vor Vertraulichkeit
* Eingeschränkte Netzwerktransparenz durch flache Architekturen und unverwaltete Switches
* Heterogene Umgebungen mit mehreren Herstellern und Protokollen
* IT/OT-Konvergenz schafft neue Angriffsflächen

Das Verständnis dieser Unterschiede ist entscheidend für effektive Netzwerksegmentierung und Sicherheitsüberwachung.

### Lernziele

Am Ende dieses Workshops werden Sie in der Lage sein:

* Gängige OT-Protokolle und ihre Sicherheitsimplikationen zu identifizieren und zu charakterisieren
* Überwachungspunkte basierend auf Risikobewertung und Netzwerkarchitektur auszuwählen
* Traffic-Erfassung und -Weiterleitung für zonenübergreifende Transparenz zu konfigurieren
* Assets zu entdecken und Kommunikationsmuster in OT-Netzwerken abzubilden
* Die Grenzen verschiedener Ansätze zur Netzwerk-Sicherheitsüberwachung in industriellen Umgebungen zu erkennen
* Segmentierungstechniken an IT/OT-Grenzen und Edge-Geräten anzuwenden

---

## Kursanforderungen

**Niveau:** Fortgeschritten, praxisorientierter Workshop

**Was Sie wissen sollten:**

* Netzwerkgrundlagen (IP-Adressierung, VLANs, grundlegendes Routing)
* Kommandozeilen-Grundkenntnisse (ssh, ping, grundlegende Dateioperationen)
* Container-Konzepte hilfreich, aber nicht erforderlich (wir erklären, was Sie brauchen)

**Laborumgebung:**

Das Labor läuft in vorkonfigurierten Cloud- oder lokalen Umgebungen. Siehe die [Laboranleitung](lab.md#erste-schritte) für Setup-Anweisungen.

---

## Vorbereitung vor dem Workshop

Keine Softwareinstallation erforderlich. Das Labor läuft in der Cloud über GitHub Codespaces oder lokal über DevPod.

**Vor dem Workshop:**

1. GitHub-Konto erstellen
2. [Laboranleitung](lab.md#erste-schritte) für Setup-Optionen lesen
3. **Wichtig:** Codespace vor dem Workshop starten. Der erste Start dauert 5-10 Minuten für Download und Konfiguration der Container-Images. Nachfolgende Starts dauern unter 1 Minute.
4. Optional: Grundkenntnisse in Linux-Kommandozeile und Netzwerkkonzepten auffrischen

---

## Was Sie mitnehmen werden

* Praktische Erfahrung mit Open-Source-OT-Laborumgebungen
* Praxiswissen zur OT-Protokollanalyse
* Erkennungs-Use-Cases, die auf Ihre Umgebung anwendbar sind
* Praktische Laborumgebung, die Sie nach dem Workshop weiterverwenden können
* Netzwerk mit Kollegen, die ähnliche OT-Security-Herausforderungen haben

---

## Fragen?

Bei technischen Fragen oder besonderen Anforderungen kontaktieren Sie die Referenten:

* Martin Scheu: martin.scheu@switch.ch
* Mischa Diehm: mischa.diehm@narrowin.com
