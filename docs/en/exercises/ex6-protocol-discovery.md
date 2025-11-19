# Exercise 6: OT Protocol Discovery

**Time Estimate**: 30-35 minutes  
**Lab Topology**: `ot-sec-flat` (recommended) or `ot-sec-segmented`  
**Difficulty**: :material-signal-cellular-2: Intermediate

## Learning Objectives

- Discover OT protocols in live traffic
- Identify Modbus TCP communication
- Analyze CODESYS runtime protocol
- Understand security implications of cleartext protocols

## Prerequisites

- Exercise 1 (Wireshark basics)
- Basic understanding of TCP/IP

## Scenario

In an OT environment, visibility is key. Unlike IT networks where HTTPS and SSH dominate, OT networks often rely on cleartext protocols like Modbus TCP and proprietary protocols like CODESYS. In this exercise, you will generate traffic between an HMI and a PLC, capture it, and analyze the protocol details to understand what an attacker could see.

## Tasks

### Task 1: Generate OT Traffic

To analyze traffic, we first need to generate it. We will use the HMI to interact with a PLC's web visualization.

1.  Open the **HMI Desktop** in your browser:
    *   URL: `http://localhost:5800`
    *   This connects you to the `pilz-hmi01-vlan40` desktop via VNC.

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

### Task 5: Security Implications

Based on your observations:

1. **Confidentiality**: Could an attacker passively listening on the network know the state of the process (e.g., "Temperature is 80°C")?
2. **Integrity**: Could an attacker inject packets to change values (e.g., "Set Speed to 10000")?
3. **Availability**: What happens if an attacker floods port 502?

## Conclusion

OT protocols like Modbus TCP were designed for reliability, not security. They lack encryption and authentication. This exercise demonstrated how easily this traffic can be captured and analyzed, highlighting the need for network segmentation (to limit who can sniff) and monitoring (to detect unauthorized commands).
