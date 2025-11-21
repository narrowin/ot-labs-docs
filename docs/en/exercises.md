# OT Security Lab Exercises

## Overview

These hands-on exercises provide practical experience with OT network security concepts. Each exercise focuses on fundamental network security principles commonly encountered in industrial control system environments.

**Total Time**: 2-3 hours for all exercises

**Lab Requirements**: You need a running lab environment (GitHub Codespaces or DevPod). See the [Lab Environment](lab.md) page for setup instructions.

**Exercise Approach**: Exercises are independent and can be completed in any order, though following the sequence provides a logical progression of concepts.

## Prerequisites

### Required Knowledge

- Basic Linux command line
- Network fundamentals (IP addressing, VLANs, routing)
- SSH basics
- Wireshark basics

### Lab Access

Before starting, ensure you can:

1. Access your lab environment (Codespaces or DevPod)
2. Deploy a lab topology using Containerlab extension
3. SSH into lab devices
4. Capture traffic using Wireshark via Edgeshark

### Topology Selection

These exercises use two lab topologies:

- `ot-sec-flat.clab.yml` - Flat network with VLANs but minimal segmentation
- `ot-sec-segmented.clab.yml` - Segmented network with firewall and bonding

Each exercise specifies which topology to use.

## Exercise List

| Exercise | Topic | Time | Topology | Difficulty |
|----------|-------|------|----------|------------|
| 1 | VLAN Discovery | 20-30 min | Both | Beginner |
| 2 | ARP Spoofing Attack | 30-40 min | Both (flat easier) | Intermediate |
| 3 | Spanning Tree Analysis | 25-35 min | Both | Intermediate |
| 4 | Link Aggregation | 20-30 min | Segmented only | Intermediate |
| 5 | Remote Access Assessment | 35-45 min | Both | Intermediate |

---

## Exercise 1: VLAN Discovery and Traffic Analysis

**Time Estimate**: 20-30 minutes

**Lab Topology**: `ot-sec-flat` or `ot-sec-segmented`

**Difficulty**: Beginner

### Learning Objectives

- Understand 802.1Q VLAN tagging on trunk ports
- Differentiate between trunk and access port behavior
- Recognize VLAN isolation in packet captures
- Identify security implications of VLAN segmentation

### Prerequisites

- Lab deployed and running
- Basic Wireshark knowledge
- Understanding of ping command

### Scenario

Industrial networks use VLANs to logically segment different operational zones. In this lab, VLANs separate hygiene systems (VLAN 10), process control (VLAN 20), disposal systems (VLAN 30), control room (VLAN 40), and engineering workstations (VLAN 50). Understanding how VLANs work and how to identify them in traffic is essential for OT security monitoring.

### Tasks

#### Task 1: Verify PLC connectivity within same VLAN

1. SSH into `wago-plc2a-vlan10`:

```bash
ssh admin@192.168.100.53
```

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

#### Task 2: Capture traffic on trunk port between switches

1. In Containerlab extension TopoViewer, right-click on the link between `sw-dist` and `sw-acc1`
2. Select capture from `sw-dist` side
3. Wireshark opens via Edgeshark

4. Apply Wireshark filter:

```
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

#### Task 3: Capture traffic on access port

1. Stop the previous capture
2. Right-click on link between `sw-acc1` and `wago-plc2a-vlan10`
3. Select capture from switch side
4. Generate traffic again:

```bash
ping -c 10 10.10.0.1
```

5. Apply same filter: `vlan`

**Questions**:

- Do you see any VLAN tags now?
- What is the difference between trunk and access ports regarding VLAN tags?
- Why does the switch remove VLAN tags on access ports?

### Verification

Success indicators:

- Pings within same VLAN succeed
- Pings across VLANs succeed (flat) or fail (segmented with firewall)
- Trunk port captures show 802.1Q headers
- Access port captures show untagged frames
- Multiple VLAN IDs visible on trunk (10, 20, 30, 40, 50)

### Questions to Consider

1. If an attacker gains access to a trunk port, what additional attack surface does this create?
2. How would you detect unauthorized VLAN access?
3. What is VLAN hopping and how could it be prevented?

### Common Issues

- **Issue**: Cannot see VLAN tags in capture
  - **Solution**: Make sure you are capturing on trunk port between switches, not on access port to device

- **Issue**: Wireshark does not open
  - **Solution**: First time may require accepting Edgeshark prompt. Close and retry.

- **Issue**: Pings fail within same VLAN
  - **Solution**: Check lab is fully deployed. Run `scripts/lab-test.py ot-sec-flat` to verify.

??? success "Solution: Exercise 1 - VLAN Discovery"
    #### Task 1: Connectivity testing

    SSH to wago-plc2a-vlan10:

    ```bash
    ssh admin@192.168.100.53
    ```

    Ping within VLAN 10 - SUCCEEDS:

    ```bash
    ping -c 4 10.10.0.12
    ```

    Expected output:

    ```
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

    #### Task 2: Trunk port capture

    In Wireshark on trunk link between switches, with filter `vlan`:

    **Observed**: 802.1Q headers visible with VLAN ID 10 for traffic from VLAN 10 devices.

    Frame structure:

    ```
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

    #### Task 3: Access port capture

    On access port to `wago-plc2a-vlan10`, filter `vlan` shows NO results.

    Frames are untagged. Switch adds VLAN tag when forwarding to trunk, removes it when forwarding to access port.

    **Answers to questions**:

    - Trunk ports show multiple VLANs: 10, 20, 30, 40, 50
    - VLAN tag is inserted after source MAC, before EtherType
    - Access ports have no VLAN tags - switch handles tagging
    - This allows end devices to be VLAN-unaware

---

## Exercise 2: ARP Spoofing Attack

**Time Estimate**: 30-40 minutes

**Lab Topology**: `ot-sec-flat` (recommended) or `ot-sec-segmented`

**Difficulty**: Intermediate

### Learning Objectives

- Execute a Layer 2 man-in-the-middle attack
- Understand ARP protocol vulnerabilities
- Capture cleartext traffic during MITM attack
- Learn detection and mitigation strategies

### Prerequisites

- Understanding of ARP protocol
- Basic knowledge of network attacks
- Wireshark experience
- Root/sudo concepts

### Scenario

ARP (Address Resolution Protocol) maps IP addresses to MAC addresses on local networks. Because ARP has no authentication, an attacker can send false ARP replies to poison the ARP cache of target devices. This redirects traffic through the attacker's system, enabling man-in-the-middle attacks. In OT environments, this can intercept PLC communications, engineering workstation traffic, or HMI connections.

### Tasks

#### Task 1: Observe normal ARP behavior

1. SSH into victim PLC `wago-plc2b-vlan10`:

```bash
ssh admin@192.168.100.54
```

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

#### Task 2: Set up traffic capture

1. Open new terminal/SSH session
2. SSH into attacker PLC `wago-plc2a-vlan10`:

```bash
ssh admin@192.168.100.53
```

3. Start Wireshark capture on link between `wago-plc2a-vlan10` and `sw-acc1`
4. Apply filter:

```
icmp
```

**Observation**: Currently you should see NO traffic because ping is not going through the attacker.

#### Task 3: Execute ARP spoofing attack

1. On attacker (`wago-plc2a-vlan10`), run the ARP spoofing script:

```bash
sudo /home/admin/arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
```

Parameters:

- `eth1` - interface connected to network
- `10.10.0.12` - victim IP (wago-plc2b-vlan10)
- `10.10.0.1` - gateway IP

**Expected output**:

```
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

#### Task 4: Capture cleartext credentials

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

### Verification

Success indicators:

- Attacker sees victim's ping traffic in Wireshark
- Victim's ARP table shows attacker's MAC for gateway
- Script output shows poisoning messages
- TCP stream reveals cleartext credentials

### Questions to Consider

1. How can you detect ARP spoofing on a production network?
2. What mitigation strategies exist (DAI, port security, static ARP)?
3. Why is this attack particularly dangerous in OT environments?
4. What protocols are vulnerable to MITM credential theft?

### Common Issues

- **Issue**: Script fails with "must be run as root"
  - **Solution**: Use `sudo` before the script command

- **Issue**: No traffic visible in Wireshark during attack
  - **Solution**: Check IP forwarding is enabled (script does this automatically)

- **Issue**: Attack does not work in segmented topology
  - **Solution**: Firewall rules may block inter-VLAN traffic. Use flat topology for easier testing.

### Advanced Challenge

Use `dsniff` tool to automatically capture credentials:

```bash
sudo dsniff -i eth1
```

While running, generate HTTP or FTP traffic and observe automatic credential extraction.

??? success "Solution: Exercise 2 - ARP Spoofing"
    #### Pre-attack state

    Victim's ARP table before attack:

    ```bash
    arp -n
    ```

    Output shows:

    ```
    Address          HWtype  HWaddress           Flags Mask   Iface
    10.10.0.1        ether   02:42:ac:14:00:0b   C            eth1
    ```

    Note this MAC address.

    #### Attack execution

    From attacker `wago-plc2a-vlan10`:

    ```bash
    sudo /home/admin/arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
    ```

    Output:

    ```
    [*] Enabling IP forwarding...
    [*] Poisoning 10.10.0.12 (telling it we are 10.10.0.1)...
    [*] Poisoning 10.10.0.1 (telling it we are 10.10.0.12)...
    ```

    Script continues running.

    #### Verification

    On victim, check ARP again:

    ```bash
    arp -n
    ```

    MAC for `10.10.0.1` now shows attacker's MAC instead of real gateway MAC.

    In Wireshark on attacker's interface, ICMP traffic from victim now visible.

    #### Detection methods

    1. Monitor ARP tables for unexpected changes
    2. Detect gratuitous ARP packets
    3. Use ARP monitoring tools (arpwatch)
    4. Compare MAC-to-IP mappings against known good state

    #### Mitigation strategies

    1. Dynamic ARP Inspection (DAI) on switches
    2. Port security limiting MAC addresses per port
    3. Static ARP entries for critical infrastructure
    4. Network segmentation reducing attack scope
    5. 802.1X authentication before network access

---

## Exercise 3: Spanning Tree Protocol Analysis

**Time Estimate**: 25-35 minutes

**Lab Topology**: `ot-sec-flat` or `ot-sec-segmented`

**Difficulty**: Intermediate

### Learning Objectives

- Understand Spanning Tree Protocol operation
- Identify root bridge and port states
- Observe STP BPDU exchange
- (Segmented only) Analyze LACP bonding with STP

### Prerequisites

- Understanding of Layer 2 switching
- Knowledge of STP/RSTP concepts
- Basic MikroTik RouterOS commands
- SSH access to switches

### Scenario

Spanning Tree Protocol (STP) prevents loops in switched networks by blocking redundant paths. Industrial networks use RSTP (Rapid Spanning Tree Protocol) for faster convergence. Both lab topologies run STP on all switches. The segmented lab additionally uses LACP (Link Aggregation Control Protocol) bonding between switches, which works in coordination with STP for both redundancy and increased bandwidth.

### Tasks

#### Task 1: Identify switch topology

1. SSH into distribution switch:

```bash
ssh admin@192.168.100.12
```

Password: `admin`

2. View bridge configuration:

```
/interface bridge print
```

3. List all bridge ports:

```
/interface bridge port print
```

**Questions**:

- What is the bridge name?
- Which interfaces are bridge members?
- Which interfaces have VLAN filtering enabled?

#### Task 2: Examine link topology

1. Check which interfaces connect to other switches:

```
/interface bridge port print
```

2. **Segmented topology only**: Check bonding configuration:

```
/interface bonding print detail
```

**Expected in segmented**:

- Bond name: `bond-sw-acc1`
- Mode: `802.3ad` (LACP)
- Slaves: `ether9`, `ether10`

**Expected in flat**:

- No bonding interfaces
- Single link: ether10 connects to sw-acc1

**Questions**:

- How many links connect sw-dist to sw-acc1?
- (Segmented) Are both bond members active?

#### Task 3: Capture BPDU frames

1. In TopoViewer, start Wireshark capture on link between `sw-dist` and `sw-acc1`
2. Apply filter:

```
stp
```

Or alternatively:

```
eth.dst == 01:80:c2:00:00:00
```

**Observations**:

- BPDUs sent to multicast address `01:80:c2:00:00:00`
- Root bridge identifier
- Port roles (designated, root)
- Bridge priority values

3. Check bridge settings on both switches:

On `sw-dist`:

```
/interface bridge print detail
```

On `sw-acc1` (SSH to 192.168.100.13):

```
/interface bridge print detail
```

**Question**: Which switch has lower priority and becomes root bridge?

#### Task 4: Understand redundancy approach

**In flat topology**:

- Single link between switches
- STP prevents loops between other switch connections
- No link-level redundancy between sw-dist and sw-acc1

**In segmented topology**:

- LACP bond provides link redundancy
- STP still runs on other ports
- Two complementary redundancy mechanisms

From distribution switch, check configuration:

```
/interface ethernet print
/interface bridge port print
```

**Question**: What happens if the single uplink fails in flat topology?

### Verification

Success indicators:

- Bridge configuration shows vlan-filtering enabled
- BPDU frames visible in Wireshark captures
- Root bridge identified from bridge settings
- (Segmented) Bond interface exists with LACP active
- (Flat) Single link topology documented
- Understanding of how STP prevents loops

### Questions to Consider

1. Why does STP run even when there's no redundant path (flat)?
2. (Segmented) What happens if one bond member fails?
3. How quickly does STP reconverge after topology change?
4. What security risks does STP present (BPDU attacks)?

### Common Issues

- **Issue**: No bonding interface visible in flat
  - **Solution**: Correct - flat topology has no bonding, only single links

- **Issue**: Cannot SSH to switches
  - **Solution**: Use management IPs 192.168.100.11-14, not data plane IPs

- **Issue**: No BPDUs in capture
  - **Solution**: BPDUs are sent periodically (every 2 seconds). Wait or generate topology change.

- **Issue**: Exercise says "segmented only" but I'm in flat
  - **Solution**: Exercise 3 works in both topologies - bonding tasks are marked as segmented-only within the exercise

??? success "Solution: Exercise 3 - STP Analysis"
    #### Bridge configuration

    SSH to `sw-dist`:

    ```bash
    ssh admin@192.168.100.12
    ```

    View bridge:

    ```
    /interface bridge print
    ```

    Output:

    ```
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
    
    **Segmented topology** shows ether12, bond-sw-acc1, ether8, ether2 as bridge members with VLAN settings.

    #### Bonding verification (Segmented topology only)

    ```
    /interface bonding print detail
    ```

    Output shows:

    ```
    0   name="bond-sw-acc1" mtu=auto mode=802.3ad primary=none 
        slaves=ether9,ether10 link-monitoring=mii arp-interval=100ms 
        lacp-rate=30secs transmit-hash-policy=layer-2-and-3
    ```

    Monitor status:

    ```
    /interface bonding monitor bond-sw-acc1 once
    ```

    Output:

    ```
                  mode: 802.3ad
                active: yes
          active-ports: ether9 ether10
       lacp-negotiated: yes
    ```

    In flat topology, bonding is not used - each switch connects via a single physical interface.

    #### Root bridge identification

    Check priority on both switches. Lower priority wins. Default priority is 32768.

    BPDU captures show root bridge ID in protocol fields.

---

## Exercise 4: Link Aggregation and Load Balancing

**Time Estimate**: 20-30 minutes

**Lab Topology**: `ot-sec-segmented` only

**Difficulty**: Intermediate

### Learning Objectives

- Analyze LACP link aggregation operation
- Observe traffic distribution across bonded links
- Test link failover behavior
- Understand load balancing algorithms

### Prerequisites

- Completed Exercise 3 (STP Analysis)
- Understanding of LACP concepts
- Wireshark experience
- Basic MikroTik commands

### Scenario

Link aggregation combines multiple physical links into one logical link for increased bandwidth and redundancy. LACP (802.3ad) dynamically negotiates bonding. Traffic is distributed across member links using a hash algorithm based on source/destination MAC, IP, or ports. Understanding how this works is important for capacity planning and troubleshooting in OT networks.

### Tasks

#### Task 1: Verify LACP negotiation

1. SSH to `sw-dist`:

```bash
ssh admin@192.168.100.12
```

2. Monitor LACP protocol operation:

```
/interface bonding monitor bond-sw-acc1 once
```

**Expected output includes**:

- `lacp-partner-system-id` - MAC of partner switch
- `active-ports` - Number of active links
- `lacp-negotiated` - True/false

3. Check detailed bonding configuration:

```
/interface bonding print detail where name=bond-sw-acc1
```

**Questions**:

- What is the bonding mode?
- Which physical interfaces are slaves?
- What is the transmit hash policy?

#### Task 2: Observe traffic distribution

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

#### Task 3: Test hash algorithm

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

```
/interface monitor-traffic ether9,ether10
```

**Question**: Are different source IPs distributed to different links?

#### Task 4: Test link failover

1. Keep pings running from previous step
2. On `sw-dist`, disable one bond member:

```
/interface ethernet disable ether9
```

3. Observe:
   - Do pings continue without interruption?
   - How quickly does failover occur?
   - Check Wireshark on remaining active link

4. Re-enable the interface:

```
/interface ethernet enable ether9
```

5. Check bond status:

```
/interface bonding monitor bond-sw-acc1 once
```

**Question**: Does both links return to active state automatically?

### Verification

Success indicators:

- LACP shows negotiated status
- Both bond members active
- Traffic visible on member links
- Failover occurs in under 1 second
- Disabled link automatically rejoins when enabled
- Packet count increases on both interfaces

### Questions to Consider

1. Why might traffic use only one link for a specific flow?
2. What are advantages and disadvantages of different hash algorithms?
3. How does LACP compare to static bonding?
4. What happens if LACP hello packets are lost?

### Common Issues

- **Issue**: Traffic only uses one link
  - **Solution**: Hash algorithm makes per-flow decisions. Single source/destination uses one link. Use multiple sources to see distribution.

- **Issue**: Cannot monitor traffic on interface
  - **Solution**: Use `/interface monitor-traffic ether9,ether10` command format

- **Issue**: Failover takes several seconds
  - **Solution**: This is normal for LACP. RSTP must also reconverge.

??? success "Solution: Exercise 4 - Link Aggregation"
    #### Traffic distribution observation

    With single source ping, traffic typically uses one link consistently due to hash algorithm using source/destination IPs.

    With multiple sources (different source IPs), traffic distributes across links.

    Hash policy `layer-2-and-3` uses source/dest MAC and IP addresses.

    #### Packet count verification

    ```
    /interface monitor-traffic ether9,ether10
    ```

    Shows rx-packets-per-second and tx-packets-per-second for each interface.

    With diverse traffic, both interfaces show increasing packet counts.

    #### Failover test

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

---

## Exercise 5: Remote Access and Security Assessment

**Time Estimate**: 35-45 minutes

**Lab Topology**: `ot-sec-flat` (easier) or `ot-sec-segmented` (realistic)

**Difficulty**: Intermediate

### Learning Objectives

- Conduct remote access testing from DMZ jumphost
- Identify exposed services using port scanning
- Discover potential security vulnerabilities
- Practice professional security assessment methodology

### Prerequisites

- Understanding of TCP/IP fundamentals
- Basic nmap knowledge
- SSH and service concepts
- Security assessment awareness

### Scenario

A security assessor has remote access to the jumphost in the DMZ. The objective is to map the OT network, identify running services, and document potential security issues. This simulates real-world remote access scenarios where engineers or attackers gain initial foothold and attempt to expand access into OT zones.

### Tasks

#### Task 1: Initial remote access from jumphost

1. SSH to jumphost:

```bash
ssh admin@192.168.100.52
```

Password: `admin`

2. Check your network position:

```bash
ip addr show
ip route show
```

**Questions**:

- What IP address does the jumphost have?
- What networks are reachable according to routing table?

3. Test basic connectivity:

```bash
ping -c 3 10.10.0.1
ping -c 3 10.10.0.11
```

**Note**: In segmented topology, firewall may block some traffic.

#### Task 2: Host discovery

1. Scan VLAN 10 (Hygiene) for live hosts:

```bash
nmap -sn 10.10.0.0/24
```

`-sn` performs ping scan (no port scan)

**Expected output**:

```
Nmap scan report for 10.10.0.1
Host is up.

Nmap scan report for 10.10.0.11
Host is up.

Nmap scan report for 10.10.0.12
Host is up.
```

2. Try other VLANs:

```bash
nmap -sn 10.20.0.0/24
nmap -sn 10.40.0.0/24
```

**Question**: Which VLANs are accessible from jumphost?

#### Task 3: Service identification

1. Scan common OT ports on discovered PLC:

```bash
nmap -sV -p22,80,502,8080,2455 10.10.0.11
```

Port reference:

- 22: SSH
- 80: HTTP
- 502: Modbus TCP
- 8080: CODESYS WebVisu
- 2455: CODESYS runtime

**Expected output**:

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.x
8080/tcp open  http    nginx
```

2. Scan all PLCs in VLAN 10:

```bash
nmap -p8080 10.10.0.11-12
```

**Questions**:

- Which services are exposed?
- Are any services running that should not be?
- What can you infer about the devices from service banners?

#### Task 4: Access testing

1. Test SSH access to PLC:

```bash
ssh admin@10.10.0.11
```

Try password: `admin`

**Question**: Does this succeed? Should PLCs be directly accessible via SSH from jumphost?

2. Test web interface access:

```bash
curl -I http://10.10.0.11:8080
```

**Observation**: Check HTTP response headers for version information.

3. If `nc` available, test Modbus port:

```bash
nc -zv 10.10.0.11 502
```

#### Task 5: Document findings

Create a simple report of findings:

```bash
cat > assessment_findings.txt << 'EOF'
OT Network Security Assessment

Date: $(date)
Assessor: $(whoami)

FINDINGS:

1. Accessible Networks:
   - VLAN 10: [list discovered hosts]
   - VLAN 20: [accessible? yes/no]

2. Exposed Services:
   - Host 10.10.0.11:
     * Port 22 (SSH): OPEN
     * Port 8080 (HTTP): OPEN
     * Port 502 (Modbus): [status]

3. Security Issues:
   - [List any concerns]

4. Recommendations:
   - [Suggested mitigations]
EOF

cat assessment_findings.txt
```

### Verification

Success indicators:

- Host discovery identifies live PLCs
- Service scans reveal open ports
- SSH access tested (may succeed or fail depending on firewall)
- HTTP interfaces discovered
- Findings documented

### Questions to Consider

1. What services should never be directly accessible from DMZ?
2. How would you restrict jumphost access in production?
3. What additional security controls would improve this architecture?
4. How could these remote access activities be detected?

### Common Issues

- **Issue**: nmap command not found
  - **Solution**: Use `apt-get update && apt-get install nmap` or tool is pre-installed

- **Issue**: All hosts appear down
  - **Solution**: In segmented topology, firewall may block ICMP. Use `-Pn` flag to skip ping.

- **Issue**: Cannot reach any 10.x networks
  - **Solution**: Check routing with `ip route`. Verify lab deployed correctly.

### Advanced Challenge

1. Write a shell script to automate discovery of all VLANs
2. Generate a CSV report of all discovered services
3. Test for default credentials on discovered services
4. Create network diagram based on discovered topology

??? success "Solution: Exercise 5 - Remote Access"
    #### Host discovery results

    From jumphost:

    ```bash
    nmap -sn 10.10.0.0/24
    ```

    Expected discoveries in flat topology:

    ```
    10.10.0.1 (gateway)
    10.10.0.11 (wago-plc2a-vlan10)
    10.10.0.12 (wago-plc2b-vlan10)
    ```

    Segmented topology may show fewer hosts due to firewall rules.

    #### Service scan results

    ```bash
    nmap -sV -p22,80,502,8080,2455 10.10.0.11
    ```

    Expected output:

    ```
    PORT     STATE SERVICE  VERSION
    22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu
    8080/tcp open  http     nginx 1.18.0
    ```

    Other ports may be filtered or closed depending on PLC configuration.

    #### Access testing

    SSH to PLC:

    ```bash
    ssh admin@10.10.0.11
    ```

    In flat topology: Likely succeeds with password `admin`.
    In segmented topology: May be blocked by firewall rules.

    Web interface:

    ```bash
    curl -I http://10.10.0.11:8080
    ```

    Returns HTTP headers showing CODESYS WebVisu interface.

    #### Security findings

    Typical issues discovered:

    1. PLCs accessible via SSH with default credentials
    2. Web interfaces exposed without authentication
    3. Modbus TCP accessible without authentication
    4. No network segmentation (flat topology)
    5. Default passwords in use
    6. Unnecessary services running

    #### Recommended mitigations

    1. Implement zero-trust network segmentation
    2. Require multi-factor authentication for remote access
    3. Disable unused services on PLCs
    4. Change default credentials
    5. Implement application-aware firewall rules
    6. Deploy IDS/IPS for anomaly detection
    7. Require VPN for remote access
    8. Implement jump host hardening

---

## Appendix: Wireshark Filters Quick Reference

### VLAN Analysis

```
vlan                          # All 802.1Q tagged frames
vlan.id == 10                 # Only VLAN 10 traffic
vlan.id in {10 20 30}        # Multiple VLANs
```

### ARP Analysis

```
arp                           # All ARP traffic
arp.opcode == 1               # ARP requests only
arp.opcode == 2               # ARP replies only
arp.duplicate-address-detected  # ARP conflicts
```

### STP/RSTP

```
stp                           # All spanning tree
rstp                          # Rapid spanning tree only
eth.dst == 01:80:c2:00:00:00  # STP multicast address
```

### Protocol-Specific

```
icmp                          # ICMP (ping) traffic
tcp.port == 502               # Modbus TCP
tcp.port == 2455              # CODESYS
tcp.port == 8080              # Web interfaces
```

### Combining Filters

```
vlan.id == 10 && icmp         # VLAN 10 ICMP traffic
tcp && !tcp.port == 22        # All TCP except SSH
ip.src == 10.10.0.11 || ip.dst == 10.10.0.11  # Specific host
```

---

## Troubleshooting Common Issues

### Lab Not Starting

**Symptom**: Containers fail to start or remain unhealthy

**Solutions**:

1. Check container status: `docker ps -a | grep clab`
2. View container logs: `docker logs <container-name>`
3. Verify images pulled: `docker images | grep narrowin`
4. Run health check: `scripts/lab-test.py ot-sec-flat`

### Cannot SSH to Devices

**Symptom**: Connection refused or timeout

**Solutions**:

1. Verify using management IP (192.168.100.x), not data plane IP
2. Wait 2-3 minutes after lab deploy for SSH to start
3. Check container is running: `docker ps | grep <device-name>`
4. Test from VS Code terminal, not external system

### Wireshark Not Opening

**Symptom**: Edgeshark does not launch

**Solutions**:

1. Accept VS Code prompt to open with Edgeshark
2. Close and retry capture
3. Check Edgeshark extension installed
4. View container logs for errors

### No Traffic in Captures

**Symptom**: Wireshark shows no packets

**Solutions**:

1. Verify capture started before generating traffic
2. Check capturing on correct interface/link
3. Remove display filters temporarily
4. Verify devices can actually communicate (ping test)

### Ping Failures

**Symptom**: Destination unreachable

**Solutions**:

1. Check routing: `ip route show`
2. Verify VLAN assignment
3. In segmented topology, check if firewall blocks traffic
4. Confirm both devices in same VLAN or proper routing configured

---

## Additional Resources

- [Containerlab Documentation](https://containerlab.dev/)
- [MikroTik RouterOS Manual](https://help.mikrotik.com/)
- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [CODESYS Documentation](https://www.codesys.com/)
- [IEC 62443 Standards Overview](https://www.isa.org/standards-and-publications/isa-standards/isa-iec-62443-series-of-standards)

---

**Document Version**: 1.0
**Last Updated**: 2025-11-19
**Lab Compatibility**: ot-sec-flat v1.x, ot-sec-segmented v1.x
