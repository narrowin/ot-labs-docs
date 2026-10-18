# Student exercise review, 18 October 2026

This records the initial review and deployment. The subsequent executed VLAN build, ARP/osquery repeat, command-copy repair and final course qualification are recorded in [the follow-up validation](2026-10-18-vlan-practical.md). Exercise 7 below describes the earlier design-only version.

Walkthrough environment: VM222, `ot-student-lab.int.narrowin.com`, current flat student lab. Checks use its standard installation account, configured terminals, actual endpoint commands and the deployed capture service. This account has system-administrator permissions; these results do not qualify a separate read-only role.

## Approach

**Reversibility:** Two-way door for course text and named links. Changes to shared switch/firewall configuration require a separate operator decision.

**Problem:** The course mixes older navigation and topology assumptions with the deployed reader.
**Desired outcome:** Students can follow the seven lessons in order, reach the correct tools and establish each result from observable evidence in either language.

**Options:**

- **A: Revise the canonical lessons and reuse deployed tools.** Impact 9 (directly removes incorrect steps). Confidence 8 (read the link contract and topology; exercised real terminals, captures and protocols). Ease 8 (contained Markdown changes and existing services). **ICE = 576.**
- **B: Build a new guided exercise application.** Impact 6 (could add guidance beyond this request). Confidence 4 (no student evidence establishes a need for a new application). Ease 3 (new UI, state and qualification). **ICE = 72.**

**Recommendation: A.** Finish each lesson's plan, implementation, validation and review, then commit that lesson before completing the next. No test suites, fixtures, mocks or failure-injection infrastructure are added.

## Exercise 1

- **Plan:** Keep the four observation tasks; add the named VLAN view, standalone navigation and two bounded extensions after the solution.
- **Implementation:** English/German instructions, collapsed solution, frame-size puzzle and next-hop-MAC comparison. Named links retain network/snapshot through the deployed reader.
- **Validation:** WAGO gateway and DMZ pings succeeded. Access captures showed untagged lengths 74/170 bytes for 32/128-byte payloads; tagged uplink frames showed 78/174 bytes. Requests to gateway and DMZ used the gateway's destination MAC on the access link. Both language sites built successfully.
- **Review:** Commands match between translations; exactly two extras follow the solution. Hostname, discovery membership and captured tagging are distinguished. No switch configuration change is required.

## Exercise 2

- **Plan:** Preserve ARP interception, cleartext inspection, detection and mitigation objectives while bounding the target pair and making recovery observable.
- **Implementation:** Named terminals/capture, a harmless TCP marker, timed helper, exact cleanup, standalone navigation and two evidence extensions in both languages.
- **Validation:** The victim gateway mapping changed to WAGO A's MAC. Captured requests and the received `LAB-ARP-DEMO` marker established interception. The direct CtrlX path remained distinct. Corrective ARP plus the targeted neighbor refresh restored the original gateway MAC; no arpspoof process remained and forwarding retained its original value. All four reader links resolved in the selected live network; both MkDocs builds passed.
- **Review:** The lesson accounts for observed ICMP redirects and ping loss, distinguishes ARP claims from authenticated identity, and does not promise decryption of encrypted traffic. The extras follow the solution and the EN/DE commands agree.

## Exercise 3

- **Plan:** Preserve operational root/port analysis, BPDU interpretation and LACP/RSTP coordination; replace the fixed external demo URL with the deployed instance-0 view.
- **Implementation:** Named switch terminals, direct spanning-tree view, interface-selected capture, standalone navigation, root-election and healthy-link diagnosis extensions.
- **Validation:** All four bridges reported gw as root, ID ending `09:11`. The capture decoded 42 RSTP BPDUs advertising the same root. sw-acc1's bond was a forwarding root port; sw-acc2 ether11 was physically link-ok but alternate/non-forwarding, with increasing BPDU counters. Both MkDocs builds passed.
- **Review:** Configuration is separated from operational election state; the bond is one logical bridge port. A received frame is not mistaken for proof of forwarding through a discarding port. No topology or priority changes were made.

## Exercise 4

- **Plan:** Use the flat lab's actual LACP bond, bounded traffic and separate request-direction evidence; retain redundancy reasoning without interrupting the shared links.
- **Implementation:** Named member captures and correct source endpoints, reciprocal system-ID checks, payload/hash and actor/partner extensions, standalone steps in both languages.
- **Validation:** Both switches reported two active members, 802.3ad and layer-2-and-3 hashing. Captures showed synchronized/collecting/distributing LACP with reciprocal actor/partner identities. CtrlX requests used ether9 and WAGO requests ether10; WAGO payloads 32/128 produced tagged lengths 78/174 on the same member. Both MkDocs builds passed.
- **Review:** WAGO B is not incorrectly used as a source behind sw-acc1. Return-direction hashing is treated separately. Counters and pings do not become a throughput or failover measurement. A manual outage trial requires an operator-approved isolated lab and was not performed here.

## Exercise 5

- **Plan:** Retain remote-access assessment, discovery, service identification and reporting with explicit lab scope and evidence limits.
- **Implementation:** Configured jumphost access, four target IPs, bounded TCP checks, an HTTP request, a reproducible finding template and two vantage-point/packet-state extensions in both languages.
- **Validation:** The jumphost used source 2.2.2.2 via 2.2.2.1; all four listed discovery targets responded. WAGO ports 22/502/4840/8080 were open, HTTP returned 200 and 3S_WebServer, and port 65000 returned closed. The capture showed SYN/ACK for 8080 and RST/ACK for 65000. WAGO B reached the same services through a direct route. Both MkDocs builds passed.
- **Review:** Port database labels are not application/version proofs. The report does not invent accepted credentials or a vulnerability. No password guessing, broad scans or tool installation are required.

## Exercise 6

- **Plan:** Preserve protocol discovery and security comparison using real decoded exchanges; replace screenshot/menu detours with the deployed named WebVisu link.
- **Implementation:** Passive hierarchy/discovery, one read-only Modbus poll, direct HTTP and WebVisu, local listeners, a bounded OPC UA Hello, standalone navigation, register-numbering and Layer 2 discovery extensions.
- **Validation:** Matched Modbus function-3 request/response, address 0 and quantity 2; both mbpoll numbering modes read the same wire address. HTTP GET returned 200. The named WebVisu link rendered its native canvas and repeated application POSTs returned 200. The OPC UA greeting returned ACKF/28 bytes and decoded Hello/Acknowledge frames. DCP frames had EtherType 0x8892, station name pn-device and no IP header. Both MkDocs builds passed.
- **Review:** Background writes are separated from the student's reads. Steady GUI values are allowed. Port labels and transport greetings do not become claims about session encryption or full PROFINET IO capability. No dependencies were installed and no PLC controls/programs were changed.

## Exercise 7

- **Plan:** Preserve segmentation, directional policy, stateful firewall and verification objectives; ground the design in the actual flat forwarding path.
- **Implementation:** Named VLAN view and terminals, observable baseline probes, rule-order analysis, shared Toolbox inspection, standalone steps and two extensions in both languages. The complete VLAN build sheet includes access/trunks, gateway CPU, a non-overlapping subnet, explicit routes, scoped policy and restoration.
- **Validation:** ABB used a direct eth1 path to WAGO; the jumphost used 2.2.2.1. Fresh TCP 22/8080 were open and HTTP returned 200. The gateway's active final drop followed broad DMZ/private accepts, and established/related replies were accepted earlier. Toolbox retained revision 20 and its two-zone baseline. WAGO routed the proposed 172.22.70.12 through its gateway, while the candidate endpoint had explicit old-gateway routes requiring replacement. Both language builds passed.
- **Review:** Saving Toolbox does not enforce device rules; same-VLAN traffic bypasses the gateway. The two extras distinguish replies/new initiations and account for the broad 172.16/12 match. No live policy, VLAN, address or link was changed. The isolated configuration/reset prerequisite and acceptance plan are recorded in [the VLAN feasibility analysis](2026-10-18-vlan-build-plan.md).
- **Specific app gap:** The reader does not interpret `/toolbox/` as an app action. Exercise 7 therefore uses the verified product launcher and Policies card. No new link handler is required for the lesson.

## Setup, references and capability gaps

- Replaced both lab guides and quick references with the deployed appliance workflow, short readiness checks, named links and standalone manual navigation. Updated entry pages, conference task links, exercise navigation, overview and filter reference.
- Readiness check: WAGO A reported 10.10.0.11/8 and four gateway replies. Browser actions offered WAGO A, CtrlX, Schneider and ABB; their proxy pages were checked. WAGO B TCP 8080 refused a direct lab request and no browser action was offered, so it is explicitly absent from the visualization task.
- The reader currently supports the seven canonical lessons, not arbitrary setup/cheatsheet pages. Those pages are available in the standalone site with manual navigation. Toolbox also needs the product launcher. An isolated VLAN configuration/reset exercise is not provisioned by the app; Exercise 7 supplies the complete design and acceptance plan.
- The course contains fourteen optional sections, two after each solution. No new automated test infrastructure, dependencies or broad/failure traffic was introduced.
- Replaced the searchable English monolith with canonical lesson links while retaining its main exercise headings, so standalone search cannot send students to contradictory instructions.

## Final deployed review

- All fourteen English/German API documents returned 200 with revisions matching the committed source. Both active controller and assistant resolved the same fourteen hashes from `/opt/ot-labs-docs`; the two course images also matched all 27 packaged page hashes.
- Reviewed every lesson in both languages in the deployed reader: two extras each, no blocked links. A final visual check found the Markdown renderer blocking bare relative lesson URLs; the reader now normalizes those specific destinations to its existing scoped routes. Actual German prerequisite and English next-lesson clicks retained network 1 / live snapshot 1. The generic outdated-content banner is removed.
- The complete frontend build, lint and existing 144 test files / 1,006 tests plus 14 script checks passed. The full pre-commit gate passed with existing per-service environments. Its initial default attempt could not resolve uncached dependencies offline; no packages were installed.
- Course-only activation preserved the other 22 container identities. The subsequent controller-only link correction preserved the course fingerprint and all 23 other containers, including the assistant. Both activated services reported healthy. Guarded rollback inputs are under `/nw/data/exercise-course-review-20261018/` and `/nw/data/exercise-course-links-20261018/`; revert the later link layer first.
- Final cleanup found no arpspoof process; WAGO B's gateway mapping was the original `4c:5e:0c:09:09:11`. Review tool tabs were closed and the reader was left at English Exercise 1. Shared switch/firewall configuration and Toolbox revision 20 were preserved.

## Documentation checks

Run from this repository with the existing environment:

```bash
uv run --no-sync mkdocs build -f mkdocs.en.yml --site-dir /tmp/ot-course-site-en
uv run --no-sync mkdocs build -f mkdocs.de.yml --site-dir /tmp/ot-course-site-de
git diff --check
```

Both language sites build successfully. Their internal page/anchor links resolve; absolute reader links are intentionally retained for LNE and have manual standalone instructions. The installed Material theme emits its existing emoji-extension deprecation warning.
