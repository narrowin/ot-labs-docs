# Exercise 4: Follow flows through an LACP bond

**Time:** 30–40 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Guided practice.

Prove that two physical links form one active LACP bond, then find which member carries a particular flow. Use the LACP bond in your own flat lab. Compare the traffic while both members are active.

**Learning objectives:** Verify LACP negotiation, observe flow distribution, explain the transmit hash, and evaluate redundancy and the evidence needed for a bounded failover test.

## Your monitoring task

A cell has two uplink cables. A capture on one cable shows only part of the expected traffic, although both cables are healthy. Decide whether communication is missing or your observation point sees only one member of the connection.

A **bond** combines physical links into one logical connection. **LACP** (Link Aggregation Control Protocol) checks which members can work together. A **flow** is a related stream of packets; the **hash policy** selects a member using address fields. It does not normally send alternating packets down alternating cables.

**Predict:** Does “two active members” mean that a capture on either member sees every packet?

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

## Task 4: Find the bond in the spanning tree

On **sw-dist**:

```routeros
/interface bridge port monitor [find interface=bond-sw-acc1] once
```

Compare this logical port with the two active members from Task 1. Does RSTP see one bridge connection or two independent ones?

**Checkpoint:** The logical bond is a forwarding bridge port. Its two active physical members carry the flows you observed, while RSTP handles the bond as one connection. A stable flow normally stays on one member; multiple flows can use both members. The small ping samples show placement, not available throughput.

Close the capture and terminals unless continuing with an extra section.

## Your monitoring handover

Record both active members and where each test source's requests appeared. Specify a capture point that can see the gateway-bound exchange without relying on one bond member. Explain why neither uneven traffic nor four successful pings establish a capacity or failover problem.

## Solution

??? success "Compare your evidence"
    The bond is active at both ends and the reciprocal system IDs agree. LACP exchanges describe member eligibility; the transmit hash chooses a member for data. The current policy uses MAC and IP information. With an unchanged bond, repeated requests for the same endpoint pair remain on a member; adding another source may change that selection, but does not guarantee even distribution.

    In the walkthrough, CtrlX requests used sw-dist ether9 and WAGO A requests used ether10. Record your measured result rather than treating this assignment as permanent. RSTP operates on the logical bond; LACP operates on its physical members.

## Extra 1: Find the capture blind spot

**Optional, 10 minutes.** An analyst captured only sw-dist ether9 and concluded that a missing PLC flow had stopped.

1. Use your request counts from Task 3. For each source, state whether ether9 alone saw its requests. A hash collision can put both test sources on one member; that does not prove complete coverage.
2. [Capture the gateway uplink](/network/capture?node=sw-dist&interface=ether12), start it and repeat the four gateway pings from WAGO A and CtrlX. Apply Task 3's request filter.
3. Compare source coverage with the single-member capture. Mark this observation point on your path notes.

**Checkpoint:** Both gateway-bound test sources are visible on ether12. This point is suitable for these flows, but does not see all same-VLAN PLC-to-PLC communication. Monitoring coverage follows the traffic path. A real bond observation needs both relevant members/directions or a verified aggregate observation point; this lab does not configure a physical TAP or switch mirroring.

Close the capture before continuing.

## Extra 2: Decide what the redundancy actually covers

**Optional, 10 minutes.** Maintenance will replace an uplink cable, then service sw-dist. Use Netmap and the bond/root-port outputs to identify which communication can continue during each step.

Complete this table as a **prediction**, using Netmap and your observed bond state:

| Proposed loss | What could remain? | What still needs checking? |
| --- | --- | --- |
| One bond member | The other member between the same switches | Member detection and actual application loss |
| Both bond members | The RSTP route through sw-acc2 | Alternate path readiness and application recovery |
| sw-dist itself | Access switches may still connect to each other | The gateway is attached to sw-dist: its path is lost |

**Checkpoint:** Two cables between the same switches do not provide independent switches, power or gateway paths. Explain the expected interruption before approving the proposed production maintenance. Specify an application-level success criterion and recovery steps; use measured lab results, rather than configured timers, if you state a recovery time.

Close terminals. Continue with [Exercise 5](ex5-reconnaissance.md).

## Explore the topic

- [MikroTik bonding and transmit hashes](https://help.mikrotik.com/docs/spaces/ROS/pages/8323193/Bonding): find `layer-2-and-3` and the monitoring examples.
- [Wireshark LACP fields](https://www.wireshark.org/docs/dfref/l/lacp.html): search for `lacp.actor.sysid`, `lacp.actor.key` and `lacp.actor.state.collecting`.
