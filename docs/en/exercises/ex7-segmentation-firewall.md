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
