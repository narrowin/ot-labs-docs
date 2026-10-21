# Übung 3: Den aktiven Spanning Tree erklären

**Zeitbedarf:** 30–40 Minuten. **Lab:** dein eigenes `ot-sec-flat`-Lab. **Niveau:** angeleitete praktische Übung.

Bestimme die gewählte Root Bridge aus Betriebsdaten. Erkläre anschliessend, wie RSTP eine Schleife im Switch-Dreieck verhindert. Diese Übung verändert weder Prioritäten noch Ports oder Kabel.

**Lernziele:** Bridge-Mitgliedschaft, gewählte Root und Portrollen bestimmen, BPDUs auswerten und das Zusammenspiel von RSTP und LACP für Redundanz erklären.

## Dein Instandhaltungsauftrag

Ein Techniker meldet: «Das redundante Kabel steckt, aber ein Port leitet nicht weiter. Soll ich die Weiterleitung aktivieren?» Kläre, ob ein Fehler vorliegt oder RSTP den Port gezielt am Weiterleiten hindert, um eine Schleife zu vermeiden. Werden beide Pfade zur Weiterleitung gezwungen, kann eine Switching-Schleife entstehen.

**Status lesen:** **RSTP** (Rapid Spanning Tree Protocol) wählt schleifenfreie Pfade. Die **Root Bridge** ist der Referenz-Switch, nicht zwingend das IP-Gateway. Ein **Root Port** führt zu dieser Referenz; ein **Alternate Port** ist ein Bereitschaftspfad. **BPDUs** sind die Abstimmungsnachrichten der Switches. Bereitschaftsports können Kontrollnachrichten austauschen, ohne normale Anwendungsdaten weiterzuleiten.

**Vorhersage:** Kann eine physisch intakte Verbindung absichtlich keine Anwendungsdaten transportieren?

## Vorbereitung

Beende die ARP-Wiederherstellung aus [Übung 2](ex2-arp-spoofing.md). Öffne [Netmap mit hervorgehobenen Switches](/network/?search=sw-) und bei Bedarf die konfigurierten Terminals für [sw-dist](/network/terminal?node=sw-dist), [sw-acc1](/network/terminal?node=sw-acc1), [sw-acc2](/network/terminal?node=sw-acc2) und [gw](/network/terminal?node=gw-firewall). Dort läuft RouterOS, nicht Linux.

[Spanning-Tree-Instanz 0 öffnen](/network/?viewMode=spanning-tree&stpInstanceId=0&dataType=All). Vergleiche angezeigte Root und Portrollen mit den Betriebsbefehlen unten.

??? info "Du verwendest die eigenständige Dokumentation?"
    Öffne **Netmap** deines zugewiesenen Live-Labs, wähle **Spanning Tree** im Ansichtsmenü und Instanz **0**. Suche den genannten Switch und wähle im Aktionsmenü **Open terminal** oder **Capture packets**. Für Aufgabe 3 wählst du sw-dist **ether8**. Links in der Übungsansicht behalten Netzwerk und Snapshot automatisch bei.

## Aufgabe 1: Physische Links und Bridge-Ports unterscheiden

Auf **sw-dist**:

```routeros
/interface bridge print detail
/interface bridge port print
/interface bonding print detail
```

Notiere `protocol-mode`, `vlan-filtering`, Bridge-Namen und Bridge-Ports. Ordne die Kabel in Netmap den logischen Ports zu.

**Kontrollpunkt:** Die Bridge heisst `bridge`; RSTP und VLAN-Filtering sind aktiv. `bond-sw-acc1` ist ein Bridge-Port. Dessen physische Mitglieder `ether9` und `ether10` bilden eine logische Verbindung, keine zwei unabhängigen Bridge-Ports. `ether12` führt zu gw, `ether8` zu sw-acc2.

## Aufgabe 2: Die gewählte Root Bridge bestimmen

Auf **jedem Switch** und anschliessend auf **gw**:

```routeros
/interface bridge monitor bridge once
```

Notiere für alle vier Geräte `bridge-id`, `root-bridge`, `root-bridge-id`, `root-port` und `root-path-cost`.

**Kontrollpunkt:** Alle Geräte melden dieselbe Root-ID. Die Root selbst meldet `root-bridge: yes`, Kosten 0 und keinen Root-Port. Im unveränderten Lab ist **gw** die Root, nicht sw-dist. Prüfe deine Ausgabe und entscheide nicht nach der Position im Diagramm.

??? info "Hinweis: Gleiche Prioritäten"
    Die Bridge-ID besteht aus Priorität und MAC. Die niedrigere Priorität gewinnt; bei Gleichstand die niedrigere MAC. `print detail` zeigt die Konfiguration, `monitor` das Ergebnis der Wahl.

## Aufgabe 3: BPDU und Betriebszustand vergleichen

[Öffne den sw-dist-Link zu sw-acc2](/network/capture?node=sw-dist&interface=ether8). Wähle **Start Capture**, warte etwa zehn Sekunden und filtere in Wireshark `stp`.

Wähle ein RSTP-Paket. Klappe **Spanning Tree Protocol** auf und suche **Root Identifier**, **Root Path Cost** und **Bridge Identifier**. Vergleiche die angekündigte Root mit Aufgabe 2. Die Bridge-ID des Senders muss nicht die Root-ID sein.

**Kontrollpunkt:** BPDUs sind sichtbar und kündigen dieselbe Root wie `monitor` an. Ihre Ziel-MAC ist `01:80:c2:00:00:00`. Notiere Frame-Nummer und beide IDs. Schliesse den Mitschnitt-Tab.

??? info "Hinweis: Keine BPDUs"
    Leere den Filter und kontrolliere sw-dist `ether8` Richtung sw-acc2. Warte mehrere periodische Übertragungen ab. Erzeuge keine Topologieänderung. Ein falsches Interface oder ein Filter beweist nicht, dass RSTP fehlt.

## Aufgabe 4: Die Schleifenvermeidung finden

Auf **sw-acc1**:

```routeros
/interface bridge port monitor [find interface=bond-sw-dist] once
/interface bridge port monitor [find interface=ether10] once
```

Auf **sw-acc2**:

```routeros
/interface bridge port monitor [find interface=ether11] once
/interface bridge port monitor [find interface=ether12] once
```

**Kontrollpunkt:** Im Ausgangszustand ist der Bond von sw-acc1 sein Root-Port. sw-acc2 verwendet `ether12` als Root-Port. sw-acc2 `ether11` zu sw-acc1 ist ein **Alternate-Port** mit `forwarding: no`. Das andere Kabelende kann Designated und Forwarding sein. Notiere die Rollen an beiden Kabelenden, statt die Verbindung als ausgefallen zu bezeichnen.

Erkläre, welche unterschiedlichen Probleme LACP und RSTP lösen. Schliesse anschliessend die Terminals.

## Deine Übergabe zur Wartung

Entscheide für sw-acc2 ether11 zwischen «wie vorgesehen belassen» und «untersuchen». Begründe mit physischem Zustand, RSTP-Rolle und gemeinsam bestätigter Root-ID. Benenne, was ein Umschalttest messen müsste: Wie lange ist die Anwendung bei einem Ausfall unterbrochen, und wann funktioniert die Kommunikation wieder? Ein Reservepfad im Diagramm beweist keine gemessene Wiederherstellungszeit.

## Lösung

??? success "Vergleiche deine Belege"
    gw meldet die Bridge-ID, mit der es die Root-Wahl gewinnt. Bei der gemeinsamen Priorität `0x8000` gewinnt seine administrative MAC mit Endung `09:11` gegen sw-dist `09:12`, sw-acc1 `09:13` und sw-acc2 `09:14`. Der Live-Monitor muss dies bestätigen; Änderungen können eine andere Root ergeben.

    sw-dist erreicht gw über `ether12`. Die Access-Switches erreichen sw-dist über ihre jeweiligen Root-Ports. Der zusätzliche Link sw-acc1–sw-acc2 würde eine Schleife schliessen. Deshalb leitet der Alternate-Port von sw-acc2 keine normalen Anwendungsdaten weiter. RSTP behandelt den LACP-Bond als einen logischen Port. Eine redundante Verbindung kann physisch intakt sein, während RSTP die Datenweiterleitung an einem Ende verhindert.

## Extra 1: Einen Switch-Tausch vor dem Ausfall prüfen

**Optional, 10 Minuten.** Ein Ersatz-Switch erhält RSTP-Standardeinstellungen. Der Wartungsplan nimmt an, dass die bisherigen Weiterleitungspfade erhalten bleiben.

1. Prüfe mit den vier Bridge-IDs aus Aufgabe 2, ob Priorität oder MAC die aktuelle Root bestimmt hat.
2. Lies auf **sw-acc2** `/interface bridge print detail`. Vergleiche konfigurierte Priorität und betriebliche Root-ID.
3. Ersetze auf Papier die heutige Root durch einen Switch mit gleicher Priorität, aber grösserer MAC als die anderen drei. Bleibt derselbe physische Standort Root? Ändere keine Geräte.
4. Lege fest, was du vor und nach dem Austausch prüfst: die vorgesehene Root Bridge, die von allen Switches gemeldete Root Bridge, die Root- und Alternate-Ports sowie die Erreichbarkeit der Anwendung.

**Kontrollpunkt:** Gleiche Prioritäten überlassen die Wahl den MAC-Adressen; Ersatzhardware kann diese Wahl ändern. Stimme die vorgesehene Root Bridge und die Ersatz-Root mit dem Netzwerkverantwortlichen ab. Lege ihre Prioritäten vor einem realen Umbau gezielt fest. Wähle die vorgesehene Root anhand der Topologie und der Verfügbarkeitsanforderungen; ihre Position in der Zeichnung reicht als Begründung nicht aus. Feste Lab-MACs machen das aktuelle Ergebnis reproduzierbar; reale Ersatzgeräte brauchen eine eigene Prüfung.

## Extra 2: Einen intakten Link ohne Datenweiterleitung diagnostizieren

**Optional, 10 Minuten.** Auf **sw-acc2**:

```routeros
/interface ethernet monitor ether11 once
/interface bridge port monitor [find interface=ether11] once
```

Warte zehn Sekunden und wiederhole den Bridge-Port-Befehl. Vergleiche `role`, `forwarding` und `tx-rx-bpdu`.

**Kontrollpunkt:** Der Ethernet-Link läuft, während RSTP Alternate/Non-Forwarding meldet. Die BPDU-Zähler steigen. Steuerverkehr hält die Schleifenvermeidung aktiv; das ist kein abgezogenes Kabel. Ein Mitschnitt kann ankommende Daten zeigen, obwohl die Bridge sie an diesem Port verwirft. Ihr Empfang beweist keine Weiterleitung.

Schreibe zwei Sätze mit physischem Zustand, RSTP-Rolle und der Veränderung eines Zählers. Schliesse die Terminals. Weiter mit [Übung 4](ex4-link-aggregation.md).

## Weiterforschen

- [MikroTik RSTP: Monitoring und Wahl](https://help.mikrotik.com/docs/spaces/ROS/pages/21725254/Spanning+Tree+Protocol).
- [Wireshark-STP-Felder](https://www.wireshark.org/docs/dfref/s/stp.html): Suche `stp.root`, `stp.bridge` und `stp.flags`.
