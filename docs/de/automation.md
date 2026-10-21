# Ausblick Automatisierung: Konfigurationsdrift erkennen

Nach einer Wartung wurde eine Ersatz-SPS dem falschen VLAN zugeordnet. Automatisierung soll Soll und Ist des Ports vergleichen und eine vorgeschlagene Korrektur prüfbar machen. Ein erfolgreicher Automatisierungsjob allein beweist nicht, dass die SPS-Anwendung funktioniert.

**Optional, 15 Minuten nach den Hauptübungen.** Verwende die Switch-Port-Belege aus Übung 1. Softwareinstallation und Geräteänderungen sind nicht nötig.

## Das gewünschte Ergebnis festlegen

Trenne für sw-acc1 ether2 Soll-Aussagen von Beobachtungen:

| Soll-Aussage | Zu vergleichender Beleg |
| --- | --- |
| Der Port bedient `wago-plc2a-vlan10` | Portbeschreibung, Topologie und Endgeräteidentität |
| Ungetaggter Endgeräteverkehr erhält VLAN 10 | PVID und tatsächliche ungetaggte Mitgliedschaft |
| Der Pfad transportiert VLAN 10 zum Gateway | Uplink-Mitgliedschaft und getaggter Ping-Mitschnitt |
| Die Anwendung ist verfügbar | Erfolgreiche, freigegebene Anwendungsanfrage |

Lies im **konfigurierten Terminal von sw-acc1** die ersten beiden Konfigurationsbelege:

```routeros
/interface bridge port print detail where interface=ether2
/interface bridge vlan print detail
```

Ein passender Ausgangszustand ist ein gültiges Ergebnis: Melde «keine Änderung nötig». Nimm auf Papier an, die erste Ausgabe zeige stattdessen PVID 20. Beschreibe die Abweichung, den möglicherweise betroffenen Verkehr und warum ein pauschales Zurücksetzen aller Ports keine geeignete Korrektur wäre.

## Einen prüfbaren Änderungsvorschlag schreiben

Notiere genaues Ziel, beobachteten Altwert, Sollwert, Grund, Verantwortlichen, erwartete Auswirkung, Anwendungsprüfung und Rollback. Die Automatisierung soll eine Änderung verweigern, wenn sich der beobachtete Altwert seit der Prüfung geändert hat. Trenne Zustandsabfrage, Diff-Vorschau, genehmigte Umsetzung und Ergebnisprüfung.

**Kontrollpunkt:** Dein Vorschlag verwaltet nur das vorgesehene Feld, erhält andere Einstellungen und enthält einen Anwendungsbeleg nach der Änderung. Wiederholung einer bereits korrekten Operation soll nichts mehr ändern; das ist **Idempotenz**.

Der Automatisierungskreislauf im Handout lautet Ist-Konfiguration sammeln → mit Soll vergleichen → validieren → anwenden. Diese Übung bereitet die Spezifikation vor; sie installiert kein Werkzeug und behauptet keine ausgeführte automatische Korrektur. Zugangsdaten und Konfigurationssicherungen gehören in den vorgesehenen privaten Speicher der Kursleitung, nicht in Kursnotizen oder Git.

Weiter zur [Übungsübersicht](exercises/index.md).
