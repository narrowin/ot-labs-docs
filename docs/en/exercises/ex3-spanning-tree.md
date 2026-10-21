# Exercise 3: Explain the live spanning tree

**Time:** 30–40 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Guided practice.

Find the elected root from operational evidence, then explain how RSTP prevents a loop in the triangle of switches. The main tasks observe the running network. Optional Extra 2 temporarily disables one uplink in your own lab and restores it.

**Learning objectives:** Identify bridge membership, elected root and port roles; interpret BPDUs; explain how RSTP and LACP provide complementary redundancy.

## Your maintenance task

A technician reports: “The redundant cable is connected, but one port is not forwarding. Should I enable forwarding?” Determine whether this is a fault or the mechanism keeping the cell available. Forcing both paths to forward can create a switching loop.

**Read the status:** **RSTP** (Rapid Spanning Tree Protocol) selects a loop-free set of paths. The **root bridge** is its reference switch, not necessarily the IP gateway. A **root port** leads towards that reference; an **alternate port** is a standby path. **BPDUs** are the control messages switches use to agree. Standby ports can exchange control messages while not forwarding ordinary application data.

**Predict:** Can a physically healthy cable be intentionally unused for application traffic?

## Before you start

Finish ARP cleanup from [Exercise 2](ex2-arp-spoofing.md). Open [Netmap with the switches highlighted](/network/?search=sw-), then the configured terminals for [sw-dist](/network/terminal?node=sw-dist), [sw-acc1](/network/terminal?node=sw-acc1), [sw-acc2](/network/terminal?node=sw-acc2) and [`gw-firewall`](/network/terminal?node=gw-firewall) as needed. These terminals run RouterOS, not Linux.

[Open spanning-tree instance 0](/network/?viewMode=spanning-tree&stpInstanceId=0&dataType=All). Compare the displayed root and port roles with the operational commands below.

??? info "Using standalone documentation?"
    Open your assigned lab's live **Netmap**, choose **Spanning Tree** in the view selector and instance **0**. Search the named switch and use its action menu's **Open terminal** or **Capture packets**. For Task 3 choose sw-dist **ether8**. Reader links retain the selected network and snapshot automatically.

## Task 1: Separate physical links from bridge ports

On **sw-dist**:

```routeros
/interface bridge print detail
/interface bridge port print
/interface bonding print detail
```

Record `protocol-mode`, `vlan-filtering`, bridge name and bridge ports. Match the cables on Netmap with the logical ports.

**Checkpoint:** The bridge is `bridge`, RSTP and VLAN filtering are enabled, and `bond-sw-acc1` is a bridge port. Its physical members `ether9` and `ether10` form one logical bridge connection; they are not two independent bridge ports. `ether12` faces `gw-firewall` and `ether8` faces sw-acc2.

## Task 2: Find the elected root

On **each switch**, then **`gw-firewall`**:

```routeros
/interface bridge monitor bridge once
```

Make a four-row note with `bridge-id`, `root-bridge`, `root-bridge-id`, `root-port` and `root-path-cost`.

**Checkpoint:** All four devices agree on the root ID. The root reports `root-bridge: yes`, cost 0 and no root port. In the unchanged baseline the root is **`gw-firewall`**, not sw-dist. Verify your own output rather than choosing the visually central node.

??? info "Hint: priorities can tie"
    A bridge ID includes priority and MAC address. A lower priority wins; if priorities tie, the lower MAC wins. `print detail` shows configuration, while `monitor` shows the election result.

## Task 3: Match a BPDU to the operational state

[Open the sw-dist link to sw-acc2](/network/capture?node=sw-dist&interface=ether8). Choose **Start Capture**, allow about ten seconds, and apply `stp` in Wireshark.

Select an RSTP packet. Expand **Spanning Tree Protocol** and find **Root Identifier**, **Root Path Cost** and **Bridge Identifier**. Compare its advertised root with Task 2. The sender's bridge ID need not be the root's ID.

**Checkpoint:** BPDUs are present and their advertised root matches the monitor output. Their destination is `01:80:c2:00:00:00`. Record a frame number and both identifiers. Close the capture tab.

??? info "Hint: no BPDUs"
    Clear the display filter and verify sw-dist `ether8`, facing sw-acc2. Wait for several periodic transmissions. Do not create a topology change to generate packets. A filter or capture on the wrong port cannot establish that RSTP is absent.

## Task 4: Locate the loop-prevention decision

On **sw-acc1**:

```routeros
/interface bridge port monitor [find interface=bond-sw-dist] once
/interface bridge port monitor [find interface=ether10] once
```

On **sw-acc2**:

```routeros
/interface ethernet monitor ether11 once
/interface bridge port monitor [find interface=ether11] once
/interface bridge port monitor [find interface=ether12] once
```

**Checkpoint:** In the baseline, sw-acc1's bond is its root port; sw-acc2 `ether12` is its root port. sw-acc2 `ether11`, facing sw-acc1, is an **alternate port** with `forwarding: no`. The other end of that cable can be designated and forwarding. Record both ends instead of saying the cable is down.

Wait ten seconds and repeat the ether11 bridge-port command. Its physical link stays up and its `tx-rx-bpdu` counters increase, while its RSTP role remains alternate/non-forwarding. Control messages still pass on this standby link. Explain why LACP and RSTP solve different problems. Close terminals when finished.

## Your maintenance handover

Write “leave as designed” or “investigate” for sw-acc2 ether11. Support the decision with its physical state, RSTP role and agreed root ID. State what a failover trial would need to measure: actual recovery and application interruption after a failure. A standby path in a diagram does not prove a measured recovery time.

## Solution

??? success "Compare your evidence"
    `gw-firewall` advertises the winning bridge ID. With the baseline priority `0x8000` tied, its administrative MAC ending in `09:11` wins over sw-dist `09:12`, sw-acc1 `09:13` and sw-acc2 `09:14`. The live monitor must confirm this; changed configurations can elect another root.

    sw-dist reaches `gw-firewall` through `ether12`. Each access switch reaches sw-dist through its chosen root port. The additional sw-acc1–sw-acc2 connection would close a switching loop, so sw-acc2's alternate port does not forward ordinary data. RSTP treats the LACP bond as one logical port. A redundant link can be healthy and deliberately non-forwarding.

## Extra 1: Review a switch replacement before the outage

**Optional, 10 minutes.** A replacement switch will use default RSTP settings. The maintenance plan assumes that the current forwarding paths will stay the same.

1. Use Task 2's four bridge IDs to identify whether priority or MAC decided the current root.
2. On **sw-acc2**, read `/interface bridge print detail`. Compare configured priority with the operational root ID.
3. On paper, replace the current root with a switch of the same priority but a MAC greater than the other three. Predict whether the same physical location would still be root. Do not change any device.
4. Write a pre/post-change acceptance check: intended root, actual root seen by all switches, root/alternate ports and application reachability.

**Checkpoint:** Equal priorities leave the choice to MAC addresses; replacement hardware can change that choice. Agree the intended root/backup with the network owner and manage priorities deliberately before a real change. Choosing a root requires topology and availability requirements, not simply picking the middle of the drawing. This lab's fixed MACs make the current result reproducible; real replacements need their own check.

## Extra 2: Test recovery after one uplink fails

**Optional, 15–20 minutes.** Verify that `wago-plc2b-vlan10` can still reach `gw-firewall` and `wago-plc2a-vlan10` when sw-acc2 loses its direct uplink. Use only your own lab. Read all four steps first and keep both terminals ready. Open the [sw-acc2 management terminal](/network/terminal?node=sw-acc2) and keep it open for restoration.

1. Confirm Task 4's starting roles on **sw-acc2**: ether12 is root/forwarding; ether11 is alternate/non-forwarding. On [`wago-plc2b-vlan10`](/network/terminal?node=wago-plc2b-vlan10), start:

    ```bash
    ping -D -i 0.5 -c 120 10.10.0.1
    ```

    This sends probes for about one minute. Leave it running while using the switch terminal.

2. On **sw-acc2**, disable only the uplink to sw-dist:

    ```routeros
    /interface disable ether12
    /interface bridge port monitor [find interface=ether11] once
    ```

    Repeat the monitor command after a few seconds. Ether11 should become root/forwarding, providing the path sw-acc2 → sw-acc1 → sw-dist → `gw-firewall`.

3. While ether12 remains disabled, wait for the bounded ping to finish. Record any missing sequence numbers and its loss summary. On **`wago-plc2b-vlan10`**, also check the PLC application:

    ```bash
    curl -sS --max-time 5 -o /dev/null -w 'HTTP %{http_code}\n' http://10.10.0.11:8080/webvisu.htm
    ```

4. **Restore ether12 before leaving this extra, even if a check failed.** On **sw-acc2**:

    ```routeros
    /interface enable ether12
    /interface ethernet monitor ether12 once
    /interface bridge port monitor [find interface=ether11] once
    /interface bridge port monitor [find interface=ether12] once
    ```

    After convergence, ether12 must be running/root/forwarding again and ether11 alternate/non-forwarding. On **`wago-plc2b-vlan10`**, repeat `ping -c 4 10.10.0.1` and the HTTP command; both must succeed. If the roles or communication do not recover, keep the terminal open and ask the instructor to restore the baseline.

**Checkpoint:** During the uplink loss, the alternate path became active and the HTTP request returned 200. Record observed ping loss; zero observed loss means only that these probes missed no replies, not that every application experienced zero interruption. After restoration, the original roles and communication return. Close terminals. Continue with [Exercise 4](ex4-link-aggregation.md).

## Explore the topic

- [MikroTik RSTP monitoring and election](https://help.mikrotik.com/docs/spaces/ROS/pages/21725254/Spanning+Tree+Protocol).
- [Wireshark STP fields](https://www.wireshark.org/docs/dfref/s/stp.html): search for `stp.root`, `stp.bridge` and `stp.flags`.
