# Das VLAN-Praktikum vorbereiten

Übung 7 ändert einen Endpunkt, zwei Switches und das Gateway im eigenen Lab jeder Person. Bereite einen Checkpoint seines flachen Ausgangszustands für die Wiederherstellung vor. Eine zusätzliche Appliance ist für diese Übung nicht erforderlich.

## Vor dem Start

1. Verwende die vorhandene `ot-sec-flat`-Installation der jeweiligen Person und deren zugewiesene Zugriffs-URL. Zugangsdaten und privater Checkpoint bleiben bei dieser Appliance.
2. Öffne den Exercises-Reader des Labs und wähle sein Live-Netzwerk. Prüfe unabhängige Terminals zu `hirschmann-edge1`, `wago-plc2a`, `sw-acc1`, `sw-dist` und `gw-firewall`. `hirschmann-edge1` muss Management auf `eth0` und `10.50.0.12/8` auf `eth1` haben.
3. Prüfe Adressen, VLAN-Mitgliedschaften und Regelanker aus Aufgabe 1 der Übung 7. VLAN 70 und `172.30.70.0/24` müssen ungenutzt sein. Von `hirschmann-edge1` müssen `wago-plc2a` TCP 22 und 8080 offen sein und `/webvisu.htm` HTTP 200 liefern. Von `wago-plc2a` muss `hirschmann-edge1` TCP 22 offen sein. Behebe fehlgeschlagene Ausgangsprüfungen vor Beginn.
4. Schliesse Mitschnitt- und Terminal-Tabs. Erstelle auf der **Appliance-Konsole des studentischen Labs** einen privaten Checkpoint mit dem installierten Lebenszyklus-Befehl:

    ```bash
    sudo nwos-ot-student checkpoint ex7-flat
    sudo nwos-ot-student start
    sudo nwos-ot-student status
    ```

    Checkpoint stoppt das zugehörige Lab; `start` startet es wieder. Verwende einen neuen Namen, falls `ex7-flat` schon existiert, notiere ihn und ersetze `ex7-flat` im Restore-Befehl unten durch denselben Namen. Warte auf gesunde Dienste und funktionierende Knotenterminals sowie Ausgangsprüfungen.
5. Notiere den Checkpoint-Namen für das Lab der jeweiligen Person. Halte für die Wiederherstellung einen Konsolenzugang der Kursleitung bereit.

## Abschliessen und zurücksetzen

Studierende führen nach den zwei optionalen Extras die manuelle Bereinigung aus und wiederholen die Ausgangsprüfungen in beiden Richtungen. Die Linux-Adressänderungen sind vorübergehend; RouterOS-Änderungen bleiben gespeichert. Ein Neustart während der Übung kann deshalb einen gemischten Zustand erzeugen. Stelle bei einem Fehler oder Geräteneustart den ganzen Checkpoint wieder her.

Auf der **Appliance-Konsole desselben studentischen Labs**:

```bash
sudo nwos-ot-student restore ex7-flat
sudo nwos-ot-student status
```

Restore stoppt und ersetzt den zugehörigen Installationszustand und startet das Lab. Warte auf gesunde Dienste, öffne den Reader erneut und wiederhole Aufgabe 1. `hirschmann-edge1` muss wieder `10.50.0.12/8` haben, VLAN 70 und `ex7-`-Regeln müssen fehlen, HTTP muss 200 liefern und beide ursprünglichen SSH-Proben müssen offen sein. RAM-Logs werden nicht zurückgesetzt; ordne Belege über die Zeit zu, statt das Verschwinden alter Logeinträge vorauszusetzen.

Bewahre Checkpoints als root mit Modus `0600` ausserhalb von Git und Verteilungsartefakten auf: Sie enthalten private Installationsdaten. Der Lebenszyklus-Befehl benötigt das passende installierte Release und dessen festgelegte Images. Bei fehlgeschlagenem Restore prüft die Kursleitung zuerst die Konsole.

Der Reader bietet Terminals, Mitschnitt und VLAN-Ansichten. Er erstellt oder restauriert keine Appliance für Studierende. Toolbox speichert einen Entwurf und wendet diese VLAN-Konfiguration nicht an.
