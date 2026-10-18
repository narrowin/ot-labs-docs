# Exercise 7: Design a policy that reaches the traffic

**Time:** 35–45 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Intermediate.

Prove where a flow is forwarded before deciding where to filter it. Build a small, justified communication matrix from your evidence. This lesson inspects the running lab and drafts a design; it does not apply firewall rules to the shared appliance.

**You will learn to:** distinguish switched and routed paths, build a directional service policy, explain stateful replies and rule order, and specify evidence for a future enforcement test.

## Before you start

Use your observations from [Exercise 5](ex5-reconnaissance.md) and [Exercise 6](ex6-protocol-discovery.md). Open terminals for [ABB](/network/terminal?node=abb-800xa-vlan40), the [DMZ jumphost](/network/terminal?node=proxmox-jumphost) and [gw](/network/terminal?node=gw-firewall). Open [VLAN 10 in Netmap](/network/?viewMode=vlan&vlanIds=10).

**Standalone documentation:** Open your assigned LNE, select the student network and live snapshot, then Explorer → Netmap → VLAN view → VLAN 10. Search the stable node names above and use their configured SSH actions. Open Toolbox from the product launcher. Reader links carry the selected network and snapshot automatically.

The baseline OT prefix is `10.0.0.0/8` in VLAN 10. DMZ is `2.2.2.0/24` in VLAN 60. Hostname suffixes do not create VLANs 20–50. Keep your proposed policy in your own notes: Toolbox's saved LNE workspace is shared with other students.

## Task 1: Find the actual enforcement path

On **ABB**:

```bash
ip route get 10.10.0.11
```

On the **jumphost**:

```bash
ip route get 10.10.0.11
```

On **gw**:

```routeros
/interface vlan print
/ip address print
```

**Checkpoint:** ABB's route to WAGO A is direct on `eth1`; the jumphost's route is via `2.2.2.1`. ABB `10.40.0.11/8` and WAGO A `10.10.0.11/8` are on the same OT VLAN in this baseline. A gateway forward-chain rule cannot restrict a flow that stays on that switched path. Gateway-destined traffic uses `input`; routed transit traffic uses `forward`.

## Task 2: Separate the baseline matrix from the desired matrix

Open **Toolbox** from the product launcher. In the **Network segmentation / Netzwerksegmentierung** card, choose **Policies** to open the communication matrix. Inspect the selected network's existing zones and direction of each policy. Return to these instructions in the original tab.

The original **Student Lab Flat Starting Project** has OT Network and DMZ with both directions allowed. A saved design may already contain student edits. Record what is there; do not clear, import over, edit or save the shared project during this observation task.

Draft this **proposed policy** in your own notes, adapting its justifications to what you actually observed:

| Initiator | Destination | Service | Proposed action | Reason / evidence |
| --- | --- | --- | --- | --- |
| DMZ jumphost | WAGO A | TCP 8080 | Allow | Operator WebVisu access, if approved |
| DMZ jumphost | WAGO A | TCP 22 | Deny new initiation | No maintenance need established |
| DMZ jumphost | WAGO A | TCP 502 | Deny new initiation | No DMZ Modbus client requirement established |
| ABB/HMI | WAGO A | TCP 502 | Allow after creating an enforcement boundary | Observed application flow; confirm required read/write functions |
| PLCs | Simulated external network | Any | Deny new initiation unless justified | An exception needs an owner and purpose |

**Checkpoint:** Every allow has a business reason, initiator and destination. The reverse row represents a new connection in the opposite direction, not the reply to an allowed stateful connection. Treat services such as DNS and time synchronization as separately justified dependencies.

## Task 3: Read the rules in their real order

On **gw**:

```routeros
/ip firewall filter print detail where chain=forward
/ip firewall filter print stats where chain=forward
/ip firewall address-list print where list=mgmt
```

Locate the established/related accept, broad DMZ-to-VLAN10 accept, private-source accept and final drop. Use comments and match conditions to identify rules; displayed row numbers can change.

**Checkpoint:** The baseline's final drop is enabled, but earlier accepts still permit the measured DMZ/OT access. The list named `mgmt` includes all of `10.0.0.0/8`, `172.16.0.0/12` and `192.168.0.0/16`; its name does not mean it contains only management hosts. The earlier `action=log` FIN/RST rules observe flags and continue processing; they are not accept verdicts.

??? info "Hint: adding a deny at the bottom would not help"
    Follow the first packet of a new jumphost-to-WAGO TCP connection. Find the first matching terminal verdict. A deny after an earlier broad accept cannot reverse that verdict. Established sessions may continue through the earlier state rule, so future retests must create fresh connections.

## Task 4: Verify the baseline and specify the future retest

On the **jumphost**, repeat only this bounded check:

```bash
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

Write a before/desired-after row for each test. The **current** expected result is open SSH and working HTTP. Under your **proposed** policy, a fresh HTTP connection should still work while a new DMZ SSH connection should be denied. No policy was applied here, so do not report the desired result as measured.

**Checkpoint:** Your retest identifies the source, destination, protocol, expected application result, enforcement device and matching rule/log evidence. Shared counters include other students' traffic; a counter delta alone cannot attribute a packet to your probe. Toolbox is a design tool; saving a matrix does not configure the gateway.

Close the Toolbox and terminal tabs when done.

## Solution

??? success "Compare your reasoning"
    The DMZ-to-OT test crosses gw and is currently broadly allowed. The ABB-to-WAGO application flow stays inside VLAN 10 and needs a real enforcement boundary before a gateway policy can control it. Different IP ranges inside the current `/8` do not supply that boundary.

    A useful design allows documented initiations, permits appropriate established/related replies and denies unapproved new flows at a device the traffic actually traverses. It also protects access to the gateway itself. Neither a final drop nor an attractive matrix proves enforcement without path, ordering and fresh-connection evidence.

## Extra 1: Predict an established reply versus a new connection

**Optional, 10 minutes.** Draw two packet conversations in your notes using the proposed DMZ-to-WAGO HTTP allow:

1. Jumphost ephemeral port → WAGO TCP 8080, followed by the response from 8080 → that ephemeral port.
2. WAGO initiates a separate TCP connection to jumphost port 8080.

Use the existing forward-chain output to identify which kind of rule handles each packet. Explain why permitting conversation 1's reply need not permit conversation 2's new initiation. Compare this with the two directions in Toolbox; inspect without saving changes.

**Checkpoint:** You distinguish connection direction, response traffic and the two independent initiators. You also explain why an already-open session is a poor test of a newly tightened policy. No connection tracking table or live rules need to be cleared.

## Extra 2: Design a complete new VLAN path

**Optional, 15–20 minutes.** Prepare a build sheet for a future isolated configuration lab. Do not apply it to this shared appliance.

Use **VLAN 70**, client **hirschmann-edge1-vlan50**, proposed data address **172.22.70.12/24**, gateway **172.22.70.1**. The existing hostname can remain a label. On [WAGO A](/network/terminal?node=wago-plc2a-vlan10), compare:

```bash
ip route get 10.70.0.12
ip route get 172.22.70.12
```

**Checkpoint:** `10.70.0.12` is direct inside WAGO's existing `/8`; `172.22.70.12` is routed through its gateway. This is why the proposed VLAN needs a non-overlapping subnet.

Complete every row before calling the design connected:

| Component | Required change in the isolated lab |
| --- | --- |
| Endpoint | New `/24` data address and gateway; replace explicit old-gateway routes; preserve management `eth0` |
| sw-acc1 access | `ether5` assigns untagged ingress to PVID 70; VLAN 70 membership includes this access port |
| sw-acc1 uplink | Carry VLAN 70 tagged on `bond-sw-dist`, preserving VLAN 10 |
| sw-dist | Carry VLAN 70 tagged between `bond-sw-acc1` and gateway-facing `ether12` |
| Gateway | Tagged VLAN 70 membership includes `ether12` **and bridge CPU**; a VLAN interface on `bridge` owns `172.22.70.1/24` |
| Firewall and return path | Permit client → WAGO A TCP 8080; deny fresh TCP 22 to the same known-open server. Scope rules to the new interface, retain established replies and protect gateway input before broad accepts |
| Evidence and undo | Untagged access capture, tagged VLAN 70 uplink capture, successful allowed flow, denied new flow, original values and reverse-order restoration |

The current `172.16.0.0/12` private-source accept also covers this proposed subnet. A new VLAN would therefore not automatically isolate it. WAGO A already routes replies to `172.22.70.12` through `10.10.0.1`. Using the DMZ jumphost as a server instead would also require its return route through `2.2.2.1`; its default route uses management. Do not add VLAN 70 to every port or attach individual LACP members to the bridge.

**Deliverable:** Describe the path and the failure symptom if any one row is omitted. A future build should prove local gateway reachability first, then an explicitly allowed cross-zone TCP exchange, then denial of an unapproved fresh connection, followed by restoration. This extension is design analysis; configuration and connectivity of VLAN 70 have not been trialled on the shared lab.

Close tool tabs. You have finished the seven lessons.

## Explore the topic

- [MikroTik bridge VLAN table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): find PVID, tagged ports and CPU access.
- [RouterOS firewall filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/): compare input and forward.
- [RouterOS connection tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/): find established/related behavior.
- [NIST SP 800-82r3](https://csrc.nist.gov/pubs/sp/800/82/r3/final): read network segmentation guidance alongside OT availability requirements.
