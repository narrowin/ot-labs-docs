# Übungsreferenz

## Wireshark-Filter als Kurzreferenz

Füge jeweils einen Ausdruck in die **Anzeigefilter**-Leiste ein und drücke Enter. Das sind keine Mitschnittfilter. Eine leere gefilterte Liste bedeutet nicht, dass keine Pakete aufgezeichnet wurden.

| Filter | Verwendung |
| --- | --- |
| `icmp && ip.addr == 10.10.0.11` | ICMP-Verkehr von `wago-plc2a-vlan10` |
| `icmp.type == 8 && ip.src == 10.10.0.11` | Nur Anfragen für richtungsbezogene LACP-Prüfungen |
| `vlan` | Frames mit 802.1Q-Header |
| `vlan.id == 10 && icmp` | Getaggtes VLAN-10-ICMP |
| `icmp && !vlan` | Ungetaggtes ICMP |
| `arp.opcode == 2 && arp.src.proto_ipv4 == 10.10.0.1` | ARP-Antworten mit behaupteter Gateway-IP |
| `stp` | Spanning-Tree-BPDUs einschliesslich RSTP |
| `stp.version == 2` | RSTP-BPDUs |
| `lacp` | LACP-Aushandlung |
| `modbus && ip.addr == 10.10.0.12` | Begrenzter Modbus-Lesezugriff von `wago-plc2b-vlan10` mit Antwort |
| `modbus.func_code == 3` | Read-Holding-Registers-Nachrichten |
| `http.request && ip.src == 10.10.0.12` | Direkte HTTP-Anfragen von `wago-plc2b-vlan10` |
| `tcp.port == 2323 && tcp.len > 0` | Klartext-Marker-Daten |
| `tcp.flags.syn == 1` | SYN und SYN/ACK; zusätzlich ACK-Flag prüfen |
| `tcp.flags.reset == 1` | TCP-Resets |
| `pn_dcp` | PROFINET-DCP-Geräteerkennung |
| `opcua` | Dekodiertes OPC UA, sofern ausgetauscht |

[Suche in Wiresharks Feldreferenz](https://www.wireshark.org/docs/dfref/) nach der genauen Schreibweise. Portfilter finden Verkehr, beweisen aber keine Anwendung oder deren Sicherheitsmodus.

## Häufige Fehler eingrenzen

| Symptom | Nächste sinnvolle Prüfung |
| --- | --- |
| Werkzeug-Link nicht verfügbar | Zugewiesenes **Live**-Netzwerk wählen; Kursleitung soll die Zuordnung zum Lab-Gerät und die Berechtigungen prüfen. |
| SSH verbindet nicht | Konfigurierte **Open terminal**-Aktion verwenden; bei einem Fehler Knoten und Meldung für die Kursleitung notieren. |
| Kein Ping im Mitschnitt | Zuerst Capture starten, begrenzten Ping wiederholen, Knoten und Port-Nachbar prüfen, Filter leeren und einige Sekunden warten. |
| Viewer zeigt ERF/LEGACY und unsinnige Zeiten statt Ethernet | Auf der Seite **Restart Capture** wählen, gültige Ethernet/IP-Dekodierung abwarten und Proben wiederholen. Bleibt der Fehler, Knoten/Interface und Screenshot an die Kursleitung melden. |
| Datenstrom nur auf einem LACP-Mitglied | Bei unverändertem Bond ist dies zu erwarten. Denselben begrenzten Testverkehr erneut erzeugen und dabei am anderen Mitglied aufzeichnen; Anfragen und Antworten getrennt betrachten. |
| `vlan` am Access-Port leer | Filter leeren und ICMP suchen. Ungetaggte Frames am Endgerät sind hier zu erwarten. |
| ARP-Hilfsprogramm endet mit Code 124 | Sicherheits-Timer abgelaufen; Wiederherstellung durchführen oder begrenzte Aufgabe wie beschrieben wiederholen. |
| Redirects/Ping-Verlust bei ARP-Spoofing | Zuordnung und abgefangene Nutzlast prüfen. Für den Nachweis der Umleitung müssen nicht alle Pings erfolgreich sein. |
| Modbus meldet Exception | Ergebnis notieren; genaue Lese-Unit, Typ, Adresse und Anzahl prüfen. Keine Werte zum Schreiben angeben. |
| HTTP liefert 404 | Der HTTP-Dienst antwortet, findet aber den angefragten Pfad nicht. `/webvisu.htm` und Port 8080 prüfen. |
| GUI-Wert bleibt gleich | Stabile Prozesswerte sind möglich. Fortlaufenden Anwendungsaustausch prüfen; keine Bedienelemente betätigen, nur um eine Wertänderung zu erzwingen. |
| Zugriff trotz abschliessendem Firewall-Drop | Davorstehende Freigaberegeln und den tatsächlichen Verkehrspfad prüfen. Verkehr im selben VLAN kann das Gateway vollständig umgehen. |
| Ergebnis weicht vom Durchgang ab | Tatsächliche Ausgabe festhalten und dein Lab mit dem Ausgangszustand der Übung vergleichen. Frühere Bereinigung abschliessen; falls nötig, die Konfiguration aus dem gespeicherten Checkpoint wiederherstellen. |

## Weitere Quellen

- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/).
- [RouterOS-Handbuch](https://manual.mikrotik.com/docs/introduction/).
- [Nmap-Portzustände](https://nmap.org/book/man-port-scanning-basics.html).
- [NIST-Leitfaden für OT-Sicherheit](https://csrc.nist.gov/pubs/sp/800/82/r3/final).

Jede Übung enthält Fachquellen und zwei optionale Zusatzaufgaben nach der Lösung. Die [Übersicht](index.md) führt dich der Reihe nach weiter.
