# Executed course follow-up, 18 October 2026

This follows the [initial seven-lesson review](2026-10-18-exercises.md). It records the command-rendering repair, fresh browser walkthroughs and executed VLAN practical. Student instructions are committed in English and German; the instructor preparation/reset guide is included in both language sites.

## Scope and method

Exercises 1–6 use the flat `ot-sec-flat` baseline. Exercise 7 changes one endpoint on a separate reserved appliance and returns it to that baseline. Historical hostname suffixes do not define the active VLANs. Each lesson has a paced main result, collapsed solution and exactly two optional extras; students can keep evidence and arrange lab access to continue later.

Browser operations used the current repository `scripts/ui-driver/ui_driver.py`, managed Explorer terminals and embedded Wireshark. Independent RouterOS/Linux checks also used real appliance CLI/SSH. This was actual tool and packet validation, not a claim that every command was entered exclusively through the UI. No new test suites, fixtures, mock services or failure-injection infrastructure were added.

## Command display and browser input

The reported concatenated commands were a renderer defect: adjacent syntax-highlighter spans lost line boundaries in the deployed styles. Literal `pre/code` rendering now preserves the authored newlines and a Copy action writes the exact block. Browser selection and the clipboard write were checked; headless clipboard read permission was unavailable, so an OS-level clipboard readback is not claimed.

The repository driver now brings its selected tab to the foreground so terminal rendering continues. Its native `typekeys` action was verified with punctuation and spaces in actual Wireshark display filters. Terminal commands were submitted one line at a time with Enter. Direct DOM text insertion is unsuitable for the VNC viewer and does not preserve multiline terminal submission.

## Fresh flat-lab walkthrough

Exercises 1 and 3–6 were repeated through Explorer with their actual Linux and RouterOS commands, alongside the packet/application evidence in the initial review. This confirmed tagged/untagged observations, operational RSTP, reciprocal LACP state, bounded service checks, HTTP, read-only Modbus and OPC UA Hello/Acknowledge. No live STP priority change, member-link outage, PLC program change or process-control operation was required or performed.

The Container101 introduction now explains the deployed appliance and the difference between a node terminal and the instructor's appliance console. Its read-only Docker examples were run. Old Codespaces, VS Code forwarding and local deployment instructions were removed. Internal historical segmented-topology analysis is excluded from both published sites and search indexes.

## ARP exercise and both extras

The complete repeat ran on isolated VM223 through Explorer terminals, the osquery panels and embedded Wireshark:

- Before interception, WAGO B mapped `10.10.0.1` to `4c:5e:0c:09:09:11`. WAGO A's data MAC was `00:30:de:10:00:11`; forwarding was `1`.
- Preparing the 300-second receiver before the 120-second helper removed the timing race. During interception, both the live neighbor table and collected **ARP Cache** mapped the gateway to WAGO A. The receiver and Follow TCP Stream showed the same `LAB-ARP-DEMO` marker, 13 bytes including its terminating newline.
- Extra 1 compared forged ARP frame 7482 with corrective frame 14170. Extra 2 was repeated from a clean baseline and fresh capture: direct CtrlX traffic bypassed A, while routed DMZ requests crossed it.
- Automatic expiry and early Ctrl+C cleanup were both exercised. No arpspoof process remained, forwarding stayed `1`, the original gateway mapping returned and gateway pings succeeded. Explorer also showed the recovered mapping at 18:57:53 UTC. Capture and terminal tabs were closed.

The collected ARP table is latest reported state, not a retained incident timeline. It can lag or miss a short event; Last Seen is discovery metadata, not the individual ARP query time. The current profile has no Active Processes panel, and Open Connections does not identify raw ARP spoofing. The lesson uses terminal process/neighbor checks for cleanup and collected data as corroborating evidence.

## Executed VLAN 70 practical

VM223 was a full isolated copy of VM222. Its inherited bridged networking was removed before boot. Restricted QEMU user networking and a host-loopback HTTPS forward provided separate operator access; no data bridge joined it to the source or another lab. Only the copy's guest firewall received the scoped HTTPS allowance. The source stayed running.

The final subnet is `172.30.70.0/24`. The earlier `172.22.70.0/24` proposal was rejected because the actual appliance already has an overlapping Docker network. The practical includes that check rather than assuming RFC1918 space is free.

The final lesson's 18 command blocks match between English and German. They were executed on Hirschmann, sw-acc1, sw-dist and gw:

| Check | Actual result |
| --- | --- |
| Flat baseline | Hirschmann `10.50.0.12/8`; WAGO TCP 22/8080 open, HTTP 200; reverse SSH open |
| Access and trunks | sw-acc1 ether5 PVID 70; VLAN 70 tagged over the logical bond and gateway ether12; gateway bridge CPU included |
| New endpoint path | Hirschmann `172.30.70.12/24`; WAGO routed through `172.30.70.1`; gateway ping 4/4 |
| Allowed application | WAGO TCP 8080 open and HTTP 200; matching `EX7-HTTP` rule/log evidence |
| Forward denial | WAGO TCP 22 filtered; matching `EX7-FROM` SYN denial |
| Extra 2, reverse denial | WAGO → Hirschmann TCP 22 filtered; matching `EX7-TO` evidence |
| Extra 2, gateway input | Hirschmann → gateway TCP 22 filtered; matching `EX7-INPUT`; HTTP still 200 |
| Independent management | Hirschmann Explorer terminal stayed connected through the move and cleanup |
| Manual cleanup | Original addresses and all routes restored, both original SSH paths open, HTTP 200; RouterOS export hashes match baseline |
| Checkpoint reset | Actual `checkpoint`, `start` and `restore` completed; restore automatically restarted all 14 healthy nodes |

Extra 1 matched the same 90-byte HTTP request, TCP sequence `2019501892:2019501982`, across the gateway uplink. VLAN 70 had TTL 64 and Hirschmann→gateway MACs; VLAN 10 had TTL 63 and gateway→WAGO MACs. IP addresses and payload stayed unchanged. Access traffic was untagged; VLAN 70 appeared on the uplink and bond members. Bounded passive capture files remain privately on the copy.

The first manual cleanup exposed a real defect in the draft address order: deleting the interface's last IPv4 address flushed its unrelated multicast routes. Both move and cleanup now add the replacement address first. The final full build/probe/cleanup repeat preserved `224.0.0.0/4` and `255.255.255.255` and restored the exact original route set. No manual route repair was needed in that final run.

After cleanup, comment-header-stripped RouterOS exports matched these original hashes on both copy and untouched source:

| Device | SHA-256 |
| --- | --- |
| sw-acc1 | `db7ec9b6540c55a25e2fff746a6305444ffac638d6152e45696002fc8715934d` |
| sw-dist | `d2fba353c69202e1b0a75b52c785d6b6499847b29eb195356150db0b30ca7151` |
| gw-firewall | `75276c40180155f2050975510067c3a86551c8f381f027d76b6cd71a9e9a1aaf` |

All 14 source lab-node container identities also remained unchanged.

## Specific app gaps and verified recovery

- **Reported VLAN membership:** Actual **Data Collector → Devices → Collect new data** completed and refreshed membership. Reloading Netmap alone does not poll devices. The working collapsed sw-dist–sw-acc1 bond can remain gray in the VLAN view; the lesson uses reported tables and tagged packets to verify that hop.
- **Capture startup:** One viewer decoded a new capture as ERF/LEGACY with invalid times. **Restart Capture** recovered valid Ethernet/VLAN/TCP decoding and the exact lesson filter. A separate viewer container was used, so viewer reuse did not explain the failure. The original bad temporary file was gone after restart; the underlying cause remains unproven. The lesson/appendix give this targeted recovery and escalation if it recurs.
- **Private practical:** The reader provides terminals, views and capture, but does not provision or reset isolated appliances. Instructor guidance covers private access, checkpoint ownership, mandatory cleanup and whole-checkpoint recovery after a restart. Toolbox does not apply device configuration.

The shared storage review rejected an attempted clone-speed increase. The clone completed at its original bounded rate; no rejected action was retried or bypassed.

## Publication and checks

The final course was committed as `c509b53`, following the ARP/osquery update `8a9f0a3`, Container101/search correction `7f82d11` and Exercise 6 clarification `085c7eb`. Course-only activation on VM222 preserved all 22 other container identities. Both active images are healthy and each passed all 38 packaged Markdown-file hashes:

- Controller: `sha256:a7cabc2c5297409913038322e5b85580ab8f27e2aff04a0322cc9e1a7b2163c2`.
- Assistant agent: `sha256:d6e3e83f69fbe1a5035b968add595af5ab88ca63fdd828f9b7d7a07364969f78`.

Inputs and guarded rollback are private on VM222 at `/nw/data/exercise-practical-course-20261018/`; revert this course layer before the earlier reader-codeblock layer. The same final course was published to the isolated copy.

VM223 retains the clean final-course checkpoint `/nw/data/ot-student/checkpoints/ex7-flat-final.tar`, root-owned with mode `0600`. It is powered off to release resources; VM222 remains running. The earlier `ex7-flat` checkpoint was the restore trial and contains the earlier course image, so use `ex7-flat-final` when resuming the finished practical. Continued student access still needs an instructor-assigned URL/login and reset support.

Both existing MkDocs language builds and the complete existing nwos-build pre-commit gate passed. English/German command blocks match, shell syntax checks pass, and every lesson has two extras after its solution. No packages were installed. The installed Material theme emits its existing emoji-extension deprecation warning.

The final deployed reader review passed all 14 English/German pages against committed source hashes. All 110 rendered code blocks preserve literal newlines, including collapsed hints; each lesson has exactly two extras after its solution. All 78 app-link occurrences resolve with the selected network/snapshot and authored STP/VLAN parameters. The real Copy action on the reported German ARP block wrote all three commands with all three line feeds. Browser evidence and hashes are retained locally under `/private/tmp/ot-final-rendered-reader/`.

Final source cleanup found no arpspoof process, forwarding `1`, the original gateway MAC on WAGO B and zero active capture viewers. All owned walkthrough browser/terminal sessions and the isolated-copy tunnel were closed.
