# OT-Netzwerke verstehen & den OT-Netzwerkdschungel lichten

**Workshop**

**Dozenten:**

- [Martin Scheu](mailto:martin.scheu@switch.ch) - OT Security Engineer, [SWITCH](https://switch.ch)
- [Mischa Diehm](mailto:mischa.diehm@narrowin.ch) - CTO und Network Engineer, [Narrowin](https://narrowin.ch)

---

## Workshop-Materialien

<a href="slides/CPH2025%20-%20Unfold%20the%20OT%20Network%20Jungle%20-%20Diehm%20-%20Scheu%20.pdf" class="md-button md-button--primary">Workshop-Folien herunterladen (PDF)</a>

---

## Überblick

### Über diesen Workshop

Operational-Technology-Netzwerke (OT) unterscheiden sich grundlegend von herkömmlichen IT-Netzwerken. Sie verbinden industrielle Steuerungssysteme, SPSen, HMIs und Feldgeräte, die kritische Infrastrukturen und Fertigungsprozesse steuern. Dennoch haben viele Organisationen Schwierigkeiten, Transparenz über diese Umgebungen zu gewinnen, was sie anfällig für Sicherheitsbedrohungen macht und Compliance-Lücken hinterlässt.

Dieser praxisorientierte Workshop adressiert die Herausforderung, OT-Netzwerke mit praktischen Open-Source-Tools zu verstehen und abzusichern. Sie lernen, wie Sie die Komplexität von OT-Protokollen bewältigen, Geräte und Datenflüsse identifizieren, blinde Flecken aufdecken und eine effektive Netzwerk-Sicherheitsüberwachung für industrielle Umgebungen implementieren.

Im Gegensatz zu herstellerzentrierten Ansätzen betont dieser Workshop pragmatische Lösungen, die sofort in realen OT-Umgebungen eingesetzt werden können.

### Was macht OT anders?

OT-Netzwerke stellen einzigartige Herausforderungen dar:

* Veraltete Protokolle, die ohne Sicherheit im Hinterkopf entwickelt wurden (Modbus, S7, CIP, BACnet)
* Lange Lebenszyklen mit Geräten, die nicht einfach gepatcht oder ersetzt werden können
* Echtzeitanforderungen, bei denen Verfügbarkeit Vorrang vor Vertraulichkeit hat
* Eingeschränkte Netzwerktransparenz durch flache Architekturen und unverwaltete Switches
* Heterogene Umgebungen mit mehreren Herstellern und Protokollen
* IT/OT-Konvergenz, die neue Angriffsflächen schafft

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

1. Stellen Sie sicher, dass Sie ein GitHub-Konto haben
2. Lesen Sie die [Laboranleitung](lab.md#erste-schritte) für Umgebungs-Setup-Optionen
3. **Wichtig:** Starten Sie Ihren Codespace möglichst vor dem Workshop. Der erste Setup dauert 5-10 Minuten zum Herunterladen und Konfigurieren der Container-Images. Nachfolgende Starts dauern unter 1 Minute.
4. Optional: Machen Sie sich mit grundlegenden Linux-Kommandozeilen- und Netzwerkkonzepten vertraut

---

## Was Sie mitnehmen werden

* Praktische Erfahrung mit Open-Source-OT-Laborumgebungen
* Praxiswissen zur OT-Protokollanalyse
* Erkennungs-Use-Cases, die auf Ihre Umgebung anwendbar sind
* Praktische Laborumgebung, die Sie nach dem Workshop weiterverwenden können
* Netzwerk mit Kollegen, die ähnliche OT-Security-Herausforderungen haben

---

## Fragen?

Bei technischen Fragen oder besonderen Anforderungen kontaktieren Sie die Referenten vor dem Workshop:

* Martin Scheu: martin.scheu@switch.ch
* Mischa Diehm: mischa.diehm@narrowin.com

Wir freuen uns auf die Zusammenarbeit mit Ihnen im Workshop.
