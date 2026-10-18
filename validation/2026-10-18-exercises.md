# Student exercise review, 18 October 2026

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

## Documentation checks

Run from this repository with the existing environment:

```bash
uv run --no-sync mkdocs build -f mkdocs.en.yml --site-dir /tmp/ot-course-site-en
uv run --no-sync mkdocs build -f mkdocs.de.yml --site-dir /tmp/ot-course-site-de
git diff --check
```

The installed Material theme emits an existing emoji-extension deprecation warning. English agenda links to old lab headings are being reconciled with the setup guidance; MkDocs reports these as informational messages rather than build failures.
