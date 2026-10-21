# OT student exercises — changes and joint review

Review draft, 21 October 2026. The intended students are OT professionals who understand their process but have limited network experience. This records the changes already made locally, why they were made, what was kept, and the tradeoffs to review together. These choices are proposals for the course owner, not proof that the exercises are pedagogically optimal.

**Current status:** This snapshot includes revisions to all seven lessons and their fourteen optional extras in English and German, along with the overview, agenda, lab guidance and automation outlook. Publication to the deployed student reader is pending. Joint review has reached Exercise 2. The course owner confirmed that every student has their own lab and requested removal of the draft's shared-lab assumptions across the course. That correction is applied; remaining exercise choices are still open for joint review.

## What “read-only validation” meant

It meant checking configuration, packets and application responses without changing switch or firewall configuration. It did not mean converting the course into read-only exercises. Exercises 1 and 3–6 already primarily used observation and bounded diagnostic traffic. Exercise 2 still runs a timed ARP interception. Exercise 7 still configures the VLAN, routing and firewall in the student's own lab and then restores the baseline.

A ping, TCP connection scan, Modbus read or HTTP request still generates traffic. Calling those checks “read-only” does not make them passive captures or suitable for every production device.

## Basis and approach

The primary teaching reference is [OT Network Security — Design and Defend handout](</Users/md/Downloads/OT Network Security - Design and Defend handout (1).pdf>). References below count PDF pages, because the printed slide numbers diverge later in the deck. The water-treatment process and allocation of functions to controllers informed Exercise 7; the actual lab topology and packets determine what can be demonstrated.

The design priorities are operational purpose, a small number of new terms, evidence the student can obtain, a useful decision, and recovery after changes. OT availability and safety constraints are grounded in [NIST SP 800-82 Rev. 3](https://csrc.nist.gov/pubs/sp/800/82/r3/final); the broader focus on operational context, people and segregation follows the [joint principles of OT cyber security](https://www.cyber.gov.au/publication/principles-of-operational-technology-cyber-security). These sources support the selection criteria, not a claim that a particular teaching scenario has a measured prevalence or learning benefit.

**Reversibility:** These local documentation edits are a two-way door. Deployment and changes to actual student lab state are separate actions.

**Problem:** Several extras rewarded packet trivia or tool execution without a useful OT decision.

**Desired outcome:** Students can connect network evidence to commissioning, maintenance, availability and access decisions, using the available lab.

**Options:**

- Keep the verified practical mechanics and replace weak extras/context. Impact 9: improves relevance across the existing course. Confidence 8: the commands, topology and handout have been inspected and substantial lab evidence exists, though no student pilot exists. Ease 8: contained bilingual documentation edits. **ICE 576.**
- Rebuild the course around new fault scenarios. Impact 9: could provide stronger troubleshooting practice. Confidence 4: new faults, recovery and teaching timings have not been qualified. Ease 3: would require additional lab implementation and validation. **ICE 108.**

**Recommendation used for this draft:** The first option. Its limitation is that some scenarios now have better context without introducing an actual fault. That limitation is explicit below and should influence our joint review.

## Exercise 1 — VLANs and a replacement PLC connection

[English lesson](../docs/en/exercises/ex1-vlan-discovery.md) · [German lesson](../docs/de/exercises/ex1-vlan-discovery.md). Handout: PDF pp. 38–52.

**Kept:** Locate WAGO and its network path, ping the gateway, capture the gateway uplink and the PLC access port, and compare tagged with untagged frames. No switch changes were added.

**Changed:**

- Added a replacement-PLC commissioning situation: link is up, but the team must establish the actual network connection. Defined VLAN, access port and trunk before the capture work.
- Added a prediction about whether the PLC needs to send tags, and a handover identifying the address, ports and packet evidence. Ping is explicitly insufficient to declare the PLC application operational.
- Replaced Extra 1, **“Solve the four-byte puzzle,”** with **“Check a replacement PLC connection.”** Students read sw-acc1 ether2's PVID and VLAN membership, then WAGO's address/prefix and route. Their output is a reconnection checklist.
- Kept Extra 2, **“Follow the next-hop MAC.”** Different destination IPs can use the same gateway MAC. This prepares students for ARP and routing.
- Increased the estimate from 20–30 to 30–40 minutes.

**Why:** The four-byte difference is a valid fact, but calculating it twice contributes less to an OT maintenance decision than knowing the correct switch port, tagging expectation and endpoint address. The replacement checklist reuses the same capture and connects it to a task an OT technician can recognize.

**Validation:** Actual switch output showed ether2 PVID 10 and untagged membership; WAGO was `10.10.0.11/8`. Gateway reachability and the direct versus routed paths were checked. Tagged/untagged capture behavior also has the previous executed walkthrough's evidence.

**Tradeoff to review:** This checks a healthy connection; no broken replacement PLC was introduced. The current main task already uses the replacement scenario and Extra 1 extends it, so there is some repetition. Extra 1 also introduces RouterOS output early. We should decide whether the checklist belongs in the core exercise, remains an extra, or is too much for the first lesson. The broader `/8` is an existing lab constraint, not a recommended plant addressing scheme.

**Joint decision:** Pending.

## Exercise 2 — ARP interception and incident handling

[English](../docs/en/exercises/ex2-arp-spoofing.md) · [German](../docs/de/exercises/ex2-arp-spoofing.md). Handout: PDF pp. 61–64.

**Kept:** The actual timed interception, victim neighbor-table comparison, receiver, cleartext marker, capture analysis and cleanup. The nine remaining fenced command blocks are unchanged from the previous version.

**Changed:**

- Framed the exercise as investigating a changed gateway MAC; introduced ARP before asking students to interpret it.
- Explained that `LAB-ARP-DEMO` is a synthetic diagnostic marker, not an industrial protocol or real credential.
- Removed the draft's group reservation and instructor-demonstration requirement after the course owner confirmed that each student has their own lab. Students perform the full interception in their own lab.
- Added an incident handover: original mapping, changed mapping and payload evidence, recovery, and verification of an approved replacement or failover.
- Kept Extra 1, the incident timeline.
- Replaced Extra 2, **“Find the boundary of this interception,”** which repeated the attack against direct/routed destinations, with **“Choose protection that survives maintenance.”** Students compare MAC port restrictions, fixed ARP entries, ARP inspection bindings and monitoring in a static-addressed cell undergoing gateway replacement.
- Increased 30–40 to 40–50 minutes.

**Why:** One full interception is enough to establish the mechanism. The new extra makes students connect prevention to maintenance: MAC port security alone does not validate an IP-to-MAC claim, while static bindings need an update procedure. The binding requirement is supported by [Cisco's DAI documentation](https://www.cisco.com/c/en/us/support/docs/switches/lan-switch-software/222274-troubleshoot-dynamic-arp-inspection-dai.html).

**Validation:** The previous complete isolated walkthrough proved changed mapping, received/captured marker and recovery. This review checked the baseline and helper implementation. A fresh active run was started on the isolated copy but stopped when we switched to joint review; it did not complete a new marker/payload validation. It is not counted as a fresh full pass.

**Tradeoff to review:** The new protection extra is discussion, not configured protection. Removing the old extra loses a hands-on demonstration that poisoning the gateway mapping does not intercept direct neighbors. The synthetic marker remains deliberately artificial; its purpose is unambiguous packet proof. We should decide whether that is the right balance or whether the direct/routed comparison should stay available.

**Joint decision:** Pending.

## Exercise 3 — RSTP and a healthy standby cable

[English](../docs/en/exercises/ex3-spanning-tree.md) · [German](../docs/de/exercises/ex3-spanning-tree.md). Handout: PDF pp. 87–97.

**Kept:** Inspect operational root and port roles, decode BPDUs, and explain the relationship between RSTP and the LACP bond. The main procedure remains observational.

**Changed:**

- Added the maintenance question: a cable is physically connected but not forwarding; is this a fault or intended loop prevention?
- Defined root bridge, root port, alternate port and BPDU. Clarified that root bridge and IP gateway are different concepts even though this lab elects the gateway.
- Replaced Extra 1's root-election ranking puzzle with a switch-replacement preflight: equal priority can make a replacement MAC change the elected root and paths. Students specify intended root and pre/post-change checks.
- Kept Extra 2, diagnosing the healthy non-forwarding link.
- Added an evidence-backed maintenance handover; increased 25–35 to 30–40 minutes.

**Why:** The election rules matter when replacing equipment or investigating a topology change. The revised task connects those rules to maintenance instead of asking students to rank IDs for its own sake. [MikroTik's RSTP election documentation](https://help.mikrotik.com/docs/spaces/ROS/pages/21725254/Spanning+Tree+Protocol) supports the priority/MAC explanation.

**Validation:** Operational output showed the agreed root ending `09:11`; sw-acc2 ether11 was physically up, alternate and non-forwarding, with BPDUs present. Passive capture agreed with the root/cost information.

**Tradeoff to review:** Students still do not disconnect a link or measure process interruption and convergence. Both the main scenario and Extra 2 address a healthy standby port; their overlap may justify consolidating them. No failover time is claimed from timer settings.

**Joint decision:** Pending.

## Exercise 4 — LACP monitoring coverage and failure scope

[English](../docs/en/exercises/ex4-link-aggregation.md) · [German](../docs/de/exercises/ex4-link-aggregation.md). Handout: PDF pp. 85–86, 109, 154–156.

**Kept:** Inspect both bond members, reciprocal LACP state and flow distribution. Keep all links enabled during this exercise.

**Changed:**

- Introduced a monitoring problem: capturing one of two healthy uplinks can miss traffic. Defined bond, LACP, flow and hash policy.
- Replaced Extra 1's packet-size/hash experiment with **“Find the capture blind spot.”** Reuse both sources' gateway pings and compare a member capture with the gateway uplink.
- Replaced Extra 2's detailed actor/partner key audit with **“Decide what the redundancy actually covers.”** Evaluate a work order to power down sw-dist because it has two uplink cables. Compare one member lost, both members lost and the distribution switch lost.
- Added a monitoring handover; increased 25–35 to 30–40 minutes.

**Why:** The student must distinguish absent traffic from incomplete observation, and cable redundancy from switch/power/gateway redundancy. These decisions are more useful at this level than repeating packet-size arithmetic or auditing LACP key fields. The core still includes negotiation evidence. [MikroTik's bonding reference](https://help.mikrotik.com/docs/spaces/ROS/pages/8323193/Bonding) provides the underlying hash and link-monitoring behavior.

**Validation:** Both members were active with reciprocal system IDs; packet flags showed collecting/distributing. The new gateway-uplink observation saw all four requests from each of the two sources, tagged VLAN 10.

**Tradeoff to review:** The gateway uplink covers those gateway-bound flows, not all local PLC traffic. Extra 2 is a prediction exercise; no cable or switch outage was executed. A practical outage could teach more, but requires a separately qualified procedure and process-level acceptance criterion. Removing the key audit reduces deeper miswiring diagnosis practice.

**Joint decision:** Pending.

## Exercise 5 — Supplier access from the DMZ

[English](../docs/en/exercises/ex5-reconnaissance.md) · [German](../docs/de/exercises/ex5-reconnaissance.md). Handout: PDF pp. 100–104.

**Kept:** All main diagnostic commands and both extras: compare two vantage points, then prove open/closed port states with packets.

**Changed:**

- Gave the checks a specific reason: a supplier needs WAGO WebVisu for diagnostics; determine what else the existing jump server can reach.
- Defined DMZ, jump server and TCP port; separated network reachability from permission to authenticate or operate a process.
- Added an access decision comparing TCP 8080 with unnecessary SSH, Modbus and OPC UA reachability, with a responsible owner and expiry.
- Added the distinction between simulator-scoped scans and production OT discovery planning. Increased 30–40 to 35–45 minutes.

**Why:** The original evidence-gathering mechanics were useful. A concrete access request makes students justify which connections the job requires instead of presenting a list of open ports as a security finding.

**Validation:** The jumphost route, four named target responses, WAGO ports 22/502/4840/8080, HTTP 200 and closed port 65000 were checked. The earlier capture walkthrough established SYN/ACK versus RST/ACK evidence for the retained packet extra.

**Tradeoff to review:** This assesses the internal access available from a jump server. It does not establish an external vendor VPN, test MFA or verify application permissions. The situation is hypothetical; no supplier identity is provisioned. The scan is not a vulnerability assessment. This is mainly a contextual improvement, not a new practical.

**Joint decision:** Pending.

## Exercise 6 — Application evidence and commissioning

[English](../docs/en/exercises/ex6-protocol-discovery.md) · [German](../docs/de/exercises/ex6-protocol-discovery.md). Handout: PDF pp. 36–41, 50.

**Kept:** Passive protocol inspection, one bounded Modbus read, HTTP/WebVisu observation, local listeners and both extras' technical checks. Students do not change process controls or PLC registers.

**Changed:**

- Added a commissioning question: the PLC answers ping, but does the required application exchange work? Defined protocol, register and transaction ID.
- Asked students to identify one actual background client/server exchange before generating their own read. The lab's ABB → WAGO traffic is distinguished from the student's WAGO B → A diagnostic read.
- Moved the handcrafted Python OPC UA greeting out of the required path into a collapsed optional student task. Added a guard for an incomplete reply header. A listener with unverified session security is now an acceptable core finding. The optional task does not require an instructor demonstration.
- Reframed Extra 1 as replacement-HMI register numbering: display labels 0 and 1 can refer to the same wire address.
- Reframed Extra 2 as remote engineering discovery failure: PROFINET DCP is local Ethernet discovery, so opening a TCP port does not solve that problem.
- Added a communication record usable in Exercise 7; increased 35–45 to 40–50 minutes.

**Why:** Hand-building a binary greeting is unnecessary cognitive load for the target audience. Students should learn the difference between reachability, a listener, a valid transaction and a supported security claim. Register mapping and local discovery are concrete commissioning issues. The [PI PROFINET commissioning guide](https://www.profibus.com/fileadmin/media/downloadsection/installation_guide/PROFINET_Commissioning_8082_V153_Sep22.pdf) supports the device addressing/naming context.

**Validation:** Both Modbus numbering modes returned the same addressed data; listeners and HTTP were confirmed. Captures contained background ABB requests, including writes, and DCP Identify messages without IP. The optional greeting returned `ACKF`, 28 bytes.

**Tradeoff to review:** The core now has less hands-on OPC UA content. It does not establish a real OPC UA application session or compare secure/insecure session settings. The HMI is a scenario framing the command comparison, not an actual HMI replacement. Unknown register units remain unknown without a map.

**Joint decision:** Pending.

## Exercise 7 — Plant zoning and an enforced maintenance conduit

[English](../docs/en/exercises/ex7-segmentation-firewall.md) · [German](../docs/de/exercises/ex7-segmentation-firewall.md). Handout: PDF pp. 33–34, 128–148, 162–164.

**Kept:** All 18 command blocks are unchanged. Students still build VLAN 70, configure trunks and gateway policy, move the client, prove allowed HTTP and denied SSH, inspect logs, and restore the original configuration. Both extras remain: follow the routed request; distinguish replies from new reverse/gateway connections. The exercise now uses each student's existing lab with its recovery checkpoint instead of requiring an additional copy.

**Changed:**

- Added 20–30 minutes of design using the handout's water-treatment process before configuration. Students can work individually or discuss with a colleague, then configure their own lab.
- Required students to respect controller boundaries. WAGO controls several treatment functions; Schneider spans process and auxiliary/disposal functions. Assigning a VLAN to one PLC interface cannot create enforced separation between that PLC's internal outputs.
- Required a zone sketch and four directional communication rows, with observed facts distinguished from dependencies still to verify.
- Framed the implementation as a temporary diagnostic client's one approved conduit. Explicitly explained that the Hirschmann-labelled Linux node is a test-client stand-in, not a normal operator-workstation role for an industrial switch.
- Added a six-row intended communication matrix, ownership/expiry and change acceptance record.
- Explicitly stated that permitting WebVisu TCP 8080 also permits any controls exposed by the application. The firewall does not enforce viewing-only access.
- Increased 60–90 to 90–120 minutes, including zone design.

**Why:** The handout teaches design and defend, so students should justify a boundary before implementing commands. The controller-allocation constraint prevents an attractive drawing from claiming impossible isolation. Keeping the tested small conduit makes the implemented part tractable and recoverable.

**Validation:** The 18 blocks match the previously executed build/denial/log/cleanup procedure exactly. That earlier isolated run restored identical RouterOS export hashes and original endpoint routes. This review rechecked the flat baseline and documented command parity. The newly planned full repeat was not performed before switching to joint review; it is not a new full execution pass.

**Tradeoff to review:** The practical implements one small conduit, not the entire water-plant design. The endpoint label remains awkward. At 90–120 minutes, design plus configuration may still be too much for beginners; timing needs a student trial. Reverse and gateway denial checks are optional extras even though the matrix describes them. We should decide whether these acceptance checks belong in the required path.

**Joint decision:** Pending.

## Supporting pages changed

- **Overview:** Maps lessons to handout topics, introduces terminal versus filter use, and separates prediction, observation, explanation and next action. Main-task estimates total about 5–6½ hours; all extras add about 2–2½ hours. These are planning estimates, not measured student timings.
- **Agenda:** Replaced the older conference schedule with a two-day sequence aligned to the handout.
- **Landing page:** Describes OT professionals new to network investigation as the target audience.
- **Lab guide:** States that each student has their own appliance. Removed the draft's ARP reservation section and the incorrect claim that restarting a student's lab interrupts other students.
- **Automation outlook:** Replaced the networka installation/command tutorial with a 15-minute configuration-drift/change-proposal exercise using the actual switch port. It teaches intended versus observed state, a scoped correction, verification, rollback and idempotence. **Scope tradeoff:** This removes hands-on networka introduction; it should be reviewed separately rather than silently accepted as part of the seven lessons.

## Verification and remaining work

Both English and German MkDocs strict builds passed after the lesson/supporting edits. The installed theme's existing emoji-extension deprecation warning remains. All seven lessons' fenced commands/filters match between languages, shell syntax checks passed, and each lesson has two extras after its solution. Representative rendered EN/DE pages were visually inspected. No new test suite or package installation was added.

Actual lab evidence includes endpoint addressing/routes, switch VLAN membership, RSTP and LACP state, passive packets, scoped reachability, HTTP, Modbus and the optional OPC greeting. Earlier complete active evidence remains in [18 October practical validation](2026-10-18-vlan-practical.md); it is not relabelled as newly executed evidence.

After stopping the fresh ARP attempt, cleanup confirmed no arpspoof process, unchanged forwarding value `1`, the original gateway MAC `4c:5e:0c:09:09:11` and successful gateway pings. The isolated copy VM223 was shut down and verified stopped. The temporary review tunnels and preview server were closed. No VLAN configuration was changed during this interrupted repeat.

The supplied slides were reviewed and the drafted scenarios aligned to them, but no representative student has completed this revised sequence. Technical execution does not establish clarity, pacing or learning outcomes. Those are part of our joint review and a later student pilot.

## Correction after confirming individual student labs

The course owner confirmed that each student has their own lab. The draft incorrectly imposed reservations, shared-appliance restrictions and extra cloning on that setup. The correction covers both languages and the setup, overview, agenda, troubleshooting, cheatsheet and container introduction.

**Reversibility:** Two-way door; local documentation only.

**Problem:** Shared-lab assumptions blocked practical work and added needless setup.

**Desired outcome:** Each student completes the exercises in their own existing lab and restores its baseline after changes.

**Options:**

- Remove shared-lab restrictions and use the student's existing lab with its recovery checkpoint. Impact 9: removes barriers across the practical course. Confidence 10: the course owner explicitly confirmed the deployment model, and the affected instructions were read. Ease 9: text changes reuse existing commands and recovery. **ICE 810.**
- Keep additional reserved copies as a requirement. Impact 2: duplicates isolation each student already has. Confidence 9: the owner's clarification establishes separate labs. Ease 3: unnecessary provisioning and access management. **ICE 54.**

**Recommendation applied:** Remove the false restrictions. Keep baseline checks, cleanup and checkpoint recovery because they address actual state changes within each student's own lab.

| Exercise | Result of this audit |
| --- | --- |
| 1 | Reviewed both extras; no shared-lab restriction needed removal. |
| 2 | Removed reservation and forced instructor-demonstration requirements. Every student performs the full interception in their own lab. |
| 3 | Removed the maintenance-window wording from the lab measurement discussion. The main task remains observing RSTP state. |
| 4 | Removed shared-appliance link restrictions and instructor approval for a lab failover trial. The authored captures still compare a stable two-member bond; no executed failover result is claimed. |
| 5 | Reviewed both extras; no shared-lab restriction needed removal. The four-target scope is the task's defined investigation, not a sharing constraint. |
| 6 | Made the optional OPC UA greeting available to the student rather than labelling it an instructor demonstration. |
| 7 | Uses the student's existing lab and recovery checkpoint. Removed the extra-copy requirement, group reservation, access-extension administration and warning about overwriting another group's design. Individual design work is supported; discussion with peers is optional. |

Supporting pages now use the same individual-lab model. Appliance-console recovery remains instructor-operated because the student browser reader does not expose that console. The 18 VLAN practical command blocks and the other exercise commands are unchanged by this correction.

**Verification of this correction:** Both MkDocs strict language builds passed. Fenced command blocks match between English and German for all seven lessons; all 18 VLAN practical blocks also match the previously committed version. `git diff --check` passed. A course-wide text audit found no remaining shared-lab reservation or extra-appliance requirements. No live lab state was changed for this correction.

### Joint review sequence

For each exercise, agree the OT situation, intended skill, core student actions, both extras and acceptable evidence. Record what to keep, change or remove below, then update both languages together. Publication follows acceptance of the revised course.

| Exercise | Joint review status | Agreed change |
| --- | --- | --- |
| 1 | Discussed | Remaining exercise choices pending |
| 2 | In review | Remove shared-lab requirements; remaining choices pending |
| 3 | Pending | Pending |
| 4 | Pending | Pending |
| 5 | Pending | Pending |
| 6 | Pending | Pending |
| 7 | Pending | Pending |
| Automation outlook | Separate scope decision | Pending |

## Diode microsegmentation addon

Added a 35–45 minute optional practical after Exercise 7's VLAN cleanup in both languages. It can also be attempted directly from the flat baseline. The source is the supplied handout's microsegmentation section: PDF pages 149–152, printed slides 158–161, particularly the unpatched disposal-system example.

The real routed diode protects Linux behind `10.10.0.21`. Students retain Dell engineering access with one `/32` exception, block Hirschmann, deny new endpoint-initiated connections while retaining replies, then remove the incoming exception and restore the baseline. The task uses the existing Diodes profile GUI, configured terminals and installed tools. It distinguishes address policy from user authentication, per-service filtering and a physical one-way diode.

The addon remains inside the existing Exercise 7 reader entry because the deployed course reader allowlists seven lesson IDs. It introduces three shell blocks; the original 18 VLAN blocks are unchanged. Both overviews and agendas point to the addon. See [the addon validation](2026-10-21-diode-addon.md) for the live policy checks and cleanup evidence.
