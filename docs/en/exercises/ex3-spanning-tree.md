# Exercise 3: Spanning Tree Protocol Analysis

**Time Estimate**: 25-35 minutes  
**Lab Topology**: `ot-sec-flat` or `ot-sec-segmented`  
**Difficulty**: :material-signal-cellular-2: Intermediate

## Learning Objectives

- Understand Spanning Tree Protocol operation
- Identify root bridge and port states
- Observe STP BPDU exchange
- (Segmented only) Analyze LACP bonding with STP

## Prerequisites

- Understanding of Layer 2 switching
- Knowledge of STP/RSTP concepts
- Basic MikroTik RouterOS commands
- SSH access to switches

## Scenario

Spanning Tree Protocol (STP) prevents loops in switched networks by blocking redundant paths. Industrial networks use RSTP (Rapid Spanning Tree Protocol) for faster convergence. Both lab topologies run STP on all switches. The segmented lab additionally uses LACP (Link Aggregation Control Protocol) bonding between switches, which works in coordination with STP for both redundancy and increased bandwidth.

## Tasks

### Task 1: Identify switch topology

1. SSH into distribution switch:

```bash
ssh admin@192.168.100.12
```

!!! info "Default Credentials"
    Password: `admin`

2. View bridge configuration:

```routeros
/interface bridge print
```

3. List all bridge ports:

```routeros
/interface bridge port print
```

**Questions**:

- What is the bridge name?
- Which interfaces are bridge members?
- Which interfaces have VLAN filtering enabled?

### Task 2: Examine link topology

1. Check which interfaces connect to other switches:

```routeros
/interface bridge port print
```

**Expected output**:

- **Flat topology**: ether10, ether12, ether8, ether2 as bridge members
- **Segmented topology**: ether12, ether10, ether8, ether2 as bridge members

**Questions**:

- How many links connect sw-dist to sw-acc1?
- (Segmented) Are both bond members active?

### Task 3: Capture BPDU frames

1. In TopoViewer, start Wireshark capture on link between `sw-dist` and `sw-acc1`
2. Apply filter:

```wireshark
stp
```

Or alternatively:

```wireshark
eth.dst == 01:80:c2:00:00:00
```

**Observations**:

- BPDUs sent to multicast address `01:80:c2:00:00:00`
- Root bridge identifier
- Port roles (designated, root)
- Bridge priority values

3. Check bridge settings on both switches:

On `sw-dist`:

```routeros
/interface bridge print detail
```

On `sw-acc1` (SSH to 192.168.100.13):

```routeros
/interface bridge print detail
```

**Question**: Which switch has lower priority and becomes root bridge?

### Task 4: Understand redundancy approach

**In flat topology**:

- Single link between switches
- STP prevents loops between other switch connections
- No link-level redundancy between sw-dist and sw-acc1

**In segmented topology**:

- LACP bond provides link redundancy
- STP still runs on other ports
- Two complementary redundancy mechanisms

From distribution switch, check configuration:

```routeros
/interface ethernet print
/interface bridge port print
```

**Question**: What happens if the single uplink fails in flat topology?

## Verification

Success indicators:

- Bridge configuration shows vlan-filtering enabled
- BPDU frames visible in Wireshark captures
- Root bridge identified from bridge settings
- (Segmented) Bond interface exists with LACP active
- (Flat) Single link topology documented
- Understanding of how STP prevents loops

## Questions to Consider

1. Why does STP run even when there's no redundant path (flat)?
2. (Segmented) What happens if one bond member fails?
3. How quickly does STP reconverge after topology change?
4. What security risks does STP present (BPDU attacks)?

## Common Issues

- **Issue**: No bonding interface visible in flat
  - **Solution**: Correct - flat topology has no bonding, only single links

- **Issue**: Cannot SSH to switches
  - **Solution**: Use management IPs 192.168.100.11-14, not data plane IPs

- **Issue**: No BPDUs in capture
  - **Solution**: BPDUs are sent periodically (every 2 seconds). Wait or generate topology change.

- **Issue**: Exercise says "segmented only" but I'm in flat
  - **Solution**: Exercise 3 works in both topologies - bonding tasks are marked as segmented-only within the exercise

---

## Solution

??? success "Click to reveal solution"
    ### Bridge configuration

    SSH to `sw-dist`:

    ```bash
    ssh admin@192.168.100.12
    ```

    View bridge:

    ```
    /interface bridge print
    ```

    Output:

    ```text
    Flags: X - disabled, R - running
     0 R name="bridge" mtu=auto actual-mtu=1500 l2mtu=65535 arp=enabled 
         arp-timeout=auto mac-address=XX:XX:XX:XX:XX:XX auto-mac=no 
         protocol-mode=rstp vlan-filtering=yes
    ```

    Bridge ports:

    ```
    /interface bridge port print
    ```

    **Flat topology** shows ether10, ether12, ether8, ether2 as bridge members with VLAN settings.
    
    **Segmented topology** shows ether12, ether10, ether8, ether2 as bridge members with VLAN settings.

    !!! info "Bonding not configured"
        Link aggregation (bonding) is currently not configured in any lab topology. In both topologies, each switch connects via individual physical interfaces.

    ### Root bridge identification

    Check priority on both switches. Lower priority wins. Default priority is 32768.

    BPDU captures show root bridge ID in protocol fields.
