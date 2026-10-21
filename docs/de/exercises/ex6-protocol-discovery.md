# Übung 6: OT-Protokolle anhand von Belegen erkennen

**Zeitbedarf:** 40–50 Minuten. **Lab:** dein eigenes `ot-sec-flat`-Lab. **Niveau:** angeleitete praktische Übung.

Erkenne tatsächliche Protokollnachrichten, führe einen begrenzten Modbus-Lesezugriff aus und trenne Anwendungsbelege von Vermutungen anhand einer Portnummer. Ändere keine SPS-Werte, lade keine Programme und bediene keine Prozesssteuerung.

**Lernziele:** Modbus- und HTTP-Nachrichten erkennen, CODESYS-/OPC-UA-Listener prüfen, lokale Geräteerkennung über Ethernet von gerouteten Protokollen unterscheiden und offene Sicherheitsfragen benennen.

## Dein Inbetriebnahmeauftrag

Nach einer Wartung heisst es: «Die SPS antwortet auf Ping, also ist das Netzwerk in Ordnung.» Prüfe genauer: Erreicht der Client den benötigten Dienst, sendet er die richtige Anfrage und erhält er eine interpretierbare Antwort? Kläre auch, welcher Verkehr zur Geräteerkennung auf das lokale Ethernet-Netz beschränkt ist.

Ein **Protokoll** definiert Nachrichten zwischen Anwendungen. Ein Modbus-**Register** ist ein adressiertes Datenelement; Bedeutung, Typ und Einheit stehen in der Registertabelle des Herstellers. Die **Transaction-ID** (Transaktionskennung) hilft, eine Antwort der zugehörigen Anfrage innerhalb derselben TCP-Verbindung zuzuordnen. Ein offener Port allein beantwortet keine dieser Anwendungsfragen.

**Vorhersage:** Beweist eine erfolgreiche TCP-Verbindung zu Port 502, dass das angefragte Register existiert oder sein Wert eine Temperatur ist?

## Vorbereitung

Öffne [das Terminal von `wago-plc2b-vlan10`](/network/terminal?node=wago-plc2b-vlan10) für Client-Befehle und [das Terminal von `wago-plc2a-vlan10`](/network/terminal?node=wago-plc2a-vlan10) für lokale Socket-Prüfungen. Ziel ist `wago-plc2a-vlan10` `10.10.0.11`, Client `wago-plc2b-vlan10` `10.10.0.12`.

[Öffne den Mitschnitt am Datenport von `wago-plc2a-vlan10`](/network/capture?node=wago-plc2a-vlan10&interface=eth1) und wähle **Start Capture**. Lass ihn für die Hauptaufgaben laufen. Der Übungstitel öffnet diese Anleitung neben dem Mitschnitt.

??? info "Du verwendest die eigenständige Dokumentation?"
    Öffne **Netmap** deines zugewiesenen Live-Labs und suche `wago-plc2a-vlan10` und `wago-plc2b-vlan10`. Wähle **Open terminal** für Client-/Server-Prüfungen. Wähle auf `wago-plc2a-vlan10` **Capture packets**, Interface **eth1**, oder **Open WebVisu** für die Visualisierung. Links in der Übungsansicht wählen diese Ziele vor und behalten Netzwerk/Snapshot bei. WebVisu ist nur für genehmigte, antwortende Runtimes in der Live-Ansicht verfügbar.

## Aufgabe 1: Vor dem Prüfen beobachten

Beobachte etwa 15 Sekunden. Leere den Anzeigefilter und öffne **Statistics → Protocol Hierarchy**. Notiere erkannte Protokolle und schliesse den Dialog.

Probiere `modbus` und `pn_dcp` einzeln aus. Notiere für eine Nachricht Quelle, Ziel, Protokoll, Zeit und Frame-Nummer. Verwende bei Frames zur Geräteerkennung ohne IP die Ethernet-Adressen.

**Kontrollpunkt:** Im Ausgangszustand gibt es Modbus-Hintergrundverkehr und vom Lab erzeugte PROFINET-DCP-Identify-Nachrichten. Andere Lab-Anwendungen können Modbus lesen und schreiben; ihre Beobachtung bedeutet nicht, dass du sie ausgelöst hast. Fehlt ein Protokoll im Zeitfenster, notiere «nicht beobachtet», nicht «nicht installiert».

Suche vor deinem eigenen Test eine Hintergrundanfrage mit Antwort, sofern vorhanden. Notiere Client, Server und Funktion. In der Kursanlage ist `abb-800xa-vlan40` das Leitsystem; `wago-plc2a-vlan10` und `wago-plc2b-vlan10` sind die Endpunkte der Aufbereitungssteuerungen. Prüfe die tatsächliche Quelle anhand von Netmap; nicht jeder Modbus-Client ist automatisch `abb-800xa-vlan40`. Dein Lesezugriff von `wago-plc2b-vlan10` auf `wago-plc2a-vlan10` unten ist eine Diagnosetransaktion und kein Beweis für die normale Steuerungsbeziehung. Kläre bei einem unerwarteten Schreibzugriff zuerst, wer ihn veranlasst hat und welchem Zweck er dient. Blockiere ihn nicht allein aufgrund dieser Beobachtung.

## Aufgabe 2: Einen Modbus-Lesezugriff verfolgen

Prüfe auf **`wago-plc2b-vlan10`** das vorhandene Werkzeug und führe genau diesen Befehl aus:

```bash
command -v mbpoll
mbpoll -m tcp -a 1 -t 4 -r 1 -c 2 -1 -o 2 10.10.0.11
```

`-t 4` wählt Holding-Register, `-r 1` die erste angezeigte Referenz, `-c 2` zwei Register, `-1` eine einmalige Abfrage und `-o 2` eine Antwortwartezeit von höchstens zwei Sekunden. Es folgen keine Schreibwerte: Das ist ein Lesezugriff.

Filtere in Wireshark `modbus && ip.addr == 10.10.0.12`, um deinen Zugriff vom Hintergrund zu trennen. Klappe **Modbus/TCP** und **Modbus** auf. Ordne Anfrage und Antwort anhand der Transaction-ID zu.

**Kontrollpunkt:** Die Anfrage verwendet Funktion **3 (Read Holding Registers)**, Startadresse **0** im übertragenen Paket und Anzahl **2**. Die Antwort liefert zwei Registerwerte. Beim Durchgang waren dies Nullen; Prozesswerte können variieren. Notiere die Werte ohne unbelegte technische Einheiten.

??? info "Hinweis: TCP funktioniert, Lesen schlägt fehl"
    Auch eine Exception-Antwort belegt Modbus. Prüfe Unit 1, Holding-Register sowie genaue Adresse und Anzahl. Notiere den Fehler; probiere weder Schreibfunktionen noch ganze Registerbereiche. Startet die Runtime gerade neu, warte und wiederhole diesen einen Lesezugriff.

## Aufgabe 3: HTTP bestätigen und die Anwendung ansehen

Auf **`wago-plc2b-vlan10`**:

```bash
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

Filtere `http.request && ip.src == 10.10.0.12`. Finde den GET-Pfad und untersuche mit `http && ip.addr == 10.10.0.12` die Antwort.

[Die WebVisu von `wago-plc2a-vlan10` öffnen](/network/webvisu?node=wago-plc2a-vlan10). Notiere zweimal im Abstand einiger Sekunden einen angezeigten Wert oder Zustand, ohne Bedienelemente zu betätigen. Ein stabiler Prozesswert ist gültig. Schliesse danach die Visualisierung.

**Kontrollpunkt:** Die direkte Anfrage liefert HTTP 200; der Mitschnitt enthält GET `/webvisu.htm`. Die Visualisierung zeigt Prozesszustände und Bedienelemente an und tauscht fortlaufend Anwendungsdaten aus. Stabile Werte allein beweisen keine eingefrorene Seite. Der Browser greift über einen Proxy zu; die Anfrage von `wago-plc2b-vlan10` geht direkt an `wago-plc2a-vlan10`. Erwarte für die Browser-Anfrage nicht `wago-plc2b-vlan10` als Quelle.

## Aufgabe 4: Listener, Protokoll und Sicherheit unterscheiden

Auf **`wago-plc2a-vlan10`**:

```bash
ss -lnt
ss -lnu
```

Notiere, auf welchen Adressen und Ports Dienste auf Verbindungen oder Daten warten. Solche Dienste werden als Listener bezeichnet. Im Ausgangszustand gehören TCP 502, 4840, 8080 und 11740 sowie CODESYS UDP 1740 dazu.

Prüfe auf **`wago-plc2b-vlan10`** die TCP-Verbindung zu Port 4840:

```bash
nc -vz -w 3 10.10.0.11 4840
```

Filtere den Mitschnitt mit `ip.addr == 10.10.0.12 && tcp.port == 4840`. Finde SYN und SYN/ACK. Dieser Befehl öffnet und schliesst eine TCP-Verbindung; er sendet keine OPC-UA-Anwendungsnachricht.

| Prüfung | Was das Ergebnis belegt |
| --- | --- |
| Lokaler Listener auf TCP 4840 | Ein Dienst wartet auf dieser Adresse und diesem Port auf Verbindungen |
| TCP-Verbindung von `wago-plc2b-vlan10` erfolgreich | Der Listener ist von `wago-plc2b-vlan10` erreichbar |
| Dekodierte Modbus-Anfrage mit Antwort | Der angeforderte Modbus-Lesezugriff wurde verarbeitet |
| HTTP-Anfrage und Antwort sowie Live-WebVisu | Die Webanwendung antwortet; laufende Übertragungen zeigen die Aktualisierung |
| OPC-UA-Authentifizierung oder Verschlüsselung | Mit dieser TCP-Verbindung nicht geprüft |

**Kontrollpunkt:** Unterscheide Port-Erreichbarkeit und Anwendungsverhalten. Notiere die OPC-UA-Sitzungssicherheit als **ungeprüft**. Für diese Prüfung wären eine OPC-UA-Client-Sitzung und deren ausgehandelte Sicherheitseinstellungen nötig.

Schliesse den Mitschnitt und die Terminals oder behalte sie für eine Zusatzaufgabe offen.

## Dein Inbetriebnahmeprotokoll

Notiere für Modbus, HTTP und OPC UA jeweils Quelle und Ziel, Dienst, beobachtete Nachrichten, Ergebnis und offene Fragen. Ergänze bei Modbus Funktion, Adresse und passende Antwort; lasse die Einheit ohne Registertabelle offen. Unterscheide bei HTTP direkten Client-Test und Browser-Proxy. Bei OPC UA bleibt die Sitzungssicherheit nach der Listener- und TCP-Verbindungsprüfung ungeprüft. Daraus entsteht eine Kommunikationsmatrix für Übung 7.

## Lösung

??? success "Vergleiche deine Belege"
    Protocol Hierarchy klassifiziert aufgezeichnete Nachrichten. Deine einzelne Modbus-Anfrage verwendet Funktion 3 und eine nullbasierte Adresse, während mbpoll standardmässig Referenz 1 anzeigt. Die passende Antwort belegt das Lesen; die sichtbaren Felder zeigen, dass dieser Austausch am aufgezeichneten Link lesbar ist.

    HTTP-GET und Antwort belegen die Anwendung auf 8080. WebVisu verwendet eigene Nachrichten; eine geladene Seite beweist keine Modbus-Verwendung. Ein TCP-Listener ist ein schwächerer Beleg als eine dekodierte Transaktion. Beworbene Sicherheitsfähigkeiten eines Protokolls und offene Ports beweisen keine tatsächlichen Sitzungseinstellungen.

## Extra 1: Eine HMI-Registerzuordnung prüfen

**Optional, 10 Minuten.** Ein Ersatzbediengerät (HMI) bezeichnet das erste Register mit «0», die Wartungsnotiz mit «1». Kläre, ob dies allein einen falschen Prozesswert bedeutet. Starte gegebenenfalls einen neuen Mitschnitt am Datenport von `wago-plc2a-vlan10`. Auf **`wago-plc2b-vlan10`**:

```bash
mbpoll -m tcp -a 1 -t 4 -r 1 -c 2 -1 -o 2 10.10.0.11
mbpoll -m tcp -a 1 -t 4 -0 -r 0 -c 2 -1 -o 2 10.10.0.11
```

Filtere `modbus && ip.addr == 10.10.0.12`. Vergleiche die Startadresse und die Anzahl der Register in beiden Anfragen sowie die von mbpoll ausgegebenen Referenznummern.

**Kontrollpunkt:** Beide Befehle fragen zwei Register ab der übertragenen Registeradresse 0 ab. Der erste Befehl zeigt die Referenznummern `[1]` und `[2]`, der zweite `[0]` und `[1]`. Werte können sich zwischen den Abfragen ändern. Eine unterschiedliche Nummerierung in der Anzeige bedeutet nicht, dass eine andere Registeradresse übertragen wird. Hänge keine Werte an; das würde den Befehl zum Schreibzugriff machen.

## Extra 2: Fehlende Geräteerkennung beim Fernzugriff erklären

**Optional, 10 Minuten.** Ein Fernwartungstechniker erreicht eine SPS per IP, findet sie aber nicht über lokale PROFINET-Geräteerkennung. Würde ein weiterer offener TCP-Port helfen? Filtere den Mitschnitt von `wago-plc2a-vlan10` mit `pn_dcp`. Wähle eine Identify-Nachricht und untersuche Ethernet-Ziel, EtherType und DCP-Service-Felder. Suche in der [DCP-Feldreferenz](https://www.wireshark.org/docs/dfref/p/pn_dcp.html) `pn_dcp.service_id` und den Stationsnamen.

Vergleiche nun `pn_dcp && ip` mit `pn_dcp` allein.

**Kontrollpunkt:** Die Frames zur Geräteerkennung im Lab verwenden EtherType `0x8892` ohne IP-/TCP-Header. Eine Tabelle mit TCP-Ports kann diesen Datenverkehr nicht beschreiben. Ein angekündigter Stationsname ist eine Behauptung des Senders, keine authentifizierte Identität. Identify-Nachrichten allein beweisen nicht, dass der Simulator auch die zyklische PROFINET-IO-Kommunikation zwischen Controller und Gerät nachbildet.

Erkläre die unterschiedlichen Auswirkungen einer gerouteten VLAN-Grenze und einer IP-Firewall auf diesen lokalen Ethernet-Verkehr. Schliesse die Werkzeug-Tabs. Weiter mit [Übung 7](ex7-segmentation-firewall.md).

## Weiterforschen

- [Modbus-Spezifikationen](https://www.modbus.org/modbus-specifications): Suche Funktion 03; unterscheide klassisches Modbus TCP und Modbus Security mit TLS.
- [mbpoll-Referenz](https://github.com/epsilonrt/mbpoll): Vergleiche `-0`, `-1` und optionale Schreibwerte.
- [OPC-Foundation-Sicherheitsmodi](https://reference.opcfoundation.org/specs/OPC-10000-2/4.8): Vergleiche Integrität und Vertraulichkeit.
- [CODESYS WebVisu ausführen](https://content.helpme-codesys.com/en/CODESYS%20Visualization/_visu_execute_webvisu.html): Unterscheide Webvisualisierung und Engineering-Verbindung.
