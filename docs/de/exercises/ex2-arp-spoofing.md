# Übung 2: Eine ARP-Umleitung nachweisen

**Zeit:** 30–40 Minuten. **Lab:** aktuelles studentisches `ot-sec-flat`. **Niveau:** Fortgeschritten.

Weise eine veränderte IP-MAC-Zuordnung nach, verfolge den umgeleiteten Verkehr und stelle den ursprünglichen Zustand wieder her. Verwende ausschliesslich die drei genannten Lab-Knoten. Die Adressen gehören zum simulierten Netz dieser Appliance.

**Lernziele:** ARPs Vertrauensproblem erklären, eine begrenzte Layer-2-Umleitung durchführen und nachweisen, lesbare Anwendungsdaten erkennen sowie Erkennung und Gegenmassnahmen erklären.

## Vorbereitung

Schliesse [Übung 1](ex1-vlan-discovery.md) ab. Öffne die konfigurierten Terminals; du brauchst weder Passwörter einzugeben noch Software zu installieren:

| Rolle | Terminal | Datenadresse |
| --- | --- | --- |
| Angreifer | [WAGO A](/network/terminal?node=wago-plc2a-vlan10) | `10.10.0.11` |
| Opfer | [WAGO B](/network/terminal?node=wago-plc2b-vlan10) | `10.10.0.12` |
| Empfänger | [DMZ-Jumphost](/network/terminal?node=proxmox-jumphost) | `2.2.2.2` |

Lass die Tabs offen. Die Terminals verwenden Management-Verbindungen; das Experiment läuft über Dateninterface `eth1`. Führe jeden Block auf dem genannten Knoten aus. Der Übungstitel im Werkzeug-Tab öffnet diese Anleitung erneut.

??? info "Du verwendest die eigenständige Dokumentation?"
    Suche in **Netmap** deines zugewiesenen Live-Labs nach jedem stabilen Knotennamen aus der Tabelle. Wähle im Aktionsmenü **Open terminal**. Wähle für den Mitschnitt WAGO A, **Capture packets**, Interface **eth1**. Reader-Links treffen diese Auswahl automatisch und behalten Netzwerk und Snapshot bei.

## Aufgabe 1: Die echte Gateway-Zuordnung festhalten

Auf **WAGO B**:

```bash
ping -c 2 10.10.0.1
ip neigh show 10.10.0.1 dev eth1
```

Notiere die Gateway-MAC, nicht nur den Nachbarzustand (`REACHABLE` oder `STALE`). Auf **WAGO A**:

```bash
cat /sys/class/net/eth1/address
cat /proc/sys/net/ipv4/ip_forward
command -v arp_spoofing-simple.sh timeout
```

Notiere A's MAC und den Forwarding-Wert. Das Hilfsprogramm bewahrt den vorherigen Forwarding-Zustand.

**Kontrollpunkt:** Gateway-MAC und A's MAC unterscheiden sich. Beide Befehle sind vorhanden. Ist die Zuordnung bereits verändert, führe zuerst Aufgabe 5 aus.

## Aufgabe 2: Den normalen Weg beobachten

[Öffne einen Mitschnitt an WAGO A's Datenport](/network/capture?node=wago-plc2a-vlan10&interface=eth1), wähle **Start Capture** und führe auf **WAGO B** aus:

```bash
ping -c 3 2.2.2.2
```

Verwende in Wireshark `icmp && ip.addr == 10.10.0.12`. Warte einige Sekunden, bis die Anzeige nachgezogen hat.

**Kontrollpunkt:** Die Pings antworten, ihre Unicast-Pakete durchqueren A's geswitchten Access-Link normalerweise aber nicht. ARP-Broadcasts können trotzdem sichtbar sein. Ist Opfer-Unicast bereits sichtbar, kontrolliere Zuordnung und Capture-Interface.

## Aufgabe 3: Das ausgewählte Paar umleiten

Starte auf **WAGO A** im Vordergrund:

```bash
timeout -s INT 60 arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
```

Das Hilfsprogramm adressiert nur B und dessen Gateway und endet nach 60 Sekunden automatisch. Mit **Ctrl+C** kannst du früher stoppen. Lass dieses Terminal laufen, während du B verwendest.

Nach etwa fünf Sekunden auf **WAGO B**:

```bash
ip neigh show 10.10.0.1 dev eth1
ping -c 3 2.2.2.2
```

**Kontrollpunkt:** B ordnet `10.10.0.1` jetzt A's MAC zu. Der Mitschnitt auf A enthält B's ICMP-Verkehr. Ein Paket kann beim Empfang und bei der Weiterleitung zweimal erscheinen. ICMP-Redirects und einzelne verlorene Pings sind möglich; sie widerlegen die MAC- und Nutzlastbelege nicht. Ping allein beweist keine Umleitung.

??? info "Hinweis: Timer abgelaufen oder keine Änderung"
    Prüfe das Hilfsprogramm-Terminal auf Fehler. Ist es beendet, wiederhole den zeitlich begrenzten Befehl und anschliessend die Opfer-Befehle. Begrenze den Filter auf B und erweitere den Angriff nicht auf andere Knoten. Exit-Code 124 nach dem Timeout ist normal.

## Aufgabe 4: Einen harmlosen Klartext-Marker finden

Starte auf dem **DMZ-Jumphost**, während das Hilfsprogramm läuft:

```bash
timeout 45 nc -l 2323
```

Lass den Listener warten. Sende auf **WAGO B** einen Marker ohne echte Zugangsdaten:

```bash
printf 'LAB-ARP-DEMO\n' | nc -N -w 3 2.2.2.2 2323
```

Der Empfänger soll `LAB-ARP-DEMO` anzeigen. Filtere im Mitschnitt `tcp.port == 2323`, wähle ein Datenpaket und öffne per Rechtsklick **Follow → TCP Stream**. Finde denselben Marker und schliesse das Stream-Fenster wieder.

**Kontrollpunkt:** Notiere eine Paketnummer mit dem Marker und die veränderte Nachbarzuordnung. Ein offener Port allein beweist keine abgefangenen Anwendungsdaten.

??? info "Hinweis: Kein Marker"
    Starte zuerst den Listener. Wiederhole Aufgabe 3, falls der Timer abgelaufen ist. `tcp.port == 2323 && tcp.len > 0` zeigt Datenpakete. Sind nur SYN-Pakete sichtbar, kontrolliere Listener und Route.

## Aufgabe 5: Wiederherstellen und prüfen

1. Stoppe das Hilfsprogramm auf A gegebenenfalls mit **Ctrl+C**. Warte auf den Prompt und einige Sekunden auf korrigierende ARP-Antworten. Stoppe auch den Listener mit **Ctrl+C**, falls er noch wartet.
2. Aktualisiere auf **WAGO B** nur diesen dynamischen Gateway-Eintrag:

    ```bash
    ip neigh del 10.10.0.1 dev eth1
    ping -c 3 10.10.0.1
    ip neigh show 10.10.0.1 dev eth1
    ```

    Meldet das Löschen einen fehlenden Eintrag, fahre mit Ping fort.
3. Auf **WAGO A**:

    ```bash
    pgrep -a arpspoof
    cat /proc/sys/net/ipv4/ip_forward
    ```

**Kontrollpunkt:** Kein `arpspoof`-Prozess bleibt übrig, Forwarding hat den Ausgangswert und B verwendet bei erfolgreichen Pings wieder die ursprüngliche Gateway-MAC. Ein stilles `pgrep` mit Exit-Code 1 bedeutet: kein Treffer. Bleibt die Zuordnung falsch, stoppe hier und hole die Kursleitung dazu.

Lass den Mitschnitt für Extra 1 offen, falls du weitermachen möchtest. Schliesse sonst Capture- und Terminal-Tabs.

## Lösung

??? success "Vergleiche deine Belege"
    Vorher sendet B gerouteten Verkehr an die echte Gateway-MAC. Gefälschte ARP-Antworten ersetzen diese Zuordnung durch A's MAC. B's geroutete Pakete laufen deshalb über A. Empfänger und Mitschnitt zeigen denselben Klartext-Marker. Nach dem Stoppen und Aktualisieren kehrt die ursprüngliche Zuordnung zurück.

    Das belegt eine nicht authentifizierte ARP-Zuordnung und lesbare TCP-Anwendungsdaten im Lab. Es beweist keine Entschlüsselung verschlüsselter Inhalte. Port Security allein validiert keine IP-MAC-Zuordnung. Dynamic ARP Inspection benötigt vertrauenswürdige Bindings und einen Plan für statisch adressierte OT-Geräte.

## Extra 1: Eine ARP-Vorfall-Zeitleiste erstellen

**Optional, 10 Minuten.** Verwende den offenen Mitschnitt oder wiederhole Aufgaben 2–5.

1. Filtere `arp.opcode == 2 && arp.src.proto_ipv4 == 10.10.0.1`.
2. Klappe **Address Resolution Protocol** auf. Notiere **Sender IP address**, **Sender MAC address** und je eine Frame-Nummer vor, während und nach dem Experiment. Fehlt eine Phase, kennzeichne sie als unbeobachtet; deine Nachbartabelle ist ein separater Beleg.
3. Vergleiche die behauptete IP-Zuordnung mit A's bekannter MAC. Warum ist eine geänderte Zuordnung aussagekräftiger als die Zahl der ARP-Pakete?

**Kontrollpunkt:** Deine Zeitleiste unterscheidet echte Gateway-Zuordnung, gefälschte Behauptung und Wiederherstellung. Gratuitous ARP allein ist kein Angriffsbeweis. [Suche in den ARP-Feldern](https://www.wireshark.org/docs/dfref/a/arp.html) nach `arp.src.hw_mac` und `arp.isgratuitous`.

## Extra 2: Die Grenze dieser Umleitung finden

**Optional, 10 Minuten.** Überlege zuerst: Erfasst eine falsche Gateway-Zuordnung jedes Ziel in `10.0.0.0/8`?

1. Starte einen neuen Mitschnitt an A's Datenport und wiederhole das zeitlich begrenzte Hilfsprogramm aus Aufgabe 3.
2. Während es läuft, auf **WAGO B**:

    ```bash
    ip route get 10.20.0.11
    ping -c 2 10.20.0.11
    ping -c 2 2.2.2.2
    ```

3. Vergleiche auf A `icmp.type == 8 && ip.src == 10.10.0.12 && ip.dst == 10.20.0.11` mit `icmp.type == 8 && ip.src == 10.10.0.12 && ip.dst == 2.2.2.2`.

**Kontrollpunkt:** Die Route zu CtrlX `10.20.0.11` ist im flachen `/8` direkt über `eth1`. Die gefälschte Gateway-Zuordnung verändert diese direkte Nachbarzuordnung nicht; der geroutete Jumphost-Verkehr läuft hingegen über A. Beide Ziele können über unterschiedliche Ethernet-Wege erreichbar sein. Der Hostname `vlan20` erzeugt kein eigenes VLAN.

Wiederhole **Aufgabe 5 vollständig** und schliesse die Werkzeug-Tabs. Weiter mit [Übung 3](ex3-spanning-tree.md).

## Weiterforschen

- [ARP-Spezifikation, RFC 826](https://www.rfc-editor.org/rfc/rfc826.html): Suche das Aktualisieren der Zuordnung im Empfangsalgorithmus.
- [arpspoof-Handbuch](https://manpages.debian.org/bullseye/dsniff/arpspoof.8.en.html): Vergleiche Forwarding und Zielauswahl.
- [Cisco Dynamic ARP Inspection](https://www.cisco.com/c/en/us/support/docs/switches/lan-switch-software/222274-troubleshoot-dynamic-arp-inspection-dai.html): Welche Informationen benötigt der Switch zur Validierung?
