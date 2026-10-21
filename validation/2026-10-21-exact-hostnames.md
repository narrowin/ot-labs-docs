# Exact hostnames and a concrete connection record

21 October 2026. Follow-up to the student's naming complaint in Exercise 1. The preceding clarity review missed device abbreviations spread across the course.

## Decision

**Reversibility:** Two-way door. Documentation changes and a versioned course mount; no device, interface, VLAN, firewall or topology changes.

**Problem:** Names such as “WAGO A”, “WAGO B”, lone “A/B”, “CtrlX”, “Hirschmann” and “Dell” require students to translate a teaching alias into a real terminal target. Exercise 1's extra also failed to specify a concrete output.

**Desired outcome:** A student can identify the exact host for each action and knows which observations to record to finish the connection extra.

**Options:**

- Replace device aliases throughout both languages and make the existing connection extra explicit. Impact 9: fixes target ambiguity without changing the practical mechanics. Confidence 9: source and target URLs were inspected, and command blocks are compared byte-for-byte with the prior committed course. Ease 8: contained prose changes and existing release publication. **ICE 648.**
- Add one alias glossary while retaining shorthand in the steps. Impact 4: students still need to translate names while switching between tools. Confidence 9: the existing role tables already failed to prevent this confusion. Ease 10: a small table. **ICE 360.**

**Recommendation implemented:** Use exact hostnames in the instructions, links, evidence tables and solutions.

## Changes by exercise

| Exercise | Changed | Why |
| --- | --- | --- |
| 1 | Replaced the vague maintenance quotation with a connection record for `wago-plc2a-vlan10`. Added seven named fields, the source for each, a completion criterion and a collapsed expected record. Named `sw-acc1`, `ether2` and PLC interface `eth1` explicitly. | Students know which PLC and switch to inspect, what to write down and how to check the answer. The settings remain unchanged. |
| 2 | Named `wago-plc2a-vlan10`, `wago-plc2b-vlan10`, `proxmox-jumphost` and `gw-firewall` in roles, command headings, ARP evidence and cleanup. Removed lone A/B references. | Attack execution, victim evidence and receiver actions must not depend on remembering letter aliases. |
| 3 | Named `gw-firewall` as the root device and the exact PLC hostnames in the optional recovery trial. | The student can find the root terminal and both application endpoints without translating labels. |
| 4 | Named `wago-plc2a-vlan10`, `ctrlx-plc3-vlan20` and `wago-plc2b-vlan10` in source selection, flow results and capture checks. | Source selection determines whether a flow traverses the tested bond. |
| 5 | Named `proxmox-jumphost` as the remote source and the exact PLC hostnames in target lists and the two-vantage-point extra. | The route and exposure result depend on the source and target. |
| 6 | Named the exact client, server, supervisory host, capture target and WebVisu target throughout. | A client read, a server listener and a browser-proxied visualization are different observations and must identify their hosts. |
| 7 | Named `hirschmann-edge1-vlan50`, `wago-plc2a-vlan10`, `gw-firewall`, `dell-ews1-vlan50` and `protected-linux` in actions and communication tables. Kept PLC2A and PLC2B separate alongside their full hostnames. | Address moves, policy changes, allow/deny results and ARP-cache claims must identify the actual endpoints. |

The lab guide, practical preparation, cheatsheet, exercise appendix, overview, Container101 and automation page use the same hostnames. The five-entry PLC mapping and supplied process image are retained.

`gw` remains only where the actual Netmap/neighbor label must be located; Exercise 1 explicitly maps it to `gw-firewall`. The real port description `wago-plc2a` and the actual HTTP page title **Diode protected Linux endpoint** are retained as observed values, rather than relabelling the GUI or service.

## Exercise 1 extra: student output

The connection record contains:

1. PLC hostname and data interface.
2. Data IP address and prefix.
3. Switch hostname and physical port.
4. Port PVID.
5. Untagged VLAN membership.
6. Tagged/untagged frames from the access-port observation.
7. Route to the gateway.

The expected baseline identifies `wago-plc2a-vlan10`, `eth1`, `10.10.0.11/8`, `sw-acc1`, `ether2`, PVID/VLAN 10, untagged endpoint frames and direct gateway reachability through eth1. The student reads configuration and packet evidence; no replacement, reconnection or configuration change is performed.

## Verification and publication

All existing command blocks and existing link destinations are unchanged from `cc0557c`; the added terminal link selects the already used `wago-plc2a-vlan10`. English/German command blocks still match. Every lesson retains two extras, all lesson files fit the reader limit, and both strict MkDocs builds pass. The installed theme's existing deprecation notice remains unrelated to these content edits.

Publication details and live reader checks will be recorded after activation. No networking practical is rerun for this prose-only correction.
