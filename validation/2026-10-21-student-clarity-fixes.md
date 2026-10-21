# Student exercise clarity corrections

21 October 2026. This report records the accepted follow-up to the wording audit. Every change applies to English and German. Each numbered lesson retains two optional extras; Exercise 7 also retains the separate diode addon.

## Decision

**Reversibility:** Two-way door. Course changes are versioned; publication uses a new persistent directory and retains the prior course mount for rollback. The new optional RSTP trial changes one uplink in the student's own lab and explicitly restores it.

**Problem:** Internal naming history, repeated tasks and unnecessary implementation detail obscure the intended network lesson.

**Desired outcome:** An OT professional can follow a concrete task, obtain the stated result and explain its operational meaning without knowing the lab's development history or constructing protocol packets in Python.

**Options:**

- Correct the wording, consolidate repeated work and replace the repeated RSTP extra with a bounded recovery trial. Impact 9: addresses every reported finding while preserving the course sequence and two useful extras. Confidence 8: both language versions were read, the new commands ran in the actual lab, and the RSTP trial returned to its baseline; no student pilot has been conducted. Ease 8: contained course edits using existing lab tools. **ICE 576.**
- Remove weak extras without replacements. Impact 5: shortens the course but loses recovery practice and the requested two-extra structure. Confidence 9: the overlapping tasks are directly visible in the source. Ease 10: simple deletions. **ICE 450.**

**Recommendation implemented:** Correct the source and replace the repeated extra with the verified recovery trial.

## Exercise 1 — identify the actual VLAN connection

[English](../docs/en/exercises/ex1-vlan-discovery.md) · [German](../docs/de/exercises/ex1-vlan-discovery.md)

**Changed:** Replaced comparison with “historical device names” with finding PLC2A and checking its VLAN 10 connection to sw-acc1. Removed repeated hostname warnings from the hints and checkpoints. The PLC list and supplied process image are unchanged.

**Why:** Students need to identify an actual port and VLAN, then compare tagged uplink packets with untagged access packets. The lab's naming history does not help that task.

**Extras:** Replacement-PLC connection checklist; next-hop MAC comparison. Both retained because they extend the observation to commissioning settings and IP/MAC behaviour.

## Exercise 2 — keep the working ARP investigation

[English](../docs/en/exercises/ex2-arp-spoofing.md) · [German](../docs/de/exercises/ex2-arp-spoofing.md)

**Reviewed and retained:** The timed interception, payload evidence, GUI ARP-cache hint and restoration procedure. No exercise text or commands changed in this pass.

**Why:** These tasks demonstrate a concrete network mechanism and distinguish an observed interception from an unexplained MAC change. They do not depend on historical naming or shared-lab reservations.

**Extras:** ARP incident timeline; maintenance-compatible protection. Both retained: one relates packet evidence to incident phases; the other addresses static OT bindings and legitimate gateway replacement.

## Exercise 3 — diagnose once, then test recovery

[English](../docs/en/exercises/ex3-spanning-tree.md) · [German](../docs/de/exercises/ex3-spanning-tree.md)

**Changed:** Moved the physical-link and increasing-BPDU-counter checks into Task 4. Replaced the repeated healthy-standby-link extra with a controlled loss of sw-acc2's ether12 uplink. Students run timestamped ping probes from PLC2B, observe ether11 becoming root/forwarding, request PLC2A's WebVisu page, re-enable ether12 and verify both original port roles and communication. The introduction, overview and cleanup guidance now describe the optional change. The extra allows 15–20 minutes.

**Why:** The main task establishes why a healthy link is not forwarding. The replacement extra demonstrates what that standby path actually does during a failure. Ping loss is recorded at the probe interval; the exercise does not turn zero observed loss into an application-wide recovery-time claim.

**Extras:** Switch-replacement election review; actual uplink failover and restoration.

## Exercise 4 — remove an invalid rate comparison

[English](../docs/en/exercises/ex4-link-aggregation.md) · [German](../docs/de/exercises/ex4-link-aggregation.md)

**Changed:** Removed the instantaneous rate snapshot requested after the short ping flows had ended, together with the extended hypothetical timer discussion. Task 4 now verifies the forwarding logical bond as a single RSTP bridge port and connects it to the two physical members already observed. The second extra uses a direct maintenance-planning task rather than a technician's implausible assertion.

**Why:** A later rate sample cannot describe the earlier ping traffic. The logical-port check supports the intended relationship between LACP and RSTP without implying a throughput measurement.

**Extras:** Capture blind spot; cable versus switch/gateway failure scope. Retained because monitoring coverage and shared failure points require different operational decisions.

## Exercise 5 — one access assessment

[English](../docs/en/exercises/ex5-reconnaissance.md) · [German](../docs/de/exercises/ex5-reconnaissance.md)

**Changed:** Removed the historical-name aside. Combined the findings worksheet and repeated access-decision section into one service table and one recommendation. Students record the source, target, time and commands once, then distinguish the required WebVisu access from other reachable services.

**Why:** The supplier's diagnostic request should produce one clear assessment rather than two overlapping handovers. The result still states what the checks leave unverified.

**Extras:** Compare DMZ and local vantage points; prove open versus closed ports with packets. Both retained.

## Exercise 6 — use a simple connection check

[English](../docs/en/exercises/ex6-protocol-discovery.md) · [German](../docs/de/exercises/ex6-protocol-discovery.md)

**Changed:** Removed the optional Python code that constructs OPC UA binary Hello messages, its message-specific reference, and the warning about an “old assumption” concerning CODESYS ports. Added an installed `nc` TCP connection check to port 4840 and a SYN/SYN-ACK observation. Updated the objectives, evidence table and commissioning record accordingly.

**Why:** Students can connect a local listener, remote reachability and captured handshake without understanding binary packet construction. This check deliberately establishes TCP reachability only; OPC UA session authentication and encryption remain unverified. Modbus and HTTP tasks still provide real application request/response evidence.

**Extras:** HMI zero/one-based register mapping; local PROFINET discovery across a routing boundary. Both retained because they address recognizable commissioning and remote-maintenance problems.

## Exercise 7 — preserve separate controller identities

[English](../docs/en/exercises/ex7-segmentation-firewall.md) · [German](../docs/de/exercises/ex7-segmentation-firewall.md)

**Changed:** Replaced the ambiguous “PLC2” controller reference with explicit separate PLC2A and PLC2B entries in the asset list and communication matrix. Students verify their functions against the handout. Replaced the hostname-suffix aside with the actual change from VLAN 10 to VLAN 70.

**Why:** The network design must use the same endpoint identities as the course reference. Assigning a VLAN to one PLC still cannot separate functions implemented inside that PLC.

**Extras:** Follow a request before/after routing; distinguish replies from new initiations. Both retained. The diode addon, corrected WAN-side ARP interception question, PLC mapping and process image remain unchanged.

## Supporting pages

Removed the Container101 naming-history paragraph, the obsolete screenshot/account comparison and references to old default passwords. The overview states the current network baseline, lists the replacement RSTP extra and includes its restoration requirement. Historical network snapshots remain documented because that is a real product feature, unrelated to old hostnames. Earlier review notes now point to this report and the completed publication record.

## Validation and publication

- All seven English/German pairs have matching command blocks, exactly two numbered extras and files below the reader's 65,536-byte limit.
- Both strict MkDocs builds pass. The installed theme emits its existing emoji-extension deprecation notice; this course change does not alter the theme.
- The exact five PLC mappings and original supplied PNG are retained.
- Actual lab: WAGO B's `nc -vz -w 3 10.10.0.11 4840` succeeded. A bounded capture on its eth1 recorded SYN, SYN/ACK, ACK and connection teardown: six packets with zero application payload, matching the lesson's TCP-only claim.
- Actual RSTP trial: ether11 changed from alternate/non-forwarding to root/forwarding when ether12 was disabled. The bounded ping completed with 120/120 replies over 60.287 seconds; PLC2A HTTP returned 200 while the uplink was disabled. After re-enabling ether12, ether11 returned to alternate/non-forwarding and ether12 to root/forwarding. The subsequent four pings and HTTP request succeeded; all 16 nodes were healthy.

Published source: `db4a87b62ef77fe0e7571046943031f85f35cb02`.

- Target: [live student reader](https://ot-student-lab.int.narrowin.com/network/exercises/ex1-vlan-discovery?network=1&snapshot=1), VM222, network 1/live snapshot 1.
- Archive: 33 source files, including all seven lessons in each language, supporting pages and supplied image; 510,677 bytes; SHA-256 `ae1af6a61355926ebf735bcc522a4248f2b0e820510ecaaa2cfd3d5252bbb766`.
- Persistent course directory: `/nw/data/ot-student/course-releases/db4a87b/docs`. The controller and assistant-agent both mount it read-only at `/opt/ot-labs-docs`; all release checksums pass independently in each service.
- Only controller and assistant-agent were recreated. Their deployed image identities are unchanged; every other service and all 16 lab nodes retain their original container identities.
- The active Compose overlay retains its other entries. SHA-256 before: `b40aa6ae1a1c277c70909ab6bb9bd12a4c32f411db4f7c88a342a09a549e74d3`; after: `d0982e90dc39fbe1abadf1479ba35fdc8d1cf3b8ab9126618262822d6cd621f6`.
- The established Chrome/UI driver opened all 14 lesson/language combinations in the real reader. Every view had the correct title, language and two extras, and the exercise-specific revised text. The obsolete task wording and binary Python construction were absent.
- The rendered German recovery extra was visually inspected; both switch-restoration commands render correctly and its tool links are available. Exercise 1 retains the exact separate PLC2A/PLC2B mapping, and its original image loads at 1365 by 1284 pixels. The corrected diode A5 remains in both served Exercise 7 files.

Rollback: the previous overlay is retained at `/nw/data/ot-student/course-releases/db4a87b/backup/compose.ot-student-release.before.yml`. Restore that overlay and use the existing lab helper's two-service Compose activation to return both services to the previous `e276eca` course mount. Container and image identity records are in the same backup directory. No appliance reboot or image rebuild was required.

This pass does not rerun the unchanged ARP attack or the complete VLAN 70 build, and makes no student-learning measurement claim.
