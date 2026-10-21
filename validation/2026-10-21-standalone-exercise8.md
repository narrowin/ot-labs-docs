# Standalone Exercise 8: diode microsegmentation

21 October 2026. The course owner requested a separate exercise instead of an addon buried after Exercise 7's cleanup.

## Decision

**Reversibility:** Two-way door. Course organization, one explicit catalog entry and the reader's lesson-link patterns; existing lesson IDs and device configurations remain unchanged.

**Problem:** Students cannot find the diode practical in the lesson list because it is embedded inside the VLAN exercise.

**Desired outcome:** Microsegmentation appears as Exercise 8 in both languages, with its own tasks, two extras, solution and mandatory cleanup.

**Options:**

- Move the practical into Exercise 8 and register its ID in the reader's shared catalog. Impact 10: directly supplies the independent lesson requested. Confidence 9: reader/API/assistant catalog and existing bilingual procedure were inspected; both strict documentation builds pass. Ease 8: preserves commands and targets and extends the existing catalog and link handling. **ICE 720.**
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
- The controller and assistant share the same explicit catalog. Adding Exercise 8 there makes it selectable and gives the assistant the new lesson's own context. Both existing frontend lesson-link patterns now recognize Exercise 8, so the link from Exercise 7 is usable in the reader as well as the standalone site.

## Verification

Both strict MkDocs builds pass. Every command block and configured tool target from the former Exercise 7 is retained unchanged across the new Exercise 7/8 pair. English/German command blocks match. All eight lessons in each language retain exactly two extras and fit the reader's 65,536-byte limit.

Controller formatting, lint and changed-file strict typing pass. The final full pre-commit gate passes all 12 hooks. The frontend's existing 144 test files pass all 1,006 tests; its 14 script checks, lint and production build also pass. No tests, fixtures or mock infrastructure were added.

Both prepared native service images resolved all 16 lessons from the staged course. All 909 frontend files in the prepared controller image match the checked build. Only the three-line catalog/link change is applied to committed application sources; the derived runtime images preserve the previously deployed controller and agent as their bases.

The final complete controller test run passed: **7,395 passed, 106 skipped**, using the existing default two-worker setting. An earlier full run failed because the temporary dependency cache was empty; after correcting that environment, an eight-worker run hit a fractional-second JWT-lifetime assertion in an unrelated assistant test (328.954 seconds remaining versus a required value greater than 329). The complete suite was rerun successfully with its normal settings. No test, tolerance or unrelated behavior was changed. The first pre-commit run overlapped the final link edit; the subsequent full run against stable files passed all 12 hooks.

The policy sequence was already executed against the running lab and restored, as recorded in [the original diode validation](2026-10-21-diode-addon.md). This change moves that same procedure; it does not introduce or rerun a network policy change.

## Final publication

- Application source: `nwos-build` commit `d22c1d74b`; initial bilingual split: `389290a`; final course: `b86745f96e92296c150fccef222d16188dee084b`.
- [Live Exercise 8](https://ot-student-lab.int.narrowin.com/network/exercises/ex8-diode-microsegmentation?network=1&snapshot=1) has its own lesson-list entry in English and German. Selecting it activates **Exercise 8 / Übung 8** for its tools.
- Final persistent course mount: `/nw/data/ot-student/course-releases/b86745f/docs`, read-only at `/opt/ot-labs-docs` in both services. All 35 published source-file checksums pass in both running services.
- Final course archive: 513,251 bytes; SHA-256 `70e3c338803db2b233b4aecc2d8a35ce098d890a376e9a566641a6c87c4d9901`.
- Controller image: `sha256:90799aff36fdaa281e5392c50efd9fe7ade99f11d5020740e12e5acefbd0b757`; assistant-agent image: `sha256:4c7f9aec0e7d55f0334775187240b0205e43157711dd4a6833f5819b52a1f904`. Both are healthy. Their image locks and the persistent course overlay retain the update across ordinary restarts.
- Final Compose overlay SHA-256: `f25eca030ff677d6a3c4c29236150724debeb95f95191d6c42c43c7fc04a9ba5`. The preceding course overlay is saved under `/nw/data/ot-student/course-releases/b86745f/backup/compose.ot-student-release.before.yml`.

All 16 live lesson views passed title, language and two-extra checks. Visual inspection then caught unsupported cleanup-anchor and overview links in the new lesson. Those were corrected in both languages without changing commands. Final English/German Exercise 8 views contain no blocked-link marker; the cleanup reference opens Exercise 7, and its three terminals resolve to `dell-ews1`, `hirschmann-edge1` and `protected-linux`, carrying the Exercise 8 context. The corrected ARP explanation remains in Extra 1.

## Operational incident during publication

The controller and assistant were recreated twice: first for the catalog/link application update and initial course, then for the corrected course mount. No VM or lab-device restart was requested. Read-only comparison confirms that all **16 lab-node container identities are unchanged**.

The second activation reused an older course-only publication procedure without the active-capture guard used by the first activation. An active `sw-dist` / `eth1` capture was interrupted. Worker logs show it started at **21:38:43 Europe/Zurich** and was removed at **21:39:13** on 21 October 2026. A controller epoch change invokes capture cleanup, deleting the temporary GUI/analyser and its RAM-backed capture data. The snapshot file was absent when checked; no retained capture could be recovered from the worker.

The course-only post-check exited nonzero because these two capture containers disappeared, after both application services had become healthy and their 35 source checksums had passed. The final course remains active; it was not rolled back. The operator was informed of both restarts and the capture interruption. No further restart or capture operation was performed afterward; subsequent environment checks were read-only.

This interruption was a publication mistake. Every controller activation must check for an active capture immediately before restarting and defer while one is active; the course-only path must not omit that guard. The existing guarded initial activation and rollback inputs are retained at `/nw/data/ot-student/course-releases/389290a/`.
