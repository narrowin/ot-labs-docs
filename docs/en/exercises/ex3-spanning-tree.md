# Exercise 3: Explain the live spanning tree

**Time:** 25–35 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Intermediate.

Find the elected root from operational evidence, then explain how RSTP prevents a loop in the triangle of switches. This is an observation exercise: leave priorities, ports and cables unchanged.

**Learning objectives:** Identify bridge membership, elected root and port roles; interpret BPDUs; explain how RSTP and LACP provide complementary redundancy.

## Before you start

Finish ARP cleanup from [Exercise 2](ex2-arp-spoofing.md). Open [Netmap with the switches highlighted](/network/?search=sw-), then the configured terminals for [sw-dist](/network/terminal?node=sw-dist), [sw-acc1](/network/terminal?node=sw-acc1), [sw-acc2](/network/terminal?node=sw-acc2) and [gw](/network/terminal?node=gw-firewall) as needed. These terminals run RouterOS, not Linux.

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

**Checkpoint:** The bridge is `bridge`, RSTP and VLAN filtering are enabled, and `bond-sw-acc1` is a bridge port. Its physical members `ether9` and `ether10` form one logical bridge connection; they are not two independent bridge ports. `ether12` faces gw and `ether8` faces sw-acc2.

## Task 2: Find the elected root

On **each switch**, then **gw**:

```routeros
/interface bridge monitor bridge once
```

Make a four-row note with `bridge-id`, `root-bridge`, `root-bridge-id`, `root-port` and `root-path-cost`.

**Checkpoint:** All four devices agree on the root ID. The root reports `root-bridge: yes`, cost 0 and no root port. In the unchanged baseline the root is **gw**, not sw-dist. Verify your own output rather than choosing the visually central node.

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
/interface bridge port monitor [find interface=ether11] once
/interface bridge port monitor [find interface=ether12] once
```

**Checkpoint:** In the baseline, sw-acc1's bond is its root port; sw-acc2 `ether12` is its root port. sw-acc2 `ether11`, facing sw-acc1, is an **alternate port** with `forwarding: no`. The other end of that cable can be designated and forwarding. Record both ends instead of saying the cable is down.

Explain why the LACP bond and RSTP solve different problems. Close terminals when finished.

## Solution

??? success "Compare your evidence"
    gw advertises the winning bridge ID. With the baseline priority `0x8000` tied, its administrative MAC ending in `09:11` wins over sw-dist `09:12`, sw-acc1 `09:13` and sw-acc2 `09:14`. The live monitor must confirm this; changed configurations can elect another root.

    sw-dist reaches gw through `ether12`. Each access switch reaches sw-dist through its chosen root port. The additional sw-acc1–sw-acc2 connection would close a switching loop, so sw-acc2's alternate port does not forward ordinary data. RSTP treats the LACP bond as one logical port. A redundant link can be healthy and deliberately non-forwarding.

## Extra 1: Win the root-election detective challenge

**Optional, 10 minutes.** Use the four bridge IDs from Task 2.

1. Rank them from lowest to highest; show which part of each comparison decided the order.
2. Predict the root if sw-acc2 had priority `0x1000` and all other settings stayed the same. This is a paper prediction; do not change the devices.
3. Read `/interface bridge print detail` on sw-acc2 and compare its configured priority with its operational `root-bridge-id`.

**Checkpoint:** You explain both the current MAC tie-break and why a lower numerical priority would override that tie-break. A configured priority alone does not prove who is currently root. [Find the election rules](https://help.mikrotik.com/docs/spaces/ROS/pages/21725254/Spanning+Tree+Protocol) under **(R)STP Election Process**.

## Extra 2: Diagnose a healthy link that does not forward

**Optional, 10 minutes.** On **sw-acc2**:

```routeros
/interface ethernet monitor ether11 once
/interface bridge port monitor [find interface=ether11] once
```

Wait ten seconds and repeat the bridge-port command. Compare `role`, `forwarding` and `tx-rx-bpdu`.

**Checkpoint:** The Ethernet link is running while RSTP reports alternate/non-forwarding, and BPDU counters increase. Control traffic still supports loop prevention. This is different from a disconnected cable. A wire capture may even contain data arriving at a discarding port; arrival does not prove that the bridge forwarded it.

Write a two-sentence diagnosis containing physical state, RSTP role and one counter observation. Close terminals. Continue with [Exercise 4](ex4-link-aggregation.md).

## Explore the topic

- [MikroTik RSTP monitoring and election](https://help.mikrotik.com/docs/spaces/ROS/pages/21725254/Spanning+Tree+Protocol).
- [Wireshark STP fields](https://www.wireshark.org/docs/dfref/s/stp.html): search for `stp.root`, `stp.bridge` and `stp.flags`.
