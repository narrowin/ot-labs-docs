# Exercise 4: Follow flows through an LACP bond

**Time:** 25–35 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Intermediate.

Prove that two physical links form one active LACP bond, then find which member carries a particular flow. The current flat lab has LACP; no separate segmented topology is needed. Keep member ports enabled throughout this shared-lab exercise.

**Learning objectives:** Verify LACP negotiation, observe flow distribution, explain the transmit hash, and evaluate redundancy and the evidence needed for a bounded failover test.

## Before you start

Complete [Exercise 3](ex3-spanning-tree.md). Open [sw-dist](/network/terminal?node=sw-dist) and [sw-acc1](/network/terminal?node=sw-acc1) terminals. For traffic generation, use [WAGO A](/network/terminal?node=wago-plc2a-vlan10) and [CtrlX](/network/terminal?node=ctrlx-plc3-vlan20). Both attach to sw-acc1; WAGO B attaches to sw-acc2 and is not a suitable second source for this bond test.

| End | Logical bond | Physical members |
| --- | --- | --- |
| sw-dist | `bond-sw-acc1` | `ether9`, `ether10` |
| sw-acc1 | `bond-sw-dist` | `ether11`, `ether12` |

The cables pair sw-dist ether9 with sw-acc1 ether11, and ether10 with ether12. Capture one member at a time and repeat the same commands on the second; the lesson's capture links reuse one tab.

??? info "Using standalone documentation?"
    In your assigned lab's live **Netmap**, search the stable switch/endpoint names above. Use **Open terminal** from a node's action menu. Use **Capture packets** on sw-dist and select **ether9**, then **ether10**, as directed. Reader links retain network and snapshot and preselect the named node/port.

## Task 1: Verify both ends

On **sw-dist**:

```routeros
/interface bonding print detail where name=bond-sw-acc1
/interface bonding monitor bond-sw-acc1 once
```

On **sw-acc1**:

```routeros
/interface bonding print detail where name=bond-sw-dist
/interface bonding monitor bond-sw-dist once
```

Record mode, members, hash policy, active/inactive ports, local system ID and partner system ID.

**Checkpoint:** Mode is `802.3ad`, both members are active, and `transmit-hash-policy` is `layer-2-and-3`. Each end's partner ID equals the other end's local system ID. Read the actual IDs; they are not necessarily the fixed bridge MACs from Exercise 3.

??? info "Hint: a bond is configured but a member is inactive"
    Stop the traffic comparison and show the monitor output to the instructor. A configured bond alone does not prove successful negotiation. Do not reset the switch or reimport startup configuration.

## Task 2: Inspect LACP on the first member

[Capture sw-dist ether9](/network/capture?node=sw-dist&interface=ether9). Choose **Start Capture**, wait about ten seconds and use `lacp` in Wireshark.

Expand **Link Aggregation Control Protocol**. Compare **Actor** and **Partner** system IDs with Task 1; inspect the state flags for **Synchronization**, **Collecting** and **Distributing**.

**Checkpoint:** LACP frames are present; a healthy negotiated member reports synchronized, collecting and distributing. A frame's Ethernet destination is `01:80:c2:00:00:02`. Record one frame number.

## Task 3: Locate two small flows

Keep the first capture running. On **WAGO A**:

```bash
ping -c 4 10.10.0.1
```

On **CtrlX**:

```bash
ping -c 4 10.10.0.1
```

Wait a few seconds, then inspect requests only:

```wireshark
icmp.type == 8 && (ip.src == 10.10.0.11 || ip.src == 10.20.0.11)
```

Record the request count for each source and the captured member. Close the first capture tab. [Capture sw-dist ether10](/network/capture?node=sw-dist&interface=ether10), start it and repeat both ping commands and the filter.

**Checkpoint:** Each source's requests normally stay on one member while the bond is stable. The two sources can select different members, or collide on the same one. Do not expect equal counts or packet-by-packet alternation. Keep requests and replies separate: each sending switch selects its own transmit member.

## Task 4: Interpret rates without claiming a speed test

On **sw-dist**:

```routeros
/interface monitor-traffic ether9,ether10 once
/interface bridge port monitor [find interface=bond-sw-acc1] once
```

Compare the instantaneous rates with your filtered test traffic. Explain why background traffic can make both counters increase even if a test flow used one member.

**Checkpoint:** The bond forwards as one bridge port. Multiple flows can share aggregate capacity, but these pings do not measure throughput. One flow is not guaranteed twice one link's bandwidth. Active members alone also do not measure failover time; this lesson makes no subsecond or zero-loss claim.

Read `link-monitoring`, `mii-interval` and `lacp-rate` in the bond configuration. Explain the expected response to losing one member versus both. A surviving member can keep the logical bond forwarding without an RSTP topology change. Losing the whole bond can require an alternate RSTP path. For an instructor-approved isolated failover trial, specify a bounded ping, before/during/after active-member evidence, observed loss and recovery to both active members. Do not infer a measured outage time from configured timers alone or disable links on this shared lab.

Close the capture and terminals unless continuing with an extra section.

## Solution

??? success "Compare your evidence"
    The bond is active at both ends and the reciprocal system IDs agree. LACP exchanges describe member eligibility; the transmit hash chooses a member for data. The current policy uses MAC and IP information. With an unchanged bond, repeated requests for the same endpoint pair remain on a member; adding another source may change that selection, but does not guarantee even distribution.

    In the walkthrough, CtrlX requests used sw-dist ether9 and WAGO A requests used ether10. Record your measured result rather than treating this assignment as permanent. RSTP operates on the logical bond; LACP operates on its physical members.

## Extra 1: Does a larger packet change the chosen member?

**Optional, 10 minutes.** Predict the effect of payload size under `layer-2-and-3` hashing.

1. Capture ether9 and, on **WAGO A**, run:

    ```bash
    ping -c 2 -s 32 10.10.0.1
    ping -c 2 -s 128 10.10.0.1
    ```

2. Close that capture, repeat on ether10 and use `icmp.type == 8 && ip.src == 10.10.0.11`.
3. Compare the member and frame lengths for both sizes.

**Checkpoint:** Packet size changes frame length, while this policy's source/destination MAC and IP inputs remain the same. Both sizes should use the same request member in a stable bond. A topology or member-state change invalidates that comparison. For the same payload, compare with Exercise 1's access frame to find the four-byte VLAN tag.

## Extra 2: Audit both ends without trusting a green status

**Optional, 10 minutes.** Use the two monitor outputs and one LACP packet from each member.

1. Make a table of local system ID, partner system ID, member name, actor key and partner key.
2. Confirm reciprocal system IDs across switches. On a given switch, members in the same aggregate should advertise a consistent actor key. The remote side has its own key; local and remote key numbers need not generally be equal.
3. Check synchronization, collecting and distributing flags. Explain how one incorrect partner ID could reveal a wrongly connected cable even while Ethernet reports link-up.

**Checkpoint:** You can distinguish physical link-up, LACP agreement and actual data forwarding using three separate observations. No interface changes are needed. Close captures and terminals. Continue with [Exercise 5](ex5-reconnaissance.md).

## Explore the topic

- [MikroTik bonding and transmit hashes](https://help.mikrotik.com/docs/spaces/ROS/pages/8323193/Bonding): find `layer-2-and-3` and the monitoring examples.
- [Wireshark LACP fields](https://www.wireshark.org/docs/dfref/l/lacp.html): search for `lacp.actor.sysid`, `lacp.actor.key` and `lacp.actor.state.collecting`.
