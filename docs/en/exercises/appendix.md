# Appendix: Reference Materials

## Wireshark Filters Quick Reference

### VLAN Analysis

```wireshark
vlan                          # All 802.1Q tagged frames
vlan.id == 10                 # Only VLAN 10 traffic
vlan.id in {10 20 30}        # Multiple VLANs
```

### ARP Analysis

```wireshark
arp                           # All ARP traffic
arp.opcode == 1               # ARP requests only
arp.opcode == 2               # ARP replies only
arp.duplicate-address-detected  # ARP conflicts
```

### STP/RSTP

```wireshark
stp                           # All spanning tree
rstp                          # Rapid spanning tree only
eth.dst == 01:80:c2:00:00:00  # STP multicast address
```

### Protocol-Specific

```wireshark
icmp                          # ICMP (ping) traffic
tcp.port == 502               # Modbus TCP
tcp.port == 2455              # CODESYS
tcp.port == 8080              # Web interfaces
```

### Combining Filters

```wireshark
vlan.id == 10 && icmp         # VLAN 10 ICMP traffic
tcp && !tcp.port == 22        # All TCP except SSH
ip.src == 10.10.0.11 || ip.dst == 10.10.0.11  # Specific host
```

---

## Troubleshooting Common Issues

### Lab Not Starting

**Symptom**: Containers fail to start or remain unhealthy

**Solutions**:

1. Check container status: `docker ps -a | grep clab`
2. View container logs: `docker logs <container-name>`
3. Verify images pulled: `docker images | grep narrowin`
4. Run health check: `scripts/lab-test.py ot-sec-flat`

### Cannot SSH to Devices

**Symptom**: Connection refused or timeout

**Solutions**:

1. Verify using management IP (192.168.100.x), not data plane IP
2. Wait 2-3 minutes after lab deploy for SSH to start
3. Check container is running: `docker ps | grep <device-name>`
4. Test from VS Code terminal, not external system

### Wireshark Not Opening

**Symptom**: Edgeshark does not launch

**Solutions**:

1. Accept VS Code prompt to open with Edgeshark
2. Close and retry capture
3. Check Edgeshark extension installed
4. View container logs for errors

### No Traffic in Captures

**Symptom**: Wireshark shows no packets

**Solutions**:

1. Verify capture started before generating traffic
2. Check capturing on correct interface/link
3. Remove display filters temporarily
4. Verify devices can actually communicate (ping test)

### Ping Failures

**Symptom**: Destination unreachable

**Solutions**:

1. Check routing: `ip route show`
2. Verify VLAN assignment
3. In segmented topology, check if firewall blocks traffic
4. Confirm both devices in same VLAN or proper routing configured

---

## Additional Resources

### Documentation

- [Containerlab Documentation](https://containerlab.dev/) - Network lab automation
- [MikroTik RouterOS Manual](https://help.mikrotik.com/) - Switch configuration reference
- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/) - Packet analysis
- [CODESYS Documentation](https://www.codesys.com/) - PLC programming

### Standards and Best Practices

- [IEC 62443 Standards](https://www.isa.org/standards-and-publications/isa-standards/isa-iec-62443-series-of-standards) - Industrial cybersecurity framework
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework) - Security best practices

### Tools

- [nmap](https://nmap.org/) - Network discovery and security auditing
- [tcpdump](https://www.tcpdump.org/) - Command-line packet analyzer
- [Edgeshark](https://github.com/siemens/edgeshark) - Container network traffic capture

---

## Advanced Topics and Extensions

These topics are mentioned in the original workshop objectives but are considered advanced or require additional infrastructure not currently present in the standard lab environment.

### IDS/IPS Deployment

- **Zeek**: Deploying Zeek for deep protocol analysis and logging.
- **Suricata**: Setting up Suricata IDS with custom rules for OT protocols.

### Advanced Monitoring

- **ELK Stack**: Integrating Elasticsearch, Logstash, and Kibana for centralized log management.
- **SIEM**: Concepts of integrating OT logs into a Security Information and Event Management system.

### Operational Security

- **Detection Rules**: Configuring specific detection rules for anomalies (e.g., "New device detected", "PLC stop command").
- **Alerting**: Setting up email or webhook alerts for security events.
- **Playbooks**: Creating response playbooks for detected incidents.
- **Tuning**: Methodologies for reducing false positives in detection rules.
- **Firewall Management**: Tools and processes for managing complex firewall rulesets over time.

---

**Document Version**: 1.0  
**Last Updated**: 2025-11-19  
**Lab Compatibility**: ot-sec-flat v1.x, ot-sec-segmented v1.x
