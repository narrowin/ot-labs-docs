# OT Security Lab Übungen

## Überblick

Diese praxisorientierten Übungen bieten praktische Erfahrung mit OT-Netzwerksicherheitskonzepten. Jede Übung konzentriert sich auf grundlegende Netzwerksicherheitsprinzipien, die häufig in Umgebungen mit industriellen Steuerungssystemen anzutreffen sind.

**Gesamtdauer**: 3-4 Stunden für alle Übungen

**Lab-Anforderungen**: Sie benötigen eine laufende Lab-Umgebung (GitHub Codespaces oder DevPod). Siehe die Seite [Lab-Umgebung](../lab.md) für Installationsanweisungen.

**Übungsansatz**: Die Übungen sind unabhängig und können in beliebiger Reihenfolge durchgeführt werden, wobei die Reihenfolge eine logische Progression der Konzepte bietet.

## Voraussetzungen

### Erforderliches Wissen

- Grundlegende Linux-Kommandozeile
- Netzwerkgrundlagen (IP-Adressierung, VLANs, Routing)
- SSH-Grundlagen
- Wireshark-Grundlagen

### Lab-Zugang

Stellen Sie vor dem Start sicher, dass Sie:

1. Auf Ihre Lab-Umgebung zugreifen können (Codespaces oder DevPod)
2. Eine Lab-Topologie mit der Containerlab-Erweiterung bereitstellen können
3. Per SSH auf Lab-Geräte zugreifen können
4. Datenverkehr mit Wireshark über Edgeshark erfassen können

### Topologie-Auswahl

Diese Übungen verwenden zwei Lab-Topologien:

- `ot-sec-flat.clab.yml` - Flaches Netzwerk mit VLANs, aber minimaler Segmentierung
- `ot-sec-segmented.clab.yml` - Segmentiertes Netzwerk mit Firewall-Regeln

Jede Übung gibt an, welche Topologie verwendet werden soll.

## Übungsliste

| Nr. | Übung | Zeit | Topologie | Schwierigkeit | Link |
|---|----------|------|----------|------------|------|
| 1 | VLAN-Erkennung | 20-30 Min. | Beide | Anfänger | [Start :material-arrow-right:](ex1-vlan-discovery.md) |
| 2 | ARP-Spoofing-Angriff | 30-40 Min. | Beide (flat einfacher) | Fortgeschritten | [Start :material-arrow-right:](ex2-arp-spoofing.md) |
| 3 | Spanning-Tree-Analyse | 25-35 Min. | Beide | Fortgeschritten | [Start :material-arrow-right:](ex3-spanning-tree.md) |
| 4 | Link-Aggregation | 20-30 Min. | Nur segmentiert | Fortgeschritten | [Start :material-arrow-right:](ex4-link-aggregation.md) |
| 5 | Netzwerk-Reconnaissance | 35-45 Min. | Beide | Fortgeschritten | [Start :material-arrow-right:](ex5-reconnaissance.md) |
| 6 | OT-Protokoll-Erkennung | 30-35 Min. | Beide (flat einfacher) | Fortgeschritten | [Start :material-arrow-right:](ex6-protocol-discovery.md) |
| 7 | Segmentierung & Firewall-Design | 35-45 Min. | Segmentiert (primär) | Fortgeschritten/Experte | [Start :material-arrow-right:](ex7-segmentation-firewall.md) |

## Zusätzliche Ressourcen

- [Anhang: Wireshark-Filter](appendix.md#wireshark-filter-kurzreferenz)
- [Troubleshooting-Leitfaden](appendix.md#troubleshooting-häufige-probleme)
- [Externe Ressourcen](appendix.md#zusätzliche-ressourcen)
