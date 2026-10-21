# Exercise 8: Microsegment a legacy endpoint with the diode

**35–45 minutes.** Use your own lab with `vlan10-diode` and `protected-linux`. Start from the flat baseline. Exercise 7 is not a prerequisite; if you performed it, finish its [VLAN 70 cleanup](ex7-segmentation-firewall.md#cleanup-restore-the-flat-baseline) first. Complete this exercise's cleanup even if you skip the two optional extras.

**Your task:** The handout's disposal-system example has an unpatched asset that creates risk for the rest of its zone (PDF pages 149–152, printed slides 158–161). Here, `protected-linux` represents a legacy computer in that system. Keep maintenance access from the `dell-ews1` engineering workstation, block access from another cell computer, and contain new connections initiated by the legacy computer.

**You will learn:** enforce a boundary around one endpoint, choose a single-address exception, test both directions and verify that an allowed application's replies still work.

The lab uses a **routed diode** with configurable firewall policy. It is not a physical one-way data diode. Its profile filters addresses/subnets rather than individual service ports. Linux is the stand-in for this task; the diode does not patch it or establish an IEC 62443 security level.

## Task 1: Find the boundary in the GUI

1. Open the product launcher → **Diodes**, then `vlan10-diode`. Record its assigned **Microsegmentation - baseline** profile. Before taking baseline measurements, wait for **Production current** and a **Last confirmed** time under **Configuration Apply Status**. Record the **Config Revision** and compare it with **ASSIGNED REV**.
2. In **Inventory**, inspect the diode interfaces; in **Netmap**, follow the path from sw-acc2 to the diode and `protected-linux`. The dotted switch link is a saved wiring annotation. Interface and ARP reports provide additional evidence; a line on the drawing alone does not prove filtering.
3. Identify the two sides: WAN `10.10.0.21` faces VLAN 10; LAN `192.168.254.254/24` faces `protected-linux` `192.168.254.1`. The two clients below both remain in VLAN 10. They connect to **`10.10.0.21`**, which forwards permitted traffic to Linux.

| Role | Terminal | Baseline data address |
| --- | --- | --- |
| Approved engineering workstation | [`dell-ews1`](/network/terminal?node=dell-ews1-vlan50) | `10.50.0.11/8` |
| Other cell computer | [`hirschmann-edge1`](/network/terminal?node=hirschmann-edge1-vlan50) | `10.50.0.12/8` |
| Legacy endpoint stand-in | [`protected-linux`](/network/terminal?node=protected-linux) | `192.168.254.1/24` |

Use these configured management terminals throughout. An open management terminal is a recovery path, not proof that the diode's data policy permits a connection. Do not use management addresses to test the policy.

## Task 2: Establish the application baseline

On **`dell-ews1`**, then **`hirschmann-edge1`**, record the actual address and selected route:

```bash
ip -4 -br address show dev eth1
ip route get 10.10.0.21
```

On **each client**, run:

```bash
curl --interface eth1 --connect-timeout 3 --max-time 5 -sS http://10.10.0.21/
nc -vz -w 3 10.10.0.21 22
```

Both must receive the **Diode protected Linux endpoint** page and establish the TCP 22 connection. `nc` checks the connection; it does not authenticate an SSH user.

On **`protected-linux`**, establish an outgoing application check using `wago-plc2a`:

```bash
curl --interface eth1 --connect-timeout 3 --max-time 5 -sS -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

**Checkpoint:** HTTP returns `200`. In **Diodes → Profiles → Microsegmentation - baseline**, record both policy directions: **From Network** and **From Endpoint** are **Default allow**, with no address exceptions. Preserve the profile's other settings. If this starting state differs, reconcile your previous work before continuing; a missing service cannot demonstrate a newly enforced block.

## Task 3: Permit only the engineering workstation

1. Open the assigned profile and choose **Edit**.
2. Under **From Network**, select **Default deny**. Choose **Add allowed source**, enter `dell-ews1`'s observed data address and select **/32**. In this baseline the entry is `10.50.0.11/32`; `/32` means one IPv4 address. Leave **From Endpoint** at **Default allow** for now.
3. Choose **Save**, then confirm **Save & apply**. Return to the diode details and wait for the changed configuration to be confirmed/in sync. Saving a profile is not yet proof that the device applied it.
4. Predict each client's result, then repeat both fresh HTTP and TCP 22 commands from Task 2.

**Checkpoint:** `dell-ews1` still reaches HTTP and TCP 22. `hirschmann-edge1` reaches neither. The clients did not need different VLANs; their access differs because the diode enforces policy at the protected endpoint's boundary. A failed request alone is insufficient: compare it with the permitted client's success and the confirmed configuration.

Why not `10.50.0.0/24`? It includes both clients and would admit `hirschmann-edge1` too. Why not call this a user allowlist? An IP address identifies a source for this rule; it does not authenticate the person or make a compromised allowed workstation trustworthy. HTTP and SSH follow the same source policy; this profile cannot express “HTTP allowed, SSH denied” for that same source.

## Task 4: Contain new connections from the legacy computer

The slide's risk also runs in the other direction: an unpatched computer could initiate connections to the rest of the plant.

1. Edit the profile again. Keep **From Network** at **Default deny** with `dell-ews1`'s `/32` exception. Set **From Endpoint** to **Default deny**, with no allowed destinations. Save, confirm **Save & apply**, and wait for configuration confirmation.
2. Repeat the outgoing HTTP command to `wago-plc2a` on **`protected-linux`**. Then repeat `dell-ews1`'s HTTP and TCP 22 checks.

**Checkpoint:** The new `protected-linux` → `wago-plc2a` request fails, while new `dell-ews1` → `protected-linux` requests and their replies still work. This separates a new connection initiated by the endpoint from replies belonging to an allowed incoming connection. Do not claim that the hardware passes packets in only one direction.

## Solution

??? success "Solution and operational conclusion"
    The diode places an enforceable boundary around the legacy endpoint while the rest of the cell keeps its existing VLANs. The `dell-ews1` `/32` exception preserves the chosen maintenance path. Restricting new endpoint-initiated connections reduces one route for lateral movement. Stateful replies allow the permitted incoming application to keep working. Removing the exception demonstrates containment, and restoring the recorded profile proves recovery.

    The protection depends on the traffic passing through this boundary. A separate unfiltered interface, a compromised allowed source or forged address identity can undermine the assumptions. The lab's management path is a deliberate platform exception. A production design must account for all paths, required process dependencies and real application authorization. The evidence demonstrates this lab policy, not immunity from exploitation or a certified security level.

## Extra 1: Can the diode's VLAN 10 traffic still be intercepted?

**Optional, 10 minutes.**

Compare this boundary with Exercise 2. On **`protected-linux`**, inspect the route back to `dell-ews1` and the local gateway mapping:

```bash
ip route get 10.50.0.11
ip neigh show 192.168.254.254 dev eth1
```

The route uses `192.168.254.254`. `dell-ews1`, in contrast, reaches the diode's WAN address `10.10.0.21` directly on VLAN 10, as recorded in Task 2. An empty neighbor row means no entry is currently cached; it does not prove protection.

Before opening the solution, answer:

1. Why can forged ARP replies from VLAN 10 not directly replace `protected-linux`'s local gateway mapping?
2. While `dell-ews1` is allowed in Task 3, which ARP entries on `dell-ews1` and on the diode's VLAN 10 side could an attacker poison to intercept their exchange?
3. Would relaying `dell-ews1`'s packets with their original source IP violate the allowlist? Would intercepting SSH traffic make its encrypted contents readable?

??? success "Solution: separate the ARP boundary from the application path"
    **Direct poisoning from VLAN 10: no.** `protected-linux` and VLAN 10 are on different Ethernet segments. This routed diode does not forward VLAN 10 ARP frames to its LAN. Linux resolves its local gateway `192.168.254.254`, not the external gateway used in Exercise 2. The protection comes from the routing boundary, not from the `/32` rule.

    **The diode's VLAN 10 traffic can still be intercepted.** Its WAN interface shares VLAN 10 with `dell-ews1` and a possible attacker. Poisoning the diode's ARP entry for `dell-ews1` (`10.50.0.11`) can redirect its replies to the attacker. Poisoning `dell-ews1`'s entry for the diode (`10.10.0.21`) can redirect requests; relaying the exchange can put the attacker between `dell-ews1` and the diode. These forged ARP replies stay on VLAN 10: they do not need to reach the protected endpoint or change its ARP cache.

    **The source-IP policy is not ARP protection.** If the relayed IP source remains `dell-ews1`'s allowed address, the source-IP rule still matches. The permitted HTTP exchange can be read on this path. Intercepted SSH packets remain encrypted when SSH authentication and host-key verification are correct; see [SSH host-key verification](https://www.rfc-editor.org/rfc/rfc4251.html#section-9.3.4). ARP spoofing alone does not turn a blocked source into an allowed one; the remaining exposure is interception or disruption of traffic on the diode's VLAN 10 side.

    Choose a control for the remaining risk: switch ARP validation with maintained IP/MAC bindings on VLAN 10, and authenticated encryption such as HTTPS or SSH with certificate/host-key verification. Encryption protects application data; it does not prevent an attacker from interrupting the connection. See [RFC 826](https://www.rfc-editor.org/rfc/rfc826.html) and [the ARP inspection explanation](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9400/software/release/17-13/configuration_guide/sec/b_1713_sec_9400_cg/configuring_dynamic_arp_inspection.html).

This is a topology and threat-analysis task. The route and neighbor checks establish the two sides; they do not demonstrate a successful interception. Record whose ARP entry would be targeted and which traffic could be exposed.

## Extra 2: Isolate the endpoint and verify recovery

**Optional, 10 minutes.** Start with the policy from Task 4. Prove that removing the last incoming exception isolates the endpoint's data path while preserving the management recovery path.

For one final isolation check, remove `dell-ews1`'s **allowed source** while keeping both directions at **Default deny**. Apply and wait for confirmation. Fresh HTTP and TCP 22 checks from both clients must now fail. The managed Linux terminal remains available through its separate management path.

Record both clients' results in the **Extra 2** row below. Then perform the cleanup and verify that all three application checks work again.

??? success "Expected result"
    Both clients' fresh HTTP and TCP 22 checks fail. The separate managed terminal remains reachable; it is not evidence that data access is still allowed. Restoring the baseline must restore both clients' access and the endpoint's outgoing HTTP request.

## Cleanup: restore the baseline and hand over

Restore the recorded baseline profile: **From Network = Default allow**, **From Endpoint = Default allow**, no exceptions in either direction. Save, confirm **Save & apply**, and wait for confirmation. Repeat both clients' HTTP/TCP 22 checks and the protected endpoint's HTTP check to `wago-plc2a`. All must work again. Keep profile assignment, DNS, NTP and logging settings as recorded in Task 2.

Record this evidence table with actual results and times:

| Policy state | `dell-ews1` → `protected-linux` HTTP / TCP 22 | `hirschmann-edge1` → `protected-linux` HTTP / TCP 22 | `protected-linux` → `wago-plc2a` HTTP |
| --- | --- | --- | --- |
| Baseline | Works / works | Works / works | HTTP 200 |
| `dell-ews1`-only incoming | Works / works | Blocked / blocked | HTTP 200 |
| `dell-ews1`-only incoming; no new outgoing | Works / works | Blocked / blocked | Blocked |
| Extra 2: no incoming exceptions; no new outgoing | Blocked / blocked | Blocked / blocked | Blocked |
| Restored baseline | Works / works | Works / works | HTTP 200 |

Each command creates a fresh connection. An existing SSH session can retain connection-tracking state and is unsuitable evidence for a newly applied block. If a result differs, record the source address, target, time, applied revision and actual error before changing more settings.

Close your terminal tabs. Return to the [exercise overview](index.md).
