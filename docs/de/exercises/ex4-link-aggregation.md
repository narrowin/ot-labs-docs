# Übung 4: Datenströme durch einen LACP-Bond verfolgen

**Zeit:** 25–35 Minuten. **Lab:** aktuelles studentisches `ot-sec-flat`. **Niveau:** Fortgeschritten.

Weise nach, dass zwei physische Links einen aktiven LACP-Bond bilden. Finde danach das Mitglied, über das ein bestimmter Datenstrom läuft. Das aktuelle flache Lab besitzt LACP; eine separate segmentierte Topologie ist nicht nötig. Lass in diesem gemeinsam genutzten Lab alle Mitgliedsports aktiviert.

**Lernziele:** LACP-Aushandlung prüfen, Verteilung von Datenströmen beobachten, den Sende-Hash erklären sowie Redundanz und benötigte Belege für einen begrenzten Umschalttest beurteilen.

## Vorbereitung

Schliesse [Übung 3](ex3-spanning-tree.md) ab. Öffne die Terminals von [sw-dist](/network/terminal?node=sw-dist) und [sw-acc1](/network/terminal?node=sw-acc1). Erzeuge Verkehr auf [WAGO A](/network/terminal?node=wago-plc2a-vlan10) und [CtrlX](/network/terminal?node=ctrlx-plc3-vlan20). Beide hängen an sw-acc1. WAGO B hängt an sw-acc2 und eignet sich deshalb nicht als zweite Quelle für diesen Bond-Test.

| Ende | Logischer Bond | Physische Mitglieder |
| --- | --- | --- |
| sw-dist | `bond-sw-acc1` | `ether9`, `ether10` |
| sw-acc1 | `bond-sw-dist` | `ether11`, `ether12` |

Die Kabel verbinden sw-dist ether9 mit sw-acc1 ether11 sowie ether10 mit ether12. Zeichne jeweils ein Mitglied auf und wiederhole dieselben Befehle am zweiten. Die Capture-Links verwenden einen gemeinsamen Tab.

??? info "Du verwendest die eigenständige Dokumentation?"
    Suche in **Netmap** deines zugewiesenen Live-Labs nach den stabilen Switch-/Endpunktnamen oben. Wähle im Aktionsmenü **Open terminal**. Wähle für **Capture packets** auf sw-dist zuerst **ether9**, dann **ether10**, wie beschrieben. Reader-Links behalten Netzwerk und Snapshot bei und wählen Knoten/Port vor.

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

Notiere Modus, Mitglieder, Hash-Verfahren, aktive/inaktive Ports sowie lokale und Partner-System-ID.

**Kontrollpunkt:** Modus `802.3ad`, beide Mitglieder aktiv, `transmit-hash-policy=layer-2-and-3`. Jede Partner-ID entspricht der lokalen System-ID des anderen Endes. Lies die tatsächlichen IDs ab; sie müssen nicht den festen Bridge-MACs aus Übung 3 entsprechen.

??? info "Hinweis: Bond konfiguriert, Mitglied inaktiv"
    Stoppe den Verkehrsvergleich und zeige die Monitor-Ausgabe der Kursleitung. Konfiguration allein beweist keine erfolgreiche Aushandlung. Setze keinen Switch zurück und importiere keine Startkonfiguration erneut.

## Aufgabe 2: LACP am ersten Mitglied untersuchen

[Zeichne sw-dist ether9 auf](/network/capture?node=sw-dist&interface=ether9). Wähle **Start Capture**, warte etwa zehn Sekunden und filtere `lacp` in Wireshark.

Klappe **Link Aggregation Control Protocol** auf. Vergleiche die **Actor**- und **Partner**-System-IDs mit Aufgabe 1. Prüfe die Zustandsflags **Synchronization**, **Collecting** und **Distributing**.

**Kontrollpunkt:** LACP-Frames sind sichtbar. Ein gesund ausgehandeltes Mitglied meldet synchronisiert, empfangend und sendend. Die Ethernet-Zieladresse ist `01:80:c2:00:00:02`. Notiere eine Frame-Nummer.

## Aufgabe 3: Zwei kleine Datenströme zuordnen

Lass den ersten Mitschnitt laufen. Auf **WAGO A**:

```bash
ping -c 4 10.10.0.1
```

Auf **CtrlX**:

```bash
ping -c 4 10.10.0.1
```

Warte einige Sekunden und filtere nur Anfragen:

```wireshark
icmp.type == 8 && (ip.src == 10.10.0.11 || ip.src == 10.20.0.11)
```

Notiere die Zahl der Anfragen pro Quelle und das aufgezeichnete Mitglied. Schliesse den ersten Capture-Tab. [Zeichne sw-dist ether10 auf](/network/capture?node=sw-dist&interface=ether10), starte den Mitschnitt und wiederhole beide Ping-Befehle und den Filter.

**Kontrollpunkt:** Bei stabilem Bond bleiben die Anfragen einer Quelle normalerweise auf einem Mitglied. Die Quellen können verschiedene oder dasselbe Mitglied treffen. Erwarte keine gleiche Verteilung und keinen Wechsel nach jedem Paket. Trenne Anfragen und Antworten: Jeder sendende Switch trifft seine eigene Auswahl.

## Aufgabe 4: Raten richtig einordnen

Auf **sw-dist**:

```routeros
/interface monitor-traffic ether9,ether10 once
/interface bridge port monitor [find interface=bond-sw-acc1] once
```

Vergleiche die momentanen Raten mit deinem gefilterten Testverkehr. Warum können beide Zähler durch Hintergrundverkehr steigen, obwohl dein Datenstrom nur ein Mitglied nutzt?

**Kontrollpunkt:** Der Bond leitet als ein Bridge-Port weiter. Mehrere Datenströme können die Gesamtkapazität nutzen; diese Pings messen aber keinen Durchsatz. Ein einzelner Datenstrom erhält nicht garantiert die doppelte Link-Bandbreite. Aktive Mitglieder messen auch keine Umschaltzeit. Diese Übung verspricht weder unterbrechungsfreie noch subsekundenschnelle Umschaltung.

Lies `link-monitoring`, `mii-interval` und `lacp-rate` in der Bond-Konfiguration. Erkläre den erwarteten Unterschied beim Verlust eines Mitglieds beziehungsweise beider. Ein verbleibendes Mitglied kann den logischen Bond ohne RSTP-Topologieänderung weiterleiten lassen. Der Verlust des ganzen Bonds kann einen alternativen RSTP-Weg erfordern. Lege für einen von der Kursleitung genehmigten isolierten Umschalttest begrenzten Ping, aktive Mitglieder vorher/während/nachher, beobachteten Verlust und Wiederherstellung beider Mitglieder fest. Leite aus Timern keine gemessene Ausfallzeit ab und deaktiviere keine Links im geteilten Lab.

Schliesse Capture und Terminals, sofern du kein Extra bearbeitest.

## Lösung

??? success "Vergleiche deine Belege"
    Der Bond ist an beiden Enden aktiv; die gegenseitigen System-IDs stimmen überein. LACP beschreibt die Eignung der Mitglieder, der Sende-Hash wählt das Datenmitglied. Die aktuelle Einstellung verwendet MAC- und IP-Informationen. Bei unverändertem Bond bleiben Anfragen desselben Endpunktpaars auf einem Mitglied. Eine zusätzliche Quelle kann anders zugeordnet werden, garantiert aber keine gleichmässige Verteilung.

    Beim Durchgang liefen CtrlX-Anfragen über sw-dist ether9 und WAGO-A-Anfragen über ether10. Notiere dein Ergebnis; diese Zuordnung ist nicht dauerhaft garantiert. RSTP betrachtet den logischen Bond, LACP dessen physische Mitglieder.

## Extra 1: Ändert die Paketgrösse das Mitglied?

**Optional, 10 Minuten.** Sage die Wirkung einer anderen Nutzlast bei `layer-2-and-3` voraus.

1. Zeichne ether9 auf und führe auf **WAGO A** aus:

    ```bash
    ping -c 2 -s 32 10.10.0.1
    ping -c 2 -s 128 10.10.0.1
    ```

2. Schliesse den Mitschnitt, wiederhole ihn an ether10 und filtere `icmp.type == 8 && ip.src == 10.10.0.11`.
3. Vergleiche Mitglied und Frame-Länge für beide Grössen.

**Kontrollpunkt:** Die Nutzlast ändert die Frame-Länge, aber nicht die MAC-/IP-Eingaben dieses Hash-Verfahrens. Beide Grössen sollten bei stabilem Bond dasselbe Anfragemitglied verwenden. Topologie- oder Mitgliedszustandsänderungen machen den Vergleich ungültig. Vergleiche bei gleicher Nutzlast mit dem Access-Frame aus Übung 1, um das vier Byte lange VLAN-Tag zu finden.

## Extra 2: Beide Enden genauer als nur anhand einer grünen Anzeige prüfen

**Optional, 10 Minuten.** Verwende beide Monitor-Ausgaben und je ein LACP-Paket von beiden Mitgliedern.

1. Erstelle eine Tabelle mit lokaler System-ID, Partner-ID, Mitglied, Actor-Key und Partner-Key.
2. Prüfe die gegenseitigen System-IDs. Mitglieder desselben Aggregats sollen auf einem Switch einen konsistenten Actor-Key ankündigen. Die Gegenseite hat ihren eigenen Key; lokale und entfernte Key-Nummern müssen allgemein nicht gleich sein.
3. Prüfe Synchronization, Collecting und Distributing. Wie könnte eine falsche Partner-ID ein falsch verbundenes Kabel verraten, obwohl Ethernet Link-up meldet?

**Kontrollpunkt:** Du unterscheidest physischen Link, LACP-Einigung und tatsächliche Datenweiterleitung anhand dreier Belege. Interface-Änderungen sind nicht nötig. Schliesse die Werkzeug-Tabs. Weiter mit [Übung 5](ex5-reconnaissance.md).

## Weiterforschen

- [MikroTik Bonding und Sende-Hashes](https://help.mikrotik.com/docs/spaces/ROS/pages/8323193/Bonding): Suche `layer-2-and-3` und die Monitoring-Beispiele.
- [Wireshark-LACP-Felder](https://www.wireshark.org/docs/dfref/l/lacp.html): Suche `lacp.actor.sysid`, `lacp.actor.key` und `lacp.actor.state.collecting`.
