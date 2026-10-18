# Exercise 7: Build and verify a routed VLAN

**Time:** 60–90 minutes. **Lab:** an instructor-assigned disposable copy of `ot-sec-flat`. **Difficulty:** Intermediate.

Create one new security zone, carry it through the switches, route it at the gateway and enforce a small policy. Prove the application works and an unapproved connection is blocked, then restore the flat baseline.

**You will learn to:** configure an access VLAN and tagged path, include the gateway's bridge CPU, choose a non-overlapping subnet, order stateful firewall rules and verify both directions with packets and application results.

## Before you start

Use the **separate copy reserved for you or your practical group**, with no unrelated users. The instructor must have saved its flat checkpoint and confirmed the node bindings. Exercises 1–6 remain on the original flat appliance. If you only have the shared course appliance, ask for the practical copy before entering configuration commands.

Open terminals for [Hirschmann](/network/terminal?node=hirschmann-edge1-vlan50), [WAGO A](/network/terminal?node=wago-plc2a-vlan10), [sw-acc1](/network/terminal?node=sw-acc1), [sw-dist](/network/terminal?node=sw-dist) and [gw](/network/terminal?node=gw-firewall). Keep the Hirschmann terminal open: its management connection uses `eth0`, while you change only data interface `eth1`.

Complete the practical in one running session. Linux address/route commands below change the current node; its startup configuration remains flat. If a node restarts during the build, have the instructor restore the checkpoint before continuing.

**Standalone documentation:** Open the assigned practical LNE, select its network and live snapshot, then Explorer → Netmap. Search the stable names above and use their configured SSH actions. For captures select the named switch/interface; for VLAN views select VLAN 10 or 70. Reader links retain the selected network/snapshot. Use the site's lesson navigation when reading outside LNE.

The name `hirschmann-edge1-vlan50` stays unchanged. This exercise creates **one new VLAN 70**; it does not create VLANs 20–50 from hostname suffixes. Toolbox is a design tool; saving a matrix does not apply the commands below.

| State | Hirschmann data address | VLAN | Path to WAGO A | Expected access |
| --- | --- | --- | --- | --- |
| Start | `10.50.0.12/8` | 10 | Direct, switched | HTTP and SSH open |
| After build | `172.30.70.12/24` | 70 | Via `172.30.70.1` on gw | HTTP allowed; fresh SSH blocked |
| After cleanup | `10.50.0.12/8` | 10 | Direct again | Original access restored |

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

Filter the capture with `ip.addr == 172.30.70.12 && (tcp.port == 22 || tcp.port == 8080)`. Match your source/destination and ports with the rule logs. Logs and counters may also contain other probes from your practical group.

Use the newest log entries from your test period; earlier `EX7-` entries can remain after a previous cleanup.

**Checkpoint:** TCP 8080 is open and HTTP returns 200. TCP 22 is **filtered**, although the server was proved open before the change. Your SSH SYN is visible on VLAN 70, and `ex7-deny-from` records the matching denial. Successful HTTP and gateway ping establish a working path; a timeout by itself would not identify a firewall rule.

Open **Data Collector → Devices → Collect new data**. Wait until collection finishes and the **Last poll** values update, then [inspect VLAN 70 in Netmap](/network/?viewMode=vlan&vlanIds=70). Hirschmann and the path through sw-acc1, sw-dist and gw should now carry VLAN 70. Compare reported membership with the configuration and packets; reloading Netmap alone does not poll the switches.

The current VLAN view can leave the collapsed sw-dist–sw-acc1 bond edge gray even when both switches report VLAN 70. Verify that hop with the bond's VLAN table and tagged packets; the edge color does not establish whether traffic passes.

**Main result complete.** Keep VLAN 70 configured for the optional extras below. Afterwards, perform **Cleanup** even if you skip the extras. Stop the capture before changing its target.

You can do the extras after the course. Arrange continued access to your practical copy and instructor help with its reset before leaving. Clean up now, then start from the flat checkpoint and repeat Tasks 1–5 when you return. Keep your earlier results for comparison.

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

If cleanup differs from the recorded baseline, leave the copy reserved and have the instructor restore its saved checkpoint. Close all terminal tabs after verification.

## Explore the topic

- [MikroTik bridge VLAN table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): trace PVID, tagged ports and CPU access.
- [RouterOS firewall filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/): compare input and forward.
- [RouterOS connection tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/): explain why the earlier state rule permits replies.
- [NIST SP 800-82r3](https://csrc.nist.gov/pubs/sp/800/82/r3/final): relate enforced boundaries to OT availability.
