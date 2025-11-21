# Exercise 7: Network Segmentation and Firewall Design

**Time Estimate**: 35-45 minutes  
**Lab Topology**: `ot-sec-segmented` (primary), `ot-sec-flat` (comparison)  
**Difficulty**: :material-signal-cellular-3: Intermediate to Advanced

## Learning Objectives

- Understand zone-based segmentation (Purdue Model concepts)
- Create a communication matrix for OT devices
- Design firewall rules to restrict traffic to necessary protocols
- Compare security posture of flat vs. segmented networks

## Prerequisites

- Exercise 1 & 6
- Basic understanding of firewalls

## Scenario

In the `ot-sec-flat` topology, all devices were in the same broadcast domain. If one device was compromised, the attacker could access everything. In `ot-sec-segmented`, we have VLANs separating functions:
- **VLAN 10 (Hygiene)**: PLCs
- **VLAN 20 (Process)**: PLCs
- **VLAN 40 (Control Room)**: HMI
- **VLAN 50 (Engineering)**: Workstations

However, segmentation alone isn't enough if the firewall allows all traffic between zones. We need to implement "Least Privilege".

## Tasks

### Task 1: Map Communication Flows

1.  Identify the legitimate communication paths needed for the plant to operate:
    *   **HMI (VLAN 40)** needs to read/write data to **PLCs (VLAN 10, 20, 30)**.
    *   **Engineering Station (VLAN 50)** needs to program **PLCs**.
    *   **PLCs** usually do *not* need to talk to the Internet.
    *   **PLCs** might need to talk to each other (inter-controller communication).

2.  **Question**: Does the HMI need to SSH into the PLC? Does it need to ping it? Does it need to access the web server?
    *   *Answer*: Typically, HMI only needs Modbus TCP (502) or a proprietary protocol (e.g., 1217/2455 for CODESYS). WebVisu uses HTTP (80/8080).

### Task 2: Create a Communication Matrix

A communication matrix defines allowed traffic. Fill in the table below (mentally or on paper):

| Source | Destination | Protocol | Port | Action |
| :--- | :--- | :--- | :--- | :--- |
| HMI (10.40.0.12) | PLC (10.10.0.11) | Modbus TCP | 502 | Allow |
| HMI (10.40.0.12) | PLC (10.10.0.11) | HTTP (WebVisu) | 8080 | Allow |
| HMI (10.40.0.12) | PLC (10.10.0.11) | SSH | 22 | **Block** |
| Any | Any | Any | Any | **Block** |

### Task 3: Analyze Current Firewall Rules

1.  SSH into the `gw-firewall` (Mikrotik):
    ```bash
    ssh admin@192.168.100.11
    ```
    (Password: `admin`)

2.  View the current forwarding rules:
    ```bash
    /ip firewall filter print where chain=forward
    ```

3.  Look for the rule allowing Control Room (VLAN 40) to Hygiene (VLAN 10).
    *   It likely looks like: `action=accept chain=forward in-interface=vlan40 out-interface=vlan10`
    *   **Analysis**: This rule allows *all* protocols. This is too permissive.

### Task 4: Design and Implement Restrictive Rules (Optional/Advanced)

*Note: This task requires familiarity with RouterOS syntax. Proceed with caution.*

We want to replace the broad "Allow VLAN 40 to VLAN 10" rule with specific rules for Modbus and HTTP.

#### Expert View: Complexity vs. Maintainability

**Warning**: With firewall rule sets, granularity can get very complex very fast. It's tempting to create extremely fine-grained rules (e.g., separate rules for every single device, every port, every protocol variant), but this leads to rule sets with hundreds of entries that nobody understands or can maintain anymore.

**Common Pitfalls**:

- Too many specific host-to-host rules instead of zone-based rules
- Multiple overlapping rules where it's unclear which one applies
- Missing documentation or outdated comments
- "Emergency" rules that were supposed to be temporary but become permanent
- Rules that nobody dares to delete because it's unknown what they do

**Recommendation: Keep It Simple and Manageable**:

1. **Prefer zone-based policies** over individual hosts (e.g., "VLAN 40 to VLAN 10" instead of "Host A to Host B")
2. **Group service objects** (e.g., "OT-Base-Services" = Modbus + OPC-UA + HTTP)
3. **Clear naming conventions** (e.g., "ALLOW_ControlRoom_to_Hygiene_OT-Services")
4. **Regular cleanup** - Remove unused rules
5. **Document** - Record business justification in comments
6. **Test** - Validate in test environment before production

**Rule of thumb**: If your rule set has more than 50 rules without clear structure, it's time to simplify. A well-designed rule set with 20 understandable rules is safer than a chaotic one with 200 rules.

1.  **Add specific rules** (placed at the top or before the drop rule):

    ```bash
    # Allow Modbus TCP from HMI to PLC
    /ip firewall filter add chain=forward action=accept src-address=10.40.0.12 dst-address=10.10.0.11 protocol=tcp dst-port=502 comment="Allow HMI to PLC Modbus" place-before=0
    
    # Allow HTTP (WebVisu) from HMI to PLC
    /ip firewall filter add chain=forward action=accept src-address=10.40.0.12 dst-address=10.10.0.11 protocol=tcp dst-port=8080 comment="Allow HMI to PLC Web" place-before=0
    ```

2.  **Disable the broad rule**:
    Find the number of the rule `allow Control Room to Hygiene` using `/ip firewall filter print`. Let's assume it is number `X`.
    ```bash
    /ip firewall filter disable X
    ```

3.  **Verify**:
    *   Can you still access the WebVisu from the HMI? (Should be YES)
    *   Can you SSH from the HMI to the PLC? (Should be NO, if the default policy is drop or if no other rule matches).
    *   *Note*: The current lab config has a `default forward drop` rule that is `disabled=yes`. To fully enforce this, you would need to enable that drop rule at the end.

    ```bash
    # Enable the default drop rule at the bottom
    /ip firewall filter enable [find comment="default forward drop"]
    ```

### Task 5: Compare Flat vs. Segmented

1.  **Flat Network**:
    *   Attacker on HMI can ARP spoof the PLC.
    *   Attacker can access any port on the PLC.
    *   No central point to filter traffic.

2.  **Segmented Network (with Firewall)**:
    *   ARP spoofing is contained within the VLAN.
    *   Firewall enforces policy (e.g., "Only Modbus allowed").
    *   Central logging of denied attempts (if logging is enabled).

## Conclusion

Segmentation is the foundation of OT security, but it must be accompanied by strict firewall rules. By moving from "Allow Any" to "Allow Specific Ports", we significantly reduce the attack surface.

---

## Solution

??? success "Click to view solution"
    ### Task 1: Map Communication Flows

    **Legitimate Communication Paths:**

    1. **HMI (VLAN 40) → PLCs (VLAN 10, 20, 30)**
        - Modbus TCP (Port 502): YES - for process control
        - HTTP/WebVisu (Port 8080): YES - for visualization
        - OPC UA (Port 4840): YES - modern alternative
        - SSH (Port 22): NO - only for engineering, not HMI

    2. **Engineering Station (VLAN 50) → PLCs**
        - CODESYS Runtime (Port 2455, 1217): YES - for programming
        - SSH (Port 22): YES - for maintenance
        - HTTP (Port 8080): YES - for WebVisu access

    3. **PLCs → Internet**
        - NO - PLCs should not have direct Internet access
        - Exception: NTP for time synchronization (controlled via firewall)

    4. **PLCs to Each Other**
        - YES - Inter-controller communication possible
        - Protocols: Modbus TCP, OPC UA, proprietary protocols

    **Answer to the Question:**

    - **Must HMI SSH into PLC?** NO - SSH is for engineering/maintenance, not normal HMI functions
    - **Must HMI ping PLC?** YES - ICMP is useful for availability checks
    - **Must HMI access web server?** YES - WebVisu runs on HTTP (Port 8080)

    ### Task 2: Create Communication Matrix

    **Extended Communication Matrix:**

    | Source | Destination | Protocol | Port | Action | Justification |
    |--------|-------------|----------|------|--------|---------------|
    | HMI (10.40.0.12) | PLC (10.10.0.11) | Modbus TCP | 502 | ALLOW | Read/write process data |
    | HMI (10.40.0.12) | PLC (10.10.0.11) | HTTP | 8080 | ALLOW | WebVisu access |
    | HMI (10.40.0.12) | PLC (10.10.0.11) | OPC UA | 4840 | ALLOW | Alternative to Modbus |
    | HMI (10.40.0.12) | PLC (10.10.0.11) | ICMP | - | ALLOW | Availability check |
    | HMI (10.40.0.12) | PLC (10.10.0.11) | SSH | 22 | BLOCK | Not needed for HMI operation |
    | EWS (10.50.0.11) | PLC (10.10.0.11) | SSH | 22 | ALLOW | Engineering/maintenance |
    | EWS (10.50.0.11) | PLC (10.10.0.11) | CODESYS | 2455, 1217 | ALLOW | Programming |
    | EWS (10.50.0.11) | PLC (10.10.0.11) | HTTP | 8080 | ALLOW | WebVisu configuration |
    | PLC (10.10.0.11) | Internet | Any | Any | BLOCK | No direct Internet connection |
    | Any | Any | Any | Any | BLOCK | Default-deny policy |

    ### Task 3: Analyze Current Firewall Rules

    **SSH to Firewall:**

    ```bash
    ssh admin@192.168.100.11
    ```

    Password: `admin`

    Alternatively with sshpass (for automation):
    ```bash
    sshpass -p admin ssh admin@192.168.100.11
    ```

    **Display Firewall Rules:**

    ```bash
    /ip firewall filter print where chain=forward
    ```

    **Expected Output (ot-sec-flat):**

    ```text
    Flags: X - disabled, I - invalid; D - dynamic 
     0    ;;; allow established,related connections
          chain=forward action=accept connection-state=established,related 

     1    ;;; allow forwarding ICMP
          chain=forward action=accept protocol=icmp 

     2    ;;; allow SSH to the jumphost from internet
          chain=forward action=accept protocol=tcp dst-address=2.2.2.2 
          in-interface-list=ext dst-port=22 

     3    ;;; allow all traffic from DMZ to VLAN10
          chain=forward action=accept in-interface=dmz-ext out-interface=vlan10 

     4    ;;; default forward drop
          chain=forward action=accept log=yes log-prefix="default forward drop"
    ```

    **Expected Output (ot-sec-segmented):**

    ```text
    Flags: X - disabled, I - invalid; D - dynamic 
     0    ;;; allow established,related connections
          chain=forward action=accept connection-state=established,related

     1    ;;; allow forwarding ICMP
          chain=forward action=accept protocol=icmp

     2    ;;; allow SSH to the jumphost from internet
          chain=forward action=accept protocol=tcp dst-address=2.2.2.2
          in-interface-list=ext dst-port=22

     3    ;;; allow Control Room to Hygiene
          chain=forward action=accept in-interface=vlan40 out-interface=vlan10

     4    ;;; allow Control Room to Process
          chain=forward action=accept in-interface=vlan40 out-interface=vlan20

     5    ;;; allow Control Room to Disposal
          chain=forward action=accept in-interface=vlan40 out-interface=vlan30

     6    ;;; allow Parameterization to Hygiene
          chain=forward action=accept in-interface=vlan50 out-interface=vlan10

     ... more rules ...
    ```

    **Analysis:**

    - **Flat Topology**: Rule 3 allows ALL traffic from DMZ to VLAN 10 - too permissive
    - **Segmented Topology**: Rule 3 allows ALL traffic from VLAN 40 to VLAN 10 - too permissive
    - **Problem**: No port restrictions, no protocol specification
    - **Risk**: SSH, unnecessary services are reachable

    ### Task 4: Implement Restrictive Rules

    **WARNING**: These changes can break network connectivity. Always perform in production during maintenance window with rollback plan.

    **Step 1: Add Specific Rule for Modbus TCP**

    ```bash
    /ip firewall filter add chain=forward action=accept \
        src-address=10.40.0.12 dst-address=10.10.0.11 \
        protocol=tcp dst-port=502 \
        comment="Allow HMI to PLC Modbus" place-before=0
    ```

    **Step 2: Add Specific Rule for WebVisu HTTP**

    ```bash
    /ip firewall filter add chain=forward action=accept \
        src-address=10.40.0.12 dst-address=10.10.0.11 \
        protocol=tcp dst-port=8080 \
        comment="Allow HMI to PLC WebVisu" place-before=0
    ```

    **Step 3: Add Specific Rule for OPC UA**

    ```bash
    /ip firewall filter add chain=forward action=accept \
        src-address=10.40.0.12 dst-address=10.10.0.11 \
        protocol=tcp dst-port=4840 \
        comment="Allow HMI to PLC OPC-UA" place-before=0
    ```

    **Step 4: Verify Rules**

    ```bash
    /ip firewall filter print where chain=forward
    ```

    You should now see the new specific rules at the top.

    **Step 5: Identify Broad "Allow Any" Rule**

    Find the rule number of the broad rule:

    ```bash
    /ip firewall filter print where chain=forward
    ```

    In `ot-sec-segmented`: Look for "allow Control Room to Hygiene" (e.g. rule #3)
    In `ot-sec-flat`: Look for "allow all traffic from DMZ to VLAN10" (e.g. rule #3)

    **Step 6: Disable Broad Rule**

    ```bash
    /ip firewall filter disable [find comment="allow Control Room to Hygiene"]
    ```

    Or for flat:
    ```bash
    /ip firewall filter disable [find comment="allow all traffic from DMZ to VLAN10"]
    ```

    **Step 7: Enable Default-Drop (Optional)**

    In `ot-sec-flat` the default-drop rule is actually set to "accept". For real security:

    ```bash
    /ip firewall filter set [find comment="default forward drop"] action=drop
    ```

    **CAUTION**: This blocks all traffic not explicitly allowed. Ensure all necessary rules are in place!

    **Step 8: Test**

    From another terminal (not via SSH to firewall!):

    ```bash
    # Test 1: HMI can reach WebVisu (should work)
    docker exec clab-ot-sec-flat-pilz-hmi01-vlan40 curl -I --connect-timeout 3 http://10.10.0.11:8080

    # Test 2: HMI can reach Modbus (should work)
    docker exec clab-ot-sec-flat-pilz-hmi01-vlan40 nc -zv 10.10.0.11 502

    # Test 3: HMI cannot reach SSH (should be blocked if default-drop active)
    docker exec clab-ot-sec-flat-pilz-hmi01-vlan40 nc -zv -w 2 10.10.0.11 22
    ```

    **Expected Results:**

    Test 1 & 2: Successful
    Test 3: Connection timed out or refused (if default-drop active)

    **Rollback (if needed):**

    ```bash
    # Re-enable broad rule
    /ip firewall filter enable [find comment="allow Control Room to Hygiene"]

    # Set default-drop back to accept
    /ip firewall filter set [find comment="default forward drop"] action=accept
    ```

    ### Task 5: Compare Flat vs. Segmented

    **Comparison Table:**

    | Aspect | Flat Network | Segmented Network (with Firewall) |
    |--------|--------------|-----------------------------------|
    | **Broadcast Domain** | One large domain | Multiple small VLANs |
    | **ARP Spoofing Reach** | All devices affected | Limited to VLAN |
    | **Lateral Movement** | Easy - direct access | Difficult - firewall blocks |
    | **Attack Surface** | All ports on all devices | Only allowed ports/protocols |
    | **Traffic Filtering** | None | Central at firewall |
    | **Monitoring** | Distributed, complex | Central at firewall |
    | **Incident Response** | Hard to isolate | Can isolate zones |
    | **Compliance** | IEC 62443 SR 5.1 FAIL | IEC 62443 SR 5.1 PASS |

    **Attack Scenario - Flat Network:**

    1. Attacker compromises HMI (10.40.0.12)
    2. Performs ARP spoofing → Man-in-the-Middle on entire network
    3. Direct access to PLC via SSH (Port 22)
    4. Direct access to all engineering ports
    5. Can move laterally to all other devices
    6. No central monitoring of traffic

    **Attack Scenario - Segmented Network:**

    1. Attacker compromises HMI (10.40.0.12 in VLAN 40)
    2. ARP spoofing only possible in VLAN 40
    3. Attempts to access PLC (10.10.0.11 in VLAN 10)
    4. Firewall blocks SSH (Port 22) → Access denied
    5. Firewall only allows Modbus, HTTP, OPC UA
    6. Firewall logs all access attempts
    7. IDS/IPS can detect anomalies
    8. Security team alerted
    9. VLAN 10 can be isolated without affecting VLAN 40

    **Conclusion:**

    - **Flat Network**: Easy to manage but catastrophic when compromised
    - **Segmented Network**: More complex but significantly more secure and IEC 62443 compliant
    - **Best Practice**: Segmentation + specific firewall rules + monitoring

    ### Additional Best Practices

    **1. Enable Logging:**

    ```bash
    /ip firewall filter set [find comment="default forward drop"] log=yes log-prefix="fw-drop"
    ```

    **2. Address Lists for Better Management:**

    ```bash
    # Define HMI devices
    /ip firewall address-list add list=hmi-devices address=10.40.0.12
    /ip firewall address-list add list=hmi-devices address=10.40.0.13

    # Define PLC devices
    /ip firewall address-list add list=plc-devices address=10.10.0.11
    /ip firewall address-list add list=plc-devices address=10.10.0.12
    /ip firewall address-list add list=plc-devices address=10.20.0.11

    # Rule with Address Lists
    /ip firewall filter add chain=forward action=accept \
        src-address-list=hmi-devices dst-address-list=plc-devices \
        protocol=tcp dst-port=502 \
        comment="Allow HMI to PLCs Modbus"
    ```

    **3. Regular Audits:**

    ```bash
    # Show all rules with counters
    /ip firewall filter print stats

    # Identify unused rules (bytes=0)
    /ip firewall filter print stats where bytes=0
    ```

    **4. Change Management:**

    - Document all firewall changes
    - Create backup before changes:
      ```bash
      /system backup save name=before-firewall-change
      ```
    - Validate in test environment
    - Roll out incrementally in production

    ### Summary: Lessons Learned

    1. **Segmentation alone is not enough** - Firewall rules are essential
    2. **Default-deny is critical** - "Allow any" makes segmentation worthless
    3. **Enforce least privilege** - Only allow absolutely necessary ports
    4. **Monitoring is mandatory** - Enable logging and alerting
    5. **Documentation is crucial** - Know why each rule exists
    6. **Test, test, test** - Validate before production deployment
    7. **IEC 62443 SR 5.1** - Network segmentation is fundamental requirement
