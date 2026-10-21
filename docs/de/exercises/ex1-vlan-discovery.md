# Übung 1: Getaggten und ungetaggten Datenverkehr vergleichen

**Zeitbedarf:** 30–40 Minuten

**Lab:** dein eigenes `ot-sec-flat`-Lab

**Niveau:** Einstieg ohne Netzwerk-Vorkenntnisse

Vergleiche denselben Ping-Datenverkehr an zwei Switch-Ports: am Access-Port von `wago-plc2a` und am Uplink zum Gateway. Finde heraus, wo ein Ethernet-Frame ein VLAN-Tag trägt.

## Anlagenübersicht und SPS-Namen

- **PLC1**: `abb-800xa`
- **PLC2A**: `wago-plc2a`
- **PLC2B**: `wago-plc2b`
- **PLC3**: `ctrlx-plc3`
- **PLC4**: `schneider-plc4`

![Übersicht der Wasseraufbereitung aus der Lab-Visualisierung](../img/water-treatment-overview.png)

## Dein Instandhaltungsauftrag

Eine Ersatz-SPS wurde angeschlossen. Der Port ist aktiv, aber das Team ist unsicher, ob die SPS im vorgesehenen Netzwerk liegt. Prüfe zuerst den bestehenden Netzwerkpfad, bevor jemand die SPS oder den Switch umkonfiguriert. Vergleiche dazu wenige Ping-Pakete am SPS-Port und am Gateway-Uplink.

**Drei Begriffe:** Ein **VLAN** ist ein eigener Ethernet-Broadcast-Bereich. Ein **Access-Port** transportiert für das Endgerät üblicherweise ein VLAN ohne Tag. Ein **Trunk** transportiert getaggte VLANs zwischen Netzwerkgeräten. Auch ein ungetaggter SPS-Frame kann also zu einem VLAN gehören; der Switch ordnet ihn beim Empfang zu.

**Vorhersage:** Muss eine gewöhnliche SPS an diesem Access-Port selbst VLAN-Tags ergänzen? Notiere deine Antwort vor Aufgabe 3.

## Bevor du beginnst

- Wähle das aktive Netzwerk deines Übungslabs aus.
- Die Terminal-Zugangsdaten des Labs müssen bereits eingerichtet sein.

Die folgenden Links öffnen das passende Lab-Werkzeug. Klicke in einem Werkzeug-Tab neben **Aktive Übung** auf den Übungstitel, um diese Anleitung wieder zu öffnen. Deine Leseposition, geöffneten Hinweise und das Gespräch mit dem Assistenten bleiben beim Wechsel zwischen den Werkzeugen erhalten.

Diese Übung verwendet das unveränderte Flat-Lab: `wago-plc2a` `10.10.0.11`, Gateway `10.10.0.1`, VLAN 10. Wurde dein Lab umkonfiguriert, können die Ergebnisse abweichen.

??? info "Liest du diese Anleitung ausserhalb von Network Explorer?"
    Öffne dein zugewiesenes Lab, wähle dessen Live-Netzwerk und **Netmap**. Finde den Knoten mit der Suche. Sein Aktionsmenü bietet **Open terminal** und **Capture packets**; wähle das in der Aufgabe genannte Interface. Für die VLAN-Ansicht wählst du **VLAN** im Netmap-Ansichtsmenü und VLAN **10**. Links in der Übungsansicht wählen diese Ziele automatisch und behalten Netzwerk und Snapshot bei.

## Aufgabe 1: Den Pfad finden

[`wago-plc2a` auf Netmap finden](/network/?search=wago-plc2a-vlan10).

Suche `wago-plc2a`, seinen Access-Switch `sw-acc1`, den Distribution-Switch `sw-dist` und das Gateway `gw-firewall`, das in Netmap und Nachbarlisten als `gw` angezeigt wird. Verfolge ihre Verbindungen.

[VLAN 10 auf Netmap untersuchen](/network/?viewMode=vlan&vlanIds=10). Finde PLC2A (`wago-plc2a`) und prüfe, ob seine Verbindung zu sw-acc1 VLAN 10 zugeordnet ist. Kehre zum Netmap-Link zu `wago-plc2a` oben zurück, um die Ports zu bestimmen. In Aufgaben 3–4 untersuchst du die VLAN-Tags in den Paketen.

Wähle `sw-acc1` aus und öffne **Ports** in seinem Detailbereich. Finde den Port zu `wago-plc2a` anhand seiner Beschreibung. Wähle `sw-dist` aus und öffne **Neighbors**, um den Port zu `gw-firewall` zu finden.

**Beobachte:** Welcher Port von sw-acc1 führt zu `wago-plc2a`? Welcher Port von sw-dist führt zum Gateway?

??? info "Hinweis: Die Ports zuordnen"
    Suche unter **Ports** nach `wago-plc2a` in der Beschreibung. Scrolle die Tabelle seitwärts, falls diese Spalte nicht sichtbar ist. Suche unter **Neighbors** nach `gw` und lies das lokale Interface ab.

??? success "Erklärung: Der erwartete Pfad"
    Im unveränderten Lab führt Port `ether2` von sw-acc1 zu `wago-plc2a` und Port `ether12` von sw-dist zu `gw-firewall`. PLC2A gehört zu VLAN 10. Die Mitschnitte zeigen, wie dieses VLAN auf der Access-Verbindung und dem Uplink transportiert wird.

## Aufgabe 2: Datenverkehr erzeugen

[Das Terminal von `wago-plc2a` öffnen](/network/terminal?node=wago-plc2a-vlan10). Warte auf **Connected**. Lass diesen Tab für beide Paketmitschnitte geöffnet.

Gib im Terminal von `wago-plc2a` ein:

```bash
ping -c 4 10.10.0.1
```

Dieser Befehl sendet vier Ping-Anfragen an das Gateway.

**Beobachte:** Wurden alle vier Anfragen beantwortet?

??? info "Hinweis: Probleme mit Terminal oder Ping"
    Prüfe den ausgewählten Host und die Zugangsdaten, falls sich das Terminal nicht verbindet. Bleiben Ping-Antworten aus, kontrolliere, ob das Lab läuft, und prüfe den Pfad auf Netmap, bevor du fortfährst.

??? success "Erklärung: Was Ping nachweist"
    Die Antworten belegen die IP-Erreichbarkeit des Gateways. Aufgaben 3–4 zeigen, wie diese Pakete durch VLAN 10 transportiert werden.

## Aufgabe 3: Den getaggten Uplink untersuchen

1. [Den Paketmitschnitt am Gateway-Uplink öffnen](/network/capture?node=sw-dist&interface=ether12). Der Link wählt Port `ether12` von `sw-dist` aus. Er führt zum Gateway `gw-firewall`, das auf Netmap als `gw` erscheint.
2. Wähle **Start Capture** und warte, bis Wireshark angezeigt wird.
3. Wechsle zum Terminal von `wago-plc2a` und führe denselben Ping mit vier Anfragen erneut aus.
4. Wechsle zurück zum Mitschnitt-Tab. Gib in Wiresharks Anzeigefilter oberhalb der Paketliste `icmp` ein und drücke Enter. Wähle ein Ping-Paket aus. Klappe in den Paketdetails darunter **802.1Q Virtual LAN** auf und lies die VLAN-ID ab.

Warte nach dem Ping einige Sekunden, bis die Anzeige des Mitschnitts aktualisiert ist.

**Beobachte:** Gibt es einen 802.1Q-Header? Welche VLAN-ID enthält er?

??? info "Hinweis: Keine Ping-Pakete sichtbar"
    Prüfe, ob du den Mitschnitt vor dem erneuten Ping gestartet hast und ob der Ping Antworten erhalten hat. Ausgewählt sein muss sw-dist `ether12` zu `gw-firewall`. Leere den Anzeigefilter, um zu prüfen, ob überhaupt Datenverkehr aufgezeichnet wurde.

    In den Ergebnissen der Paketanalyse heisst dieser Port `eth11`; die RouterOS-Bezeichnung in der Auswahl lautet `ether12`.

??? info "Hinweis: Getaggte Pakete eingrenzen"
    Verwende `icmp && ip.addr == 10.10.0.11`, falls weitere Pings sichtbar sind. Der Filter `vlan` zeigt getaggte Pakete. Ein Anzeigefilter verändert die sichtbare Auswahl, nicht die aufgezeichneten Daten.

??? success "Erklärung: Erwartete Beobachtung am Uplink"
    Der Ping durchquert den getaggten VLAN-10-Uplink. Sein Ethernet-Frame enthält einen 802.1Q-Header mit VLAN-ID 10.

Schliesse den Mitschnitt-Tab, bevor du fortfährst. Dadurch wird der Paketmitschnitt beendet.

## Aufgabe 4: Mit dem ungetaggten Access-Port vergleichen

1. [Den Paketmitschnitt am Access-Port von `wago-plc2a` öffnen](/network/capture?node=sw-acc1&interface=ether2). Der Link öffnet einen Mitschnitt-Tab mit `sw-acc1`, Port `ether2` zu `wago-plc2a`.
2. Wähle **Start Capture** und warte, bis Wireshark angezeigt wird.
3. Wiederhole im Terminal von `wago-plc2a` `ping -c 4 10.10.0.1`.
4. Wechsle zurück zum Mitschnitt-Tab. Wende `icmp` an, wähle ein Ping-Paket aus und vergleiche seine Paketdetails mit denen des Uplink-Pakets.
5. Probiere den Anzeigefilter `vlan` aus. Leere ihn danach oder kehre zu `icmp` zurück.

**Beobachte:** Wie unterscheidet sich der Ethernet-Header dieses Pakets vom Uplink-Paket? Was geschah, als du `vlan` angewendet hast?

??? info "Hinweis: Eine leere gefilterte Ansicht"
    Der Filter `vlan` blendet ungetaggte Frames aus. Leere den Filter, um zu prüfen, ob Pakete aufgezeichnet wurden. Kontrolliere, ob du den Port zum Endgerät statt eines Uplinks oder Management-Interfaces ausgewählt hast.

    In den Ergebnissen der Paketanalyse heisst dieser Access-Port von sw-acc1 `eth1`; die RouterOS-Bezeichnung in der Auswahl lautet `ether2`.

??? success "Erklärung: Erwartete Beobachtung am Access-Port"
    `wago-plc2a` sendet und empfängt auf dieser Access-Verbindung ungetaggte Ethernet-Frames. Der Switch ordnet sie VLAN 10 zu und fügt auf dem getaggten Uplink ein 802.1Q-Tag hinzu.

Schliesse den Mitschnitt-Tab und den Tab mit dem Terminal von `wago-plc2a`.

## Abschluss

Der Hauptversuch endet nach Aufgabe 4. Halte deine Ergebnisse für die Übergabe unten fest; bearbeite danach bei Bedarf die optionalen Wartungsprüfungen.

Du hast denselben Datenverkehr an einem Uplink und einem Access-Port verglichen. Prüfe, ob du erklären kannst, wo das VLAN-Tag auftauchte und warum der Anzeigefilter `vlan` trotz erfolgreicher Pings keine Pakete zeigt.

Öffne bei Fragen den vorhandenen Assistenten in einem beliebigen Lab-Tab. Er kennt diese Übung. Nenne die Aufgabennummer und bei Fragen zu Paketmitschnitten deinen Filter oder die Frame-Nummer. Warte auf seine Antwort, bevor du aus einem anderen Tab eine weitere Frage stellst.

## Deine Übergabe

Notiere SPS-Adresse, Anschlussport, Uplink-Port und je einen Frame aus beiden Mitschnitten. Erkläre einer Kollegin oder einem Kollegen, warum der Anzeigefilter `vlan` am SPS-Port hier keine Pakete zeigt. Benenne eine zusätzliche Prüfung vor der Freigabe der SPS-Anwendung. Dabei muss die benötigte Anwendung erfolgreich Daten austauschen; Ping allein reicht nicht.

## Lösung

??? success "Vergleiche deine Belege"
    Der Access-Port von `wago-plc2a` ist sw-acc1 `ether2`, der Gateway-Uplink sw-dist `ether12`. Vier erfolgreiche Pings erzeugen vier Anfragen und vier Antworten. Am Gateway-Uplink tragen diese Pakete VLAN-ID 10; am Access-Port fehlt der 802.1Q-Header. Ein Anzeigefilter verändert die Aufzeichnung nicht.

    Notiere je eine Frame-Nummer mit Interface, Filter und VLAN-ID beziehungsweise fehlendem Tag. Beim gleichen Standard-Ping ist der getaggte Frame vier Bytes länger. Diese Mitschnitte belegen die Tag-Behandlung auf den beiden gewählten Verbindungen. Für eine Isolationsprüfung wären zusätzlich gezielte erlaubte und gesperrte Verbindungsversuche nötig.

## Extra 1: Anschlussdaten von `wago-plc2a` festhalten

**Optional, 10 Minuten.** Erstelle einen Anschlussnachweis für **`wago-plc2a` (PLC2A)**. Eine andere Person soll damit den Switch-Port eindeutig finden und die Netzwerkeinstellungen beim Wiederanschliessen dieser SPS prüfen können. Lies die bestehenden Einstellungen ab; ändere sie nicht.

1. Öffne [das Terminal von `sw-acc1`](/network/terminal?node=sw-acc1) und führe aus:

    ```routeros
    /interface bridge port print detail where interface=ether2
    /interface bridge vlan print detail
    ```

2. Öffne [das Terminal von `wago-plc2a`](/network/terminal?node=wago-plc2a-vlan10) und führe aus:

    ```bash
    ip -br address show eth1
    ip route get 10.10.0.1
    ```

3. Halte diese Angaben in einem Anschlussnachweis fest:

    | Angabe | Wo du sie abliest |
    | --- | --- |
    | SPS-Gerätename und Dateninterface | Benanntes Terminal und dessen `ip`-Ausgabe |
    | Daten-IP-Adresse und Präfix der SPS | `ip -br address show eth1` |
    | Switch-Gerätename und physischer Port | `sw-acc1`, Port aus Aufgabe 1 |
    | PVID: VLAN-Zuordnung eingehender ungetaggter Frames | Feld `pvid` in der Bridge-Port-Ausgabe für ether2 |
    | Ungetaggte VLAN-Mitgliedschaft dieses Ports | VLAN-Zeile, deren Liste `current-untagged` den Port `ether2` enthält |
    | Sendet die SPS getaggte oder ungetaggte Frames? | Deine Notiz zum Access-Port-Paket aus Aufgabe 4; wiederhole die Aufgabe bei Bedarf |
    | Weg zum Gateway `10.10.0.1` | `ip route get 10.10.0.1` |

**Fertig, wenn:** Dein Nachweis genau eine SPS, einen Switch/Port und die Angaben oben identifiziert. Unterscheide dabei `eth1` der SPS von `ether2` des Switches.

??? success "Erwartete Anschlussdaten im unveränderten Lab"
    - SPS: `wago-plc2a`; Dateninterface: `eth1`; Adresse: `10.10.0.11/8`.
    - Switch: `sw-acc1`; physischer Port: `ether2`.
    - Port-PVID: `10`; der Port ist ungetaggtes Mitglied von VLAN `10`.
    - Die SPS sendet auf dieser Verbindung ungetaggte Frames, wie in Aufgabe 4 beobachtet.
    - Die Routenprüfung erreicht `10.10.0.1` direkt über `eth1`.

    Beim Wiederanschliessen dieser SPS müssen Endgeräteadressierung und Switch-Port-Konfiguration zusammenpassen. `/8` bedeutet in diesem Lab, dass Adressen mit `10.` zum lokalen IP-Subnetz gehören. Vor der Rückgabe an den Betrieb ist zusätzlich die benötigte Anwendung zu prüfen.

Schliesse beide Terminal-Tabs und behalte den Anschlussnachweis.

## Extra 2: Verfolge die MAC-Adresse des nächsten Hops

**Optional, 10 Minuten.** Ändert eine andere Ziel-IP immer auch die Ethernet-Zieladresse?

1. Starte einen neuen Mitschnitt am Access-Port von `wago-plc2a` wie in Aufgabe 4.
2. Führe im Terminal von `wago-plc2a` aus:

    ```bash
    ip route get 2.2.2.2
    ping -c 2 10.10.0.1
    ping -c 2 2.2.2.2
    ip neigh show 10.10.0.1 dev eth1
    ```

3. Filtere mit `icmp.type == 8 && ip.src == 10.10.0.11`. Vergleiche in den Anfragen **Internet Protocol → Destination Address** mit **Ethernet II → Destination**.

**Kontrollpunkt:** Beide Ziele antworten. Die IP-Zieladressen unterscheiden sich, doch beide Anfragen gehen auf dieser Access-Verbindung an die MAC-Adresse des Gateways. Die Route zu `2.2.2.2` führt über `10.10.0.1`. `2.2.2.2` ist hier der simulierte `proxmox-jumphost` innerhalb des Labs.

??? success "Erklärung"
    Der Ethernet-Frame adressiert den nächsten Hop auf dem lokalen Link. Das IP-Paket adressiert das endgültige Ziel. Das Gateway baut für die Weiterleitung einen neuen Ethernet-Frame.

Schliesse den Mitschnitt- und den Terminal-Tab. Weiter mit [Übung 2](ex2-arp-spoofing.md).

## Weiterforschen

- [Wireshark-VLAN-Felder](https://www.wireshark.org/docs/dfref/v/vlan.html): Suche nach `vlan.id` und `vlan.priority`.
- [Wireshark-Anzeigefilter](https://www.wireshark.org/docs/man-pages/wireshark-filter): Probiere die Filter deiner Beobachtungen aus.
- [MikroTik Bridge VLAN Table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): Vergleiche Access-PVID und getaggte Mitgliedschaft.
