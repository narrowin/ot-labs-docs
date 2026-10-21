# Übung 8: Ein Altsystem mit der Diode mikrosegmentieren

**35–45 Minuten.** Verwende dein eigenes Lab mit `vlan10-diode` und `protected-linux`. Beginne im flachen Ausgangszustand. Übung 7 ist keine Voraussetzung; falls du sie durchgeführt hast, schliesse zuerst die Bereinigung in [Übung 7](ex7-segmentation-firewall.md) ab. Führe die Bereinigung dieser Übung auch durch, wenn du die beiden optionalen Extras auslässt.

**Dein Auftrag:** Im Entsorgungsbeispiel des Handouts gefährdet ein ungepatchtes System den Rest seiner Zone (PDF-Seiten 149–152, aufgedruckte Folien 158–161). Hier steht `protected-linux` für einen älteren Rechner in diesem Anlagenteil. Erhalte den Wartungszugriff vom `dell-ews1`-Engineering-Arbeitsplatz, blockiere einen anderen Zellenrechner und begrenze neue Verbindungen, die der ältere Rechner selbst aufbaut.

**Du lernst:** eine Grenze um einen einzelnen Endpunkt durchzusetzen, eine Ausnahme für genau eine Adresse zu wählen, beide Richtungen zu prüfen und funktionierende Antworten einer erlaubten Anwendung nachzuweisen.

Das Lab verwendet eine **geroutete Diode** mit konfigurierbarer Firewall-Policy. Sie ist keine physische Einweg-Datendiode. Ihr Profil filtert Adressen und Subnetze statt einzelner Dienstports. Linux ist der Stellvertreter für diese Aufgabe; die Diode patcht ihn nicht und weist ihm kein IEC-62443-Security-Level zu.

## Aufgabe 1: Die Grenze in der GUI finden

1. Öffne Produktmenü → **Diodes**, danach `vlan10-diode`. Notiere das zugewiesene Profil **Microsegmentation - baseline**. Warte vor den Ausgangsmessungen auf **Production current** und einen Zeitpunkt bei **Last confirmed** unter **Configuration Apply Status**. Notiere **Config Revision** und vergleiche mit **ASSIGNED REV**.
2. Untersuche die Schnittstellen der Diode unter **Inventory**. Verfolge in **Netmap** den Weg von sw-acc2 über die Diode zu `protected-linux`. Die gepunktete Switch-Verbindung ist eine gespeicherte Verkabelungsnotiz. Schnittstellen- und ARP-Meldungen liefern weitere Belege; eine Linie im Diagramm beweist allein keine Filterung.
3. Unterscheide die Seiten: WAN `10.10.0.21` zeigt zu VLAN 10; LAN `192.168.254.254/24` zu `protected-linux` `192.168.254.1`. Die beiden folgenden Clients bleiben in VLAN 10. Sie verbinden sich mit **`10.10.0.21`**; die Diode leitet erlaubten Verkehr an Linux weiter.

| Rolle | Terminal | Datenadresse im Ausgangszustand |
| --- | --- | --- |
| Berechtigter Engineering-Arbeitsplatz | [`dell-ews1`](/network/terminal?node=dell-ews1-vlan50) | `10.50.0.11/8` |
| Anderer Zellenrechner | [`hirschmann-edge1`](/network/terminal?node=hirschmann-edge1-vlan50) | `10.50.0.12/8` |
| Stellvertreter für das Altsystem | [`protected-linux`](/network/terminal?node=protected-linux) | `192.168.254.1/24` |

Verwende durchgehend diese eingerichteten Management-Terminals. Ein offenes Management-Terminal ermöglicht die Wiederherstellung; es beweist keine Freigabe der Datenverbindung durch die Diode. Teste die Policy nicht über Management-Adressen.

## Aufgabe 2: Die Anwendungskommunikation im Ausgangszustand prüfen

Notiere auf **`dell-ews1`**, danach auf **`hirschmann-edge1`**, die tatsächliche Adresse und die ausgewählte Route:

```bash
ip -4 -br address show dev eth1
ip route get 10.10.0.21
```

Führe auf **beiden Clients** aus:

```bash
curl --interface eth1 --connect-timeout 3 --max-time 5 -sS http://10.10.0.21/
nc -vz -w 3 10.10.0.21 22
```

Beide müssen die Seite **Diode protected Linux endpoint** erhalten und die TCP-Verbindung zu Port 22 aufbauen können. `nc` prüft die Verbindung; es meldet keinen SSH-Benutzer an.

Prüfe auf **`protected-linux`** eine ausgehende Anwendungsverbindung zu `wago-plc2a`:

```bash
curl --interface eth1 --connect-timeout 3 --max-time 5 -sS -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

**Kontrollpunkt:** HTTP liefert `200`. Notiere unter **Diodes → Profiles → Microsegmentation - baseline** beide Richtungen: **From Network** und **From Endpoint** stehen auf **Default allow**, jeweils ohne Adressausnahmen. Erhalte die anderen Profileinstellungen. Weicht der Ausgangszustand ab, gleiche zuerst deine frühere Arbeit ab; ein fehlender Dienst kann keine neu wirksame Sperre belegen.

## Aufgabe 3: Nur den Engineering-Arbeitsplatz zulassen

1. Öffne das zugewiesene Profil und wähle **Edit**.
2. Wähle unter **From Network** die Einstellung **Default deny**. Wähle **Add allowed source**, trage die beobachtete Datenadresse von `dell-ews1` ein und wähle **/32**. Im Ausgangszustand lautet der Eintrag `10.50.0.11/32`; `/32` bedeutet genau eine IPv4-Adresse. Lass **From Endpoint** zunächst auf **Default allow**.
3. Wähle **Save** und bestätige **Save & apply**. Kehre zu den Details der Diode zurück und warte auf die bestätigte beziehungsweise synchronisierte Konfiguration. Ein gespeichertes Profil beweist noch nicht, dass das Gerät es angewendet hat.
4. Sage die Ergebnisse beider Clients voraus und wiederhole danach die HTTP- und TCP-22-Befehle aus Aufgabe 2 mit neuen Verbindungen.

**Kontrollpunkt:** `dell-ews1` erreicht HTTP und TCP 22 weiterhin. `hirschmann-edge1` erreicht beide nicht mehr. Die Clients benötigten keine unterschiedlichen VLANs; ihre Zugriffe unterscheiden sich, weil die Diode die Policy an der Grenze des geschützten Endpunkts durchsetzt. Eine fehlgeschlagene Anfrage allein reicht nicht: Vergleiche mit dem erfolgreichen berechtigten Client und der bestätigten Konfiguration.

Warum nicht `10.50.0.0/24`? Dieses Subnetz enthält beide Clients und würde auch `hirschmann-edge1` zulassen. Warum ist dies keine Benutzerfreigabe? Die IP-Adresse kennzeichnet eine Quelle für die Regel; sie authentifiziert keine Person und macht einen kompromittierten zugelassenen Arbeitsplatz nicht vertrauenswürdig. HTTP und SSH folgen derselben Quelladressregel; dieses Profil kann für dieselbe Quelle nicht «HTTP erlaubt, SSH gesperrt» ausdrücken.

## Aufgabe 4: Neue Verbindungen des Altsystems begrenzen

Das Risiko aus der Folie besteht auch in Gegenrichtung: Ein ungepatchter Rechner könnte selbst Verbindungen zum Rest der Anlage aufbauen.

1. Bearbeite das Profil erneut. Lass **From Network** auf **Default deny** mit der `/32`-Ausnahme für `dell-ews1`. Setze **From Endpoint** auf **Default deny**, ohne erlaubte Ziele. Speichere, bestätige **Save & apply** und warte auf die Konfigurationsbestätigung.
2. Wiederhole auf **`protected-linux`** den ausgehenden HTTP-Befehl zu `wago-plc2a`. Wiederhole danach die HTTP- und TCP-22-Prüfungen auf `dell-ews1`.

**Kontrollpunkt:** Die neue Anfrage von `protected-linux` zu `wago-plc2a` scheitert. Neue Anfragen von `dell-ews1` zu `protected-linux` und deren Antworten funktionieren weiterhin. Damit unterscheidest du neue Verbindungen des Endpunkts von Antworten innerhalb einer erlaubten eingehenden Verbindung. Daraus folgt nicht, dass die Hardware Pakete nur in einer Richtung durchlässt.

## Lösung

??? success "Lösung und betriebliche Schlussfolgerung"
    Die Diode setzt eine Grenze um den älteren Endpunkt durch, während der Rest der Zelle seine bestehenden VLANs behält. Die `/32`-Ausnahme für die Adresse von `dell-ews1` erhält den gewählten Wartungsweg. Das Begrenzen neuer, vom Endpunkt ausgehender Verbindungen reduziert einen Weg zur seitlichen Ausbreitung. Zustandsbehaftete Antworten erhalten die erlaubte eingehende Anwendungskommunikation. Das Entfernen der Ausnahme zeigt die Isolation; das Wiederherstellen des notierten Profils belegt die Rückkehr zum Ausgangszustand.

    Der Schutz hängt davon ab, dass der Verkehr diese Grenze durchläuft. Ein weiterer ungefilterter Anschluss, eine kompromittierte zugelassene Quelle oder eine gefälschte Adressidentität können die Annahmen unterlaufen. Der Management-Weg im Lab ist eine beabsichtigte Ausnahme für die Plattform. Eine Produktionsplanung muss alle Wege, notwendige Prozessabhängigkeiten und tatsächliche Anwendungsberechtigungen berücksichtigen. Die Belege zeigen die Lab-Policy, keine Immunität gegen Angriffe und kein zertifiziertes Security Level.

## Extra 1: Kann der VLAN-10-Verkehr der Diode weiterhin abgefangen werden?

**Optional, 10 Minuten.**

Vergleiche diese Grenze mit Übung 2. Prüfe auf **`protected-linux`** die Route zurück zu `dell-ews1` und die Zuordnung des lokalen Gateways:

```bash
ip route get 10.50.0.11
ip neigh show 192.168.254.254 dev eth1
```

Die Route führt über `192.168.254.254`. `dell-ews1` erreicht dagegen die WAN-Adresse `10.10.0.21` der Diode direkt in VLAN 10, wie in Aufgabe 2 notiert. Eine leere Nachbartabelle bedeutet, dass gerade kein Eintrag gespeichert ist; sie beweist keinen Schutz.

Beantworte vor dem Öffnen der Lösung:

1. Warum können gefälschte ARP-Antworten aus VLAN 10 die lokale Gateway-Zuordnung von `protected-linux` nicht direkt ersetzen?
2. Welche ARP-Einträge auf `dell-ews1` und auf der VLAN-10-Seite der Diode könnte ein Angreifer während der `dell-ews1`-Freigabe aus Aufgabe 3 manipulieren, um deren Kommunikation abzufangen?
3. Würde das Weiterleiten der Pakete von `dell-ews1` mit ihrer ursprünglichen Quell-IP die Freigaberegel verletzen? Würden abgefangene SSH-Pakete deren verschlüsselten Inhalt lesbar machen?

??? success "Lösung: ARP-Grenze und Anwendungsweg unterscheiden"
    **Direkte Manipulation aus VLAN 10: nein.** `protected-linux` und VLAN 10 liegen in unterschiedlichen Ethernet-Segmenten. Diese geroutete Diode leitet ARP-Frames aus VLAN 10 nicht in ihr LAN weiter. Linux ermittelt die MAC-Adresse seines lokalen Gateways `192.168.254.254`, nicht die des externen Gateways aus Übung 2. Der Schutz entsteht durch die Routing-Grenze, nicht durch die `/32`-Regel.

    **Der VLAN-10-Verkehr der Diode kann weiterhin abgefangen werden.** Ihr WAN-Interface teilt VLAN 10 mit `dell-ews1` und einem möglichen Angreifer. Ein manipulierter ARP-Eintrag für `dell-ews1` (`10.50.0.11`) auf der Diode kann deren Antworten zum Angreifer umleiten. Ein manipulierter Eintrag für die Diode (`10.10.0.21`) auf `dell-ews1` kann Anfragen umleiten. Leitet der Angreifer die Kommunikation weiter, kann er zwischen `dell-ews1` und der Diode sitzen. Diese gefälschten ARP-Antworten bleiben in VLAN 10: Sie müssen weder den geschützten Endpunkt erreichen noch dessen ARP-Cache verändern.

    **Die Quell-IP-Regel ist kein ARP-Schutz.** Bleibt die Quell-IP beim Weiterleiten gleich der erlaubten Adresse von `dell-ews1`, trifft die Freigaberegel weiterhin zu. Der erlaubte HTTP-Verkehr lässt sich auf diesem Weg mitlesen. Abgefangene SSH-Pakete bleiben bei korrekter SSH-Authentifizierung und Host-Key-Prüfung verschlüsselt; siehe [SSH-Host-Key-Prüfung](https://www.rfc-editor.org/rfc/rfc4251.html#section-9.3.4). ARP-Spoofing allein macht aus einer gesperrten Quelle keine erlaubte Quelle. Das verbleibende Risiko ist das Abfangen oder Unterbrechen von Verkehr auf der VLAN-10-Seite der Diode.

    Wähle einen Schutz für das verbleibende Risiko: ARP-Prüfung am Switch mit gepflegten IP/MAC-Zuordnungen in VLAN 10 sowie authentifizierte Verschlüsselung, etwa HTTPS oder SSH mit Zertifikats- beziehungsweise Host-Key-Prüfung. Verschlüsselung schützt Anwendungsdaten; sie verhindert nicht, dass ein Angreifer die Verbindung unterbricht. Siehe [RFC 826](https://www.rfc-editor.org/rfc/rfc826.html) und [die Erklärung zur ARP-Prüfung](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9400/software/release/17-13/configuration_guide/sec/b_1713_sec_9400_cg/configuring_dynamic_arp_inspection.html).

Dies ist eine Aufgabe zur Topologie und zur Einschätzung des Angriffswegs. Die Routen- und Nachbarprüfungen belegen die beiden Seiten; sie weisen kein erfolgreiches Abfangen nach. Notiere, wessen ARP-Eintrag angegriffen würde und welcher Verkehr betroffen wäre.

## Extra 2: Endpunkt isolieren und Wiederherstellung prüfen

**Optional, 10 Minuten.** Beginne mit der Policy aus Aufgabe 4. Weise nach, dass das Entfernen der letzten eingehenden Ausnahme den Datenweg des Endpunkts isoliert und der Management-Weg zur Wiederherstellung verfügbar bleibt.

Entferne für eine abschliessende Isolationsprüfung den **allowed source**-Eintrag für `dell-ews1`. Lass beide Richtungen auf **Default deny**. Wende die Änderung an und warte auf die Bestätigung. Neue HTTP- und TCP-22-Prüfungen beider Clients müssen nun scheitern. Das verwaltete Linux-Terminal bleibt über den separaten Management-Weg erreichbar.

Notiere die Ergebnisse beider Clients in der Zeile **Extra 2** unten. Führe danach die Bereinigung durch und prüfe, dass alle drei Anwendungsprüfungen wieder funktionieren.

??? success "Erwartetes Ergebnis"
    Neue HTTP- und TCP-22-Prüfungen beider Clients scheitern. Das separate verwaltete Terminal bleibt erreichbar; es belegt keine Freigabe des Datenzugriffs. Das Wiederherstellen des Ausgangszustands muss den Zugriff beider Clients und die ausgehende HTTP-Anfrage des Endpunkts wieder ermöglichen.

## Bereinigung: Ausgangszustand wiederherstellen und übergeben

Stelle das notierte Ausgangsprofil wieder her: **From Network = Default allow**, **From Endpoint = Default allow**, keine Ausnahmen in beiden Richtungen. Speichere, bestätige **Save & apply** und warte auf die Bestätigung. Wiederhole HTTP/TCP 22 von beiden Clients sowie die HTTP-Anfrage des geschützten Endpunkts an `wago-plc2a`. Alles muss wieder funktionieren. Profilzuordnung, DNS, NTP und Logging müssen den in Aufgabe 2 notierten Einstellungen entsprechen.

Ergänze tatsächliche Ergebnisse und Zeitpunkte in dieser Tabelle:

| Policy-Zustand | `dell-ews1` → `protected-linux` HTTP / TCP 22 | `hirschmann-edge1` → `protected-linux` HTTP / TCP 22 | `protected-linux` → `wago-plc2a` HTTP |
| --- | --- | --- | --- |
| Ausgangszustand | Funktioniert / funktioniert | Funktioniert / funktioniert | HTTP 200 |
| Eingehend nur `dell-ews1` | Funktioniert / funktioniert | Gesperrt / gesperrt | HTTP 200 |
| Eingehend nur `dell-ews1`; keine neuen ausgehenden Verbindungen | Funktioniert / funktioniert | Gesperrt / gesperrt | Gesperrt |
| Extra 2: keine eingehenden Ausnahmen; keine neuen ausgehenden Verbindungen | Gesperrt / gesperrt | Gesperrt / gesperrt | Gesperrt |
| Wiederhergestellter Ausgangszustand | Funktioniert / funktioniert | Funktioniert / funktioniert | HTTP 200 |

Jeder Befehl baut eine neue Verbindung auf. Eine bestehende SSH-Sitzung kann ihren Zustand in der Verbindungsverfolgung behalten und eignet sich nicht als Beleg für eine neu angewendete Sperre. Weicht ein Ergebnis ab, notiere Quelladresse, Ziel, Zeitpunkt, angewendete Revision und tatsächlichen Fehler, bevor du weitere Einstellungen änderst.

Schliesse deine Terminal-Tabs.
