# Übung 1: Getaggten und ungetaggten Datenverkehr vergleichen

**Zeitbedarf:** 20–30 Minuten

**Lab:** Studentenlab `ot-sec-flat`

**Schwierigkeit:** Anfänger

Vergleiche denselben Ping-Datenverkehr an zwei Switch-Ports: am Access-Port des WAGO-Geräts und am Uplink zum Gateway. Finde heraus, wo ein Ethernet-Frame ein VLAN-Tag trägt.

## Bevor du beginnst

- Wähle das aktive Netzwerk deines Studentenlabs aus.
- Die Terminal-Zugangsdaten des Labs müssen bereits eingerichtet sein.

Die folgenden Links öffnen das passende Lab-Werkzeug. Klicke in einem Werkzeug-Tab neben **Aktive Übung** auf den Übungstitel, um diese Anleitung wieder zu öffnen. Deine Leseposition, geöffneten Hinweise und das Gespräch mit dem Assistenten bleiben beim Wechsel zwischen den Werkzeugen erhalten.

Diese Übung verwendet das unveränderte Flat-Lab: WAGO `10.10.0.11`, Gateway `10.10.0.1`, VLAN 10. Wurde dein Lab umkonfiguriert, können die Ergebnisse abweichen.

??? info "Liest du diese Anleitung ausserhalb von Network Explorer?"
    Die Werkzeug-Links funktionieren im Übungsreader von Network Explorer. Öffne in der veröffentlichten Dokumentation das genannte Werkzeug und wähle den in der Aufgabe beschriebenen Host oder Port aus.

## Aufgabe 1: Den Pfad finden

[Das WAGO-Gerät auf Netmap finden](/network/?search=wago-plc2a-vlan10).

Suche `wago-plc2a-vlan10`, seinen Access-Switch `sw-acc1`, den Distribution-Switch `sw-dist` und das Gateway `gw`. Verfolge ihre Verbindungen.

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

**Beobachte:** Gibt es einen 802.1Q-Header? Welche VLAN-ID enthält er?

??? info "Hinweis: Keine Ping-Pakete sichtbar"
    Prüfe, ob du den Mitschnitt vor dem erneuten Ping gestartet hast und ob der Ping Antworten erhalten hat. Ausgewählt sein muss sw-dist `ether12` zu `gw-firewall`. Leere den Anzeigefilter, um zu prüfen, ob überhaupt Datenverkehr aufgezeichnet wurde.

    In den Ergebnissen der Paketanalyse heisst dieser Port `eth11`; die RouterOS-Bezeichnung in der Auswahl lautet `ether12`.

??? info "Hinweis: Getaggte Pakete eingrenzen"
    Verwende `icmp && ip.addr == 10.10.0.11`, falls weitere Pings sichtbar sind. Der Filter `vlan` zeigt getaggte Pakete. Ein Anzeigefilter verändert die sichtbare Auswahl, nicht die aufgezeichneten Daten.

??? success "Erklärung: Erwartete Beobachtung am Uplink"
    Der Ping durchquert den getaggten VLAN-10-Uplink. Sein Ethernet-Frame enthält einen 802.1Q-Header mit VLAN-ID 10.

Wähle **Stop Capture**, bevor du fortfährst.

## Aufgabe 4: Mit dem ungetaggten Access-Port vergleichen

1. [Den Paketmitschnitt am WAGO-Access-Port öffnen](/network/capture?node=sw-acc1&interface=ether2). Derselbe Capture-Tab wählt nun `sw-acc1`, Port `ether2` zu `wago-plc2a-vlan10` aus.
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

Wähle **Stop Capture**. Gib im WAGO-Terminal `exit` ein, um die SSH-Verbindung zu schliessen.

## Abschluss

Übung 1 endet nach Aufgabe 4. Es gibt keine Aufgabe 5 und keine Switch-Konfiguration zu ändern.

Du hast denselben Datenverkehr an einem Uplink und einem Access-Port verglichen. Prüfe, ob du erklären kannst, wo das VLAN-Tag auftauchte und warum ein leerer `vlan`-Filter mit erfolgreichen Pings vereinbar ist.

Öffne bei Fragen den vorhandenen Assistenten in einem beliebigen Lab-Tab. Er kennt diese Übung. Nenne die Aufgabennummer und bei Fragen zu Paketmitschnitten deinen Filter oder die Frame-Nummer. Warte auf seine Antwort, bevor du aus einem anderen Tab eine weitere Frage stellst.
