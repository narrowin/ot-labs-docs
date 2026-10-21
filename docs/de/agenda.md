# Lab-Reihenfolge für den zweitägigen Kurs

Nutze das Handout **OT Network Security – Design and Defend** für die Theorie und diese Übungen für die Praxis. Plane über beide Tage etwa 5–6½ Stunden für Hauptaufgaben und Diskussion; Pausen und Theorie kommen hinzu. Optionale Zusatzaufgaben können später bearbeitet werden.

## Tag 1: Verbindungen, Grenzen und Verfügbarkeit

1. Wasseraufbereitungsanlage, Folgen von Kontrollverlust und Unterschied zwischen Prozess- und Netzwerkdiagramm einführen. Den [Lab-Zugriff prüfen](lab.md#die-ersten-funf-minuten).
2. Ethernet, IP-Adressierung und VLANs erklären, danach [Übung 1](exercises/ex1-vlan-discovery.md): den tatsächlichen SPS-Netzwerkanschluss nachweisen.
3. ARP und Layer-2-Vertrauen erklären, danach [Übung 2](exercises/ex2-arp-spoofing.md) im eigenen Lab durchführen. Vor dem nächsten Schritt die Bereinigung prüfen.
4. Link Aggregation und RSTP einführen, danach [Übung 3](exercises/ex3-spanning-tree.md) und [Übung 4](exercises/ex4-link-aggregation.md). Reservepfade, Capture-Abdeckung und gemeinsame Ausfallpunkte diskutieren, ohne einen Ausfall auszulösen.

## Tag 2: Fernzugriff, Belege und Zonendesign

1. Lebenszyklus des Fernzugriffs erklären, danach [Übung 5](exercises/ex5-reconnaissance.md): Lieferantenbedarf mit tatsächlicher DMZ-Erreichbarkeit vergleichen.
2. Monitoring-Platzierung und Protokollbelege diskutieren, danach [Übung 6](exercises/ex6-protocol-discovery.md): Anlagenkommunikation und einen kontrollierten Lesezugriff dokumentieren.
3. [Übung 7](exercises/ex7-segmentation-firewall.md) für den Zonenentwurf der Wasseraufbereitung verwenden, selbstständig oder mit Diskussion. Gemischte SPS-Zuständigkeiten vor dem Zeichnen von Zonengrenzen prüfen. Jede Person setzt im eigenen Lab einen kleinen Conduit um, prüft ihn und stellt anschliessend den Ausgangszustand wieder her.
4. Mit installierter gerouteter Diode die optionale **Zusatzübung zur Mikrosegmentierung** in Übung 7 durchführen (35–45 Minuten). Eine Engineering-Quelle zulassen, eine andere sperren und neue Verbindungen des Endpunkts begrenzen; mit der VLAN-Grenze vergleichen und den Ausgangszustand wiederherstellen.
5. Ergebnisse mit Logging, Roadmap und [Automatisierungsausblick](automation.md) des Handouts verbinden. Jede Person benennt eine Verbesserung, Verantwortlichen, Abhängigkeit und Abnahmeprüfung.

## Auswahl für einen kurzen Workshop

Für kurze Veranstaltungen ein vollständiges Ergebnis auswählen, statt alle sieben Übungen zu überfliegen: die zwei Mitschnitte aus Übung 1, den Monitoring-Blindspot aus Übung 4 oder die einzelne Modbus-Transaktion aus Übung 6. Die Gruppenplanung aus Übung 7 ist eigenständig möglich. Die Konfiguration erst beginnen, wenn Zeit und Zugriff für Prüfung und Bereinigung reichen.
