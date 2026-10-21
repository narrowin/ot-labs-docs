# Exercise 2: Trace an ARP interception

**Time:** 40–50 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Guided practice.

Prove a change in the victim's IP-to-MAC mapping, follow the resulting traffic, and restore the original path. Run the interception from WAGO A against only B and its gateway. Use the named lab targets for connectivity checks. These addresses refer to the simulated network inside this appliance.

**Learning objectives:** Explain ARP's trust problem, execute and prove a bounded Layer 2 interception, identify readable application data, and explain detection and mitigation.

## Your incident task

Monitoring reports a changed gateway MAC address in a production cell. Is this an approved network change, an addressing fault or an interception? Reproduce one known cause in the simulator, then build the evidence an operations team would need before taking action.

**ARP in plain language:** To send an IPv4 packet on Ethernet, a host needs the next hop's MAC address. ARP asks who owns that IP and caches the answer. A forged answer can make a different device receive the frames. The IP destination does not have to change.

The TCP marker below is a disposable test message, not an industrial protocol or a stolen credential. It makes the redirection visible without manipulating a process. Exercise 6 examines actual lab OT protocol messages.

## Before you start

Complete [Exercise 1](ex1-vlan-discovery.md). Open the configured terminals; no password entry or software installation is needed:

| Role | Terminal | Data address |
| --- | --- | --- |
| Attacker | [WAGO A](/network/terminal?node=wago-plc2a-vlan10) | `10.10.0.11` |
| Victim | [WAGO B](/network/terminal?node=wago-plc2b-vlan10) | `10.10.0.12` |
| Receiver | [DMZ jumphost](/network/terminal?node=proxmox-jumphost) | `2.2.2.2` |

Keep the tabs open. The terminals use management connections; the experiment uses data interface `eth1`. Run commands in the role named above each block. Use the exercise title in a tool tab to reopen these instructions.

??? info "Using standalone documentation?"
    In your assigned lab's live **Netmap**, search each stable node name in the table. Use its action menu's **Open terminal**. For the capture, select WAGO A, **Capture packets**, interface **eth1**. Reader links make those selections automatically and retain the selected network and snapshot.

## Task 1: Record the real gateway mapping

On **WAGO B**:

```bash
ping -c 2 10.10.0.1
ip neigh show 10.10.0.1 dev eth1
```

Record the gateway MAC, not just the neighbor state (`REACHABLE` or `STALE`). On **WAGO A**:

```bash
cat /sys/class/net/eth1/address
cat /proc/sys/net/ipv4/ip_forward
command -v arp_spoofing-simple.sh timeout
```

Record A's MAC and forwarding value. The last command must print one path for `arp_spoofing-simple.sh` and one for `timeout`. The helper preserves the previous forwarding setting.

**Checkpoint:** The real gateway MAC differs from A's MAC. Both commands exist. If this is already a changed mapping, complete the three cleanup steps in Task 5 before proceeding.

??? info "Also inspect the mapping in Explorer"
    Open [WAGO B's host details](/network/inventory/host?node=wago-plc2b-vlan10) and expand **ARP Cache**. Find **address** `10.10.0.1` on **interface** `eth1`; compare **mac** with your gateway MAC. On [WAGO A](/network/inventory/host?node=wago-plc2a-vlan10), **Interface Details** shows the `eth1` MAC and **Network Interfaces** shows `10.10.0.11` with mask `255.0.0.0`. **Interface ether2** in the header is the switch port, not Linux `eth1`. B's header MAC identifies B itself and stays unchanged; the gateway's **ARP Cache** row is the mapping you are testing.

    These are collected endpoint reports. ARP is collected about every 60 seconds; reloading shows the latest received report, not a new query. An early-stopped experiment can fall between reports, and delivery can lag. Use the terminal and capture for immediate proof; an unchanged Explorer row does not rule out interception. Missing panels: tell the instructor and continue with terminal/capture evidence.

    Standalone: **Inventory → Hosts**, search the stable node name, press **Enter**, then click its name.

## Task 2: Establish the normal path

[Open a capture on WAGO A's data port](/network/capture?node=wago-plc2a-vlan10&interface=eth1), choose **Start Capture**, then run on **WAGO B**:

```bash
ping -c 3 2.2.2.2
```

In Wireshark use `icmp && ip.addr == 10.10.0.12`. Wait a few seconds for the view to catch up.

**Checkpoint:** The pings reply, but their unicast requests and replies normally do not cross A's switched access link. Background ARP broadcasts can still appear. If victim unicast is already present, inspect the ARP mapping and selected interface before interpreting the experiment.

Read Tasks 3–5 before starting the timer. Keep A, B, the receiver and the capture ready. On the **DMZ jumphost**, prepare the marker receiver:

```bash
timeout 300 nc -l 2323
```

Leave it waiting. It exits after one connection or at most five minutes. Now switch to A. Complete the short commands in Tasks 3–4 first; inspect the recorded packets after cleanup.

## Task 3: Redirect the selected pair

On **WAGO A**, run in the foreground:

```bash
timeout -s INT 120 arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
```

The helper targets only B and its gateway and stops automatically after 120 seconds. Once you have the mapping and marker evidence, stop it earlier with **Ctrl+C**. Leave this terminal running while using B.

After about five seconds, on **WAGO B**:

```bash
ip neigh show 10.10.0.1 dev eth1
ping -c 3 2.2.2.2
```

**Checkpoint:** B now associates `10.10.0.1` with A's MAC. The capture on A contains B's ICMP traffic. A packet may appear twice on A's interface as it arrives and is forwarded. ICMP redirects and some ping loss are possible; they do not invalidate the MAC and payload evidence. Ping replies alone would not prove interception.

??? info "Hint: the timer expired or nothing changed"
    Check the helper terminal for errors. Exit status 124 means the 120-second limit expired. If you need another attempt, complete Task 5's cleanup first. Restart the receiver from Task 2, then repeat Tasks 3–5 when ready. Keep the same three nodes and capture filter. There is no need to rush packet analysis while the helper runs.

## Task 4: Find a harmless cleartext marker

While the helper is running and the prepared receiver is still waiting, send this marker on **WAGO B**, using no real credentials:

```bash
printf 'LAB-ARP-DEMO\n' | nc -N -w 3 2.2.2.2 2323
```

The receiver should print `LAB-ARP-DEMO`. Record the changed neighbor mapping, then go straight to Task 5. The capture retains the packets for analysis after the helper stops.

**Checkpoint:** The receiver got the marker while B's gateway mapping pointed at A. Packet evidence is checked after cleanup below.

??? info "Hint: no marker"
    If the receiver or helper exited, complete Task 5's cleanup, restart the receiver from Task 2 and repeat Tasks 3–5. Use `tcp.port == 2323 && tcp.len > 0` to isolate data. If only SYN packets appear, check the listener and route rather than claiming payload capture.

## Task 5: Restore and verify

1. Press **Ctrl+C** in A's helper terminal if it is still running. Wait for its prompt; allow a few seconds for corrective ARP replies. Stop the receiver with **Ctrl+C** if it is still waiting.
2. On **WAGO B**, refresh only this dynamic gateway entry:

    ```bash
    ip neigh del 10.10.0.1 dev eth1
    ping -c 3 10.10.0.1
    ip neigh show 10.10.0.1 dev eth1
    ```

    If deletion reports that the entry does not exist, continue with ping.
3. On **WAGO A**:

    ```bash
    pgrep -a arpspoof
    cat /proc/sys/net/ipv4/ip_forward
    ```

**Checkpoint:** No `arpspoof` process remains, forwarding equals your initial value, and B again uses the original gateway MAC with successful pings. A silent `pgrep` with exit status 1 means no matching process. If the mapping remains wrong, stop here and ask the instructor; do not leave an active interception behind.

Now inspect the retained capture. Filter `tcp.port == 2323`, select a data packet and right-click **Follow → TCP Stream**. Find `LAB-ARP-DEMO`, record its packet number and close the stream window.

**Checkpoint:** The receiver and captured TCP payload contain the same marker. This, together with the changed neighbor mapping, proves the interception. Seeing an open port alone would not establish that application data was intercepted.

Keep the capture open for Extra 1 if desired. Otherwise close the capture and terminal tabs.

## Your incident handover

Write four lines: original gateway MAC and time; changed mapping and captured marker; recovered mapping and cleanup evidence; the next operational check. Verify approved gateway replacement/failover and the owner of the observed MAC before labelling a real alert malicious. Escalate with this evidence; a changed ARP entry alone is not a reason to disconnect a running PLC.

## Solution

??? success "Compare your evidence"
    Before the experiment B sends routed traffic to the real gateway MAC. The forged ARP replies replace that mapping with A's MAC, so B's routed packets pass through A. The receiver and capture both show the plaintext marker. On stopping the helper and refreshing B's entry, the original mapping returns.

    This demonstrates unauthenticated ARP mapping and readable TCP application data in this lab. It does not establish that encrypted application contents can be decrypted. Port security alone is not an IP-to-MAC validator. Dynamic ARP Inspection needs trusted bindings, including a deliberate plan for static-addressed OT devices.

## Extra 1: Build an ARP incident timeline

**Optional, 10 minutes.** Use the retained capture, or repeat Tasks 2–5.

1. Apply `arp.opcode == 2 && arp.src.proto_ipv4 == 10.10.0.1`.
2. Expand **Address Resolution Protocol** and record **Sender IP address**, **Sender MAC address** and a frame number before, during and after the experiment. If one phase is missing, label it unobserved; use your recorded neighbor table as separate evidence.
3. Compare the claimed IP with A's known MAC. Explain why a changed mapping is stronger evidence than simply counting ARP packets.
4. Reopen B's **ARP Cache** in Explorer. Compare the latest available row with your recorded phases. Explorer retains the latest report, not this packet timeline; label any phase it did not collect as unobserved.

**Checkpoint:** Your timeline distinguishes the real gateway mapping, the forged claim and recovery. A normal gratuitous ARP is not by itself proof of an attack. [Search the ARP field reference](https://www.wireshark.org/docs/dfref/a/arp.html) for `arp.src.hw_mac` and `arp.isgratuitous`.

## Extra 2: Choose protection that survives maintenance

**Optional, 10 minutes.** The cell uses static PLC addresses. Tomorrow the gateway is replaced with hardware that has a different MAC. Use your recorded gateway/attacker mappings to review these proposals; do not configure them.

| Proposal | Question to answer |
| --- | --- |
| Allow only each PLC's MAC on its switch port | Did A need a different source MAC to claim the gateway IP? |
| Configure a fixed gateway ARP entry on B | What must change when the legitimate gateway MAC changes? |
| Validate ARP claims at the switch | Where will trusted IP/MAC bindings for static PLCs come from? |
| Alert on changes only | Who checks the maintenance record and decides the response? |

**Checkpoint:** A can lie about the gateway IP while using A's own allowed MAC, so simple source-MAC port security is insufficient. A fixed ARP mapping needs a maintained replacement procedure. Dynamic ARP Inspection needs platform support and trustworthy bindings; a DHCP-only database does not automatically cover static devices. Monitoring detects evidence but does not itself block the forged claim.

Choose one preventive measure and one detection/response measure. Name their owner and a replacement-day acceptance check, including restored application communication. Do not enable an unqualified protection feature on a running cell. Close tools; continue with [Exercise 3](ex3-spanning-tree.md).

## Explore the topic

- [ARP specification, RFC 826](https://www.rfc-editor.org/rfc/rfc826.html): find the receive algorithm's mapping update.
- [arpspoof manual](https://manpages.debian.org/bullseye/dsniff/arpspoof.8.en.html): compare forwarding and target selection with the helper.
- [Cisco Dynamic ARP Inspection](https://www.cisco.com/c/en/us/support/docs/switches/lan-switch-software/222274-troubleshoot-dynamic-arp-inspection-dai.html): identify what a switch needs to validate an ARP claim.
- [osquery ARP table fields](https://github.com/osquery/osquery/blob/master/specs/arp_cache.table): compare the reported `address`, `mac` and `interface` with the live neighbor table.
