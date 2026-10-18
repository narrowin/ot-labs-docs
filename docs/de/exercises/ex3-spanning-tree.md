# Übung 3: Den aktiven Spanning Tree erklären

**Zeit:** 25–35 Minuten. **Lab:** aktuelles studentisches `ot-sec-flat`. **Niveau:** Fortgeschritten.

Bestimme die gewählte Root Bridge aus Betriebsdaten. Erkläre anschliessend, wie RSTP eine Schleife im Switch-Dreieck verhindert. Diese Übung verändert weder Prioritäten noch Ports oder Kabel.

**Lernziele:** Bridge-Mitgliedschaft, gewählte Root und Portrollen bestimmen, BPDUs auswerten und das Zusammenspiel von RSTP und LACP für Redundanz erklären.

## Vorbereitung

Beende die ARP-Wiederherstellung aus [Übung 2](ex2-arp-spoofing.md). Öffne [Netmap mit hervorgehobenen Switches](/network/?search=sw-) und bei Bedarf die konfigurierten Terminals für [sw-dist](/network/terminal?node=sw-dist), [sw-acc1](/network/terminal?node=sw-acc1), [sw-acc2](/network/terminal?node=sw-acc2) und [gw](/network/terminal?node=gw-firewall). Dort läuft RouterOS, nicht Linux.

[Spanning-Tree-Instanz 0 öffnen](/network/?viewMode=spanning-tree&stpInstanceId=0&dataType=All). Vergleiche angezeigte Root und Portrollen mit den Betriebsbefehlen unten.

??? info "Du verwendest die eigenständige Dokumentation?"
    Öffne **Netmap** deines zugewiesenen Live-Labs, wähle **Spanning Tree** im Ansichtsmenü und Instanz **0**. Suche den genannten Switch und wähle im Aktionsmenü **Open terminal** oder **Capture packets**. Für Aufgabe 3 wählst du sw-dist **ether8**. Reader-Links behalten Netzwerk und Snapshot automatisch bei.

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

Auf **jedem Switch** und anschliessend **gw**:

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

**Kontrollpunkt:** BPDUs sind sichtbar und kündigen dieselbe Root wie `monitor` an. Ihre Ziel-MAC ist `01:80:c2:00:00:00`. Notiere Frame-Nummer und beide IDs. Schliesse den Capture-Tab.

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

**Kontrollpunkt:** Im Ausgangszustand ist sw-acc1's Bond sein Root-Port. sw-acc2 verwendet `ether12` als Root-Port. sw-acc2 `ether11` zu sw-acc1 ist ein **Alternate-Port** mit `forwarding: no`. Das andere Kabelende kann Designated und Forwarding sein. Notiere beide Enden, statt den Link als ausgefallen zu bezeichnen.

Erkläre, welche unterschiedlichen Probleme LACP und RSTP lösen. Schliesse anschliessend die Terminals.

## Lösung

??? success "Vergleiche deine Belege"
    gw kündigt die gewinnende Bridge-ID an. Bei der gemeinsamen Priorität `0x8000` gewinnt seine administrative MAC mit Endung `09:11` gegen sw-dist `09:12`, sw-acc1 `09:13` und sw-acc2 `09:14`. Der Live-Monitor muss dies bestätigen; Änderungen können eine andere Root ergeben.

    sw-dist erreicht gw über `ether12`. Die Access-Switches erreichen sw-dist über ihre jeweiligen Root-Ports. Der zusätzliche Link sw-acc1–sw-acc2 würde eine Schleife schliessen. Deshalb leitet sw-acc2's Alternate-Port keine gewöhnlichen Daten weiter. RSTP behandelt den LACP-Bond als einen logischen Port. Ein redundanter Link kann physisch funktionieren und absichtlich nicht weiterleiten.

## Extra 1: Die Root-Wahl als Detektivaufgabe

**Optional, 10 Minuten.** Verwende die vier Bridge-IDs aus Aufgabe 2.

1. Sortiere sie aufsteigend. Zeige bei jedem Vergleich, welcher Teil entscheidet.
2. Sage die Root voraus, falls sw-acc2 Priorität `0x1000` hätte und alles andere unverändert bliebe. Das ist eine Papieraufgabe; ändere kein Gerät.
3. Vergleiche auf sw-acc2 `/interface bridge print detail` mit dem betrieblichen `root-bridge-id`.

**Kontrollpunkt:** Du erklärst den aktuellen MAC-Gleichstandsentscheid und warum eine niedrigere Priorität diesen überstimmen würde. Eine konfigurierte Priorität allein beweist keine aktuelle Root-Rolle. [Suche die Wahlregeln](https://help.mikrotik.com/docs/spaces/ROS/pages/21725254/Spanning+Tree+Protocol) unter **(R)STP Election Process**.

## Extra 2: Einen gesunden, nicht weiterleitenden Link diagnostizieren

**Optional, 10 Minuten.** Auf **sw-acc2**:

```routeros
/interface ethernet monitor ether11 once
/interface bridge port monitor [find interface=ether11] once
```

Warte zehn Sekunden und wiederhole den Bridge-Port-Befehl. Vergleiche `role`, `forwarding` und `tx-rx-bpdu`.

**Kontrollpunkt:** Der Ethernet-Link läuft, während RSTP Alternate/Non-Forwarding meldet. Die BPDU-Zähler steigen. Steuerverkehr hält die Schleifenvermeidung aktiv; das ist kein abgezogenes Kabel. Ein Mitschnitt kann sogar ankommende Daten am verwerfenden Port zeigen. Ankunft beweist keine Weiterleitung durch die Bridge.

Schreibe zwei Sätze mit physischem Zustand, RSTP-Rolle und einer Zählerbeobachtung. Schliesse die Terminals. Weiter mit [Übung 4](ex4-link-aggregation.md).

## Weiterforschen

- [MikroTik RSTP: Monitoring und Wahl](https://help.mikrotik.com/docs/spaces/ROS/pages/21725254/Spanning+Tree+Protocol).
- [Wireshark-STP-Felder](https://www.wireshark.org/docs/dfref/s/stp.html): Suche `stp.root`, `stp.bridge` und `stp.flags`.
