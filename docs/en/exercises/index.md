# Student lab exercises

Work through these seven lessons in your own student `ot-sec-flat` lab. Exercises 1–6 start from its flat baseline; Exercise 7 builds a routed VLAN and then restores that baseline. Allow about 5–6½ hours for the main tasks, discussion and handovers. Each lesson has a collapsed solution followed by **two optional extra sections**; allow another 2–2½ hours if you do all extras.

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
| 1 | [Compare tagged and untagged traffic](ex1-vlan-discovery.md) | 30–40 min | Replacement PLC checklist; next-hop MAC |
| 2 | [Trace an ARP interception](ex2-arp-spoofing.md) | 40–50 min | Incident timeline; maintainable ARP protection |
| 3 | [Explain the live spanning tree](ex3-spanning-tree.md) | 30–40 min | Switch replacement; healthy standby link |
| 4 | [Follow flows through LACP](ex4-link-aggregation.md) | 30–40 min | Capture blind spot; shared failure points |
| 5 | [Assess access from the DMZ](ex5-reconnaissance.md) | 35–45 min | Compare vantage points; prove TCP port states |
| 6 | [Identify OT protocols](ex6-protocol-discovery.md) | 40–50 min | HMI register mapping; remote discovery limits |
| 7 | [Build and verify a routed VLAN](ex7-segmentation-firewall.md) | 90–120 min | Follow a routed request; replies versus fresh connections |

## Connect the work to the course

The course plant is a water-treatment example. ABB provides supervision; WAGO controls treatment functions; CtrlX controls pumps; Schneider also spans process and disposal functions. Treat these as the handout's teaching roles. Verify a network flow before calling it a proven control dependency.

| Course handout topic | Lab result |
| --- | --- |
| Ethernet, VLANs and boundaries (PDF pp. 38–52) | Exercise 1: explain the PLC's actual connection |
| ARP attacks and mitigation (pp. 61–64) | Exercise 2: distinguish evidence, protection and recovery |
| Loop prevention and availability (pp. 87–97) | Exercise 3: justify a healthy standby path |
| Link aggregation and monitoring coverage (pp. 85–86, 109, 154–156) | Exercise 4: explain missing capture traffic and shared failures |
| Remote access (pp. 100–104) | Exercise 5: bound a supplier's required access |
| Protocol layers and local discovery (pp. 36–41, 50) | Exercise 6: separate network, service and process evidence |
| Plant zones, controller dependencies and logs (pp. 128–148, 162–164) | Exercise 7: justify a zone design, enforce one conduit and prove cleanup |

Page numbers count PDF pages in **OT Network Security – Design and Defend handout**; printed slide numbers later differ. Keep the numbered lab order for independent study. In the taught course, the instructor can introduce the LACP concept before RSTP, then use Exercises 3–4 together once both terms are known.

The practical does not configure MRP, VPN/MFA, 802.1X or industrial DPI, and it does not qualify real-time performance. Those handout topics remain discussion/design work unless the instructor provides a separate qualified environment.

## Use the tools without prior network CLI experience

- A **terminal** runs commands on the named lab device. Confirm that name before entering a block. Linux commands such as `ip` and RouterOS commands beginning `/interface` belong in different terminals.
- Enter commands one line at a time and read the result. Do not type the Markdown fences (the lines with three backticks). Stop at an unexpected error before applying later configuration lines.
- A Wireshark **display filter** goes in the filter bar, not the terminal. It narrows the visible packets; it does not create traffic or change the network.
- For each handover, keep **prediction → observation → explanation → next action** separate. Record the device, interface, time and frame/output. “Not observed” is a valid result with a stated observation window.

## Lab facts that prevent wrong conclusions

- The OT baseline uses **VLAN 10 and `10.0.0.0/8`**. Historical suffixes such as `vlan20` do not prove segmentation.
- The **DMZ is VLAN 60, `2.2.2.0/24`**. Addresses such as `1.1.1.1` and `2.2.2.2` are simulated inside the lab.
- sw-dist and sw-acc1 have an **active two-member LACP bond**. A stable flow normally uses one member.
- Operational state, packets and successful application responses are different forms of evidence. A port label, diagram or hostname is not a substitute.
- Toolbox saves your LNE design and does not apply device configuration. Exercise 7 builds VLAN 70 in **your own lab**, then restores the flat baseline. It changes one endpoint, two switches and the gateway; the [practical setup](../lab-practical.md) prepares its recovery checkpoint.

## Finish each lesson

Close your capture tabs and terminal tabs. In Exercise 2 also stop the helper/listener and verify the original gateway MAC and forwarding value before moving on. Exercise 7 requires its cleanup even if you skip the extras.

The existing assistant knows the selected lesson. Give it the task number, actual command/output and capture filter/frame number. Wait for an answer before asking from another tab. Compare its advice with your recorded evidence and the local lesson.

See the [filter and troubleshooting reference](appendix.md).
