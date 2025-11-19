# Exercise 2: ARP Spoofing Attack

**Time Estimate**: 30-40 minutes  
**Lab Topology**: `ot-sec-flat` (recommended) or `ot-sec-segmented`  
**Difficulty**: :material-signal-cellular-2: Intermediate

## Learning Objectives

- Execute a Layer 2 man-in-the-middle attack
- Understand ARP protocol vulnerabilities
- Capture cleartext traffic during MITM attack
- Learn detection and mitigation strategies

## Prerequisites

- Understanding of ARP protocol
- Basic knowledge of network attacks
- Wireshark experience
- Root/sudo concepts

## Scenario

ARP (Address Resolution Protocol) maps IP addresses to MAC addresses on local networks. Because ARP has no authentication, an attacker can send false ARP replies to poison the ARP cache of target devices. This redirects traffic through the attacker's system, enabling man-in-the-middle attacks. In OT environments, this can intercept PLC communications, engineering workstation traffic, or HMI connections.

## Tasks

### Task 1: Observe normal ARP behavior

1. SSH into victim PLC `wago-plc2b-vlan10`:

```bash
ssh admin@192.168.100.54
```

!!! info "Default Credentials"
    Password: `admin`

2. View current ARP table:

```bash
arp -n
```

3. Note the MAC address for gateway `10.10.0.1`

4. Start continuous ping to external network:

```bash
ping 1.1.1.1
```

Leave this running (do not press Ctrl+C).

### Task 2: Set up traffic capture

1. Open new terminal/SSH session
2. SSH into attacker PLC `wago-plc2a-vlan10`:

```bash
ssh admin@192.168.100.53
```

3. Start Wireshark capture on link between `wago-plc2a-vlan10` and `sw-acc1`
4. Apply filter:

```wireshark
icmp
```

**Observation**: Currently you should see NO traffic because ping is not going through the attacker.

### Task 3: Execute ARP spoofing attack

1. On attacker (`wago-plc2a-vlan10`), run the ARP spoofing script:

```bash
sudo /home/admin/arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
```

Parameters:

- `eth1` - interface connected to network
- `10.10.0.12` - victim IP (wago-plc2b-vlan10)
- `10.10.0.1` - gateway IP

**Expected output**:

```text
[*] Enabling IP forwarding...
[*] Poisoning 10.10.0.12 (telling it we are 10.10.0.1)...
[*] Poisoning 10.10.0.1 (telling it we are 10.10.0.12)...
```

2. Check Wireshark - you should now see ICMP echo requests and replies flowing through attacker

3. On victim, check ARP table again:

```bash
arp -n
```

**Question**: Has the MAC address for `10.10.0.1` changed? Compare to original.

### Task 4: Capture cleartext credentials

1. Stop the ARP spoofing script (Ctrl+C on attacker)
2. On victim, start netcat connection:

```bash
echo "username:admin password:secretpass123" | nc 10.20.0.11 1234
```

3. On attacker, restart ARP spoofing:

```bash
sudo /home/admin/arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
```

4. From victim, send credentials again:

```bash
echo "username:admin password:secretpass123" | nc 10.20.0.11 1234
```

5. In Wireshark, remove ICMP filter and look for TCP traffic
6. Follow TCP stream to see cleartext data

## Verification

Success indicators:

- Attacker sees victim's ping traffic in Wireshark
- Victim's ARP table shows attacker's MAC for gateway
- Script output shows poisoning messages
- TCP stream reveals cleartext credentials

## Questions to Consider

1. How can you detect ARP spoofing on a production network?
2. What mitigation strategies exist (DAI, port security, static ARP)?
3. Why is this attack particularly dangerous in OT environments?
4. What protocols are vulnerable to MITM credential theft?

## Common Issues

- **Issue**: Script fails with "must be run as root"
  - **Solution**: Use `sudo` before the script command

- **Issue**: No traffic visible in Wireshark during attack
  - **Solution**: Check IP forwarding is enabled (script does this automatically)

- **Issue**: Attack does not work in segmented topology
  - **Solution**: Firewall rules may block inter-VLAN traffic. Use flat topology for easier testing.

## Advanced Challenge

Use `dsniff` tool to automatically capture credentials:

```bash
sudo dsniff -i eth1
```

While running, generate HTTP or FTP traffic and observe automatic credential extraction.

---

## Solution

??? success "Click to reveal solution"
    ### Pre-attack state

    Victim's ARP table before attack:

    ```bash
    arp -n
    ```

    Output shows:

    ```text
    Address          HWtype  HWaddress           Flags Mask   Iface
    10.10.0.1        ether   02:42:ac:14:00:0b   C            eth1
    ```

    Note this MAC address.

    ### Attack execution

    From attacker `wago-plc2a-vlan10`:

    ```bash
    sudo /home/admin/arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
    ```

    Output:

    ```text
    [*] Enabling IP forwarding...
    [*] Poisoning 10.10.0.12 (telling it we are 10.10.0.1)...
    [*] Poisoning 10.10.0.1 (telling it we are 10.10.0.12)...
    ```

    Script continues running.

    ### Verification

    On victim, check ARP again:

    ```bash
    arp -n
    ```

    MAC for `10.10.0.1` now shows attacker's MAC instead of real gateway MAC.

    In Wireshark on attacker's interface, ICMP traffic from victim now visible.

    ### Detection methods

    1. Monitor ARP tables for unexpected changes
    2. Detect gratuitous ARP packets
    3. Use ARP monitoring tools (arpwatch)
    4. Compare MAC-to-IP mappings against known good state

    ### Mitigation strategies

    1. Dynamic ARP Inspection (DAI) on switches
    2. Port security limiting MAC addresses per port
    3. Static ARP entries for critical infrastructure
    4. Network segmentation reducing attack scope
    5. 802.1X authentication before network access
