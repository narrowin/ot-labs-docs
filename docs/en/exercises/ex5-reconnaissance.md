# Exercise 5: Remote Access and Security Assessment

**Time Estimate**: 35-45 minutes  
**Lab Topology**: `ot-sec-flat` (easier) or `ot-sec-segmented` (realistic)  
**Difficulty**: :material-signal-cellular-2: Intermediate

## Learning Objectives

- Conduct remote access testing from DMZ jumphost
- Identify exposed services via port scanning
- Discover potential security vulnerabilities
- Practice professional security assessment methodology

## Prerequisites

- Understanding of TCP/IP fundamentals
- Basic nmap knowledge
- SSH and service concepts
- Security assessment awareness

## Scenario

A security assessor has remote access to the jumphost in the DMZ. The objective is to map the OT network, identify running services, and document potential security issues. This simulates real-world remote access scenarios where engineers or attackers gain initial foothold and attempt to expand access into OT zones.

## Tasks

### Task 1: Initial remote access from jumphost

1. SSH to jumphost:

```bash
ssh admin@192.168.100.52
```

!!! info "Default Credentials"
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

### Task 2: Host discovery

1. Scan VLAN 10 (Hygiene) for live hosts:

```bash
nmap -sn 10.10.0.0/24
```

`-sn` performs ping scan (no port scan)

**Expected output**:

```text
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

### Task 3: Service identification

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

```text
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

### Task 4: Access testing

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

### Task 5: Document findings

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

## Verification

Success indicators:

- Host discovery identifies live PLCs
- Service scans reveal open ports
- SSH access tested (may succeed or fail depending on firewall)
- HTTP interfaces discovered
- Findings documented

## Questions to Consider

1. What services should never be directly accessible from DMZ?
2. How would you restrict jumphost access in production?
3. What additional security controls would improve this architecture?
4. How could these remote access activities be detected?

## Common Issues

- **Issue**: nmap command not found
  - **Solution**: Use `apt-get update && apt-get install nmap` or tool is pre-installed

- **Issue**: All hosts appear down
  - **Solution**: In segmented topology, firewall may block ICMP. Use `-Pn` flag to skip ping.

- **Issue**: Cannot reach any 10.x networks
  - **Solution**: Check routing with `ip route`. Verify lab deployed correctly.

## Advanced Challenge

1. Write a shell script to automate discovery of all VLANs
2. Generate a CSV report of all discovered services
3. Test for default credentials on discovered services
4. Create network diagram based on discovered topology

---

## Solution

??? success "Click to reveal solution"
    ### Host discovery results

    From jumphost:

    ```bash
    nmap -sn 10.10.0.0/24
    ```

    Expected discoveries in flat topology:

    ```text
    10.10.0.1 (gateway)
    10.10.0.11 (wago-plc2a-vlan10)
    10.10.0.12 (wago-plc2b-vlan10)
    ```

    Segmented topology may show fewer hosts due to firewall rules.

    ### Service scan results

    ```bash
    nmap -sV -p22,80,502,8080,2455 10.10.0.11
    ```

    Expected output:

    ```text
    PORT     STATE SERVICE  VERSION
    22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu
    8080/tcp open  http     nginx 1.18.0
    ```

    Other ports may be filtered or closed depending on PLC configuration.

    ### Access testing

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

    ### Security findings

    Typical issues discovered:

    1. PLCs accessible via SSH with default credentials
    2. Web interfaces exposed without authentication
    3. Modbus TCP accessible without authentication
    4. No network segmentation (flat topology)
    5. Default passwords in use
    6. Unnecessary services running

    ### Recommended mitigations

    1. Implement zero-trust network segmentation
    2. Require multi-factor authentication for remote access
    3. Disable unused services on PLCs
    4. Change default credentials
    5. Implement application-aware firewall rules
    6. Deploy IDS/IPS for anomaly detection
    7. Require VPN for remote access
    8. Implement jump host hardening
