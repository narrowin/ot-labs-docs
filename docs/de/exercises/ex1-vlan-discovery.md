# Übung 1: Getaggten und ungetaggten Datenverkehr vergleichen

**Zeitbedarf:** 30–40 Minuten

**Lab:** Studentenlab `ot-sec-flat`

**Schwierigkeit:** Anfänger

Vergleiche denselben Ping-Datenverkehr an zwei Switch-Ports: am Access-Port des WAGO-Geräts und am Uplink zum Gateway. Finde heraus, wo ein Ethernet-Frame ein VLAN-Tag trägt.

## Anlagenübersicht und SPS-Namen

- **PLC1**: `abb-800xa-vlan40`
- **PLC2A**: `wago-plc2a-vlan10`
- **PLC2B**: `wago-plc2b-vlan10`
- **PLC3**: `ctrlx-plc3-vlan20`
- **PLC4**: `schneider-plc4-vlan30`

![Übersicht der Wasseraufbereitung aus der Lab-Visualisierung](../img/water-treatment-overview.png)

## Dein Instandhaltungsauftrag

Eine Ersatz-SPS wurde angeschlossen. Der Port ist aktiv, aber das Team ist unsicher, ob die SPS im vorgesehenen Netzwerk liegt. Prüfe zuerst den normalen Pfad, bevor jemand SPS oder Switch umkonfiguriert. Vergleiche einen kleinen Diagnoseaustausch am SPS-Port und am Gateway-Uplink.

**Drei Begriffe:** Ein **VLAN** ist ein eigener Ethernet-Broadcast-Bereich. Ein **Access-Port** transportiert für das Endgerät üblicherweise ein VLAN ohne Tag. Ein **Trunk** transportiert getaggte VLANs zwischen Netzwerkgeräten. Auch ein ungetaggter SPS-Frame kann also zu einem VLAN gehören; der Switch ordnet ihn beim Empfang zu.

**Vorhersage:** Muss eine gewöhnliche SPS an diesem Access-Port selbst VLAN-Tags ergänzen? Notiere deine Antwort vor Aufgabe 3.

## Bevor du beginnst

- Wähle das aktive Netzwerk deines Studentenlabs aus.
- Die Terminal-Zugangsdaten des Labs müssen bereits eingerichtet sein.

Die folgenden Links öffnen das passende Lab-Werkzeug. Klicke in einem Werkzeug-Tab neben **Aktive Übung** auf den Übungstitel, um diese Anleitung wieder zu öffnen. Deine Leseposition, geöffneten Hinweise und das Gespräch mit dem Assistenten bleiben beim Wechsel zwischen den Werkzeugen erhalten.

Diese Übung verwendet das unveränderte Flat-Lab: WAGO `10.10.0.11`, Gateway `10.10.0.1`, VLAN 10. Wurde dein Lab umkonfiguriert, können die Ergebnisse abweichen.

??? info "Liest du diese Anleitung ausserhalb von Network Explorer?"
    Öffne dein zugewiesenes Lab, wähle dessen Live-Netzwerk und **Netmap**. Finde den Knoten mit der Suche. Sein Aktionsmenü bietet **Open terminal** und **Capture packets**; wähle das in der Aufgabe genannte Interface. Für die VLAN-Ansicht wählst du **VLAN** im Netmap-Ansichtsmenü und VLAN **10**. Reader-Links wählen diese Ziele automatisch und behalten Netzwerk und Snapshot bei.

## Aufgabe 1: Den Pfad finden

[Das WAGO-Gerät auf Netmap finden](/network/?search=wago-plc2a-vlan10).

Suche `wago-plc2a-vlan10`, seinen Access-Switch `sw-acc1`, den Distribution-Switch `sw-dist` und das Gateway `gw`. Verfolge ihre Verbindungen.

[VLAN 10 auf Netmap untersuchen](/network/?viewMode=vlan&vlanIds=10). Vergleiche die erkannte Zugehörigkeit mit den historischen Namen. Kehre für die Port-Prüfung zum WAGO-Netmap-Link oben zurück; die Paketaufgaben prüfen das Tagging auf dem Draht.

Wähle `sw-acc1` aus und öffne **Ports** in seinem Detailbereich. Finde den Port zu WAGO anhand seiner Beschreibung. Wähle `sw-dist` aus und öffne **Neighbors**, um den Port zu `gw` zu finden.

**Beobachte:** Welcher Port von sw-acc1 führt zum WAGO-Gerät? Welcher Port von sw-dist führt zum Gateway?

??? info "Hinweis: Die Ports zuordnen"
    Suche unter **Ports** nach `wago-plc2a` in der Beschreibung. Scrolle die Tabelle seitwärts, falls diese Spalte nicht sichtbar ist. Suche unter **Neighbors** nach `gw` und lies das lokale Interface ab.

??? success "Erklärung: Ein Name ist keine Konfiguration"
    Im unveränderten Lab führt sw-acc1 `ether2` zu WAGO und sw-dist `ether12` zu `gw`. Hostnamen sind Bezeichnungen und kein Beweis für eine VLAN-Zugehörigkeit. Die Paketmitschnitte zeigen, ob ein Frame ein Tag trägt.

## Aufgabe 2: Datenverkehr erzeugen

[Das WAGO-Terminal öffnen](/network/terminal?node=wago-plc2a-vlan10). Warte auf **Connected**. Lass diesen Tab für beide Paketmitschnitte geöffnet.

Gib im WAGO-Terminal ein:

```bash
ping -c 4 10.10.0.1
```

Dieser Befehl sendet vier Ping-Anfragen an das Gateway.

**Beobachte:** Wurden alle vier Anfragen beantwortet?

??? info "Hinweis: Probleme mit Terminal oder Ping"
    Prüfe den ausgewählten Host und die Zugangsdaten, falls sich das Terminal nicht verbindet. Bleiben Ping-Antworten aus, kontrolliere, ob das Lab läuft, und prüfe den Pfad auf Netmap, bevor du fortfährst.

??? success "Erklärung: Was Ping nachweist"
    Antworten belegen die IP-Erreichbarkeit. Sie beweisen weder VLAN-Isolation noch die Durchsetzung von Firewall-Regeln. Unterschiedliche IP-Subnetze oder Endungen im Hostnamen belegen keine unterschiedlichen VLANs.

## Aufgabe 3: Den getaggten Uplink untersuchen

1. [Den Paketmitschnitt am Gateway-Uplink öffnen](/network/capture?node=sw-dist&interface=ether12). Der Link wählt `sw-dist`, Port `ether12` zu `gw-firewall` aus, das auf Netmap als `gw` erscheint.
2. Wähle **Start Capture** und warte, bis Wireshark angezeigt wird.
3. Wechsle zum WAGO-Terminal und führe denselben Ping mit vier Anfragen erneut aus.
4. Wechsle zurück zum Capture-Tab. Gib in Wiresharks Anzeigefilter oberhalb der Paketliste `icmp` ein und drücke Enter. Wähle ein Ping-Paket aus. Klappe in den Paketdetails darunter **802.1Q Virtual LAN** auf und lies die VLAN-ID ab.

Warte nach dem Ping einige Sekunden, bis die Capture-Anzeige nachgezogen hat.

**Beobachte:** Gibt es einen 802.1Q-Header? Welche VLAN-ID enthält er?

??? info "Hinweis: Keine Ping-Pakete sichtbar"
    Prüfe, ob du den Mitschnitt vor dem erneuten Ping gestartet hast und ob der Ping Antworten erhalten hat. Ausgewählt sein muss sw-dist `ether12` zu `gw-firewall`. Leere den Anzeigefilter, um zu prüfen, ob überhaupt Datenverkehr aufgezeichnet wurde.

    In den Ergebnissen der Paketanalyse heisst dieser Port `eth11`; die RouterOS-Bezeichnung in der Auswahl lautet `ether12`.

??? info "Hinweis: Getaggte Pakete eingrenzen"
    Verwende `icmp && ip.addr == 10.10.0.11`, falls weitere Pings sichtbar sind. Der Filter `vlan` zeigt getaggte Pakete. Ein Anzeigefilter verändert die sichtbare Auswahl, nicht die aufgezeichneten Daten.

??? success "Erklärung: Erwartete Beobachtung am Uplink"
    Der Ping durchquert den getaggten VLAN-10-Uplink. Sein Ethernet-Frame enthält einen 802.1Q-Header mit VLAN-ID 10.

Schliesse den Capture-Tab, bevor du fortfährst. Dadurch wird der Paketmitschnitt beendet.

## Aufgabe 4: Mit dem ungetaggten Access-Port vergleichen

1. [Den Paketmitschnitt am WAGO-Access-Port öffnen](/network/capture?node=sw-acc1&interface=ether2). Der Link öffnet einen Capture-Tab mit `sw-acc1`, Port `ether2` zu `wago-plc2a-vlan10`.
2. Wähle **Start Capture** und warte, bis Wireshark angezeigt wird.
3. Wiederhole im WAGO-Terminal `ping -c 4 10.10.0.1`.
4. Wechsle zurück zum Capture-Tab. Wende `icmp` an, wähle ein Ping-Paket aus und vergleiche seine Paketdetails mit denen des Uplink-Pakets.
5. Probiere den Anzeigefilter `vlan` aus. Leere ihn danach oder kehre zu `icmp` zurück.

**Beobachte:** Wie unterscheidet sich der Ethernet-Header dieses Pakets vom Uplink-Paket? Was geschah, als du `vlan` angewendet hast?

??? info "Hinweis: Eine leere gefilterte Ansicht"
    Der Filter `vlan` blendet ungetaggte Frames aus. Leere den Filter, um zu prüfen, ob Pakete aufgezeichnet wurden. Kontrolliere, ob du den Port zum Endgerät statt eines Uplinks oder Management-Interfaces ausgewählt hast.

    In den Ergebnissen der Paketanalyse heisst dieser Access-Port von sw-acc1 `eth1`; die RouterOS-Bezeichnung in der Auswahl lautet `ether2`.

??? success "Erklärung: Erwartete Beobachtung am Access-Port"
    WAGO sendet und empfängt auf dieser Access-Verbindung ungetaggte Ethernet-Frames. Der Switch ordnet sie VLAN 10 zu und fügt auf dem getaggten Uplink ein 802.1Q-Tag hinzu.

Schliesse den Capture-Tab und den Tab mit dem WAGO-Terminal.

## Abschluss

Der Hauptversuch endet nach Aufgabe 4. Erstelle die Übergabe unten; bearbeite danach bei Bedarf die optionalen Wartungsprüfungen.

Du hast denselben Datenverkehr an einem Uplink und einem Access-Port verglichen. Prüfe, ob du erklären kannst, wo das VLAN-Tag auftauchte und warum ein leerer `vlan`-Filter mit erfolgreichen Pings vereinbar ist.

Öffne bei Fragen den vorhandenen Assistenten in einem beliebigen Lab-Tab. Er kennt diese Übung. Nenne die Aufgabennummer und bei Fragen zu Paketmitschnitten deinen Filter oder die Frame-Nummer. Warte auf seine Antwort, bevor du aus einem anderen Tab eine weitere Frage stellst.

## Deine Übergabe

Notiere SPS-Adresse, Anschlussport, Uplink-Port und je einen Frame aus beiden Mitschnitten. Erkläre einer Kollegin oder einem Kollegen, warum ein leerer `vlan`-Filter am SPS-Port hier normal ist. Benenne eine zusätzliche Prüfung vor der Freigabe der SPS-Anwendung: einen erfolgreichen Anwendungsaustausch, nicht nur Ping.

## Lösung

??? success "Vergleiche deine Belege"
    Der WAGO-Access-Port ist sw-acc1 `ether2`, der Gateway-Uplink sw-dist `ether12`. Vier erfolgreiche Pings erzeugen vier Anfragen und vier Antworten. Am Gateway-Uplink tragen diese Pakete VLAN-ID 10; am Access-Port fehlt der 802.1Q-Header. Ein Anzeigefilter verändert die Aufzeichnung nicht.

    Notiere je eine Frame-Nummer mit Interface, Filter und VLAN-ID beziehungsweise fehlendem Tag. Beim gleichen Standard-Ping ist der getaggte Frame vier Bytes länger. Erreichbarkeit und Hostname beweisen keine Isolation.

## Extra 1: Anschluss einer Ersatz-SPS prüfen

**Optional, 10 Minuten.** Die Wartungsnotiz sagt «WAGO A ist in VLAN 10». Erstelle die Angaben, mit denen eine andere Person das Gerät richtig anschliessen kann. Ändere keine Einstellungen.

1. Öffne [sw-acc1's Terminal](/network/terminal?node=sw-acc1):

    ```routeros
    /interface bridge port print detail where interface=ether2
    /interface bridge vlan print detail
    ```

2. Auf **WAGO A**:

    ```bash
    ip -br address show eth1
    ip route get 10.10.0.1
    ```

3. Notiere Switch/Port, **PVID** (VLAN-Zuordnung für eingehenden ungetaggten Verkehr), ungetaggte VLAN-Mitgliedschaft, SPS-IP/Präfix und ob die SPS Tags sendet. Vergleiche die Konfiguration mit den Paketen aus Aufgabe 4.

**Kontrollpunkt:** Im Ausgangszustand ordnet ether2 ungetaggten Verkehr VLAN 10 zu. WAGO nutzt `10.10.0.11/8` auf eth1. Beim Ersatz müssen sowohl Switch-Port-Konfiguration als auch Endgeräteadressierung passen. Eine Link-LED oder ein Hostnamensuffix reicht nicht. `/8` bedeutet hier, dass Adressen mit `10.` zum selben IP-Subnetz gehören; diese breite Lab-Adressierung ist keine Empfehlung für eine Anlage.

Schliesse das Switch-Terminal. Behalte deine Notiz als Inbetriebnahme-Checkliste.

## Extra 2: Verfolge die MAC-Adresse des nächsten Hops

**Optional, 10 Minuten.** Ändert eine andere Ziel-IP immer auch die Ethernet-Zieladresse?

1. Starte einen neuen WAGO-Access-Port-Mitschnitt aus Aufgabe 4.
2. Führe im WAGO-Terminal aus:

    ```bash
    ip route get 2.2.2.2
    ping -c 2 10.10.0.1
    ping -c 2 2.2.2.2
    ip neigh show 10.10.0.1 dev eth1
    ```

3. Filtere mit `icmp.type == 8 && ip.src == 10.10.0.11`. Vergleiche in den Anfragen **Internet Protocol → Destination Address** mit **Ethernet II → Destination**.

**Kontrollpunkt:** Beide Ziele antworten. Die IP-Ziele unterscheiden sich, beide Anfragen gehen auf diesem Access-Link aber an die Gateway-MAC. Die Route zu `2.2.2.2` führt über `10.10.0.1`. `2.2.2.2` ist hier der simulierte DMZ-Jumphost innerhalb des Labs.

??? success "Erklärung"
    Der Ethernet-Frame adressiert den nächsten Hop auf dem lokalen Link. Das IP-Paket adressiert das endgültige Ziel. Das Gateway baut für die Weiterleitung einen neuen Ethernet-Frame.

Schliesse Capture- und Terminal-Tab. Weiter mit [Übung 2](ex2-arp-spoofing.md).

## Weiterforschen

- [Wireshark-VLAN-Felder](https://www.wireshark.org/docs/dfref/v/vlan.html): Suche nach `vlan.id` und `vlan.priority`.
- [Wireshark-Anzeigefilter](https://www.wireshark.org/docs/man-pages/wireshark-filter): Probiere die Filter deiner Beobachtungen aus.
- [MikroTik Bridge VLAN Table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): Vergleiche Access-PVID und getaggte Mitgliedschaft.
