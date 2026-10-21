# Übung 4: Datenströme durch einen LACP-Bond verfolgen

**Zeitbedarf:** 30–40 Minuten. **Lab:** dein eigenes `ot-sec-flat`-Lab. **Niveau:** angeleitete praktische Übung.

Weise nach, dass zwei physische Links einen aktiven LACP-Bond bilden. Finde danach den physischen Port, über den ein bestimmter Datenstrom läuft. Die einzelnen Ports eines Bonds werden im Folgenden als Mitglieder bezeichnet. Verwende den LACP-Bond in deinem eigenen flachen Lab. Vergleiche den Verkehr, während beide Mitglieder aktiv sind.

**Lernziele:** LACP-Aushandlung prüfen, Verteilung von Datenströmen beobachten, den Sende-Hash erklären sowie Redundanz und benötigte Belege für einen begrenzten Umschalttest beurteilen.

## Dein Monitoring-Auftrag

Eine Zelle hat zwei Uplink-Kabel. Ein Mitschnitt auf einem Kabel zeigt nur einen Teil des erwarteten Verkehrs, obwohl beide Kabel intakt sind. Entscheide, ob Datenverkehr fehlt oder der Mitschnitt nur eines der beiden Bond-Mitglieder erfasst.

Ein **Bond** verbindet physische Links zu einer logischen Verbindung. **LACP** (Link Aggregation Control Protocol) prüft, welche Mitglieder zusammenarbeiten können. Ein **Flow** ist ein zusammengehöriger Paketstrom; die **Hash-Policy** wählt anhand von Adressfeldern ein Mitglied. Sie verteilt normalerweise nicht abwechselnd einzelne Pakete auf beide Kabel.

**Vorhersage:** Bedeutet «zwei aktive Mitglieder», dass ein Mitschnitt auf jedem beliebigen Mitglied alle Pakete sieht?

## Vorbereitung

Schliesse [Übung 3](ex3-spanning-tree.md) ab. Öffne die Terminals von [sw-dist](/network/terminal?node=sw-dist) und [sw-acc1](/network/terminal?node=sw-acc1). Erzeuge Verkehr auf [`wago-plc2a`](/network/terminal?node=wago-plc2a-vlan10) und [`ctrlx-plc3`](/network/terminal?node=ctrlx-plc3-vlan20). Beide hängen an sw-acc1. `wago-plc2b` hängt an sw-acc2 und eignet sich deshalb nicht als zweite Quelle für diesen Bond-Test.

| Ende | Logischer Bond | Physische Mitglieder |
| --- | --- | --- |
| sw-dist | `bond-sw-acc1` | `ether9`, `ether10` |
| sw-acc1 | `bond-sw-dist` | `ether11`, `ether12` |

Die Kabel verbinden sw-dist ether9 mit sw-acc1 ether11 sowie ether10 mit ether12. Erstelle nacheinander an beiden Mitgliedern einen Mitschnitt und erzeuge dafür jeweils denselben Testverkehr. Die Links zum Paketmitschnitt verwenden einen gemeinsamen Tab.

??? info "Du verwendest die eigenständige Dokumentation?"
    Suche in **Netmap** deines zugewiesenen Live-Labs nach den stabilen Switch-/Endpunktnamen oben. Wähle im Aktionsmenü **Open terminal**. Wähle für **Capture packets** auf sw-dist zuerst **ether9**, dann **ether10**, wie beschrieben. Links in der Übungsansicht behalten Netzwerk und Snapshot bei und wählen Knoten/Port vor.

## Aufgabe 1: Beide Enden prüfen

Auf **sw-dist**:

```routeros
/interface bonding print detail where name=bond-sw-acc1
/interface bonding monitor bond-sw-acc1 once
```

Auf **sw-acc1**:

```routeros
/interface bonding print detail where name=bond-sw-dist
/interface bonding monitor bond-sw-dist once
```

Notiere Modus, Mitglieder, Hash-Verfahren, aktive und inaktive Ports sowie die lokale System-ID und die System-ID des Partners.

**Kontrollpunkt:** Modus `802.3ad`, beide Mitglieder aktiv, `transmit-hash-policy=layer-2-and-3`. Jede Partner-ID entspricht der lokalen System-ID des anderen Endes. Lies die tatsächlichen IDs ab; sie müssen nicht den festen Bridge-MACs aus Übung 3 entsprechen.

??? info "Hinweis: Bond konfiguriert, Mitglied inaktiv"
    Stoppe den Verkehrsvergleich und zeige die Monitor-Ausgabe der Kursleitung. Konfiguration allein beweist keine erfolgreiche Aushandlung. Setze keinen Switch zurück und importiere keine Startkonfiguration erneut.

## Aufgabe 2: LACP am ersten Mitglied untersuchen

[Zeichne sw-dist ether9 auf](/network/capture?node=sw-dist&interface=ether9). Wähle **Start Capture**, warte etwa zehn Sekunden und filtere `lacp` in Wireshark.

Klappe **Link Aggregation Control Protocol** auf. Vergleiche die **Actor**- und **Partner**-System-IDs mit Aufgabe 1. Prüfe die Zustandsflags **Synchronization**, **Collecting** und **Distributing**.

**Kontrollpunkt:** LACP-Frames sind sichtbar. Bei einem korrekt ausgehandelten Mitglied sind die Flags für Synchronisation, Empfang und Senden gesetzt. Die Ethernet-Zieladresse ist `01:80:c2:00:00:02`. Notiere eine Frame-Nummer.

## Aufgabe 3: Zwei kleine Datenströme zuordnen

Lass den ersten Mitschnitt laufen. Auf **`wago-plc2a`**:

```bash
ping -c 4 10.10.0.1
```

Auf **`ctrlx-plc3`**:

```bash
ping -c 4 10.10.0.1
```

Warte einige Sekunden und filtere nur Anfragen:

```wireshark
icmp.type == 8 && (ip.src == 10.10.0.11 || ip.src == 10.20.0.11)
```

Notiere die Zahl der Anfragen pro Quelle und das aufgezeichnete Mitglied. Schliesse den ersten Mitschnitt-Tab. [Zeichne sw-dist ether10 auf](/network/capture?node=sw-dist&interface=ether10), starte den Mitschnitt und wiederhole beide Ping-Befehle und den Filter.

**Kontrollpunkt:** Bei stabilem Bond bleiben die Anfragen einer Quelle normalerweise auf einem Mitglied. Die Datenströme der beiden Quellen können über unterschiedliche Mitglieder oder über dasselbe Mitglied laufen. Erwarte keine gleiche Verteilung und keinen Wechsel nach jedem Paket. Trenne Anfragen und Antworten: Jeder sendende Switch trifft seine eigene Auswahl.

## Aufgabe 4: Den Bond im Spanning Tree finden

Auf **sw-dist**:

```routeros
/interface bridge port monitor [find interface=bond-sw-acc1] once
```

Vergleiche diesen logischen Port mit den beiden aktiven Mitgliedern aus Aufgabe 1. Sieht RSTP eine Bridge-Verbindung oder zwei unabhängige Verbindungen?

**Kontrollpunkt:** Der logische Bond ist ein weiterleitender Bridge-Port. Seine zwei aktiven physischen Mitglieder transportieren die beobachteten Datenströme, während RSTP den Bond als eine Verbindung behandelt. Ein gleichbleibender Datenstrom nutzt normalerweise ein Mitglied; mehrere Datenströme können beide Mitglieder nutzen. Die wenigen Ping-Pakete zeigen die Zuordnung, nicht den verfügbaren Durchsatz.

Schliesse den Mitschnitt und die Terminals, sofern du kein Extra bearbeitest.

## Deine Übergabe zum Monitoring

Notiere beide aktiven Mitglieder und wo die Anfragen jeder Testquelle erschienen. Benenne einen Ort für den Mitschnitt, an dem du den Austausch zum Gateway unabhängig vom gewählten Bond-Mitglied erfassen kannst. Erkläre, warum weder ungleiche Verkehrsmengen noch vier erfolgreiche Pings ein Kapazitäts- oder Failover-Problem belegen.

## Lösung

??? success "Vergleiche deine Belege"
    Der Bond ist an beiden Enden aktiv; die gegenseitigen System-IDs stimmen überein. LACP prüft, welche Mitglieder im Bond zusammenarbeiten können. Das Hash-Verfahren bestimmt, über welches Mitglied ein Datenstrom gesendet wird. Die aktuelle Einstellung verwendet MAC- und IP-Informationen. Bei unverändertem Bond bleiben Anfragen desselben Endpunktpaars auf einem Mitglied. Eine zusätzliche Quelle kann anders zugeordnet werden, garantiert aber keine gleichmässige Verteilung.

    Beim Durchgang liefen `ctrlx-plc3`-Anfragen über sw-dist ether9 und `wago-plc2a`-Anfragen über ether10. Notiere dein Ergebnis; diese Zuordnung ist nicht dauerhaft garantiert. RSTP betrachtet den logischen Bond, LACP dessen physische Mitglieder.

## Extra 1: Den blinden Fleck im Mitschnitt finden

**Optional, 10 Minuten.** Ein Analyst zeichnet nur sw-dist ether9 auf und folgert, dass ein dort fehlender SPS-Datenfluss gestoppt sei.

1. Nutze die Anzahl der aufgezeichneten Anfragen aus Aufgabe 3. Hat ether9 allein die Anfragen jeder Quelle gesehen? Das Hash-Verfahren kann die Datenströme beider Testquellen demselben Mitglied zuordnen; das beweist keine vollständige Abdeckung.
2. [Zeichne den Gateway-Uplink auf](/network/capture?node=sw-dist&interface=ether12). Starte den Mitschnitt und wiederhole die vier Gateway-Pings von `wago-plc2a` und `ctrlx-plc3`. Verwende den Anfragefilter aus Aufgabe 3.
3. Vergleiche die sichtbaren Quellen mit dem Mitschnitt eines einzelnen Bond-Mitglieds. Markiere den Beobachtungspunkt in deinen Pfadnotizen.

**Kontrollpunkt:** Auf ether12 sind die Gateway-Anfragen beider Testquellen sichtbar. Dieser Mitschnittpunkt eignet sich für diese Datenflüsse, erfasst aber nicht jede direkte SPS-zu-SPS-Kommunikation im selben VLAN. Welche Pakete du erfassen kannst, hängt vom Verkehrspfad ab. Für die Überwachung eines realen Bonds musst du alle relevanten Mitglieder und Richtungen erfassen oder einen geeigneten gemeinsamen Mitschnittpunkt verwenden. Einen physischen Netzwerk-TAP oder eine Port-Spiegelung richtest du in diesem Lab nicht ein.

Schliesse den Mitschnitt vor der nächsten Aufgabe.

## Extra 2: Entscheiden, welchen Ausfall die Redundanz abdeckt

**Optional, 10 Minuten.** Bei einer Wartung soll zuerst ein Uplink-Kabel ersetzt und danach sw-dist gewartet werden. Bestimme mit Netmap und den Bond-/Root-Port-Ausgaben, welche Kommunikation während jedes Schritts weiterlaufen kann.

Ergänze diese Tabelle als **Vorhersage** anhand von Netmap und deinem beobachteten Bond-Zustand:

| Vorgesehener Ausfall | Was könnte verbleiben? | Was muss noch geprüft werden? |
| --- | --- | --- |
| Ein Bond-Mitglied | Das andere Mitglied zwischen denselben Switches | Ausfallerkennung und tatsächliche Anwendungsunterbrechung |
| Beide Bond-Mitglieder | RSTP-Pfad über sw-acc2 | Bereitschaft des Alternativpfads und Wiederherstellung der Anwendungskommunikation |
| sw-dist selbst | Access-Switches können sich weiter erreichen | Das Gateway hängt an sw-dist: Sein Pfad fällt aus |

**Kontrollpunkt:** Zwei Kabel zwischen denselben Switches schaffen keine unabhängigen Switches, Stromversorgungen oder Gateway-Pfade. Erkläre die erwartete Unterbrechung vor einer Freigabe der vorgeschlagenen Produktionswartung. Lege fest, woran du die Funktion der Anwendung erkennst und welche Schritte zur Wiederherstellung nötig sind. Verwende gemessene Lab-Ergebnisse statt konfigurierter Timer, wenn du eine Wiederherstellungszeit angibst.

Schliesse die Terminals. Weiter mit [Übung 5](ex5-reconnaissance.md).

## Weiterforschen

- [MikroTik Bonding und Sende-Hashes](https://help.mikrotik.com/docs/spaces/ROS/pages/8323193/Bonding): Suche `layer-2-and-3` und die Monitoring-Beispiele.
- [Wireshark-LACP-Felder](https://www.wireshark.org/docs/dfref/l/lacp.html): Suche `lacp.actor.sysid`, `lacp.actor.key` und `lacp.actor.state.collecting`.
