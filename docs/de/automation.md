# Netzwerk-Automatisierung

## Warum automatisieren?

Netzwerkautomatisierung reduziert manuelle Fehler, spart Zeit und gewährleistet Konsistenz über Ihre Infrastruktur. Nach manueller Exploration dieser Laborumgebung können Sie häufige Aufgaben automatisieren:

- Diagnosebefehle über mehrere Geräte ausführen
- Konfigurationen regelmässig sichern
- Systeminformationen für Analysen sammeln
- Firmware-Updates verwalten

Dieser Leitfaden zeigt, wie Sie mit dem Open-Source-Werkzeug [networka](https://narrowin.github.io/networka/) beginnen, einem Multi-Vendor-Netzwerkautomatisierungs-CLI-Toolkit.

## Schritt 1: networka installieren

Installieren Sie networka mit uv oder pipx:

```bash
uv tool install git+https://github.com/narrowin/networka.git
```

Installation überprüfen:

```bash
nw --version
```

## Schritt 2: Konfiguration initialisieren

Initialisierungsbefehl ausführen:

```bash
nw config init
```

Dies erstellt ein Konfigurationsverzeichnis unter `~/.config/networka` (Linux/macOS):

- `devices/` - Gerätedefinitionen
- `groups/` - Gerätegruppen und Tags  
- `sequences/` - Wiederverwendbare Befehlssequenzen
- `.env` - Anmeldedaten

Folgen Sie den Eingabeaufforderungen für Shell-Vervollständigungen und optionale Herstellersequenzen.

## Schritt 3: Ihre Laborgeräte definieren

Bearbeiten Sie `~/.config/networka/devices/devices.yml`, um die Laborgeräte hinzuzufügen:

```yaml
devices:
  router1:
    host: 192.168.100.11
    device_type: mikrotik_routeros
    platform: x86
    description: "Gateway-Router"
    tags:
      - gateway
      - critical

  switch1:
    host: 192.168.100.12
    device_type: mikrotik_routeros
    platform: x86
    description: "Access-Switch"
    tags:
      - switch
      - access
```

Überprüfen Sie, dass Ihre Geräte geladen sind:

```bash
nw list devices
```

Sie sollten beide Geräte mit ihren IP-Adressen und Beschreibungen sehen.

## Schritt 4: Ersten Befehl ausführen

Schnittstellenstatus auf dem Router mit vordefinierter Sequenz prüfen:

```bash
nw run router1 interface_status
```

Dies verbindet sich mit dem Gerät und zeigt:

- Alle Schnittstellen mit Status und Konfiguration
- Verkehrsstatistiken (Pakete, Bytes, Fehler, Drops)
- Ethernet-Port-Details und -Geschwindigkeiten
- Aktueller Durchsatz für jede Schnittstelle

Die Ausgabe ist formatiert und mit klaren Abschnitten für jedes Befehlsergebnis leicht lesbar.

## Schritt 5: Weitere Sequenzen erkunden

Networka enthält viele eingebaute Sequenzen für häufige Operationen. Verfügbare Sequenzen anzeigen:

```bash
nw run router1 <TAB>
```

Probieren Sie diese nützlichen Sequenzen aus:

```bash
# Schnelle Geräteübersicht
nw run router1 quick_status

# Detaillierte Systeminformationen
nw run router1 system_info

# Routing-Tabelle prüfen
nw run router1 routing_info

# Vollständiger Gesundheitscheck
nw run router1 health_check
```

## Schritt 6: Konfigurationen sichern

Erstellen Sie Konfigurationssicherungen mit herstellerbewusster Behandlung:

```bash
# Konfigurationsexport
nw backup config router1

# Umfassende Sicherung (Konfiguration + Systemdaten)
nw backup comprehensive router1
```

Die Sicherungen werden in Ihrem Ergebnisverzeichnis mit Zeitstempeln gespeichert.

## Schritt 7: Mit mehreren Geräten arbeiten

Befehle gleichzeitig auf mehreren Geräten ausführen:

```bash
# Kommagetrennte Liste
nw run router1,switch1 quick_status

# Befehl über Geräte ausführen
nw run router1,switch1 "/system/resource/print"
```

Für Produktionseinsatz Gerätegruppen erstellen und Befehle effizient über ganze Gruppen ausführen.

## Was als Nächstes?

Sobald Sie mit diesen Grundlagen vertraut sind:

1. **Benutzerdefinierte Sequenzen erstellen** - Definieren Sie Ihre eigenen Befehls-Workflows in YAML
2. **Gerätegruppen einrichten** - Organisieren Sie Geräte nach Rolle, Standort oder Funktion
3. **Sicherungen planen** - Verwenden Sie cron oder systemd-Timer für automatisierte Sicherungen
4. **Multi-Vendor erkunden** - networka unterstützt MikroTik, Cisco, Arista, Juniper und mehr

Lesen Sie die [vollständige Dokumentation](https://narrowin.github.io/networka/) für erweiterte Funktionen wie:

- Benutzerdefinierte Befehlssequenzen
- Tag-basierte Gerätefilterung
- Ergebnisspeicherung und -analyse
- CI/CD-Integration
- Multi-Vendor-Workflows
