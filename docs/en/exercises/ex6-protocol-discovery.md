# Exercise 6: OT Protocol Discovery

**Time Estimate**: 45-50 minutes  
**Lab Topology**: `ot-sec-flat` (recommended) or `ot-sec-segmented`  
**Difficulty**: :material-signal-cellular-2: Intermediate

## Learning Objectives

- Discover OT protocols in live traffic
- Identify Modbus TCP communication
- Analyze CODESYS runtime protocol
- Assess OPC UA security configurations
- Recognize PROFINET characteristics
- Understand security implications of cleartext protocols

## Prerequisites

- Exercise 1 (Wireshark basics)
- Basic understanding of TCP/IP

## Scenario

In an OT environment, visibility is key. Unlike IT networks where HTTPS and SSH dominate, OT networks often rely on cleartext protocols like Modbus TCP and proprietary protocols like CODESYS. In this exercise, you will generate traffic between an HMI and a PLC, capture it, and analyze the protocol details to understand what an attacker could see.

## Tasks

### Task 1: Generate OT Traffic

To analyze traffic, we first need to generate it. We will use the HMI to interact with a PLC's web visualization.

1.  Under Ports goto: **HMI Chromium browser**

2.  On the HMI desktop, open the **Chromium** web browser.

3.  Navigate to the WebVisu of `wago-plc2a-vlan10`:
    *   URL: `http://10.10.0.11:8080/webvisu.htm`
    *   (If that doesn't load, try `http://10.10.0.11:8080`)

4.  Interact with the visualization (click buttons, toggle switches) to generate traffic.

### Task 2: Capture Traffic with Wireshark

Now we will capture the traffic flowing to the PLC using the integrated lab tools.

1. In VS Code, go to the **Containerlab** extension (TopoViewer).

2. Locate the link between `wago-plc2a-vlan10` and `sw-acc1`.

3. **Right-click** the link (or the interface dot on the PLC side) and select **Wireshark Capture**.
    - This will launch a Wireshark instance directly in your browser/editor via Edgeshark.

4. Go back to the HMI (VNC) and interact with the WebVisu again for 10-15 seconds (click buttons, toggle switches).

5. Return to the Wireshark window. You should see packets appearing in real-time.

6. Stop the capture (red square button in Wireshark toolbar) to analyze the data.

### Task 3: Analyze Modbus TCP

1. In Wireshark, filter for Modbus TCP traffic:

    ```wireshark
    mbtcp
    ```

    *Note: If you don't see `mbtcp`, try filtering for port `tcp.port == 502`.*

2. Select a packet. Expand the **Modbus** section in the packet details.

3. **Questions**:
    - What Function Codes do you see? (e.g., Read Holding Registers, Write Single Coil)
    - Can you see the values being read or written?
    - Is there any encryption or authentication?

### Task 4: Analyze CODESYS Protocol

The WebVisu communicates with the runtime, often using proprietary protocols or HTTP/XML. However, the engineering workstation would use the CODESYS protocol. Let's look for that or other management traffic.

1. Clear the filter and look for traffic on port **2455** (UDP) or **1217** (TCP).

    ```wireshark
    udp.port == 2455 || tcp.port == 1217
    ```

2. If you don't see this traffic, it might be because we are only using WebVisu. To generate CODESYS engineering traffic, we would need the CODESYS IDE.

    *Alternative*: Look for the HTTP traffic for the WebVisu.

    ```wireshark
    http && ip.addr == 10.10.0.11
    ```

3. Analyze the HTTP streams.
    - Right-click a packet -> Follow -> HTTP Stream.
    - Observe that the visualization data is transferred in cleartext (unless HTTPS is configured, which is often not the default in older setups).

### Task 5: Analyze OPC UA Security Configuration

OPC UA is the modern Industry 4.0 protocol standard. Unlike Modbus, it was designed with security features, but these are often disabled in production environments. Your job as an OT security professional is to assess whether security is properly configured.

#### Background

OPC UA supports:

- **Encryption** (SignAndEncrypt)
- **Authentication** (Username/Password or Certificate-based)
- **Authorization** (Role-based access control)

However, most deployments run with `SecurityPolicy=None` and `Anonymous` authentication for simplicity or legacy compatibility.

#### Generate OPC UA Traffic

The lab PLCs have OPC UA servers enabled on port 4840:

- `ctrlx-plc3-vlan20` at `10.20.0.11:4840`
- `schneider-plc4-vlan30` at `10.30.0.11:4840`

We need to generate OPC UA traffic to analyze it. The simplest approach is to use the HMI's web browser.

**Generate Traffic from HMI**

1. Open the **HMI Chromium Browser** under Ports

2. In the HMI Chromium browser, open a new tab

3. Navigate to: `http://10.20.0.11:8080/webvisu.htm`
   - The WebVisu may communicate with the PLC using OPC UA in the background
   - Interact with the visualization to generate traffic

**Alternative: Test Connection from Jumphost**

If you have terminal access to the jumphost, verify OPC UA port is accessible:

```bash
# Test connectivity
nc -zv 10.20.0.11 4840

# Send a simple connection probe
timeout 2 telnet 10.20.0.11 4840
```

You should see a connection established, confirming the OPC UA server is running.

#### Capture OPC UA Traffic

1. Start Wireshark capture on the link between `ctrlx-plc3-vlan20` and `sw-acc1`
2. Execute the OPC UA client connection (Option A or B above)
3. Stop the capture after the connection completes

#### Analyze Security Configuration

1. Filter for OPC UA traffic:

    ```wireshark
    opcua
    ```

    Or by port:

    ```wireshark
    tcp.port == 4840
    ```

2. Locate the **CreateSessionRequest** packet:
    - Expand: `OpcUa Binary Protocol -> Message -> CreateSessionRequest`
    - Find: `SecurityPolicyUri`
    - Check value: `http://opcfoundation.org/UA/SecurityPolicy#None` indicates no encryption

3. Locate the **ActivateSessionRequest** packet:
    - Expand: `OpcUa Binary Protocol -> Message -> ActivateSessionRequest`
    - Check: `UserIdentityToken`
    - Look for: `AnonymousIdentityToken` indicates no authentication

4. Analyze a **BrowseRequest/Response**:
    - This shows the PLC's complete tag structure
    - All tag names are visible in cleartext
    - An attacker can map the entire process data model

5. Look at **ReadRequest/Response**:
    - Variable values are transmitted in cleartext
    - No integrity protection
    - An attacker can see real-time process values

#### Assessment Questions

1. **What SecurityPolicy is configured?** (Expected: None)
2. **Is the communication encrypted?** (Expected: No)
3. **What authentication is required?** (Expected: Anonymous - none)
4. **Can you see tag names in the Browse response?** (Expected: Yes)
5. **Can you see variable values in Read responses?** (Expected: Yes)
6. **What IEC 62443 requirements are violated?**
    - SR 1.1: Human user identification and authentication (FAIL - Anonymous)
    - SR 3.1: Communication integrity (FAIL - No signing)
    - SR 4.1: Information confidentiality (FAIL - No encryption)

#### Real-World Context

This is one of the most common findings in OT security audits. OPC UA has excellent security features, but organizations disable them because:

- Configuration complexity (perceived, not actual)
- Legacy compatibility concerns (sometimes valid)
- Performance myths (encryption overhead is negligible on modern hardware)
- Lack of awareness (most common reason)

**Audit Recommendation:** Enable OPC UA security with `SecurityPolicy=Basic256Sha256`, `SecurityMode=SignAndEncrypt`, and certificate-based authentication. Test with existing clients to verify compatibility.

### Task 6: PROFINET Protocol Recognition

PROFINET is the dominant real-time industrial Ethernet protocol in European manufacturing, particularly in automotive and process industries. Unlike the previous protocols which operate at TCP/IP layers, PROFINET real-time traffic runs directly on Layer 2 (Ethernet frames) for microsecond-level control.

#### Key Differences

| Aspect | Modbus/OPC UA | PROFINET RT |
|--------|---------------|-------------|
| OSI Layer | Layer 4 (TCP) | Layer 2 (Ethernet) |
| Can Route? | Yes | No (requires L2 adjacency) |
| Encryption | Possible | No (real-time constraints) |
| Monitoring | Routed TAP | SPAN port required |

#### Why This Matters

Understanding Layer 2 protocols changes your security assessment approach:

- **Cannot segment across subnets** - PROFINET requires L2 adjacency
- **Cannot encrypt** - Real-time performance requirements (sub-millisecond latency)
- **Different IDS placement** - Must use SPAN ports, not routed TAPs
- **Security is architectural** - Physical access control, VLAN isolation, anomaly detection

#### Capture PROFINET Traffic

The ABB 800xA system (`abb-800xa-vlan40` at `10.40.0.11`) has active PROFINET Real-Time communication running.

1. In VS Code, go to the **Containerlab** extension (TopoViewer)

2. Locate the link between `abb-800xa-vlan40` and `sw-acc2`

3. **Right-click** the link and select **Wireshark Capture**

4. Let it capture for 10-15 seconds to collect PROFINET frames

5. Stop the capture

#### Analyze PROFINET Real-Time Traffic

**Filter for PROFINET:**

```wireshark
# PROFINET Real-Time traffic
eth.type == 0x8892
```

Or in Wireshark display filter:

```wireshark
pn_io
```

**What You Should See:**

1. **Ethertype 0x8892** - Real-time cyclic I/O data
    - No TCP/IP headers (this is Layer 2, not TCP/IP)
    - Direct Ethernet frames
    - Source/Destination MAC addresses visible

2. **FrameID in hex (typically 0x8000-0x8001)**
    - 0x8000 range = RT cyclic data
    - Look at the packet details for the FrameID field

3. **Cyclic Pattern**
    - PROFINET RT runs at fixed intervals (typically 1-10ms)
    - In this lab: approximately 4ms cycle time
    - Packet timing is extremely regular
    - Select several packets and check the time delta between them

4. **Payload Data**
    - Raw I/O data (digital/analog values)
    - No encryption possible (real-time performance requirement)
    - Process values are transmitted in cleartext

#### Assessment Questions

1. **What is the Ethertype?** (Expected: 0x8892)
2. **Are there TCP/IP headers?** (Expected: No - this is Layer 2)
3. **What is the approximate cycle time?** (Measure time between packets - should be ~4ms)
4. **Can you see MAC addresses?** (Expected: Yes - 00:03:2c:40:00:11 and 00:0b:0f:20:00:11)
5. **Is the data encrypted?** (Expected: No - cannot encrypt real-time traffic)
6. **What happens if you filter for `tcp` or `udp`?** (Expected: PROFINET RT won't appear - it's Layer 2)

#### Security Implications

**Attack Scenarios:**

- **Reconnaissance**: DCP broadcasts reveal all devices, vendors, firmware versions
- **Topology Mapping**: LLDP (often used with PROFINET) exposes network architecture
- **Packet Injection**: Attacker on same L2 segment can forge RT frames
- **Timing Attacks**: Disrupting cyclic timing can cause emergency stops

**Defense Requirements:**

- **Physical Security**: Must protect L2 access (switch ports)
- **VLAN Isolation**: Separate PROFINET domains
- **Port Security**: MAC address filtering on switches
- **Anomaly Detection**: Monitor for timing irregularities, unexpected devices
- **No Application-Layer Security**: Cannot use encryption, TLS, etc.

**IEC 62443 Context:**

PROFINET predates IEC 62443 and was designed for performance, not security. Security must be implemented through:

- Network architecture (zones and conduits)
- Physical access control
- Monitoring and detection (not prevention at protocol level)

### Task 7: Protocol Comparison and Security Assessment

Now that you've analyzed multiple OT protocols, compare their security characteristics and understand when you'll encounter each in real environments.

#### Protocol Comparison Matrix

| Protocol | Layer | Port/Type | Encryption | Auth | Primary Use Case | Security Approach |
|----------|-------|-----------|------------|------|------------------|-------------------|
| **Modbus TCP** | L4 TCP | 502 | No | No | Legacy SCADA, simple PLC polling | Segmentation, monitoring, consider Modbus/TLS |
| **CODESYS** | L4 TCP/UDP | 1217, 2455 | Optional | Optional | PLC engineering, diagnostics | Disable in production, use only during maintenance |
| **OPC UA** | L4 TCP | 4840 | Yes (often disabled) | Yes (often disabled) | Vertical integration, MES/SCADA, Industry 4.0 | **Enable built-in security features** |
| **PROFINET RT** | L2 Ethernet | 0x8892 | No (impossible) | No | Real-time I/O control, automotive, discrete manufacturing | Physical + architectural controls |

#### Critical Assessment Questions

For each protocol discovered in an OT environment, ask:

1. **Is this protocol necessary?**
    - CODESYS engineering ports should be disabled in production
    - Modbus may be legacy and replaceable with OPC UA

2. **Is security configured optimally?**
    - OPC UA: SecurityPolicy and authentication should be enabled
    - Modbus: Consider upgrading to Modbus/TLS or wrapping in VPN

3. **Is network architecture appropriate?**
    - All protocols: Proper VLAN segmentation
    - PROFINET: Must have L2 isolation, cannot route
    - OPC UA: Can route, consider firewall rules between zones

4. **Is monitoring in place?**
    - Modbus/OPC UA: Can monitor at L3/L4 (routed TAP or firewall logs)
    - PROFINET: Requires L2 SPAN port to switch
    - All: Baseline normal behavior, alert on anomalies

#### Audit Checklist

When assessing an OT environment:

- [ ] Inventory all protocols in use (Wireshark on SPAN ports)
- [ ] For OPC UA: Verify SecurityPolicy is NOT None
- [ ] For OPC UA: Verify authentication is required (not Anonymous)
- [ ] For Modbus: Verify network segmentation prevents unauthorized access
- [ ] For CODESYS: Verify engineering ports are disabled or access-controlled
- [ ] For PROFINET: Verify VLAN isolation and port security
- [ ] For all: Verify monitoring/IDS is in place and tuned for OT protocols
- [ ] Document findings with specific IEC 62443 SR violations

#### IEC 62443 Security Requirements Mapping

| SR | Requirement | Modbus TCP | OPC UA (Misconfigured) | OPC UA (Proper) | PROFINET |
|----|-------------|------------|------------------------|-----------------|----------|
| SR 1.1 | Human user identification | Not Applicable | FAIL (Anonymous) | PASS (Certificate) | Not Applicable |
| SR 3.1 | Communication integrity | FAIL (No protection) | FAIL (No signing) | PASS (Signed) | FAIL (No protection) |
| SR 4.1 | Information confidentiality | FAIL (Cleartext) | FAIL (No encryption) | PASS (Encrypted) | FAIL (Impossible) |
| SR 5.1 | Network segmentation | Required | Required | Recommended | **Critical** |

#### Conclusion: Defense in Depth

No single protocol security feature provides complete protection. OT security requires:

1. **Protocol-level security** (where available) - Enable OPC UA security features
2. **Network architecture** - Proper segmentation, VLANs, access control
3. **Monitoring and detection** - Baseline normal behavior, alert on anomalies
4. **Physical security** - Particularly for L2 protocols like PROFINET
5. **Access management** - Disable unnecessary services (CODESYS in production)

**Key Takeaway:** Legacy protocols (Modbus) require compensating controls. Modern protocols (OPC UA) have security features that must be enabled. Real-time protocols (PROFINET) require architectural security because protocol-level security isn't possible.

## Conclusion

This exercise demonstrated the security reality of OT protocols:

- **Legacy protocols** (Modbus) have no security features - rely on segmentation and monitoring
- **Modern protocols** (OPC UA) have excellent security features that are often disabled - audit and enable them
- **Real-time protocols** (PROFINET) cannot use encryption - require architectural controls
- **All protocols** benefit from network segmentation, monitoring, and proper access control

As an OT security professional, your role is to:

1. Inventory what protocols are in use
2. Assess their security configuration
3. Identify gaps against IEC 62443 requirements
4. Recommend practical, operations-friendly mitigations
5. Implement monitoring to detect unauthorized protocol usage

The protocols you analyzed today represent 80% of what you'll encounter in real industrial environments. Understanding their security characteristics is fundamental to effective OT security assessment.

---

## Solution

??? success "Click to view solution"
    ### Task 1: Generate OT Traffic

    In HMI desktop, open Chromium and navigate to WebVisu:
    ```
    http://10.10.0.11:8080/webvisu.htm
    ```

    Interact with buttons and switches in the visualization to generate HTTP and potentially Modbus traffic.

    ### Task 2: Capture Traffic

    1. Open VS Code Containerlab TopoViewer
    2. Right-click on link between `wago-plc2a-vlan10` and `sw-acc1`
    3. Select "Wireshark Capture"
    4. Interact again with WebVisu in HMI
    5. Observe traffic in Wireshark

    ### Task 3: Analyze Modbus TCP

    Apply Wireshark filter:
    ```wireshark
    mbtcp
    ```

    Or alternatively:
    ```wireshark
    tcp.port == 502
    ```

    **Expected Observations:**

    - **Function Codes**: Typically Read Holding Registers (0x03), Write Single Coil (0x05), Write Multiple Registers (0x10)
    - **Values visible**: Yes - all register values visible in cleartext
    - **Encryption**: No - Modbus TCP has no encryption
    - **Authentication**: No - no authentication in protocol

    **Security Implication**: An attacker in the same network segment can:
    - Read all process values
    - Send commands to the PLC
    - Manipulate traffic (Man-in-the-Middle)

    From jumphost you can test Modbus connectivity:
    ```bash
    ssh admin@192.168.100.52
    nc -zv 10.10.0.11 502
    ```

    Expected output:
    ```text
    Connection to 10.10.0.11 502 port [tcp/*] succeeded!
    ```

    ### Task 4: Analyze CODESYS Protocol

    Filter for HTTP traffic to WebVisu:
    ```wireshark
    http && ip.addr == 10.10.0.11
    ```

    Follow HTTP stream:
    - Right-click on HTTP packet
    - Follow → HTTP Stream
    - Observe visualization data in cleartext

    **Expected Observations:**
    - WebVisu uses HTTP (Port 8080)
    - No HTTPS encryption
    - Variable values and status in cleartext
    - Session information exposed

    For CODESYS runtime protocol (normally from engineering workstation):
    ```wireshark
    udp.port == 2455 || tcp.port == 1217
    ```

    In the lab you may see little CODESYS runtime traffic as no engineering workstation is actively programming.

    **Port overview on PLC:**

    You can check open ports from jumphost or from PLC itself:
    ```bash
    ssh admin@192.168.100.53  # wago-plc2a-vlan10
    ss -tuln | grep LISTEN
    ```

    Expected output:
    ```text
    tcp   LISTEN 0      5              0.0.0.0:8080       0.0.0.0:*    # WebVisu
    tcp   LISTEN 0      128            0.0.0.0:22         0.0.0.0:*    # SSH
    tcp   LISTEN 0      63          10.10.0.11:502        0.0.0.0:*    # Modbus
    tcp   LISTEN 0      10          10.10.0.11:4840       0.0.0.0:*    # OPC UA
    ```

    ### Task 5: Analyze OPC UA Security Configuration

    **Generate OPC UA Traffic:**

    Option 1 - From HMI browser:
    ```
    http://10.20.0.11:8080/webvisu.htm  # ctrlx-plc3-vlan20
    ```

    Option 2 - Test connectivity from jumphost:
    ```bash
    ssh admin@192.168.100.52
    nc -zv 10.20.0.11 4840
    ```

    Expected output:
    ```text
    Connection to 10.20.0.11 4840 port [tcp/*] succeeded!
    ```

    **Wireshark Analysis:**

    Capture traffic between `ctrlx-plc3-vlan20` and `sw-acc1`.

    Filter for OPC UA:
    ```wireshark
    opcua
    ```

    Or:
    ```wireshark
    tcp.port == 4840
    ```

    **Analyze CreateSessionRequest:**

    Expand: `OpcUa Binary Protocol → Message → CreateSessionRequest`

    Find: `SecurityPolicyUri`

    **Expected Value:**
    ```
    http://opcfoundation.org/UA/SecurityPolicy#None
    ```

    **Meaning**: No encryption enabled.

    **Analyze ActivateSessionRequest:**

    Expand: `OpcUa Binary Protocol → Message → ActivateSessionRequest`

    Find: `UserIdentityToken`

    **Expected Type:**
    ```
    AnonymousIdentityToken
    ```

    **Meaning**: No authentication required.

    **Analyze BrowseRequest/Response:**

    - Complete OPC UA node structure visible
    - All tag names in cleartext
    - Organization structure of process data model exposed
    - Attacker can create complete asset inventory

    **Analyze ReadRequest/Response:**

    - Variable values transmitted in cleartext
    - No integrity protection
    - No confidentiality
    - Real-time process data visible to attacker

    **Assessment Results:**

    1. **SecurityPolicy**: None (FAIL)
    2. **Encryption**: No (FAIL)
    3. **Authentication**: Anonymous (FAIL)
    4. **Tag names visible**: Yes (FAIL)
    5. **Variable values visible**: Yes (FAIL)

    **IEC 62443 Violations:**

    - SR 1.1 (Identification & Authentication): FAIL
    - SR 3.1 (Communication Integrity): FAIL
    - SR 4.1 (Information Confidentiality): FAIL

    **Recommendation:**

    Change OPC UA server configuration:
    ```
    SecurityPolicy: Basic256Sha256
    SecurityMode: SignAndEncrypt
    Authentication: Certificate-based
    ```

    ### Task 6: PROFINET Protocol Discovery

    **Capture PROFINET Traffic:**

    Capture traffic between `abb-800xa-vlan40` and `sw-acc2`.

    **Wireshark Filter:**

    ```wireshark
    eth.type == 0x8892
    ```

    Or:
    ```wireshark
    pn_io
    ```

    **Expected Observations:**

    1. **Ethertype**: 0x8892 (PROFINET Real-Time)
    2. **No TCP/IP Headers**: This is Layer 2, not Layer 4
    3. **FrameID**: Typically 0x8000-0x8001 (RT cyclic data)
    4. **Cyclic Pattern**: Packets every ~4ms (very regular)
    5. **MAC Addresses visible**:
       - Source: 00:03:2c:40:00:11 (ABB system)
       - Destination: 00:0b:0f:20:00:11 (PROFINET device)

    **Timing Analysis:**

    Select multiple consecutive PROFINET packets and check time delta:

    In Wireshark:
    - View → Time Display Format → Seconds Since Previous Displayed Packet
    - Expectation: ~0.004 seconds (4ms cycle time)

    **Assessment Results:**

    1. **Ethertype**: 0x8892 (Layer 2 Real-Time)
    2. **TCP/IP Headers**: No - direct Ethernet frame
    3. **Cycle Time**: ~4ms (very deterministic)
    4. **MAC Addresses**: Yes, visible
    5. **Encryption**: No - impossible with real-time
    6. **Filter for tcp/udp**: PROFINET RT does not appear

    **Security Implications:**

    - No routing possible (Layer 2)
    - No encryption possible
    - Physical access = full control
    - VLAN isolation is critical
    - Port security required
    - Anomaly detection based on timing

    **PROFINET DCP (Discovery):**

    For PROFINET device discovery:
    ```wireshark
    pn_dcp
    ```

    DCP broadcasts reveal:
    - Device name
    - Manufacturer
    - IP configuration
    - Role in network

    ### Task 7: Protocol Comparison

    **Analysis Summary:**

    | Protocol | Encryption | Auth | Main Risk | Defense |
    |----------|------------|------|-----------|---------|
    | **Modbus TCP** | No | No | Cleartext access | Segmentation + Monitoring |
    | **CODESYS** | Optional, usually No | Optional | Engineering access | Disable in production |
    | **OPC UA** | Yes (often disabled) | Yes (often disabled) | Misconfiguration | Enable security features |
    | **PROFINET RT** | Impossible | No | Layer 2 attacks | Physical + VLAN isolation |

    **Audit Checklist Completed:**

    - [x] Protocols inventoried (Modbus, OPC UA, PROFINET, HTTP)
    - [x] OPC UA SecurityPolicy checked (None = FAIL)
    - [x] OPC UA Authentication checked (Anonymous = FAIL)
    - [x] Modbus segmentation checked
    - [x] CODESYS ports identified (should be disabled)
    - [x] PROFINET VLAN isolation checked
    - [x] IEC 62443 violations documented

    **Key Takeaways:**

    1. **Legacy Protocols** (Modbus): No security features → Compensating controls required
    2. **Modern Protocols** (OPC UA): Security present but often disabled → Audit and activation required
    3. **Real-Time Protocols** (PROFINET): Encryption impossible → Architectural security critical
    4. **All Protocols**: Benefit from segmentation, monitoring, and access control

    **Next Steps:**

    1. Enable OPC UA security features (highest priority)
    2. Enforce network segmentation (see Exercise 7)
    3. Disable CODESYS engineering ports for production
    4. Implement IDS/IPS for OT protocol anomalies
    5. Conduct regular protocol audits

```
