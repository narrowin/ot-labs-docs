# Student lab exercises

Work through these seven lessons in order. Exercises 1–6 use your assigned flat `ot-sec-flat` appliance; Exercise 7 uses a separate copy reserved for your practical group. Allow about 4–5¼ hours for the main tasks. Each lesson has a collapsed solution followed by **two optional extra sections**; allow another 2–2½ hours if you do all extras.

Times are guides. Complete the core checks at your own pace; use the hints when needed and save the extras for later if you prefer. Keep your commands, filters and results so you can resume in your lab after the course. Finish each lesson's cleanup before taking a break; repeat its baseline checks when returning.

## Start here

1. Open your assigned Network Explorer lab and select its **live** network.
2. Choose **Exercises** in the product launcher, then Exercise 1. Use **English / Deutsch** for your preferred language.
3. Use each lesson's links to open the configured terminal, filtered Netmap, target details or selected capture port. In terminal and capture tabs, the active exercise title reopens the instructions. Keep a terminal open while switching to a capture.
4. Attempt each observation before revealing its hint or solution. Record the source node, interface, command/filter and actual evidence.

You need a running instructor-provided lab and its configured node credentials. Do not deploy another topology, install extensions or enter old default passwords. If a link says **unavailable**, check the selected live network and ask the instructor to verify access/bindings.

Tool links are designed for the Network Explorer reader. In this published documentation, open the named tool in your assigned lab and select the named node/interface manually. Do not try to reach the simulated data IPs directly from your laptop.

## Lessons

| # | Lesson | Main time | Extra challenges |
| --- | --- | --- | --- |
| 1 | [Compare tagged and untagged traffic](ex1-vlan-discovery.md) | 20–30 min | Four-byte frame puzzle; next-hop MAC |
| 2 | [Trace an ARP interception](ex2-arp-spoofing.md) | 30–40 min | Incident timeline; direct versus routed interception |
| 3 | [Explain the live spanning tree](ex3-spanning-tree.md) | 25–35 min | Root-election detective; healthy non-forwarding link |
| 4 | [Follow flows through LACP](ex4-link-aggregation.md) | 25–35 min | Packet size versus hash; actor/partner audit |
| 5 | [Assess access from the DMZ](ex5-reconnaissance.md) | 30–40 min | Compare vantage points; prove TCP port states |
| 6 | [Identify OT protocols](ex6-protocol-discovery.md) | 35–45 min | Register numbering; discovery without TCP ports |
| 7 | [Build and verify a routed VLAN](ex7-segmentation-firewall.md) | 60–90 min | Follow a routed request; replies versus fresh connections |

## Lab facts that prevent wrong conclusions

- The OT baseline uses **VLAN 10 and `10.0.0.0/8`**. Historical suffixes such as `vlan20` do not prove segmentation.
- The **DMZ is VLAN 60, `2.2.2.0/24`**. Addresses such as `1.1.1.1` and `2.2.2.2` are simulated inside the lab.
- sw-dist and sw-acc1 have an **active two-member LACP bond**. A stable flow normally uses one member.
- Operational state, packets and successful application responses are different forms of evidence. A port label, diagram or hostname is not a substitute.
- Toolbox's LNE design is shared and does not apply device configuration. Exercise 7 builds VLAN 70 on an **assigned disposable copy**, then restores the flat baseline. It changes one endpoint, two switches and the gateway; use the [practical setup](../lab-practical.md) before starting.

## Finish each lesson

Close your capture tabs and terminal tabs. In Exercise 2 also stop the helper/listener and verify the original gateway MAC and forwarding value before moving on. Exercise 7 requires its cleanup even if you skip the extras. Leave the shared appliance and Toolbox policies unchanged.

The existing assistant knows the selected lesson. Give it the task number, actual command/output and capture filter/frame number. Wait for an answer before asking from another tab. Compare its advice with your recorded evidence and the local lesson.

See the [filter and troubleshooting reference](appendix.md).
