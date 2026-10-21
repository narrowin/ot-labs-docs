# Live bilingual exercise publication

21 October 2026. Target: the existing student lab at `ot-student-lab.int.narrowin.com`, VM222. This records the first publication. The current reader release and subsequent student-facing corrections are documented in [the clarity report](2026-10-21-student-clarity-fixes.md).

## Publication decision

**Reversibility:** Two-way door. Keep the prior release overlay and baked-in course content available for rollback.

**Problem:** The running reader and assistant contain the older October 18 course files inside their images.

**Desired outcome:** Both services use the same updated English and German lessons, including the separate PLC2A/PLC2B list, supplied process image and corrected diode ARP question, after normal service recreation.

**Options:**

- Mount one dedicated course directory read-only into both services using the existing Compose mount pattern. Impact 9: keeps reader and assistant content aligned across recreation. Confidence 9: both live services and the documented exercise overlay were inspected. Ease 8: adds two mounts to the existing active release overlay and recreates only those services. **ICE 648.**
- Copy files into the two running containers only. Impact 3: recreation returns to the old image content. Confidence 9: the live containers have no course bind mount. Ease 9: a short file copy. **ICE 243.**

**Recommendation:** Use the dedicated shared directory and read-only mounts. Preserve the existing release-overlay entries and image identities.

## Source and payload

Course source revision: `e276eca436e0608790cae31ecae0689b2087ebd4`. This includes the final reviewed German wording, all seven lessons in both languages, their support pages, the PLC reference list and image, and A5's explicit focus on interception of the diode's VLAN 10 traffic.

The publication archive contains the 33 course files changed since the exercise review began, plus a source manifest and checksums. Size: 512,017 bytes. Archive SHA-256: `519ac567a6209c94582a8de468ffcaaaf4b15a6da2c9853b8cf72f958f9dba07`.

Both strict MkDocs builds pass. All seven English/German command sets match, the German wording edits preserve the existing commands, and all numbered lesson files remain below the reader's 65,536-byte limit.

## Deployment evidence

Publication completed successfully.

- Published directory: `/nw/data/ot-student/course-releases/e276eca/docs` on VM222. Existing unchanged course assets were retained; all 33 updated files match the archive manifest.
- The controller and assistant-agent both mount this directory at `/opt/ot-labs-docs`, read-only. Both services are healthy and independently pass all published file checksums.
- The active release overlay retains its previous terminal/capture entries and adds only the two course mounts. Its SHA-256 changed from `0e4e20fa825b29c3a0233b35f0d6da195bd2a6ef54a5bd6c830df6497576408d` to `b40aa6ae1a1c277c70909ab6bb9bd12a4c32f411db4f7c88a342a09a549e74d3`.
- Activation used the existing lab owner with `compose up -d --no-deps --no-build --pull never --wait --wait-timeout 90 controller assistant-agent`. The image identities were retained. Only those two containers were recreated; all other services and all 16 lab nodes retain their original container identities.
- The course directory and active Compose overlay are on persistent VM storage. Both newly recreated services read the updated files through their mounts. No VM reboot was needed for publication.

The established Chrome/UI driver opened every lesson in the actual live reader and checked its language, current title and both optional extras:

| Exercise | English | German | Extras in each version |
| --- | --- | --- | --- |
| 1 | Pass | Pass | 2 |
| 2 | Pass | Pass | 2 |
| 3 | Pass | Pass | 2 |
| 4 | Pass | Pass | 2 |
| 5 | Pass | Pass | 2 |
| 6 | Pass | Pass | 2 |
| 7 | Pass | Pass | 2 |

Exercise 1 displays the exact five-item PLC list in both languages, with PLC2A and PLC2B separate. The supplied image loads at its original 1365 by 1284 pixel resolution. Exercise 7 displays the updated A5 heading in both languages. Its solution was expanded and inspected: the diode's ARP entry for Dell and Dell's ARP entry for the diode are explicitly named as VLAN 10 interception targets, while the protected endpoint's private ARP cache remains across the routed boundary. Live screenshots of the list/image and both expanded solutions were inspected. The reader was left on English Exercise 1.

## Rollback and scope

The previous release overlay is retained at `/nw/data/ot-student/course-releases/e276eca/backup/compose.ot-student-release.before.yml`; before/after container identities are in that same backup directory. Restoring that overlay and running the same two-service Compose activation returns both services to their original baked-in course files. The original images remain available.

This publication verifies the served content, language switching, image delivery and display of the updated lessons. It does not constitute a new execution of every networking practical or an active ARP interception trial. The earlier exercise validation records retain those qualifications.
