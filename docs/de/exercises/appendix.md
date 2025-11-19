# Anhang: Referenzmaterialien

## Wireshark-Filter-Kurzreferenz

### VLAN-Analyse

```wireshark
vlan                          # Alle 802.1Q-getaggten Frames
vlan.id == 10                 # Nur VLAN 10-Verkehr
vlan.id in {10 20 30}        # Mehrere VLANs
```

### ARP-Analyse

```wireshark
arp                           # Gesamter ARP-Verkehr
arp.opcode == 1               # Nur ARP-Requests
arp.opcode == 2               # Nur ARP-Replies
arp.duplicate-address-detected  # ARP-Konflikte
```

### STP/RSTP

```wireshark
stp                           # Gesamter Spanning-Tree-Verkehr
rstp                          # Nur Rapid Spanning Tree
eth.dst == 01:80:c2:00:00:00  # STP-Multicast-Adresse
```

### Protokoll-spezifisch

```wireshark
icmp                          # ICMP (Ping) Verkehr
tcp.port == 502               # Modbus TCP
tcp.port == 2455              # CODESYS
tcp.port == 8080              # Web-Interfaces
```

### Filter kombinieren

```wireshark
vlan.id == 10 && icmp         # VLAN 10 ICMP-Verkehr
tcp && !tcp.port == 22        # Gesamter TCP ausser SSH
ip.src == 10.10.0.11 || ip.dst == 10.10.0.11  # Spezifischer Host
```

---

## Troubleshooting: Häufige Probleme

### Lab startet nicht

**Symptom**: Container starten nicht oder bleiben unhealthy

**Lösungen**:

1. Container-Status prüfen: `docker ps -a | grep clab`
2. Container-Logs anzeigen: `docker logs <container-name>`
3. Images überprüfen: `docker images | grep narrowin`
4. Health-Check ausführen: `scripts/lab-test.py ot-sec-flat`

### SSH zu Geräten nicht möglich

**Symptom**: Connection refused oder Timeout

**Lösungen**:

1. Überprüfen, dass Management-IP (192.168.100.x) verwendet wird, nicht Datenebenen-IP
2. 2-3 Minuten nach Lab-Bereitstellung warten, bis SSH startet
3. Überprüfen, dass Container läuft: `docker ps | grep <device-name>`
4. Von VS Code-Terminal testen, nicht von externem System

### Wireshark öffnet sich nicht

**Symptom**: Edgeshark startet nicht

**Lösungen**:

1. VS Code-Eingabeaufforderung zum Öffnen mit Edgeshark akzeptieren
2. Schliessen und Capture erneut versuchen
3. Überprüfen, dass Edgeshark-Erweiterung installiert ist
4. Container-Logs auf Fehler prüfen

### Kein Verkehr in Captures

**Symptom**: Wireshark zeigt keine Pakete

**Lösungen**:

1. Überprüfen, dass Capture vor Verkehrsgenerierung gestartet wurde
2. Überprüfen, dass auf korrektem Interface/Link erfasst wird
3. Display-Filter vorübergehend entfernen
4. Überprüfen, dass Geräte tatsächlich kommunizieren können (Ping-Test)

### Ping-Fehler

**Symptom**: Destination unreachable

**Lösungen**:

1. Routing prüfen: `ip route show`
2. VLAN-Zuweisung überprüfen
3. In segmentierter Topologie prüfen, ob Firewall Verkehr blockiert
4. Bestätigen, dass beide Geräte im selben VLAN oder mit korrektem Routing konfiguriert sind

---

## Zusätzliche Ressourcen

### Dokumentation

- [Containerlab-Dokumentation](https://containerlab.dev/) - Netzwerk-Lab-Automatisierung
- [MikroTik RouterOS Manual](https://help.mikrotik.com/) - Switch-Konfigurationsreferenz
- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/) - Paketanalyse
- [CODESYS-Dokumentation](https://www.codesys.com/) - SPS-Programmierung

### Standards und Best Practices

- [IEC 62443 Standards](https://www.isa.org/standards-and-publications/isa-standards/isa-iec-62443-series-of-standards) - Industrielles Cybersicherheits-Framework
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework) - Sicherheits-Best-Practices

### Tools

- [nmap](https://nmap.org/) - Netzwerkerkennung und Sicherheitsprüfung
- [tcpdump](https://www.tcpdump.org/) - Kommandozeilen-Paketanalyse
- [Edgeshark](https://github.com/siemens/edgeshark) - Container-Netzwerkverkehrserfassung

---

## Erweiterte Themen und Erweiterungen

Diese Themen werden in den ursprünglichen Workshop-Zielen erwähnt, gelten aber als erweitert oder erfordern zusätzliche Infrastruktur, die derzeit nicht in der Standard-Lab-Umgebung vorhanden ist.

### IDS/IPS-Bereitstellung

- **Zeek**: Zeek für tiefe Protokollanalyse und Logging bereitstellen.
- **Suricata**: Suricata IDS mit benutzerdefinierten Regeln für OT-Protokolle einrichten.

### Erweiterte Überwachung

- **ELK Stack**: Elasticsearch, Logstash und Kibana für zentrales Log-Management integrieren.
- **SIEM**: Konzepte zur Integration von OT-Logs in ein Security Information and Event Management System.

### Betriebssicherheit

- **Erkennungsregeln**: Spezifische Erkennungsregeln für Anomalien konfigurieren (z.B. "Neues Gerät erkannt", "SPS-Stopp-Befehl").
- **Alarmierung**: E-Mail- oder Webhook-Alarme für Sicherheitsereignisse einrichten.
- **Playbooks**: Response-Playbooks für erkannte Vorfälle erstellen.
- **Tuning**: Methodologien zur Reduzierung von False Positives in Erkennungsregeln.
- **Firewall-Management**: Tools und Prozesse zur Verwaltung komplexer Firewall-Regelwerke im Laufe der Zeit.

---

**Dokumentversion**: 1.0  
**Letztes Update**: 2025-11-19  
**Lab-Kompatibilität**: ot-sec-flat v1.x, ot-sec-segmented v1.x
