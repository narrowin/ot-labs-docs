# Standalone Exercise 8: diode microsegmentation

21 October 2026. The course owner requested a separate exercise instead of an addon buried after Exercise 7's cleanup.

## Decision

**Reversibility:** Two-way door. Course organization and one explicit catalog entry; existing lesson IDs and device configurations remain unchanged.

**Problem:** Students cannot find the diode practical in the lesson list because it is embedded inside the VLAN exercise.

**Desired outcome:** Microsegmentation appears as Exercise 8 in both languages, with its own tasks, two extras, solution and mandatory cleanup.

**Options:**

- Move the practical into Exercise 8 and register its ID in the reader's shared catalog. Impact 10: directly supplies the independent lesson requested. Confidence 9: reader/API/assistant catalog and existing bilingual procedure were inspected; both strict documentation builds pass. Ease 8: preserves commands, targets and deployment images except the one-line catalog addition. **ICE 720.**
- Keep the embedded addon and add a more prominent link. Impact 2: still leaves the practical inside Exercise 7 and fails the requested organization. Confidence 10: the existing seven-ID catalog explicitly excludes a separate lesson. Ease 10: a link-only edit. **ICE 200.**

**Recommendation implemented:** A standalone `ex8-diode-microsegmentation` lesson, with a matching English/German catalog entry and navigation. The explicit exercise allowlist remains in place.

## What changed and why

- Exercise 7 retains its five VLAN tasks, solution, two existing extras and VLAN cleanup. Its embedded diode addon is replaced by a link to Exercise 8.
- Exercise 8 has four main tasks: identify the boundary, prove baseline applications, permit only `dell-ews1`, and block new endpoint-initiated connections while retaining permitted replies.
- Extra 1 preserves the corrected ARP question and solution: VLAN 10 cannot directly poison the protected endpoint's gateway entry, but the diode's VLAN 10 exchange can still be intercepted.
- Extra 2 moves the existing final isolation check into its own optional task: remove the last incoming exception, prove both clients are blocked, retain the independent management recovery path, then restore the baseline. No new policy mechanism or command is introduced.
- Mandatory cleanup restores both directions to default allow with no exceptions and repeats the existing positive application checks. The isolation row is labelled Extra 2 so students who skip it do not need to invent a result.
- The lesson starts from the flat baseline and does not require the VLAN 70 build. A direct link identifies Exercise 7's cleanup if students performed it.
- Both site navigations, overviews, agendas and lab introductions now identify eight exercises. The course timing includes the separate practical. Device names remain unambiguous without VLAN suffixes in the instructions.
- The controller and assistant share the same explicit catalog. Adding Exercise 8 there makes it selectable and gives the assistant the new lesson's own context.

## Verification

Both strict MkDocs builds pass. Every command block and configured tool target from the former Exercise 7 is retained unchanged across the new Exercise 7/8 pair. English/German command blocks match. All eight lessons in each language retain exactly two extras and fit the reader's 65,536-byte limit.

The policy sequence was already executed against the running lab and restored, as recorded in [the original diode validation](2026-10-21-diode-addon.md). This change moves that same procedure; it does not introduce or rerun a network policy change. Live reader and deployment results will be recorded after activation.
