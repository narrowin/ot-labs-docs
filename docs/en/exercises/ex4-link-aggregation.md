# Exercise 4: Link Aggregation and Load Balancing

**Time Estimate**: 20-30 minutes  
**Lab Topology**: `ot-sec-segmented` only  
**Difficulty**: :material-signal-cellular-2: Intermediate

!!! warning "Exercise Not Yet Available"
    Link aggregation is not currently configured in any lab topology. This exercise is planned for a future lab version.

## Learning Objectives

- Analyze LACP link aggregation operation
- Observe traffic distribution across bonded links
- Test link failover behavior
- Understand load balancing algorithms

## Prerequisites

- Completed Exercise 3 (STP Analysis)
- Understanding of LACP concepts
- Wireshark experience
- Basic MikroTik commands

## Scenario

Link aggregation combines multiple physical links into one logical link for increased bandwidth and redundancy. LACP (802.3ad) dynamically negotiates bonding. Traffic is distributed across member links using a hash algorithm based on source/destination MAC, IP, or ports. Understanding how this works is important for capacity planning and troubleshooting in OT networks.

## Tasks

### Task 1: Verify LACP negotiation

1. SSH to `sw-dist`:

```bash
ssh admin@192.168.100.12
```

!!! info "Default Credentials"
    Password: `admin`

2. Monitor LACP protocol operation:

```routeros
/interface bonding monitor bond-sw-acc1 once
```

**Expected output includes**:

- `lacp-partner-system-id` - MAC of partner switch
- `active-ports` - Number of active links
- `lacp-negotiated` - True/false

3. Check detailed bonding configuration:

```routeros
/interface bonding print detail where name=bond-sw-acc1
```

**Questions**:

- What is the bonding mode?
- Which physical interfaces are slaves?
- What is the transmit hash policy?

### Task 2: Observe traffic distribution

1. Start Wireshark captures on BOTH bond member links:
   - Capture link `sw-dist` ether9 to `sw-acc1`
   - Open second Wireshark for ether10

2. SSH to `wago-plc2a-vlan10`:

```bash
ssh admin@192.168.100.53
```

3. Generate continuous traffic with varying sources:

```bash
ping -f 1.1.1.1
```

Use `-f` (flood) for high packet rate to see distribution.

4. Observe both Wireshark captures

**Questions**:

- Is traffic balanced equally across both links?
- Do you see traffic on both links or just one?
- What determines which link is used for specific flows?

### Task 3: Test hash algorithm

1. Stop flood ping
2. Generate traffic from multiple sources:

Terminal 1 - `wago-plc2a-vlan10`:

```bash
ping -i 0.2 1.1.1.1
```

Terminal 2 - `wago-plc2b-vlan10`:

```bash
ssh admin@192.168.100.54
ping -i 0.2 1.1.1.1
```

3. Check packet counts on bond members:

```routeros
/interface monitor-traffic ether9,ether10
```

**Question**: Are different source IPs distributed to different links?

### Task 4: Test link failover

1. Keep pings running from previous step
2. On `sw-dist`, disable one bond member:

```routeros
/interface ethernet disable ether9
```

3. Observe:
   - Do pings continue without interruption?
   - How quickly does failover occur?
   - Check Wireshark on remaining active link

4. Re-enable the interface:

```routeros
/interface ethernet enable ether9
```

5. Check bond status:

```routeros
/interface bonding monitor bond-sw-acc1 once
```

**Question**: Does both links return to active state automatically?

## Verification

Success indicators:

- LACP shows negotiated status
- Both bond members active
- Traffic visible on member links
- Failover occurs in under 1 second
- Disabled link automatically rejoins when enabled
- Packet count increases on both interfaces

## Questions to Consider

1. Why might traffic use only one link for a specific flow?
2. What are advantages and disadvantages of different hash algorithms?
3. How does LACP compare to static bonding?
4. What happens if LACP hello packets are lost?

## Common Issues

- **Issue**: Traffic only uses one link
  - **Solution**: Hash algorithm makes per-flow decisions. Single source/destination uses one link. Use multiple sources to see distribution.

- **Issue**: Cannot monitor traffic on interface
  - **Solution**: Use `/interface monitor-traffic ether9,ether10` command format

- **Issue**: Failover takes several seconds
  - **Solution**: This is normal for LACP. RSTP must also reconverge.

---

## Solution

??? success "Click to reveal solution"
    ### Traffic distribution observation

    With single source ping, traffic typically uses one link consistently due to hash algorithm using source/destination IPs.

    With multiple sources (different source IPs), traffic distributes across links.

    Hash policy `layer-2-and-3` uses source/dest MAC and IP addresses.

    ### Packet count verification

    ```routeros
    /interface monitor-traffic ether9,ether10
    ```

    Shows rx-packets-per-second and tx-packets-per-second for each interface.

    With diverse traffic, both interfaces show increasing packet counts.

    ### Failover test

    Disable one member:

    ```
    /interface ethernet disable ether9
    ```

    Pings continue without interruption. All traffic switches to ether10.

    Re-enable:

    ```
    /interface ethernet enable ether9
    ```

    LACP renegotiates (takes few seconds), then both links active again.

    Failover is subsecond for established flows.
