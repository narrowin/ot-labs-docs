# Übung 6: OT-Protokolle anhand von Belegen erkennen

**Zeit:** 40–50 Minuten. **Lab:** aktuelles studentisches `ot-sec-flat`. **Niveau:** Geführte Praxis.

Erkenne tatsächliche Protokollnachrichten, führe einen begrenzten Modbus-Lesezugriff aus und trenne Anwendungsbelege von Vermutungen anhand einer Portnummer. Ändere keine PLC-Werte, lade keine Programme und bediene keine Prozesssteuerung.

**Lernziele:** Belege für Modbus, WebVisu/HTTP, CODESYS und OPC UA finden, lokale Ethernet-Discovery von gerouteten Protokollen unterscheiden und Authentifizierungs-/Verschlüsselungsbehauptungen mit beobachtetem Verkehr und Einstellungen vergleichen.

## Dein Inbetriebnahmeauftrag

Nach einer Wartung heisst es: «Die SPS antwortet auf Ping, also ist das Netzwerk in Ordnung.» Erstelle eine aussagekräftigere Prüfung: Erreicht der Client den benötigten Dienst, sendet er die richtige Anfrage und erhält er eine interpretierbare Antwort? Kläre auch, welcher Discovery-Verkehr nicht einfach einen Router überquert.

Ein **Protokoll** definiert Nachrichten zwischen Anwendungen. Ein Modbus-**Register** ist ein adressiertes Datenelement; Bedeutung, Typ und Einheit stehen in der Registertabelle des Herstellers. Eine **Transaction-ID** hilft, Anfrage und Antwort innerhalb derselben TCP-Verbindung zuzuordnen. Ein offener Port allein beantwortet keine dieser Anwendungsfragen.

**Vorhersage:** Beweist eine erfolgreiche TCP-Verbindung zu Port 502, dass das angefragte Register existiert oder sein Wert eine Temperatur ist?

## Vorbereitung

Öffne [WAGO B's Terminal](/network/terminal?node=wago-plc2b-vlan10) für Client-Befehle und [WAGO A's Terminal](/network/terminal?node=wago-plc2a-vlan10) für lokale Socket-Prüfungen. Ziel ist A `10.10.0.11`, Client B `10.10.0.12`.

[Öffne WAGO A's Datenport-Mitschnitt](/network/capture?node=wago-plc2a-vlan10&interface=eth1) und wähle **Start Capture**. Lass ihn für die Hauptaufgaben laufen. Der Übungstitel öffnet diese Anleitung neben dem Mitschnitt.

??? info "Du verwendest die eigenständige Dokumentation?"
    Öffne **Netmap** deines zugewiesenen Live-Labs und suche die stabilen WAGO-Knotennamen oben. Wähle **Open terminal** für Client-/Server-Prüfungen. Wähle auf WAGO A **Capture packets**, Interface **eth1**, beziehungsweise **Open WebVisu** für die Visualisierung. Reader-Links wählen diese Ziele vor und behalten Netzwerk/Snapshot bei. WebVisu ist nur für genehmigte, antwortende Runtimes in der Live-Ansicht verfügbar.

## Aufgabe 1: Vor dem Prüfen beobachten

Beobachte etwa 15 Sekunden. Leere den Anzeigefilter und öffne **Statistics → Protocol Hierarchy**. Notiere erkannte Protokolle und schliesse den Dialog.

Probiere `modbus` und `pn_dcp` einzeln aus. Notiere für eine Nachricht Quelle, Ziel, Protokoll, Zeit und Frame-Nummer. Verwende bei Discovery-Frames ohne IP die Ethernet-Adressen.

**Kontrollpunkt:** Im Ausgangszustand gibt es Modbus-Hintergrundverkehr und vom Lab erzeugte PROFINET-DCP-Identify-Nachrichten. Andere Lab-Anwendungen können Modbus lesen und schreiben; ihre Beobachtung bedeutet nicht, dass du sie ausgelöst hast. Fehlt ein Protokoll im Zeitfenster, notiere «nicht beobachtet», nicht «nicht installiert».

Suche vor deinem eigenen Test eine Hintergrundanfrage mit Antwort, sofern vorhanden. Notiere Client, Server und Funktion. In der Kursanlage ist ABB das Leitsystem; WAGO steuert Aufbereitungsfunktionen. Prüfe die tatsächliche Quelle anhand von Netmap; nicht jeder Modbus-Client ist automatisch ABB. Dein B → A-Lesezugriff unten ist eine Diagnosetransaktion und kein Beweis für die normale Steuerungsbeziehung. Ein ungeklärter Schreibzugriff braucht Verantwortlichen und Zweck, keine automatische Blockade.

## Aufgabe 2: Einen Modbus-Lesezugriff verfolgen

Prüfe auf **WAGO B** das vorhandene Werkzeug und führe genau diesen Befehl aus:

```bash
command -v mbpoll
mbpoll -m tcp -a 1 -t 4 -r 1 -c 2 -1 -o 2 10.10.0.11
```

`-t 4` wählt Holding-Register, `-r 1` die erste angezeigte Referenz, `-c 2` zwei Register, `-1` einen einzigen Poll und `-o 2` die begrenzte Antwortwartezeit. Es folgen keine Schreibwerte: Das ist ein Lesezugriff.

Filtere in Wireshark `modbus && ip.addr == 10.10.0.12`, um deinen Zugriff vom Hintergrund zu trennen. Klappe **Modbus/TCP** und **Modbus** auf. Ordne Anfrage und Antwort anhand der Transaction-ID zu.

**Kontrollpunkt:** Die Anfrage verwendet Funktion **3 (Read Holding Registers)**, Startadresse **0** auf dem Draht und Anzahl **2**. Die Antwort liefert zwei Registerwerte. Beim Durchgang waren dies Nullen; Prozesswerte können variieren. Notiere die Werte ohne unbelegte technische Einheiten.

??? info "Hinweis: TCP funktioniert, Lesen schlägt fehl"
    Auch eine Exception-Antwort belegt Modbus. Prüfe Unit 1, Holding-Register sowie genaue Adresse und Anzahl. Notiere den Fehler; probiere weder Schreibfunktionen noch ganze Registerbereiche. Startet die Runtime gerade neu, warte und wiederhole diesen einen Lesezugriff.

## Aufgabe 3: HTTP bestätigen und die Anwendung ansehen

Auf **WAGO B**:

```bash
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

Filtere `http.request && ip.src == 10.10.0.12`. Finde den GET-Pfad und untersuche mit `http && ip.addr == 10.10.0.12` die Antwort.

[WAGO A's WebVisu öffnen](/network/webvisu?node=wago-plc2a-vlan10). Notiere zweimal im Abstand einiger Sekunden einen angezeigten Wert oder Zustand, ohne Bedienelemente zu betätigen. Ein stabiler Prozesswert ist gültig. Schliesse danach die Visualisierung.

**Kontrollpunkt:** Die direkte Anfrage liefert HTTP 200; der Mitschnitt enthält GET `/webvisu.htm`. Die Visualisierung stellt Prozessbedienung/-zustände dar und tauscht weiter Anwendungsdaten aus. Stabile Werte allein beweisen keine eingefrorene Seite. Die Browser-Proxy-Verbindung und B's direkte Anfrage verwenden unterschiedliche Quellwege. Erwarte für die Browser-Anfrage nicht B als Quelle.

## Aufgabe 4: Listener, Protokoll und Sicherheit unterscheiden

Auf **WAGO A**:

```bash
ss -lnt
ss -lnu
```

Notiere die lauschenden Adressen und Ports für den Belegvergleich unten.

Vergleiche die Listener mit den aufgezeichneten Nachrichten. Fehlt eine OPC-UA-Transaktion, notiere «Listener vorhanden; Anwendung/Sitzungssicherheit ungeprüft». Damit ist die Hauptaufgabe erfüllt. Du musst weder Python schreiben noch eine OPC-UA-Sitzung aufbauen.

??? info "Optional: OPC-UA-Transport untersuchen"
    Verwende auf **WAGO B** das vorhandene `uv` und die Python-Standardbibliothek für eine begrenzte OPC-UA-Transportbegrüssung:

    ```bash
    uv run --offline --no-project python - <<'PY'
    import socket, struct
    url = b'opc.tcp://10.10.0.11:4840'
    hello = b'HELF' + struct.pack('<7I', 32 + len(url), 0, 65536, 65536, 0, 0, len(url)) + url
    with socket.create_connection(('10.10.0.11', 4840), timeout=3) as connection:
        connection.sendall(hello)
        reply = connection.recv(8, socket.MSG_WAITALL)
        if len(reply) != 8:
            raise SystemExit('Incomplete OPC UA transport header; record the capture and stop.')
        print('OPC UA transport response:', reply[:4].decode('ascii', errors='replace'))
        print('Message bytes:', struct.unpack('<I', reply[4:8])[0])
    PY
    ```

    Filtere `opcua && ip.addr == 10.10.0.12`. Finde **Hello** und **Acknowledge**. Der Durchgang lieferte `ACKF`, Nachrichtengrösse 28. Diese Begrüssung handelt Transportgrenzen aus und endet vor einer Anwendungssitzung; sie sendet keinen PLC-Lese-/Schreibzugriff. Vergleiche die Belegebenen unten:

| Aussage | Nötiger Beleg |
| --- | --- |
| TCP 4840 lauscht | Lokaler Socket oder erfolgreiche TCP-Verbindung |
| OPC UA wurde ausgetauscht | Dekodierte OPC-UA-Nachrichten statt nur TCP 4840 |
| Eine OPC-UA-Sitzung verschlüsselt Daten | Ausgehandelter Modus/Policy und passende Sitzungsbelege |
| WebVisu funktioniert | Live-Seite und erfolgreiche fortlaufende Anwendungsübertragung |
| Modbus-Lesen war erfolgreich | Anfrage und passende Antwort mit Funktion und Registerdaten |

**Kontrollpunkt:** Diese Version lauscht auf TCP 502, 4840, 8080 und 11740; lokal ist auch CODESYS UDP 1740 sichtbar. Übernimm nicht die alte Annahme, sämtlicher CODESYS-Verkehr verwende 1217/2455. Falls durchgeführt, belegt die Begrüssung OPC-UA-Transportverhalten; Sitzungssicherheit bleibt **ungeprüft**. OPC UA unterstützt `None`, `Sign` und `SignAndEncrypt`; ein offener Port oder Hello/Acknowledge verrät keinen ausgehandelten Sitzungsmodus.

Schliesse Capture und Terminals oder behalte sie für einen Zusatzteil offen.

## Dein Inbetriebnahmeprotokoll

Notiere für Modbus, HTTP und OPC UA jeweils Quelle → Ziel, Dienst, tatsächliche Nachricht/Ergebnis und offene Fragen. Ergänze bei Modbus Funktion, Adresse und passende Antwort; lasse die Einheit ohne Registertabelle offen. Unterscheide bei HTTP direkten Client-Test und Browser-Proxy. Bei OPC UA beweisen weder Listener noch optionaler Transport-Handshake die Sitzungssicherheit. Daraus entsteht eine Kommunikationsmatrix für Übung 7.

## Lösung

??? success "Vergleiche deine Belege"
    Protocol Hierarchy klassifiziert aufgezeichnete Nachrichten. Deine einzelne Modbus-Anfrage verwendet Funktion 3 und eine nullbasierte Adresse, während mbpoll standardmässig Referenz 1 anzeigt. Die passende Antwort belegt das Lesen; die sichtbaren Felder zeigen, dass dieser Austausch am Capture-Link lesbar ist.

    HTTP-GET und Antwort belegen die Anwendung auf 8080. WebVisu verwendet eigene Nachrichten; eine geladene Seite beweist keine Modbus-Verwendung. Ein TCP-Listener ist ein schwächerer Beleg als eine dekodierte Transaktion. Beworbene Sicherheitsfähigkeiten eines Protokolls und offene Ports beweisen keine tatsächlichen Sitzungseinstellungen.

## Extra 1: Eine HMI-Registerzuordnung prüfen

**Optional, 10 Minuten.** Ein Ersatz-HMI bezeichnet das erste Register mit «0», die Wartungsnotiz mit «1». Kläre, ob dies allein einen falschen Prozesswert bedeutet. Starte gegebenenfalls einen neuen Mitschnitt an A's Datenport. Auf **WAGO B**:

```bash
mbpoll -m tcp -a 1 -t 4 -r 1 -c 2 -1 -o 2 10.10.0.11
mbpoll -m tcp -a 1 -t 4 -0 -r 0 -c 2 -1 -o 2 10.10.0.11
```

Filtere `modbus && ip.addr == 10.10.0.12`. Vergleiche Startadresse und Anzahl der beiden Anfragen sowie mbpolls ausgegebene Referenzen.

**Kontrollpunkt:** Beide fragen Drahtadresse 0 mit Anzahl 2 ab. Der erste Befehl beschriftet `[1]` und `[2]`, der zweite `[0]` und `[1]`. Werte können sich zwischen den Abfragen ändern. Unterschiedliche Anzeigennummerierung bedeutet keine andere Drahtadresse. Hänge keine Werte an; das würde den Befehl zum Schreibzugriff machen.

## Extra 2: Fehlende Geräteerkennung beim Fernzugriff erklären

**Optional, 10 Minuten.** Ein Fernwartungstechniker erreicht eine SPS per IP, findet sie aber nicht über lokale PROFINET-Geräteerkennung. Würde ein weiterer offener TCP-Port helfen? Filtere A's Mitschnitt mit `pn_dcp`. Wähle eine Identify-Nachricht und untersuche Ethernet-Ziel, EtherType und DCP-Service-Felder. Suche in der [DCP-Feldreferenz](https://www.wireshark.org/docs/dfref/p/pn_dcp.html) `pn_dcp.service_id` und den Stationsnamen.

Vergleiche nun `pn_dcp && ip` mit `pn_dcp` allein.

**Kontrollpunkt:** Diese Lab-Discovery-Frames verwenden EtherType `0x8892` ohne IP-/TCP-Header. Eine reine TCP-Port-Matrix kann diesen Fluss nicht beschreiben. Ein angekündigter Stationsname ist eine Behauptung des Senders, keine authentifizierte Identität. Identify-Nachrichten beweisen keine vollständige PROFINET-IO-Controller-/Device-Beziehung des Simulators.

Erkläre die unterschiedlichen Auswirkungen einer gerouteten VLAN-Grenze und einer IP-Firewall auf diesen lokalen Ethernet-Verkehr. Schliesse die Werkzeug-Tabs. Weiter mit [Übung 7](ex7-segmentation-firewall.md).

## Weiterforschen

- [Modbus-Spezifikationen](https://www.modbus.org/modbus-specifications): Suche Funktion 03; unterscheide klassisches Modbus TCP und Modbus Security mit TLS.
- [mbpoll-Referenz](https://github.com/epsilonrt/mbpoll): Vergleiche `-0`, `-1` und optionale Schreibwerte.
- [OPC-Foundation-Sicherheitsmodi](https://reference.opcfoundation.org/specs/OPC-10000-2/4.8): Vergleiche Integrität und Vertraulichkeit.
- [OPC-UA-Hello-Nachricht](https://reference.opcfoundation.org/specs/OPC-10000-6/7.1.2.3): Finde Endpoint-URL und Puffergrenzen deiner Begrüssung.
- [CODESYS WebVisu ausführen](https://content.helpme-codesys.com/en/CODESYS%20Visualization/_visu_execute_webvisu.html): Unterscheide Webvisualisierung und Engineering-Verbindung.
