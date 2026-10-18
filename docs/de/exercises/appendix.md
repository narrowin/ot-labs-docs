# Übungsreferenz

## Wireshark-Filter als Kurzreferenz

Füge jeweils einen Ausdruck in die **Anzeigefilter**-Leiste ein und drücke Enter. Das sind keine Capture-Filter. Eine leere gefilterte Liste bedeutet keinen leeren Mitschnitt.

| Filter | Verwendung |
| --- | --- |
| `icmp && ip.addr == 10.10.0.11` | WAGO A's ICMP |
| `icmp.type == 8 && ip.src == 10.10.0.11` | Nur Anfragen für richtungsbezogene LACP-Prüfungen |
| `vlan` | Frames mit 802.1Q-Header |
| `vlan.id == 10 && icmp` | Getaggtes VLAN-10-ICMP |
| `icmp && !vlan` | Ungetaggtes ICMP |
| `arp.opcode == 2 && arp.src.proto_ipv4 == 10.10.0.1` | ARP-Antworten mit behaupteter Gateway-IP |
| `stp` | Spanning-Tree-BPDUs einschliesslich RSTP |
| `stp.version == 2` | RSTP-BPDUs |
| `lacp` | LACP-Aushandlung |
| `modbus && ip.addr == 10.10.0.12` | B's begrenzter Modbus-Lesezugriff mit Antwort |
| `modbus.func_code == 3` | Read-Holding-Registers-Nachrichten |
| `http.request && ip.src == 10.10.0.12` | B's direkte HTTP-Anfragen |
| `tcp.port == 2323 && tcp.len > 0` | Klartext-Marker-Daten |
| `tcp.flags.syn == 1` | SYN und SYN/ACK; zusätzlich ACK-Flag prüfen |
| `tcp.flags.reset == 1` | TCP-Resets |
| `pn_dcp` | PROFINET-DCP-Discovery |
| `opcua` | Dekodiertes OPC UA, sofern ausgetauscht |

[Suche in Wiresharks Feldreferenz](https://www.wireshark.org/docs/dfref/) nach der genauen Schreibweise. Portfilter finden Verkehr, beweisen aber keine Anwendung oder deren Sicherheitsmodus.

## Häufige Fehler eingrenzen

| Symptom | Nächste sinnvolle Prüfung |
| --- | --- |
| Werkzeug-Link nicht verfügbar | Zugewiesenes **Live**-Netzwerk wählen; Kursleitung soll Bindings und Berechtigungen prüfen. |
| SSH verbindet nicht | Konfigurierte **Open terminal**-Aktion verwenden; keine alten Standardpasswörter probieren oder SSH-Server ändern. |
| Terminal zeigt root statt admin im alten Bild | Verwaltete Linux-Knoten verwenden root, RouterOS sein konfiguriertes Admin-Konto. Aktueller Anleitung folgen. |
| Kein Ping im Mitschnitt | Zuerst Capture starten, begrenzten Ping wiederholen, Knoten und Port-Nachbar prüfen, Filter leeren und einige Sekunden warten. |
| Viewer zeigt ERF/LEGACY und unsinnige Zeiten statt Ethernet | Auf der Seite **Restart Capture** wählen, gültige Ethernet/IP-Dekodierung abwarten und Proben wiederholen. Bleibt der Fehler, Knoten/Interface und Screenshot an die Kursleitung melden. |
| Datenstrom nur auf einem LACP-Mitglied | Bei stabilem Hash erwartet. Gleichen begrenzten Datenstrom am anderen Mitglied wiederholen; Anfragen und Antworten trennen. |
| `vlan` am Access-Port leer | Filter leeren und ICMP suchen. Ungetaggte Endpunkt-Frames sind erwartet. |
| ARP-Hilfsprogramm endet mit Code 124 | Sicherheits-Timer abgelaufen; Wiederherstellung durchführen oder begrenzte Aufgabe wie beschrieben wiederholen. |
| Redirects/Ping-Verlust bei ARP-Spoofing | Zuordnung und abgefangene Nutzlast prüfen. Perfekter Ping ist kein notwendiger Umleitungsbeleg. |
| Modbus meldet Exception | Ergebnis notieren; genaue Lese-Unit, Typ, Adresse und Anzahl prüfen. Keine Schreibwerte erraten. |
| HTTP liefert 404 | Antwortender Dienst und fehlender Pfad sind verschieden. `/webvisu.htm` und Port 8080 prüfen. |
| GUI-Wert bleibt gleich | Stabile Prozesswerte sind möglich. Fortlaufenden Anwendungsaustausch prüfen; keine Bedienung nur zur erzwungenen Änderung. |
| Zugriff trotz abschliessendem Firewall-Drop | Frühere Accepts und tatsächlichen Weg prüfen. Verkehr im selben VLAN kann das Gateway vollständig umgehen. |
| Ergebnis weicht vom Durchgang ab | Tatsächliche Ausgabe festhalten und Lab-Änderungen erfragen. Keine Knoten zurücksetzen oder geteilten Entwürfe ersetzen. |

## Weitere Quellen

- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/).
- [RouterOS-Handbuch](https://manual.mikrotik.com/docs/introduction/).
- [Nmap-Portzustände](https://nmap.org/book/man-port-scanning-basics.html).
- [NIST-Leitfaden für OT-Sicherheit](https://csrc.nist.gov/pubs/sp/800/82/r3/final).

Jede Übung enthält Fachquellen und zwei optionale Zusatzteile nach der Lösung. Die [Übersicht](index.md) führt dich der Reihe nach weiter.
