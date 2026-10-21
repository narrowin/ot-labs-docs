# Diode microsegmentation addon validation

21 October 2026. This records the optional addon in [English Exercise 7](../docs/en/exercises/ex7-segmentation-firewall.md) and [German Exercise 7](../docs/de/exercises/ex7-segmentation-firewall.md), and the reasoning for its design.

## Teaching outcome and slide alignment

The supplied [Design and Defend handout](</Users/md/Downloads/OT Network Security - Design and Defend handout (1).pdf>) covers microsegmentation on PDF pages 149–152, printed slides 158–161. All four pages were read and rendered. The disposal-system example identifies an unpatched asset as a risk to the rest of its zone; the following slides distinguish endpoint agents from network enforcement and mention complete isolation and directional communication.

The addon uses the installed routed diode to make those concepts observable. Protected Linux stands in for the legacy computer; it is not claimed to run an actual disposal process. Students first restrict incoming access, then restrict new outgoing connections. Keeping a permitted application's replies working distinguishes stateful policy from a physical unidirectional path. The solution explains residual risks rather than repeating the slide's broad claims about preventing exploitation or achieving security levels.

**Reversibility:** Two-way door for course edits and the temporary policy changes on the single student diode; the initial profile values and assignment were recorded and restored.

**Problem:** Students need a practical microsegmentation task that uses the newly added diode and its real profile controls.

**Desired outcome:** Students enforce and prove an endpoint boundary, retain justified maintenance access, contain new outbound initiation and restore normal access.

**Options:**

- Add the existing profile workflow to Exercise 7, including positive/negative application checks. Impact 9: directly demonstrates the slide's use case with the installed hardware model. Confidence 9: topology, frontend controls, profile and live incoming/outgoing enforcement were inspected. Ease 8: reuses the enrolled diode and installed commands. **ICE 648.**
- Add a new reader application feature and an industrial protocol proxy. Impact 5: could support a different lesson, but exceeds the available address-policy controls. Confidence 3: neither a protocol proxy nor its learning procedure has been qualified here. Ease 2: adds implementation and deployment. **ICE 30.**

**Recommendation applied:** The first option. The existing reader's explicit seven-ID allowlist makes an addon inside Exercise 7 usable without a backend change. The overviews distinguish its additional 35–45 minutes from the seven main lessons and their existing extras.

## Environment and procedure

- Student appliance: VM222, `ot-student-lab.int.narrowin.com`, default network 1, live view.
- Existing lab integration: `nwos-build` commit `22eda5e40`, particularly `labs/ot-student/microsegmentation/README.md`, `exercise.md`, `baseline.json` and `topology.jq`.
- Diode: `vlan10-diode`, model `vm86`, firmware `2.0.5 debug`; WAN `10.10.0.21`, LAN `192.168.254.254/24`.
- Assigned profile: `Microsegmentation - baseline`, ID 2, used by one diode. Initially both directions were default allow with no exceptions.
- Dell data source: `10.50.0.11/8`; Hirschmann: `10.50.0.12/8`; protected Linux: `192.168.254.1/24`.

The product launcher, Diodes list, profile editor, Save and Save & apply confirmation, and live configuration status were inspected through the repository UI driver using the established Chrome wrapper. Only the two access-policy directions and the single Dell source exception were changed. Assignment, name, description, DNS, NTP and logging settings were preserved. After each apply, the device's production-current status and confirmation were checked before interpreting probes.

The exact lesson commands were executed inside the existing lab endpoints through the established guest command transport. HTTP requests bound to data interface `eth1`; TCP 22 was checked with a fresh `nc` connection, without an SSH login or credentials. The protected endpoint's outgoing check was an HTTP request to WAGO A's existing WebVisu. No PLC values or programs were changed.

## Executed results

| Applied policy | Dell → Linux HTTP / TCP 22 | Hirschmann → Linux HTTP / TCP 22 | Linux → WAGO HTTP |
| --- | --- | --- | --- |
| Original allow/allow baseline | Page received / TCP success | Page received / TCP success | HTTP 200 |
| From Network deny, Dell `/32`; From Endpoint allow | Page received / TCP success | Both rejected | HTTP 200 |
| From Network deny, Dell `/32`; From Endpoint deny | Page received / TCP success | Both rejected | Rejected, curl exit 7 |
| Both directions deny, no exceptions | Both rejected | Both rejected | Rejected, curl exit 7 |
| Restored allow/allow baseline | Page received / TCP success | Page received / TCP success | HTTP 200 |

The first allowlist was assigned at 15:08:16 Europe/Zurich and confirmed at 15:09:08. The outbound-deny revision was assigned at 15:10:49 and confirmed at 15:11:45. The no-exception deny-all revision was assigned at 15:12:55 and confirmed at 15:15:26. A probe before that last confirmation still showed the previous Dell exception working; after confirmation both clients were rejected. These observed delays reinforce the instruction to wait for actual confirmation; a saved profile is not immediate proof of enforcement.

During confirmed deny-all enforcement, the actual student **Open terminal** action to Protected Linux connected through its independent management path. The address command returned `192.168.254.1/24` on `eth1`. The owned terminal session was then exited.

Cleanup restored both directions to default allow with no exceptions. The restored revision was assigned at 15:17:20 Europe/Zurich and confirmed at 15:18:10. Both clients again received the protected page and connected to TCP 22; Protected Linux again received HTTP 200 from WAGO. Profile assignment and all non-policy settings matched the recorded baseline. The revision and confirmation timestamps advanced as expected; this was a configuration restoration, not a history rollback.

The observed rejects were connection refusals: HTTP returned curl exit 7 and TCP 22 returned a failed connection. The lesson therefore asks for successful or blocked fresh connections rather than incorrectly promising a timeout or Nmap `filtered` state.

## Content checks and limits

The addon has three fenced shell blocks. They match between English and German; shell syntax passes. The original 18 VLAN practical blocks remain unchanged, and both files remain below the reader's 65,536-byte limit. Both strict MkDocs builds pass. Rendered English and German addon pages were inspected in the existing preview browser.

Live policy results qualify the tested address-based enforcement, not port-specific rules, authentication, patching, IEC 62443 certification or protection against all address spoofing. No reboot-persistence trial or ARP attack was added to this addon validation. The new lesson text has not yet been published to the running reader, and 35–45 minutes is a planning estimate rather than a student-pilot measurement.
