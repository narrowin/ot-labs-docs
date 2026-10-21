# Übung 2: Eine ARP-Umleitung nachweisen

**Zeitbedarf:** 40–50 Minuten. **Lab:** dein eigenes `ot-sec-flat`-Lab. **Niveau:** angeleitete praktische Übung.

Weise eine veränderte IP-MAC-Zuordnung nach, verfolge den umgeleiteten Verkehr und stelle den ursprünglichen Zustand wieder her. Führe die Umleitung nur von `wago-plc2a` gegen `wago-plc2b` und das Gateway `gw-firewall` (`10.10.0.1`) aus. Verwende für Verbindungstests die genannten Lab-Ziele. Die Adressen gehören zum simulierten Netz dieser Appliance.

**Lernziele:** Erklären, warum gefälschte ARP-Antworten akzeptiert werden können, eine begrenzte Layer-2-Umleitung durchführen und nachweisen, lesbare Anwendungsdaten erkennen sowie Erkennung und Gegenmassnahmen erklären.

## Dein Auftrag bei einer Meldung

Das Monitoring meldet eine geänderte Gateway-MAC in einer Produktionszelle. Ist das eine genehmigte Netzwerkänderung, ein Adressierungsfehler oder eine Umleitung? Stelle im Simulator eine bekannte Ursache nach. Sammle danach die Belege, die der Betrieb für eine Entscheidung braucht.

**ARP einfach erklärt:** Für ein IPv4-Paket über Ethernet braucht ein Host die MAC-Adresse des nächsten Hops. ARP fragt, wem diese IP gehört, und speichert die Antwort. Eine gefälschte Antwort kann die Frames an ein anderes Gerät lenken. Die IP-Zieladresse muss sich dabei nicht ändern.

Die Testnachricht `LAB-ARP-DEMO` ist weder ein Industrieprotokoll noch ein echtes Passwort. Du sendest sie über TCP, um die Umleitung nachzuweisen, ohne in die Prozesssteuerung einzugreifen. Übung 6 untersucht tatsächliche OT-Protokollnachrichten des Labs.

## Vorbereitung

Schliesse [Übung 1](ex1-vlan-discovery.md) ab. Öffne die konfigurierten Terminals; du brauchst weder Passwörter einzugeben noch Software zu installieren:

| Rolle | Terminal | Datenadresse |
| --- | --- | --- |
| Angreifer | [`wago-plc2a`](/network/terminal?node=wago-plc2a-vlan10) | `10.10.0.11` |
| Opfer | [`wago-plc2b`](/network/terminal?node=wago-plc2b-vlan10) | `10.10.0.12` |
| Empfänger | [`proxmox-jumphost`](/network/terminal?node=proxmox-jumphost) | `2.2.2.2` |

Lass die Tabs offen. Die Terminals verwenden Management-Verbindungen; das Experiment läuft über das Dateninterface `eth1`. Führe jeden Block auf dem genannten Knoten aus. Der Übungstitel im Werkzeug-Tab öffnet diese Anleitung erneut.

??? info "Du verwendest die eigenständige Dokumentation?"
    Suche in **Netmap** deines zugewiesenen Live-Labs nach jedem stabilen Knotennamen aus der Tabelle. Wähle im Aktionsmenü **Open terminal**. Wähle für den Mitschnitt `wago-plc2a`, **Capture packets**, Interface **eth1**. Links in der Übungsansicht treffen diese Auswahl automatisch und behalten Netzwerk und Snapshot bei.

## Aufgabe 1: Die echte Gateway-Zuordnung festhalten

Auf **`wago-plc2b`**:

```bash
ping -c 2 10.10.0.1
ip neigh show 10.10.0.1 dev eth1
```

Notiere die Gateway-MAC, nicht nur den Zustand des Nachbareintrags (`REACHABLE` oder `STALE`). Auf **`wago-plc2a`**:

```bash
cat /sys/class/net/eth1/address
cat /proc/sys/net/ipv4/ip_forward
command -v arp_spoofing-simple.sh timeout
```

Notiere die MAC-Adresse von `wago-plc2a` und den Forwarding-Wert. Der letzte Befehl muss je einen Pfad für `arp_spoofing-simple.sh` und `timeout` ausgeben. Das Hilfsprogramm bewahrt den vorherigen Forwarding-Zustand.

**Kontrollpunkt:** Gateway-MAC und die MAC-Adresse von `wago-plc2a` unterscheiden sich. Beide Befehle sind vorhanden. Ist die Zuordnung bereits verändert, führe zuerst die drei Aufräumschritte aus Aufgabe 5 aus.

??? info "Die Zuordnung auch im Explorer ansehen"
    Öffne [die Host-Details von `wago-plc2b`](/network/inventory/host?node=wago-plc2b-vlan10) und klappe **ARP Cache** auf. Suche **address** `10.10.0.1` auf **interface** `eth1`; vergleiche **mac** mit deiner Gateway-MAC. Bei [`wago-plc2a`](/network/inventory/host?node=wago-plc2a-vlan10) zeigt **Interface Details** die MAC von `eth1` und **Network Interfaces** die Adresse `10.10.0.11` mit Maske `255.0.0.0`. **Interface ether2** im Kopfbereich bezeichnet den Switch-Port, nicht Linux-`eth1`. Die MAC-Adresse von `wago-plc2b` im Kopfbereich identifiziert `wago-plc2b` selbst und bleibt unverändert; untersucht wird die Gateway-Zuordnung in **ARP Cache**.

    Das sind gesammelte Endpunktdaten. ARP wird etwa alle 60 Sekunden erfasst; Neuladen zeigt die zuletzt empfangene Meldung und startet keine neue Abfrage. Ein früh gestoppter Versuch kann zwischen zwei Erfassungen liegen; die Übermittlung kann verzögert sein. Verwende das Terminal und den Mitschnitt für den unmittelbaren Nachweis. Eine unveränderte Explorer-Zeile schliesst eine Umleitung nicht aus. Fehlen Bereiche in der Anzeige, melde dies der Kursleitung und verwende die Terminalausgaben und aufgezeichneten Pakete.

    Ausserhalb der Übungsansicht: **Inventory → Hosts**, stabilen Knotennamen suchen, **Enter** drücken und den Namen anklicken.

## Aufgabe 2: Den normalen Weg beobachten

[Öffne einen Mitschnitt am Datenport von `wago-plc2a`](/network/capture?node=wago-plc2a-vlan10&interface=eth1), wähle **Start Capture** und führe auf **`wago-plc2b`** aus:

```bash
ping -c 3 2.2.2.2
```

Verwende in Wireshark `icmp && ip.addr == 10.10.0.12`. Warte einige Sekunden, bis die Anzeige aktualisiert ist.

**Kontrollpunkt:** Die Pings werden beantwortet. Ihre Unicast-Pakete laufen normalerweise aber nicht über den Access-Link von `wago-plc2a`, weil der Switch sie direkt zum Gateway weiterleitet. ARP-Broadcasts können trotzdem sichtbar sein. Siehst du bereits Unicast-Verkehr von `wago-plc2b`, kontrolliere die ARP-Zuordnung und das ausgewählte Mitschnitt-Interface.

Lies Aufgaben 3–5 vor dem Start des Timers. Halte `wago-plc2a`, `wago-plc2b`, Empfänger und Mitschnitt bereit. Bereite auf dem **`proxmox-jumphost`** den Marker-Empfänger vor:

```bash
timeout 300 nc -l 2323
```

Lass ihn warten. Er endet nach einer Verbindung oder spätestens nach fünf Minuten. Wechsle jetzt zu `wago-plc2a`. Führe zuerst die kurzen Befehle aus Aufgaben 3–4 aus; untersuche die aufgezeichneten Pakete nach dem Aufräumen.

## Aufgabe 3: Das ausgewählte Paar umleiten

Starte auf **`wago-plc2a`** im Vordergrund:

```bash
timeout -s INT 120 arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
```

Das Hilfsprogramm adressiert nur `wago-plc2b` und dessen Gateway und endet nach 120 Sekunden automatisch. Sobald du Zuordnung und Marker nachgewiesen hast, stoppe es früher mit **Ctrl+C**. Lass dieses Terminal laufen, während du `wago-plc2b` verwendest.

Nach etwa fünf Sekunden auf **`wago-plc2b`**:

```bash
ip neigh show 10.10.0.1 dev eth1
ping -c 3 2.2.2.2
```

**Kontrollpunkt:** `wago-plc2b` ordnet `10.10.0.1` jetzt die MAC-Adresse von `wago-plc2a` zu. Der Mitschnitt auf `wago-plc2a` enthält den ICMP-Verkehr von `wago-plc2b`. Ein Paket kann beim Empfang und bei der Weiterleitung zweimal erscheinen. ICMP-Redirects und einzelne verlorene Pings sind möglich; sie widerlegen die MAC- und Nutzlastbelege nicht. Ping allein beweist keine Umleitung.

??? info "Hinweis: Timer abgelaufen oder keine Änderung"
    Prüfe das Hilfsprogramm-Terminal auf Fehler. Exit-Code 124 bedeutet: Das 120-Sekunden-Limit ist abgelaufen. Räume vor einem neuen Versuch zuerst mit Aufgabe 5 auf. Starte den Empfänger aus Aufgabe 2 erneut und wiederhole Aufgaben 3–5, sobald du bereit bist. Verwende dieselben drei Knoten und denselben Mitschnittfilter. Die Paketanalyse muss nicht während des laufenden Hilfsprogramms geschehen.

## Aufgabe 4: Einen harmlosen Klartext-Marker finden

Sende diesen Marker auf **`wago-plc2b`**, während das Hilfsprogramm läuft und der vorbereitete Empfänger noch wartet. Verwende keine echten Zugangsdaten:

```bash
printf 'LAB-ARP-DEMO\n' | nc -N -w 3 2.2.2.2 2323
```

Der Empfänger soll `LAB-ARP-DEMO` anzeigen. Notiere die veränderte Nachbarzuordnung und gehe direkt zu Aufgabe 5. Die aufgezeichneten Pakete bleiben für die Analyse nach dem Stoppen des Hilfsprogramms erhalten.

**Kontrollpunkt:** Der Empfänger hat den Marker erhalten, während `wago-plc2b` der Gateway-IP die MAC-Adresse von `wago-plc2a` zuordnete. Die Paketbelege prüfst du nach dem Aufräumen unten.

??? info "Hinweis: Kein Marker"
    Ist der Empfänger oder das Hilfsprogramm beendet, räume zuerst mit Aufgabe 5 auf, starte den Empfänger aus Aufgabe 2 erneut und wiederhole Aufgaben 3–5. `tcp.port == 2323 && tcp.len > 0` zeigt Datenpakete. Sind nur SYN-Pakete sichtbar, kontrolliere Listener und Route.

## Aufgabe 5: Wiederherstellen und prüfen

1. Stoppe das Hilfsprogramm auf `wago-plc2a` gegebenenfalls mit **Ctrl+C**. Warte, bis die Eingabeaufforderung wieder erscheint. Warte anschliessend einige Sekunden auf korrigierende ARP-Antworten. Stoppe auch den Listener mit **Ctrl+C**, falls er noch wartet.
2. Aktualisiere auf **`wago-plc2b`** nur diesen dynamischen Gateway-Eintrag:

    ```bash
    ip neigh del 10.10.0.1 dev eth1
    ping -c 3 10.10.0.1
    ip neigh show 10.10.0.1 dev eth1
    ```

    Meldet das Löschen einen fehlenden Eintrag, fahre mit Ping fort.
3. Auf **`wago-plc2a`**:

    ```bash
    pgrep -a arpspoof
    cat /proc/sys/net/ipv4/ip_forward
    ```

**Kontrollpunkt:** Kein `arpspoof`-Prozess bleibt übrig, Forwarding hat den Ausgangswert und `wago-plc2b` verwendet bei erfolgreichen Pings wieder die ursprüngliche Gateway-MAC. Gibt `pgrep` nichts aus und endet mit Exit-Code 1, wurde kein passender Prozess gefunden. Bleibt die Zuordnung falsch, stoppe hier und hole die Kursleitung dazu.

Untersuche jetzt den gespeicherten Mitschnitt. Filtere `tcp.port == 2323`, wähle ein Datenpaket und öffne per Rechtsklick **Follow → TCP Stream**. Finde `LAB-ARP-DEMO`, notiere die Paketnummer und schliesse das Stream-Fenster wieder.

**Kontrollpunkt:** Empfänger und aufgezeichnete TCP-Nutzdaten enthalten denselben Marker. Zusammen mit der veränderten Nachbarzuordnung belegt dies die Umleitung. Ein offener Port allein beweist keine abgefangenen Anwendungsdaten.

Lass den Mitschnitt für Extra 1 offen, falls du weitermachen möchtest. Schliesse sonst die Mitschnitt- und Terminal-Tabs.

## Deine Übergabe zur Meldung

Schreibe vier Zeilen: ursprüngliche Gateway-MAC mit Zeitpunkt; geänderte Zuordnung mit aufgezeichnetem Marker; wiederhergestellte Zuordnung mit Bereinigungsbelegen; nächste betriebliche Prüfung. Prüfe, ob ein Gateway-Tausch oder eine automatische Umschaltung genehmigt war und zu welchem Gerät die beobachtete MAC-Adresse gehört, bevor du eine reale Meldung als Angriff einstufst. Gib diese Belege an die zuständige Stelle weiter; ein geänderter ARP-Eintrag allein rechtfertigt nicht das Abtrennen einer laufenden SPS.

## Lösung

??? success "Vergleiche deine Belege"
    Vor dem Versuch sendet `wago-plc2b` gerouteten Verkehr an die echte Gateway-MAC. Gefälschte ARP-Antworten ersetzen diese Zuordnung durch die MAC-Adresse von `wago-plc2a`. Die gerouteten Pakete von `wago-plc2b` laufen deshalb über `wago-plc2a`. Empfänger und Mitschnitt zeigen denselben Klartext-Marker. Nach dem Stoppen und Aktualisieren kehrt die ursprüngliche Zuordnung zurück.

    Das belegt eine nicht authentifizierte ARP-Zuordnung und lesbare TCP-Anwendungsdaten im Lab. Es beweist keine Entschlüsselung verschlüsselter Inhalte. Port Security allein prüft keine IP-MAC-Zuordnung. Dynamic ARP Inspection benötigt vertrauenswürdige Zuordnungen und ein Verfahren für statisch adressierte OT-Geräte.

## Extra 1: Eine ARP-Vorfall-Zeitleiste erstellen

**Optional, 10 Minuten.** Verwende den offenen Mitschnitt oder wiederhole Aufgaben 2–5.

1. Filtere `arp.opcode == 2 && arp.src.proto_ipv4 == 10.10.0.1`.
2. Klappe **Address Resolution Protocol** auf. Notiere **Sender IP address**, **Sender MAC address** und je eine Frame-Nummer vor, während und nach dem Experiment. Fehlt eine Phase, kennzeichne sie als unbeobachtet; deine Nachbartabelle ist ein separater Beleg.
3. Vergleiche die gemeldete Zuordnung der Gateway-IP mit der bekannten MAC-Adresse von `wago-plc2a`. Warum ist eine geänderte Zuordnung aussagekräftiger als die Zahl der ARP-Pakete?
4. Öffne den **ARP Cache** von `wago-plc2b` im Explorer erneut. Vergleiche die neueste verfügbare Zeile mit deinen aufgezeichneten Phasen. Der Explorer zeigt den neuesten Bericht, nicht den gesamten zeitlichen Verlauf. Kennzeichne dort nicht erfasste Phasen als unbeobachtet.

**Kontrollpunkt:** Deine Zeitleiste unterscheidet echte Gateway-Zuordnung, gefälschte Behauptung und Wiederherstellung. Gratuitous ARP allein ist kein Angriffsbeweis. [Suche in den ARP-Feldern](https://www.wireshark.org/docs/dfref/a/arp.html) nach `arp.src.hw_mac` und `arp.isgratuitous`.

## Extra 2: Schutz wählen, der auch nach Wartung funktioniert

**Optional, 10 Minuten.** Die Zelle verwendet feste SPS-Adressen. Morgen wird das Gateway durch Hardware mit anderer MAC ersetzt. Prüfe diese Vorschläge anhand deiner notierten Gateway-/Angreifer-Zuordnungen; konfiguriere sie nicht.

| Vorschlag | Zu beantwortende Frage |
| --- | --- |
| Pro Switch-Port nur die MAC der jeweiligen SPS erlauben | Musste `wago-plc2a` seine Quell-MAC ändern, um die Gateway-IP zu beanspruchen? |
| Auf `wago-plc2b` einen festen Gateway-ARP-Eintrag setzen | Was muss sich beim legitimen Wechsel der Gateway-MAC ändern? |
| ARP-Zuordnungen am Switch prüfen | Woher kommen vertrauenswürdige IP-/MAC-Zuordnungen für statische SPS-Adressen? |
| Änderungen nur melden | Wer prüft den Wartungsauftrag und entscheidet über die Reaktion? |

**Kontrollpunkt:** `wago-plc2a` kann mit seiner eigenen erlaubten MAC eine falsche Zuordnung zur Gateway-IP verbreiten. Eine Port-Security-Regel, die nur die Quell-MAC prüft, reicht deshalb nicht aus. Bei einem festen ARP-Eintrag muss geregelt sein, wer ihn beim Gateway-Tausch aktualisiert. Dynamic ARP Inspection benötigt Plattformunterstützung und vertrauenswürdige Zuordnungen; eine reine DHCP-Datenbank deckt statische Geräte nicht automatisch ab. Monitoring liefert Belege, verhindert die falsche Zuordnung aber nicht.

Wähle eine vorbeugende Massnahme und eine Massnahme zur Erkennung und Reaktion. Halte fest, wer verantwortlich ist und wie die Funktion am Austauschtag geprüft wird. Dazu gehört auch der Nachweis, dass die Anwendung wieder kommunizieren kann. Aktiviere keinen ungeprüften Schutzmechanismus in einer laufenden Zelle. Schliesse die Werkzeuge; weiter mit [Übung 3](ex3-spanning-tree.md).

## Weiterforschen

- [ARP-Spezifikation, RFC 826](https://www.rfc-editor.org/rfc/rfc826.html): Suche das Aktualisieren der Zuordnung im Empfangsalgorithmus.
- [arpspoof-Handbuch](https://manpages.debian.org/bullseye/dsniff/arpspoof.8.en.html): Vergleiche Forwarding und Zielauswahl.
- [Cisco Dynamic ARP Inspection](https://www.cisco.com/c/en/us/support/docs/switches/lan-switch-software/222274-troubleshoot-dynamic-arp-inspection-dai.html): Welche Informationen benötigt der Switch zur Validierung?
- [Felder der osquery-ARP-Tabelle](https://github.com/osquery/osquery/blob/master/specs/arp_cache.table): Vergleiche gemeldete `address`, `mac` und `interface` mit der aktuellen Nachbartabelle.
