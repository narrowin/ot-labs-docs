# Exercise 7: Build and verify a routed VLAN

**Time:** 90–120 minutes, including zone planning. **Lab:** your student `ot-sec-flat` lab. **Difficulty:** Guided practice.

Create one new security zone, carry it through the switches, route it at the gateway and enforce a small policy. Prove the application works and an unapproved connection is blocked, then restore the flat baseline.

**You will learn to:** configure an access VLAN and tagged path, include the gateway's bridge CPU, choose a non-overlapping subnet, order stateful firewall rules and verify both directions with packets and application results.

## Your change request

A temporary diagnostics station must reach one PLC visualization without gaining general access to the cell. Use `hirschmann-edge1` as the **test client** and `wago-plc2a` as the visualization server. The existing lab node `hirschmann-edge1` runs Linux and supplies the diagnostic client for this task.

The owner authorizes the WebVisu service plus a gateway diagnostic. SSH, other new cross-zone services and new connections initiated back towards the diagnostics station are outside this request. First write that intent as a communication matrix, then implement it and prove both service continuity and restriction.

A **zone** groups assets with shared security requirements. A **conduit** is an approved communication path between zones. A VLAN separates Ethernet traffic; routing and firewall policy control this lab's path between zones. **Stateful** filtering tracks existing connections, so replies need not be allowed as unrelated new initiations.

**Critical limit:** Allowing TCP 8080 permits the WebVisu application, including any controls it exposes. It does **not** enforce read-only viewing. Students do not press controls; a real viewing-only requirement also needs application roles or a suitable application-aware access mechanism. This practical qualifies network access only.

## Before you start

Use your own student lab after completing Exercises 1–6 and their cleanup. Its starting state must match Task 1. A flat-baseline checkpoint must be available for recovery; the [setup guide](../lab-practical.md) gives the instructor-console commands.

Open terminals for [`hirschmann-edge1`](/network/terminal?node=hirschmann-edge1-vlan50), [`wago-plc2a`](/network/terminal?node=wago-plc2a-vlan10), [sw-acc1](/network/terminal?node=sw-acc1), [sw-dist](/network/terminal?node=sw-dist) and [`gw-firewall`](/network/terminal?node=gw-firewall). Keep the `hirschmann-edge1` terminal open: its management connection uses `eth0`, while you change only data interface `eth1`.

Complete the practical in one running session. Linux address/route commands below change the current node; its startup configuration remains flat. If a node restarts during the build, have the instructor restore the checkpoint before continuing.

**Standalone documentation:** Open the assigned practical LNE, select its network and live snapshot, then Explorer → Netmap. Search the stable names above and use their configured SSH actions. For captures select the named switch/interface; for VLAN views select VLAN 10 or 70. Reader links retain the selected network/snapshot. Use the site's lesson navigation when reading outside LNE.

The name `hirschmann-edge1` stays unchanged. You will move this test client from VLAN 10 to **one new VLAN 70**. Toolbox is a design tool; saving a matrix does not apply the commands below.

| State | `hirschmann-edge1` data address | VLAN | Path to `wago-plc2a` | Expected access |
| --- | --- | --- | --- | --- |
| Start | `10.50.0.12/8` | 10 | Direct, switched | HTTP and SSH open |
| After build | `172.30.70.12/24` | 70 | Via `172.30.70.1` on `gw-firewall` | HTTP allowed; fresh SSH blocked |
| After cleanup | `10.50.0.12/8` | 10 | Direct again | Original access restored |

## Plan the plant zones before configuring

**Design task, 20–30 minutes.** Use the course handout's water-treatment diagram and controller allocation (PDF pages 33–34 and 130–145). Page numbers here count PDF pages, because printed slide numbers differ later in the deck. Work individually or discuss your design with a colleague before configuring your own lab.

1. Group assets by operational purpose and consequence of loss or manipulation. Start with treatment/hygiene, pumping/process, disposal, supervision and engineering/remote maintenance. These are candidate groups, not a prescribed answer or assigned security level.
2. Check the **controller boundary**. Keep `wago-plc2a` (PLC2A) and `wago-plc2b` (PLC2B) as separate endpoints in your asset list and communication rows. Check their treatment functions against the handout. `schneider-plc4` (PLC4) controls the first-stage mixer as well as auxiliary/disposal equipment. A VLAN assigned to a PLC's data interface moves that whole endpoint; it cannot independently separate its physical outputs. Do not draw a firewall between two actuators controlled internally by the same PLC and claim that it enforces the separation.
3. List required communications. Use Exercise 6's observed client/server/function as evidence, then record purpose, initiator, receiver, protocol, owner and consequence of interruption. Mark a relationship from the plant diagram as **to verify** until observed or confirmed by its owner. A flow currently present is not automatically a necessary flow.
4. Identify one dependency that crosses your proposed zone boundaries. Choose whether to regroup the controller with the functions it controls or propose an engineering redesign for later qualification. A TCP-port rule cannot resolve an internal controller dependency or turn Modbus into a read-only protocol.
5. Review your design: which compromise contains a fault without breaking required control? Compare with a colleague if available. Note one remaining risk and the responsible plant owner. This exercise does not assign or certify IEC 62443 security levels.

**Deliverable:** a zone sketch and at least four directional communication rows, including supervisory control, engineering access, supplier access and a local discovery dependency. Use your own notes or the Toolbox zone/communication planner in your lab. In LNE open the product launcher → **Toolbox** → **Policies**; saving a design does not enforce it on devices.

??? success "Check your reasoning"
    A good answer follows the actual controller allocation, preserves justified control relationships and limits maintenance initiation. It does not split UV/chlorine, mixer or pH functions into independently enforced VLANs while leaving them controlled inside a single controller. It also notices `schneider-plc4`'s mixed process/disposal responsibility. Multiple zone designs are defensible if their dependencies, constraints and residual risks are explicit.

The implementation below is **one small, testable conduit** for a temporary diagnostic client. It does not implement your complete plant design. Keeping the process controllers in place lets you practise routing and enforcement without moving a running control dependency.

| Initiator | Receiver | Service | Intended result and reason |
| --- | --- | --- | --- |
| `hirschmann-edge1` | `wago-plc2a` | TCP 8080 | Allow the approved WebVisu application |
| `hirschmann-edge1` | `wago-plc2a` | TCP 22 | Deny unapproved SSH initiation |
| `wago-plc2a` | `hirschmann-edge1` | New TCP 22 | Deny unapproved reverse initiation |
| `hirschmann-edge1` | `gw-firewall` | ICMP | Allow the selected reachability diagnostic |
| `hirschmann-edge1` | `gw-firewall` | TCP 22 | Deny gateway administration |
| Participants in an allowed connection | Each other | Established/related traffic | Permit tracked replies |

Record an owner and end-of-maintenance expiry for the proposed WebVisu access. In this practical, cleanup implements that expiry. Predict the first matching firewall rule for each row before Task 2.

## Task 1: Record the starting state

On **`hirschmann-edge1`**:

```bash
ip -br address
ip route
ip route get 10.10.0.11
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

On **`wago-plc2a`**, establish that `hirschmann-edge1`'s SSH service is open before adding policy:

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

On **`gw-firewall`**:

```routeros
/interface vlan print
/interface bridge vlan print detail
/ip address print
/ip firewall filter print detail
/ip firewall address-list print where list=mgmt
:put [:len [/ip firewall filter find where chain=forward and comment~"^allow HMI"]]
:put [:len [/ip firewall filter find where chain=input and comment~"^allow DHCP"]]
```

**Checkpoint:** `hirschmann-edge1` has `10.50.0.12/8` on `eth1`, management `192.168.100.59/24` on `eth0`, and default plus explicit `1.0.0.0/8` and `2.2.2.0/24` routes through `10.10.0.1`. It reaches `wago-plc2a` directly; ports 22/8080 are open and HTTP returns 200. `wago-plc2a` reaches `hirschmann-edge1` SSH and routes `172.30.70.12` through `10.10.0.1`.

VLAN/interface 70, subnet `172.30.70.0/24` and comments beginning `ex7-` must be unused. Each of the two final gateway queries must print `1`: these are the insertion anchors for Task 2. sw-acc1 ether5 has PVID 10 and joins VLAN 10 dynamically. Its static VLAN 10 row carries `bond-sw-dist,ether10`; sw-dist carries VLAN 10 on `bond-sw-acc1,ether12,ether8`. If these starting values differ, stop and have the instructor restore the assigned flat checkpoint.

**Policy to implement:** permit `hirschmann-edge1` → `wago-plc2a` TCP 8080 and `hirschmann-edge1` → its new gateway ICMP; retain established/related replies; deny other new forwarded connections from/to VLAN 70 and other gateway access from VLAN 70. The existing private-source list includes `172.16.0.0/12`, so it would broadly accept this subnet unless your scoped rules run first.

## Task 2: Prepare the gateway and policy

On **`gw-firewall`**, first create the interface so firewall rules can refer to it:

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

**Checkpoint:** The new path is `hirschmann-edge1` → untagged ether5 → tagged `bond-sw-dist` → sw-dist `bond-sw-acc1` → tagged ether12 → `gw-firewall` CPU/vlan70. Changing PVID removes ether5 from the dynamically derived VLAN 10 membership. Existing VLAN 10 tagged rows remain intact. The bond remains one logical bridge port.

## Task 4: Move the endpoint into its new subnet

On **`hirschmann-edge1`**, remove the old gateway routes before replacing its data address. The new default covers the destinations previously named by the two explicit routes.

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

**Checkpoint:** `eth0` retains its management address and your terminal stays connected. `eth1` has only the new data address, its `224.0.0.0/4` and `255.255.255.255` routes remain, the `wago-plc2a` route uses `172.30.70.1`, and the gateway replies. Adding the new address before removing the old one avoids flushing unrelated interface routes. A `10.70.0.0/24` choice would overlap existing OT `/8` return routes; `172.30.70.0/24` avoids that overlap.

## Task 5: Prove allowed HTTP and blocked SSH

[Open the gateway-uplink capture](/network/capture?node=sw-dist&interface=ether12) and choose **Start Capture**.

If the viewer shows ERF/LEGACY rather than Ethernet, choose **Restart Capture** once before generating traffic. Report a recurring failure to the instructor.

Run on **`hirschmann-edge1`**:

```bash
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

On **`gw-firewall`**:

```routeros
/ip firewall filter print stats where comment~"^ex7-"
/log print where message~"EX7-"
```

Filter the capture with `ip.addr == 172.30.70.12 && (tcp.port == 22 || tcp.port == 8080)`. Match your source/destination and ports with the rule logs. Logs and counters may also contain earlier attempts or background traffic in your lab.

Use the newest log entries from your test period; earlier `EX7-` entries can remain after a previous cleanup.

**Checkpoint:** TCP 8080 is open and HTTP returns 200. TCP 22 is **filtered**, although the server was proved open before the change. Your SSH SYN is visible on VLAN 70, and `ex7-deny-from` records the matching denial. Successful HTTP and gateway ping establish a working path; a timeout by itself would not identify a firewall rule.

Open **Data Collector → Devices → Collect new data**. Wait until collection finishes and the **Last poll** values update, then [inspect VLAN 70 in Netmap](/network/?viewMode=vlan&vlanIds=70). `hirschmann-edge1` and the path through sw-acc1, sw-dist and `gw-firewall` should now carry VLAN 70. Compare reported membership with the configuration and packets; reloading Netmap alone does not poll the switches.

The current VLAN view can leave the collapsed sw-dist–sw-acc1 bond edge gray even when both switches report VLAN 70. Verify that hop with the bond's VLAN table and tagged packets; the edge color does not establish whether traffic passes.

**Main result complete.** Keep VLAN 70 configured for the optional extras below. Afterwards, perform **Cleanup** even if you skip the extras. Stop the capture before changing its target.

If you pause before the extras, complete cleanup first. When you return, verify the flat baseline in Task 1 and repeat Tasks 2–5 before continuing with the extras. Keep your earlier results for comparison.

## Your change acceptance record

Record the approved source, target and service; the HTTP result; a matching denied SSH attempt and firewall log; and restored baseline evidence after cleanup. Keep the reverse-direction and gateway checks from Extra 2 if you perform them. Do not claim that every possible flow is blocked or that HTTP permission enforces a read-only HMI.

## Solution

??? success "Compare your implementation"
    VLAN 70 needs the access PVID, an untagged client port, tagged membership on both switch uplinks and gateway ether12/CPU, a VLAN interface, a non-overlapping subnet and a working return route. `wago-plc2a` sends replies for the new subnet through its existing `10.10.0.1` gateway.

    The HTTP initiation matches `ex7-http`. Replies match the earlier established/related accept. Fresh SSH matches `ex7-deny-from` before the broad private-source accept. The input chain permits the selected gateway diagnostic and blocks other VLAN 70 requests to `gw-firewall`. Existing OT-to-OT traffic remains in VLAN 10.

## Extra 1: Follow one request across the router

**Optional, 10 minutes.** Keep the gateway-uplink capture running and make one more HTTP request from `hirschmann-edge1`. Filter `ip.src == 172.30.70.12 && tcp.dstport == 8080`.

Find the same request before and after routing: compare TCP sequence number/payload, VLAN ID, Ethernet addresses, IP addresses and TTL.

**Checkpoint:** On this one uplink the request arrives tagged VLAN 70 and leaves tagged VLAN 10. Its IP source/destination stay unchanged, Ethernet addresses change and TTL decreases by one. This is routing without source NAT. A successful ping alone would not show those transformations.

Recording both sides can make Wireshark label the second copy as duplicate/out-of-order. Compare sequence, VLAN and TTL before interpreting those labels as a network fault.

## Extra 2: Prove replies and new initiations differ

**Optional, 10–15 minutes.** On **`wago-plc2a`**, test a fresh reverse SSH connection to the listener proved open in Task 1:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.12
```

On **`hirschmann-edge1`**, test gateway protection, then repeat permitted HTTP:

```bash
nmap -n -sT -Pn -p22 --max-retries 1 --host-timeout 10s 172.30.70.1
curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
```

Read the `ex7-` rule statistics/logs again. The reverse attempt must match `ex7-deny-to`, gateway SSH must match `ex7-deny-input`, and HTTP must still return 200 through stateful replies. The two SSH results are filtered. No existing sessions or global counters need to be cleared.

## Cleanup: restore the flat baseline

Stop and close the capture. On **`hirschmann-edge1`**:

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

On **`gw-firewall`**:

```routeros
/ip address remove [find where comment=ex7-vlan70]
/interface bridge vlan remove [find where comment=ex7-vlan70]
/interface vlan remove [find where name=vlan70]
/ip firewall filter remove [find where comment~"^ex7-"]
```

Repeat Task 1's address, route, service and switch checks. `hirschmann-edge1` must again use `10.50.0.12/8`, PVID 10 and a direct `wago-plc2a` route; HTTP and SSH must be open again. `wago-plc2a` must again reach `hirschmann-edge1` SSH at `10.50.0.12`. VLAN/interface 70 and the `ex7-` rules must be absent. Original VLAN 10/60 membership and management connectivity must match your notes.

Run **Collect new data** again to refresh reported port membership. Check absence in the actual RouterOS tables; a VLAN number in a catalog alone is not proof that a port still carries it.

If cleanup differs from the recorded baseline, use the saved checkpoint recovery procedure with the instructor and repeat Task 1. Close all terminal tabs after verification.

Continue with [Exercise 8: Microsegment a legacy endpoint with the diode](ex8-diode-microsegmentation.md).

## Explore the topic

- [MikroTik bridge VLAN table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): trace PVID, tagged ports and CPU access.
- [RouterOS firewall filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/): compare input and forward.
- [RouterOS connection tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/): explain why the earlier state rule permits replies.
- [NIST SP 800-82r3](https://csrc.nist.gov/pubs/sp/800/82/r3/final): relate enforced boundaries to OT availability.
