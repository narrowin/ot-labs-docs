# Exercise 1: VLAN Discovery and Traffic Analysis

**Time Estimate**: 20-30 minutes  
**Lab Topology**: `ot-sec-flat` or `ot-sec-segmented`  
**Difficulty**: :material-signal: Beginner

## Learning Objectives

- Understand 802.1Q VLAN tagging on trunk ports
- Differentiate between trunk and access port behavior
- Recognize VLAN isolation in packet captures
- Identify security implications of VLAN segmentation

## Prerequisites

- Lab deployed and running
- Basic Wireshark knowledge
- Understanding of ping command

## Scenario

Industrial networks use VLANs to logically segment different operational zones. In this lab, VLANs separate hygiene systems (VLAN 10), process control (VLAN 20), disposal systems (VLAN 30), control room (VLAN 40), and engineering workstations (VLAN 50). Understanding how VLANs work and how to identify them in traffic is essential for OT security monitoring.

## Tasks

### Task 1: Verify PLC connectivity within same VLAN

1. SSH into `wago-plc2a-vlan10`:

```bash
ssh admin@192.168.100.53
```

!!! info "Default Credentials"
    Password: `admin`

2. Test connectivity to another device in VLAN 10:

```bash
ping -c 4 10.10.0.12
```

3. Test connectivity to device in different VLAN:

```bash
ping -c 4 10.20.0.11
```

**Question**: Do both pings succeed? Why or why not?

### Task 2: Capture traffic on trunk port between switches

1. In Containerlab extension TopoViewer, right-click on the link between `sw-dist` and `sw-acc1`
2. Select capture from `sw-dist` side
3. Wireshark opens via Edgeshark

4. Apply Wireshark filter:

```wireshark
vlan
```

5. Generate traffic by pinging from a PLC (use existing SSH session):

```bash
ping -c 10 10.10.0.1
```

**Questions**:

- Which VLANs do you see in the captures?
- Look at the Ethernet frame details - where is the VLAN tag located?
- What is the VLAN ID field value?

### Task 3: Capture traffic on access port

1. Stop the previous capture
2. Right-click on link between `sw-acc1` and `wago-plc2a-vlan10`
3. Select capture from switch side
4. Generate traffic again:

```bash
ping -c 10 10.10.0.1
```

5. Apply same filter:

```wireshark
vlan
```

**Questions**:

- Do you see any VLAN tags now?
- What is the difference between trunk and access ports regarding VLAN tags?
- Why does the switch remove VLAN tags on access ports?

## Verification

Success indicators:

- Pings within same VLAN succeed
- Pings across VLANs succeed (flat) or fail (segmented with firewall)
- Trunk port captures show 802.1Q headers
- Access port captures show untagged frames
- Multiple VLAN IDs visible on trunk (10, 20, 30, 40, 50)

## Questions to Consider

1. If an attacker gains access to a trunk port, what additional attack surface does this create?
2. How would you detect unauthorized VLAN access?
3. What is VLAN hopping and how could it be prevented?

## Common Issues

- **Issue**: Cannot see VLAN tags in capture
  - **Solution**: Make sure you are capturing on trunk port between switches, not on access port to device

- **Issue**: Wireshark does not open
  - **Solution**: First time may require accepting Edgeshark prompt. Close and retry.

- **Issue**: Pings fail within same VLAN
  - **Solution**: Check lab is fully deployed. Run `scripts/lab-test.py ot-sec-flat` to verify.

---

## Solution

??? success "Click to reveal solution"
    ### Task 1: Connectivity testing

    SSH to wago-plc2a-vlan10:

    ```bash
    ssh admin@192.168.100.53
    ```

    Ping within VLAN 10 - SUCCEEDS:

    ```bash
    ping -c 4 10.10.0.12
    ```

    Expected output:

    ```text
    PING 10.10.0.12 (10.10.0.12) 56(84) bytes of data.
    64 bytes from 10.10.0.12: icmp_seq=1 ttl=64 time=0.521 ms
    64 bytes from 10.10.0.12: icmp_seq=2 ttl=64 time=0.389 ms
    64 bytes from 10.10.0.12: icmp_seq=3 ttl=64 time=0.412 ms
    64 bytes from 10.10.0.12: icmp_seq=4 ttl=64 time=0.398 ms

    --- 10.10.0.12 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss
    ```

    Ping to VLAN 20 - SUCCESS (flat) or FAIL (segmented with firewall):

    ```bash
    ping -c 4 10.20.0.11
    ```

    In `ot-sec-flat`: Succeeds because no firewall blocks inter-VLAN traffic.
    In `ot-sec-segmented`: May fail if firewall rules block this communication.

    ### Task 2: Trunk port capture

    In Wireshark on trunk link between switches, with filter `vlan`:

    **Observed**: 802.1Q headers visible with VLAN ID 10 for traffic from VLAN 10 devices.

    Frame structure:

    ```text
    Ethernet II
        Destination: [MAC]
        Source: [MAC]
        Type: 802.1Q (0x8100)
    802.1Q Virtual LAN
        Priority: 0
        CFI: 0
        ID: 10
        Type: IPv4 (0x0800)
    ```

    ### Task 3: Access port capture

    On access port to `wago-plc2a-vlan10`, filter `vlan` shows NO results.

    Frames are untagged. Switch adds VLAN tag when forwarding to trunk, removes it when forwarding to access port.

    **Answers to questions**:

    - Trunk ports show multiple VLANs: 10, 20, 30, 40, 50
    - VLAN tag is inserted after source MAC, before EtherType
    - Access ports have no VLAN tags - switch handles tagging
    - This allows end devices to be VLAN-unaware
