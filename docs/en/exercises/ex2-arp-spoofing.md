# Exercise 2: Trace an ARP interception

**Time:** 30–40 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Intermediate.

Prove a change in the victim's IP-to-MAC mapping, follow the resulting traffic, and restore the original path. Use only the three named lab nodes below. These addresses refer to the simulated network inside this appliance.

**Learning objectives:** Explain ARP's trust problem, execute and prove a bounded Layer 2 interception, identify readable application data, and explain detection and mitigation.

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

Record A's MAC and forwarding value. The helper preserves the previous forwarding setting.

**Checkpoint:** The real gateway MAC differs from A's MAC. Both commands exist. If this is already a changed mapping, finish cleanup in Task 5 before proceeding.

## Task 2: Establish the normal path

[Open a capture on WAGO A's data port](/network/capture?node=wago-plc2a-vlan10&interface=eth1), choose **Start Capture**, then run on **WAGO B**:

```bash
ping -c 3 2.2.2.2
```

In Wireshark use `icmp && ip.addr == 10.10.0.12`. Wait a few seconds for the view to catch up.

**Checkpoint:** The pings reply, but their unicast requests and replies normally do not cross A's switched access link. Background ARP broadcasts can still appear. If victim unicast is already present, inspect the ARP mapping and selected interface before interpreting the experiment.

## Task 3: Redirect the selected pair

On **WAGO A**, run in the foreground:

```bash
timeout -s INT 60 arp_spoofing-simple.sh eth1 10.10.0.12 10.10.0.1
```

The helper targets only B and its gateway and stops automatically after 60 seconds. You can stop it earlier with **Ctrl+C**. Leave this terminal running while using B.

After about five seconds, on **WAGO B**:

```bash
ip neigh show 10.10.0.1 dev eth1
ping -c 3 2.2.2.2
```

**Checkpoint:** B now associates `10.10.0.1` with A's MAC. The capture on A contains B's ICMP traffic. A packet may appear twice on A's interface as it arrives and is forwarded. ICMP redirects and some ping loss are possible; they do not invalidate the MAC and payload evidence. Ping replies alone would not prove interception.

??? info "Hint: the timer expired or nothing changed"
    Check the helper's terminal for an error. If it has stopped, repeat the bounded command and then the victim commands. Keep the filter specific to B. Do not broaden the attack to other nodes. Exit status 124 after the timeout is normal.

## Task 4: Find a harmless cleartext marker

While the helper is running, start a bounded listener on the **DMZ jumphost**:

```bash
timeout 45 nc -l 2323
```

Leave it waiting. On **WAGO B** send a marker, using no real credentials:

```bash
printf 'LAB-ARP-DEMO\n' | nc -N -w 3 2.2.2.2 2323
```

The receiver should print `LAB-ARP-DEMO`. In the capture, filter `tcp.port == 2323`, select a data packet, then right-click **Follow → TCP Stream**. Find the same marker. Close the stream window afterwards.

**Checkpoint:** Record a packet number containing the marker and the changed neighbor mapping. Seeing an open port alone would not establish that application data was intercepted.

??? info "Hint: no marker"
    Start the listener before the sender. Repeat Task 3 if its timer expired. Use `tcp.port == 2323 && tcp.len > 0` to isolate data. If only SYN packets appear, check the listener and route rather than claiming payload capture.

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

Keep the capture open for Extra 1 if desired. Otherwise close the capture and terminal tabs.

## Solution

??? success "Compare your evidence"
    Before the experiment B sends routed traffic to the real gateway MAC. The forged ARP replies replace that mapping with A's MAC, so B's routed packets pass through A. The receiver and capture both show the plaintext marker. On stopping the helper and refreshing B's entry, the original mapping returns.

    This demonstrates unauthenticated ARP mapping and readable TCP application data in this lab. It does not establish that encrypted application contents can be decrypted. Port security alone is not an IP-to-MAC validator. Dynamic ARP Inspection needs trusted bindings, including a deliberate plan for static-addressed OT devices.

## Extra 1: Build an ARP incident timeline

**Optional, 10 minutes.** Use the retained capture, or repeat Tasks 2–5.

1. Apply `arp.opcode == 2 && arp.src.proto_ipv4 == 10.10.0.1`.
2. Expand **Address Resolution Protocol** and record **Sender IP address**, **Sender MAC address** and a frame number before, during and after the experiment. If one phase is missing, label it unobserved; use your recorded neighbor table as separate evidence.
3. Compare the claimed IP with A's known MAC. Explain why a changed mapping is stronger evidence than simply counting ARP packets.

**Checkpoint:** Your timeline distinguishes the real gateway mapping, the forged claim and recovery. A normal gratuitous ARP is not by itself proof of an attack. [Search the ARP field reference](https://www.wireshark.org/docs/dfref/a/arp.html) for `arp.src.hw_mac` and `arp.isgratuitous`.

## Extra 2: Find the boundary of this interception

**Optional, 10 minutes.** Predict whether changing B's gateway mapping intercepts every destination in `10.0.0.0/8`.

1. Start a fresh A data-port capture and repeat the bounded helper from Task 3.
2. While it runs, on **WAGO B**:

    ```bash
    ip route get 10.20.0.11
    ping -c 2 10.20.0.11
    ping -c 2 2.2.2.2
    ```

3. Compare `icmp.type == 8 && ip.src == 10.10.0.12 && ip.dst == 10.20.0.11` with `icmp.type == 8 && ip.src == 10.10.0.12 && ip.dst == 2.2.2.2` on A.

**Checkpoint:** The route to CtrlX `10.20.0.11` is direct on `eth1` in this `/8` flat baseline. Spoofing the gateway pair does not redirect that direct neighbor mapping; the routed jumphost traffic does cross A. Both pings can succeed while taking different Ethernet paths. The `vlan20` hostname does not create a separate VLAN.

Repeat **all of Task 5**, then close captures and terminals. Continue with [Exercise 3](ex3-spanning-tree.md).

## Explore the topic

- [ARP specification, RFC 826](https://www.rfc-editor.org/rfc/rfc826.html): find the receive algorithm's mapping update.
- [arpspoof manual](https://manpages.debian.org/bullseye/dsniff/arpspoof.8.en.html): compare forwarding and target selection with the helper.
- [Cisco Dynamic ARP Inspection](https://www.cisco.com/c/en/us/support/docs/switches/lan-switch-software/222274-troubleshoot-dynamic-arp-inspection-dai.html): identify what a switch needs to validate an ARP claim.
