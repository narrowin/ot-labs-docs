# Exercise 7: Build and verify a routed VLAN

**Time:** 90–120 minutes, including zone planning. **Lab:** your student `ot-sec-flat` lab. **Difficulty:** Guided practice.

Create one new security zone, carry it through the switches, route it at the gateway and enforce a small policy. Prove the application works and an unapproved connection is blocked, then restore the flat baseline.

**You will learn to:** configure an access VLAN and tagged path, include the gateway's bridge CPU, choose a non-overlapping subnet, order stateful firewall rules and verify both directions with packets and application results.

## Your change request

A temporary diagnostics station must reach one PLC visualization without gaining general access to the cell. Use the Hirschmann-labelled Linux endpoint as the **test client** and WAGO A as the visualization server. These are exercise roles, not a claim that an industrial switch normally acts as an operator workstation.

The owner authorizes the WebVisu service plus a gateway diagnostic. SSH, other new cross-zone services and new connections initiated back towards the diagnostics station are outside this request. First write that intent as a communication matrix, then implement it and prove both service continuity and restriction.

A **zone** groups assets with shared security requirements. A **conduit** is an approved communication path between zones. A VLAN separates Ethernet traffic; routing and firewall policy control this lab's path between zones. **Stateful** filtering tracks existing connections, so replies need not be allowed as unrelated new initiations.

**Critical limit:** Allowing TCP 8080 permits the WebVisu application, including any controls it exposes. It does **not** enforce read-only viewing. Students do not press controls; a real viewing-only requirement also needs application roles or a suitable application-aware access mechanism. This practical qualifies network access only.

## Before you start

Use your own student lab after completing Exercises 1–6 and their cleanup. Its starting state must match Task 1. A flat-baseline checkpoint must be available for recovery; the [setup guide](../lab-practical.md) gives the instructor-console commands.

Open terminals for [Hirschmann](/network/terminal?node=hirschmann-edge1-vlan50), [WAGO A](/network/terminal?node=wago-plc2a-vlan10), [sw-acc1](/network/terminal?node=sw-acc1), [sw-dist](/network/terminal?node=sw-dist) and [gw](/network/terminal?node=gw-firewall). Keep the Hirschmann terminal open: its management connection uses `eth0`, while you change only data interface `eth1`.

Complete the practical in one running session. Linux address/route commands below change the current node; its startup configuration remains flat. If a node restarts during the build, have the instructor restore the checkpoint before continuing.

**Standalone documentation:** Open the assigned practical LNE, select its network and live snapshot, then Explorer → Netmap. Search the stable names above and use their configured SSH actions. For captures select the named switch/interface; for VLAN views select VLAN 10 or 70. Reader links retain the selected network/snapshot. Use the site's lesson navigation when reading outside LNE.

The name `hirschmann-edge1-vlan50` stays unchanged. This exercise creates **one new VLAN 70**; it does not create VLANs 20–50 from hostname suffixes. Toolbox is a design tool; saving a matrix does not apply the commands below.

| State | Hirschmann data address | VLAN | Path to WAGO A | Expected access |
| --- | --- | --- | --- | --- |
| Start | `10.50.0.12/8` | 10 | Direct, switched | HTTP and SSH open |
| After build | `172.30.70.12/24` | 70 | Via `172.30.70.1` on gw | HTTP allowed; fresh SSH blocked |
| After cleanup | `10.50.0.12/8` | 10 | Direct again | Original access restored |

## Plan the plant zones before configuring

**Design task, 20–30 minutes.** Use the course handout's water-treatment diagram and controller allocation (PDF pages 33–34 and 130–145). Page numbers here count PDF pages, because printed slide numbers differ later in the deck. Work individually or discuss your design with a colleague before configuring your own lab.

1. Group assets by operational purpose and consequence of loss or manipulation. Start with treatment/hygiene, pumping/process, disposal, supervision and engineering/remote maintenance. These are candidate groups, not a prescribed answer or assigned security level.
2. Check the **controller boundary**. WAGO PLC2 controls UV/chlorine, mixing and pH functions. Schneider PLC4 controls the first-stage mixer as well as auxiliary/disposal equipment. A VLAN assigned to either PLC's one data interface moves that whole endpoint; it cannot independently separate its physical outputs. Do not draw a firewall between two actuators controlled internally by the same PLC and claim that it enforces the separation.
3. List required communications. Use Exercise 6's observed client/server/function as evidence, then record purpose, initiator, receiver, protocol, owner and consequence of interruption. Mark a relationship from the plant diagram as **to verify** until observed or confirmed by its owner. A flow currently present is not automatically a necessary flow.
4. Identify one dependency that crosses your proposed zone boundaries. Choose whether to regroup the controller with the functions it controls or propose an engineering redesign for later qualification. A TCP-port rule cannot resolve an internal controller dependency or turn Modbus into a read-only protocol.
5. Review your design: which compromise contains a fault without breaking required control? Compare with a colleague if available. Note one remaining risk and the responsible plant owner. This exercise does not assign or certify IEC 62443 security levels.

**Deliverable:** a zone sketch and at least four directional communication rows, including supervisory control, engineering access, supplier access and a local discovery dependency. Use your own notes or the Toolbox zone/communication planner in your lab. In LNE open the product launcher → **Toolbox** → **Policies**; saving a design does not enforce it on devices.

??? success "Check your reasoning"
    A good answer follows the actual controller allocation, preserves justified control relationships and limits maintenance initiation. It does not split UV/chlorine, mixer or pH functions into independently enforced VLANs while leaving them controlled inside the same WAGO endpoint. It also notices Schneider's mixed process/disposal responsibility. Multiple zone designs are defensible if their dependencies, constraints and residual risks are explicit.

The implementation below is **one small, testable conduit** for a temporary diagnostic client. It does not implement your complete plant design. Keeping the process controllers in place lets you practise routing and enforcement without moving a running control dependency.

| Initiator | Receiver | Service | Intended result and reason |
| --- | --- | --- | --- |
| Diagnostics client | WAGO A | TCP 8080 | Allow the approved WebVisu application |
| Diagnostics client | WAGO A | TCP 22 | Deny unapproved SSH initiation |
| WAGO A | Diagnostics client | New TCP 22 | Deny unapproved reverse initiation |
| Diagnostics client | Its gateway | ICMP | Allow the selected reachability diagnostic |
| Diagnostics client | Its gateway | TCP 22 | Deny gateway administration |
| Participants in an allowed connection | Each other | Established/related traffic | Permit tracked replies |

Record an owner and end-of-maintenance expiry for the proposed WebVisu access. In this practical, cleanup implements that expiry. Predict the first matching firewall rule for each row before Task 2.

## Task 1: Record the starting state

On **Hirschmann**:

```bash
ip -br address
ip route
ip route get 10.10.0.11
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

On **WAGO A**, establish that Hirschmann's SSH service is open before adding policy:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 10.50.0.12
ip route get 172.30.70.12
```

On **sw-acc1**:

```routeros
/interface bridge port print detail where interface=ether5
/interface bridge vlan print detail
```

On **sw-dist**:

```routeros
/interface bridge vlan print detail
```

On **gw**:

```routeros
/interface vlan print
/interface bridge vlan print detail
/ip address print
/ip firewall filter print detail
/ip firewall address-list print where list=mgmt
:put [:len [/ip firewall filter find where chain=forward and comment~"^allow HMI"]]
:put [:len [/ip firewall filter find where chain=input and comment~"^allow DHCP"]]
```

**Checkpoint:** Hirschmann has `10.50.0.12/8` on `eth1`, management `192.168.100.59/24` on `eth0`, and default plus explicit `1.0.0.0/8` and `2.2.2.0/24` routes through `10.10.0.1`. It reaches WAGO directly; ports 22/8080 are open and HTTP returns 200. WAGO reaches Hirschmann SSH and routes `172.30.70.12` through `10.10.0.1`.

VLAN/interface 70, subnet `172.30.70.0/24` and comments beginning `ex7-` must be unused. Each of the two final gateway queries must print `1`: these are the insertion anchors for Task 2. sw-acc1 ether5 has PVID 10 and joins VLAN 10 dynamically. Its static VLAN 10 row carries `bond-sw-dist,ether10`; sw-dist carries VLAN 10 on `bond-sw-acc1,ether12,ether8`. If these starting values differ, stop and have the instructor restore the assigned flat checkpoint.

**Policy to implement:** permit Hirschmann → WAGO A TCP 8080 and Hirschmann → its new gateway ICMP; retain established/related replies; deny other new forwarded connections from/to VLAN 70 and other gateway access from VLAN 70. The existing private-source list includes `172.16.0.0/12`, so it would broadly accept this subnet unless your scoped rules run first.

## Task 2: Prepare the gateway and policy

On **gw**, first create the interface so firewall rules can refer to it:

```routeros
/interface vlan add name=vlan70 interface=bridge vlan-id=70 comment=ex7-vlan70
```

Add the rules in this order. The `place-before` expressions identify the first post-state forward/input rules by their baseline comments. Existing established/related accepts stay ahead of your rules; broader ICMP, management and private-source accepts stay behind them.

```routeros
/ip firewall filter add chain=forward action=accept in-interface=vlan70 out-interface=vlan10 src-address=172.30.70.12 dst-address=10.10.0.11 protocol=tcp dst-port=8080 log=yes log-prefix="EX7-HTTP " comment=ex7-http place-before=[find where chain=forward and comment~"^allow HMI"]
/ip firewall filter add chain=forward action=drop in-interface=vlan70 log=yes log-prefix="EX7-FROM " comment=ex7-deny-from place-before=[find where chain=forward and comment~"^allow HMI"]
/ip firewall filter add chain=forward action=drop out-interface=vlan70 log=yes log-prefix="EX7-TO " comment=ex7-deny-to place-before=[find where chain=forward and comment~"^allow HMI"]
/ip firewall filter add chain=input action=accept in-interface=vlan70 src-address=172.30.70.12 dst-address=172.30.70.1 protocol=icmp comment=ex7-gateway-ping place-before=[find where chain=input and comment~"^allow DHCP"]
/ip firewall filter add chain=input action=drop in-interface=vlan70 log=yes log-prefix="EX7-INPUT " comment=ex7-deny-input place-before=[find where chain=input and comment~"^allow DHCP"]
/interface bridge vlan add bridge=bridge vlan-ids=70 tagged=bridge,ether12 comment=ex7-vlan70
/ip address add address=172.30.70.1/24 interface=vlan70 comment=ex7-vlan70
/ip firewall filter print detail
```

**Checkpoint:** Forward order is established/related → `ex7-http` → `ex7-deny-from` → `ex7-deny-to` → earlier baseline policies. Input order is established/related → `ex7-gateway-ping` → `ex7-deny-input` → earlier baseline policies. The new VLAN table row includes **bridge CPU and ether12**. Adding only ether12 would omit the explicitly configured CPU path for routing.

New rule rows can briefly show `I` while the interface starts. Wait a few seconds and repeat the last print command. None of the `ex7-` rows should still carry an `I` flag before you continue; the flags legend itself always explains `I`.

## Task 3: Carry VLAN 70 through both switches

On **sw-dist**:

```routeros
/interface bridge vlan add bridge=bridge vlan-ids=70 tagged=bond-sw-acc1,ether12 comment=ex7-vlan70
/interface bridge vlan print detail where vlan-ids=70
```

On **sw-acc1**:

```routeros
/interface bridge vlan add bridge=bridge vlan-ids=70 tagged=bond-sw-dist untagged=ether5 comment=ex7-vlan70
/interface bridge port set [find where interface=ether5] pvid=70
/interface bridge vlan print detail
```

**Checkpoint:** The new path is Hirschmann → untagged ether5 → tagged `bond-sw-dist` → sw-dist `bond-sw-acc1` → tagged ether12 → gw CPU/vlan70. Changing PVID removes ether5 from the dynamically derived VLAN 10 membership. Existing VLAN 10 tagged rows remain intact. The bond remains one logical bridge port.

## Task 4: Move the endpoint into its new subnet

On **Hirschmann**, remove the old gateway routes before replacing its data address. The new default covers the destinations previously named by the two explicit routes.

```bash
ip route del 1.0.0.0/8 via 10.10.0.1 dev eth1
ip route del 2.2.2.0/24 via 10.10.0.1 dev eth1
ip route del default via 10.10.0.1 dev eth1
ip address add 172.30.70.12/24 dev eth1
ip address del 10.50.0.12/8 dev eth1
ip route add default via 172.30.70.1 dev eth1
ip -br address
ip route get 10.10.0.11
ping -c 4 172.30.70.1
```

**Checkpoint:** `eth0` retains its management address and your terminal stays connected. `eth1` has only the new data address, its `224.0.0.0/4` and `255.255.255.255` routes remain, the WAGO route uses `172.30.70.1`, and the gateway replies. Adding the new address before removing the old one avoids flushing unrelated interface routes. A `10.70.0.0/24` choice would overlap existing OT `/8` return routes; `172.30.70.0/24` avoids that overlap.

## Task 5: Prove allowed HTTP and blocked SSH

[Open the gateway-uplink capture](/network/capture?node=sw-dist&interface=ether12) and choose **Start Capture**.

If the viewer shows ERF/LEGACY rather than Ethernet, choose **Restart Capture** once before generating traffic. Report a recurring failure to the instructor.

Run on **Hirschmann**:

```bash
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

On **gw**:

```routeros
/ip firewall filter print stats where comment~"^ex7-"
/log print where message~"EX7-"
```

Filter the capture with `ip.addr == 172.30.70.12 && (tcp.port == 22 || tcp.port == 8080)`. Match your source/destination and ports with the rule logs. Logs and counters may also contain earlier attempts or background traffic in your lab.

Use the newest log entries from your test period; earlier `EX7-` entries can remain after a previous cleanup.

**Checkpoint:** TCP 8080 is open and HTTP returns 200. TCP 22 is **filtered**, although the server was proved open before the change. Your SSH SYN is visible on VLAN 70, and `ex7-deny-from` records the matching denial. Successful HTTP and gateway ping establish a working path; a timeout by itself would not identify a firewall rule.

Open **Data Collector → Devices → Collect new data**. Wait until collection finishes and the **Last poll** values update, then [inspect VLAN 70 in Netmap](/network/?viewMode=vlan&vlanIds=70). Hirschmann and the path through sw-acc1, sw-dist and gw should now carry VLAN 70. Compare reported membership with the configuration and packets; reloading Netmap alone does not poll the switches.

The current VLAN view can leave the collapsed sw-dist–sw-acc1 bond edge gray even when both switches report VLAN 70. Verify that hop with the bond's VLAN table and tagged packets; the edge color does not establish whether traffic passes.

**Main result complete.** Keep VLAN 70 configured for the optional extras below. Afterwards, perform **Cleanup** even if you skip the extras. Stop the capture before changing its target.

If you pause before the extras, complete cleanup first. When you return, verify the flat baseline in Task 1 and repeat Tasks 2–5 before continuing with the extras. Keep your earlier results for comparison.

## Your change acceptance record

Record the approved source, target and service; the HTTP result; a matching denied SSH attempt and firewall log; and restored baseline evidence after cleanup. Keep the reverse-direction and gateway checks from Extra 2 if you perform them. Do not claim that every possible flow is blocked or that HTTP permission enforces a read-only HMI.

## Solution

??? success "Compare your implementation"
    VLAN 70 needs the access PVID, an untagged client port, tagged membership on both switch uplinks and gateway ether12/CPU, a VLAN interface, a non-overlapping subnet and a working return route. WAGO A sends replies for the new subnet through its existing `10.10.0.1` gateway.

    The HTTP initiation matches `ex7-http`. Replies match the earlier established/related accept. Fresh SSH matches `ex7-deny-from` before the broad private-source accept. The input chain permits the selected gateway diagnostic and blocks other VLAN 70 requests to gw. Existing OT-to-OT traffic remains in VLAN 10.

## Extra 1: Follow one request across the router

**Optional, 10 minutes.** Keep the gateway-uplink capture running and make one more HTTP request from Hirschmann. Filter `ip.src == 172.30.70.12 && tcp.dstport == 8080`.

Find the same request before and after routing: compare TCP sequence number/payload, VLAN ID, Ethernet addresses, IP addresses and TTL.

**Checkpoint:** On this one uplink the request arrives tagged VLAN 70 and leaves tagged VLAN 10. Its IP source/destination stay unchanged, Ethernet addresses change and TTL decreases by one. This is routing without source NAT. A successful ping alone would not show those transformations.

Recording both sides can make Wireshark label the second copy as duplicate/out-of-order. Compare sequence, VLAN and TTL before interpreting those labels as a network fault.

## Extra 2: Prove replies and new initiations differ

**Optional, 10–15 minutes.** On **WAGO A**, test a fresh reverse SSH connection to the listener proved open in Task 1:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.12
```

On **Hirschmann**, test gateway protection, then repeat permitted HTTP:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.1
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Read the `ex7-` rule statistics/logs again. The reverse attempt must match `ex7-deny-to`, gateway SSH must match `ex7-deny-input`, and HTTP must still return 200 through stateful replies. The two SSH results are filtered. No existing sessions or global counters need to be cleared.

## Cleanup: restore the flat baseline

Stop and close the capture. On **Hirschmann**:

```bash
ip route del default via 172.30.70.1 dev eth1
ip address add 10.50.0.12/8 dev eth1
ip address del 172.30.70.12/24 dev eth1
ip route add default via 10.10.0.1 dev eth1
ip route add 1.0.0.0/8 via 10.10.0.1 dev eth1
ip route add 2.2.2.0/24 via 10.10.0.1 dev eth1
```

On **sw-acc1**:

```routeros
/interface bridge port set [find where interface=ether5] pvid=10
/interface bridge vlan remove [find where comment=ex7-vlan70]
```

On **sw-dist**:

```routeros
/interface bridge vlan remove [find where comment=ex7-vlan70]
```

On **gw**:

```routeros
/ip address remove [find where comment=ex7-vlan70]
/interface bridge vlan remove [find where comment=ex7-vlan70]
/interface vlan remove [find where name=vlan70]
/ip firewall filter remove [find where comment~"^ex7-"]
```

Repeat Task 1's address, route, service and switch checks. Hirschmann must again use `10.50.0.12/8`, PVID 10 and a direct WAGO route; HTTP and SSH must be open again. WAGO must again reach Hirschmann SSH at `10.50.0.12`. VLAN/interface 70 and the `ex7-` rules must be absent. Original VLAN 10/60 membership and management connectivity must match your notes.

Run **Collect new data** again to refresh reported port membership. Check absence in the actual RouterOS tables; a VLAN number in a catalog alone is not proof that a port still carries it.

If cleanup differs from the recorded baseline, use the saved checkpoint recovery procedure with the instructor and repeat Task 1. Close all terminal tabs after verification.

## Addon: Microsegment a legacy endpoint with the diode

**Optional, 35–45 minutes.** Use your own lab with the routed `vlan10-diode` and `protected-linux` endpoint. Start from the flat baseline; finish the VLAN 70 cleanup above if you performed it. The other two extras are not prerequisites.

**Your task:** The handout's disposal-system example has an unpatched asset that creates risk for the rest of its zone (PDF pages 149–152, printed slides 158–161). Here, `protected-linux` represents a legacy computer in that system. Keep maintenance access from the Dell engineering workstation, block access from another cell computer, and contain new connections initiated by the legacy computer.

**You will learn:** enforce a boundary around one endpoint, choose a single-address exception, test both directions and verify that an allowed application's replies still work.

The lab uses a **routed diode** with configurable firewall policy. It is not a physical one-way data diode. Its profile filters addresses/subnets rather than individual service ports. Linux is the stand-in for this task; the diode does not patch it or establish an IEC 62443 security level.

### A1: Find the boundary in the GUI

1. Open the product launcher → **Diodes**, then `vlan10-diode`. Record its assigned **Microsegmentation - baseline** profile. Before taking baseline measurements, wait for **Production current** and a **Last confirmed** time under **Configuration Apply Status**. Record the **Config Revision** and compare it with **ASSIGNED REV**.
2. In **Inventory**, inspect the diode interfaces; in **Netmap**, follow the path from sw-acc2 to the diode and protected Linux. The dotted switch link is a saved wiring annotation. Interface and ARP reports provide additional evidence; a line on the drawing alone does not prove filtering.
3. Identify the two sides: WAN `10.10.0.21` faces VLAN 10; LAN `192.168.254.254/24` faces protected Linux `192.168.254.1`. The two clients below both remain in VLAN 10. They connect to **`10.10.0.21`**, which forwards permitted traffic to Linux.

| Role | Terminal | Baseline data address |
| --- | --- | --- |
| Approved engineering workstation | [Dell](/network/terminal?node=dell-ews1-vlan50) | `10.50.0.11/8` |
| Other cell computer | [Hirschmann](/network/terminal?node=hirschmann-edge1-vlan50) | `10.50.0.12/8` |
| Legacy endpoint stand-in | [Protected Linux](/network/terminal?node=protected-linux) | `192.168.254.1/24` |

Use these configured management terminals throughout. An open management terminal is a recovery path, not proof that the diode's data policy permits a connection. Do not use management addresses to test the policy.

### A2: Establish the application baseline

On **Dell**, then **Hirschmann**, record the actual address and selected route:

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

On **Protected Linux**, establish an outgoing application check using WAGO A:

```bash
curl --interface eth1 --connect-timeout 3 --max-time 5 -sS -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

**Checkpoint:** HTTP returns `200`. In **Diodes → Profiles → Microsegmentation - baseline**, record both policy directions: **From Network** and **From Endpoint** are **Default allow**, with no address exceptions. Preserve the profile's other settings. If this starting state differs, reconcile your previous work before continuing; a missing service cannot demonstrate a newly enforced block.

### A3: Permit only the engineering workstation

1. Open the assigned profile and choose **Edit**.
2. Under **From Network**, select **Default deny**. Choose **Add allowed source**, enter Dell's observed data address and select **/32**. In this baseline the entry is `10.50.0.11/32`; `/32` means one IPv4 address. Leave **From Endpoint** at **Default allow** for now.
3. Choose **Save**, then confirm **Save & apply**. Return to the diode details and wait for the changed configuration to be confirmed/in sync. Saving a profile is not yet proof that the device applied it.
4. Predict each client's result, then repeat both fresh HTTP and TCP 22 commands from A2.

**Checkpoint:** Dell still reaches HTTP and TCP 22. Hirschmann reaches neither. The clients did not need different VLANs; their access differs because the diode enforces policy at the protected endpoint's boundary. A failed request alone is insufficient: compare it with the permitted client's success and the confirmed configuration.

Why not `10.50.0.0/24`? It includes both clients and would admit Hirschmann too. Why not call this a user allowlist? An IP address identifies a source for this rule; it does not authenticate the person or make a compromised allowed workstation trustworthy. HTTP and SSH follow the same source policy; this profile cannot express “HTTP allowed, SSH denied” for that same source.

### A4: Contain new connections from the legacy computer

The slide's risk also runs in the other direction: an unpatched computer could initiate connections to the rest of the plant.

1. Edit the profile again. Keep **From Network** at **Default deny** with Dell's `/32` exception. Set **From Endpoint** to **Default deny**, with no allowed destinations. Save, confirm **Save & apply**, and wait for configuration confirmation.
2. Repeat the outgoing WAGO HTTP command on **Protected Linux**. Then repeat Dell's HTTP and TCP 22 checks.

**Checkpoint:** The new Protected Linux → WAGO request fails, while new Dell → Protected Linux requests and their replies still work. This separates a new connection initiated by the endpoint from replies belonging to an allowed incoming connection. Do not claim that the hardware passes packets in only one direction.

For one final isolation check, remove Dell's **allowed source** while keeping both directions at **Default deny**. Apply and wait for confirmation. Fresh HTTP and TCP 22 checks from both clients must now fail. The managed Linux terminal remains available through its separate management path.

### A5: Can the diode's VLAN 10 traffic still be intercepted?

Compare this boundary with Exercise 2. On **Protected Linux**, inspect the route back to Dell and the local gateway mapping:

```bash
ip route get 10.50.0.11
ip neigh show 192.168.254.254 dev eth1
```

The route uses `192.168.254.254`. Dell, in contrast, reaches the diode's WAN address `10.10.0.21` directly on VLAN 10, as recorded in A2. An empty neighbor row means no entry is currently cached; it does not prove protection.

Before opening the solution, answer:

1. Why can forged ARP replies from VLAN 10 not directly replace Protected Linux's local gateway mapping?
2. While Dell is allowed in A3, which ARP entries on Dell and on the diode's VLAN 10 side could an attacker poison to intercept their exchange?
3. Would relaying Dell's packets with their original source IP violate the allowlist? Would intercepting SSH traffic make its encrypted contents readable?

??? success "Solution: separate the ARP boundary from the application path"
    **Direct poisoning from VLAN 10: no.** Protected Linux and VLAN 10 are on different Ethernet segments. This routed diode does not forward VLAN 10 ARP frames to its LAN. Linux resolves its local gateway `192.168.254.254`, not the external gateway used in Exercise 2. The protection comes from the routing boundary, not from the `/32` rule.

    **The diode's VLAN 10 traffic can still be intercepted.** Its WAN interface shares VLAN 10 with Dell and a possible attacker. Poisoning the diode's ARP entry for Dell (`10.50.0.11`) can redirect its replies to the attacker. Poisoning Dell's entry for the diode (`10.10.0.21`) can redirect requests; relaying the exchange can put the attacker between Dell and the diode. These forged ARP replies stay on VLAN 10: they do not need to reach the protected endpoint or change its ARP cache.

    **The source-IP policy is not ARP protection.** If the relayed IP source remains Dell's allowed address, the source-IP rule still matches. The permitted HTTP exchange can be read on this path. Intercepted SSH packets remain encrypted when SSH authentication and host-key verification are correct; see [SSH host-key verification](https://www.rfc-editor.org/rfc/rfc4251.html#section-9.3.4). ARP spoofing alone does not turn a blocked source into an allowed one; the remaining exposure is interception or disruption of traffic on the diode's VLAN 10 side.

    Choose a control for the remaining risk: switch ARP validation with maintained IP/MAC bindings on VLAN 10, and authenticated encryption such as HTTPS or SSH with certificate/host-key verification. Encryption protects application data; it does not prevent an attacker from interrupting the connection. See [RFC 826](https://www.rfc-editor.org/rfc/rfc826.html) and [the ARP inspection explanation](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9400/software/release/17-13/configuration_guide/sec/b_1713_sec_9400_cg/configuring_dynamic_arp_inspection.html).

This is a topology and threat-analysis task. The route and neighbor checks establish the two sides; they do not demonstrate a successful interception. Record whose ARP entry would be targeted and which traffic could be exposed.

### A6: Restore and hand over

Restore the recorded baseline profile: **From Network = Default allow**, **From Endpoint = Default allow**, no exceptions in either direction. Save, confirm **Save & apply**, and wait for confirmation. Repeat both clients' HTTP/TCP 22 checks and the protected endpoint's WAGO HTTP check. All must work again. Keep profile assignment, DNS, NTP and logging settings as recorded in A2.

Record this evidence table with actual results and times:

| Policy state | Dell → endpoint HTTP / TCP 22 | Hirschmann → endpoint HTTP / TCP 22 | Endpoint → WAGO HTTP |
| --- | --- | --- | --- |
| Baseline | Works / works | Works / works | HTTP 200 |
| Dell-only incoming | Works / works | Blocked / blocked | HTTP 200 |
| Dell-only incoming; no new outgoing | Works / works | Blocked / blocked | Blocked |
| No incoming exceptions; no new outgoing | Blocked / blocked | Blocked / blocked | Blocked |
| Restored baseline | Works / works | Works / works | HTTP 200 |

Each command creates a fresh connection. An existing SSH session can retain connection-tracking state and is unsuitable evidence for a newly applied block. If a result differs, record the source address, target, time, applied revision and actual error before changing more settings.

??? success "Addon solution and operational conclusion"
    The diode places an enforceable boundary around the legacy endpoint while the rest of the cell keeps its existing VLANs. The Dell `/32` exception preserves the chosen maintenance path. Restricting new endpoint-initiated connections reduces one route for lateral movement. Stateful replies allow the permitted incoming application to keep working. Removing the exception demonstrates containment, and restoring the recorded profile proves recovery.

    The protection depends on the traffic passing through this boundary. A separate unfiltered interface, a compromised allowed source or forged address identity can undermine the assumptions. The lab's management path is a deliberate platform exception. A production design must account for all paths, required process dependencies and real application authorization. The evidence demonstrates this lab policy, not immunity from exploitation or a certified security level.

## Explore the topic

- [MikroTik bridge VLAN table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): trace PVID, tagged ports and CPU access.
- [RouterOS firewall filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/): compare input and forward.
- [RouterOS connection tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/): explain why the earlier state rule permits replies.
- [NIST SP 800-82r3](https://csrc.nist.gov/pubs/sp/800/82/r3/final): relate enforced boundaries to OT availability.
