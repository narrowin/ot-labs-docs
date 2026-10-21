# Exercise reference

## Wireshark filters quick reference

Paste one expression into the **display-filter** bar and press Enter. These are not capture filters. A blank filtered list does not mean the recording is empty.

| Filter | Use |
| --- | --- |
| `icmp && ip.addr == 10.10.0.11` | `wago-plc2a-vlan10`'s ICMP |
| `icmp.type == 8 && ip.src == 10.10.0.11` | Requests only, for directional LACP checks |
| `vlan` | Frames with an 802.1Q header |
| `vlan.id == 10 && icmp` | Tagged VLAN 10 ICMP |
| `icmp && !vlan` | Untagged ICMP |
| `arp.opcode == 2 && arp.src.proto_ipv4 == 10.10.0.1` | ARP replies claiming the gateway IP |
| `stp` | Spanning-tree BPDUs, including RSTP |
| `stp.version == 2` | RSTP BPDUs |
| `lacp` | LACP negotiation |
| `modbus && ip.addr == 10.10.0.12` | `wago-plc2b-vlan10`'s bounded Modbus read and response |
| `modbus.func_code == 3` | Read Holding Registers messages |
| `http.request && ip.src == 10.10.0.12` | `wago-plc2b-vlan10`'s direct HTTP requests |
| `tcp.port == 2323 && tcp.len > 0` | Cleartext-marker data |
| `tcp.flags.syn == 1` | SYN and SYN/ACK; inspect ACK flag too |
| `tcp.flags.reset == 1` | TCP resets |
| `pn_dcp` | PROFINET DCP discovery |
| `opcua` | Decoded OPC UA messages, if exchanged |

[Search Wireshark's field reference](https://www.wireshark.org/docs/dfref/). Search for the field's exact spelling. Port-only filters locate traffic, but do not prove the application or its security mode.

## Troubleshooting common issues

| Symptom | Next useful check |
| --- | --- |
| Tool link unavailable | Select the assigned **live** network. Ask the instructor to verify lab bindings and permissions. |
| SSH does not connect | Use the lesson's configured **Open terminal** action. If it fails, record the node and error for the instructor. |
| Capture shows no ping | Start capture first, repeat the bounded ping, check node and neighbor-labelled port, then clear the filter. Allow a few seconds for the viewer to catch up. |
| Viewer shows ERF/LEGACY and nonsensical timestamps instead of Ethernet | Choose the page's **Restart Capture**, wait for valid Ethernet/IP decoding, then repeat the probes. If it persists, report the node/interface and screenshot to the instructor. |
| Only one LACP member has your flow | Expected for a stable hash. Repeat the same bounded flow on the other member; separate requests from replies. |
| `vlan` is empty on an access port | Clear it and look for ICMP. Untagged endpoint frames are expected. |
| ARP helper stops with exit 124 | Its safety timer expired. Complete cleanup or repeat the bounded task as directed. |
| Redirects or ping loss during ARP spoofing | Verify the mapping and captured payload. Perfect ping is not a requirement for proving interception. |
| Modbus returns an exception | Record it; verify the exact read-only unit, type, address and count. Do not guess write values. |
| HTTP returns 404 | A responding HTTP service is different from a missing path. Check `/webvisu.htm` and port 8080. |
| A GUI value stays constant | A steady process value is possible. Check ongoing application exchange; do not operate controls just to force movement. |
| A final firewall drop exists, yet access works | Check earlier accepts and the actual packet path. Same-VLAN traffic can bypass the gateway entirely. |
| Results differ from the walkthrough | Record the actual output and compare your lab with the lesson's baseline. Complete earlier cleanup; use the saved recovery checkpoint if the configuration needs restoring. |

## Additional resources

- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/).
- [RouterOS manual](https://manual.mikrotik.com/docs/introduction/).
- [Nmap port-state reference](https://nmap.org/book/man-port-scanning-basics.html).
- [NIST OT security guide](https://csrc.nist.gov/pubs/sp/800/82/r3/final).

Each lesson includes topic-specific references and two optional extensions after its solution. Use [the exercise overview](index.md) to continue in order.
