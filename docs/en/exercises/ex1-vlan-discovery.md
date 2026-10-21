# Exercise 1: Compare tagged and untagged traffic

**Time estimate:** 30–40 minutes

**Lab:** Student `ot-sec-flat`

**Difficulty:** Beginner

Compare the same ping traffic on two switch ports: the WAGO endpoint's access port and the uplink to the gateway. Find where an Ethernet frame carries a VLAN tag.

## Your maintenance task

A replacement PLC has been connected. Its port is up, but the team is unsure whether it is in the intended network. Establish the normal path before anyone changes the PLC or switch. You will compare a small diagnostic exchange at the PLC port and at the gateway uplink.

**Three terms:** A **VLAN** is a separate Ethernet broadcast domain. An **access port** normally carries one VLAN without a tag for the endpoint. A **trunk** carries tagged VLANs between network devices. An untagged PLC frame can therefore belong to a VLAN; the switch assigns it at ingress.

**Predict:** Should an ordinary PLC have to add VLAN tags itself to use this access port? Write your answer before Task 3.

## Before you start

- Select your live student lab network.
- The lab's terminal credentials must already be configured.

The links below open the right lab tool. In a tool tab, click the exercise title beside **Active exercise** to reopen these instructions. Your place, opened hints and assistant conversation follow you between tools.

This exercise uses the unchanged flat lab: WAGO `10.10.0.11`, gateway `10.10.0.1`, VLAN 10. If your lab has been reconfigured, its results may differ.

??? info "Reading this outside Network Explorer?"
    Open your assigned lab, select its live network and choose **Netmap**. Find the named node with search. Its action menu provides **Open terminal** and **Capture packets**; select the interface named in the task. For the VLAN view, select **VLAN** in Netmap's view selector and VLAN **10**. Reader links select these destinations automatically and retain your network and snapshot.

## Task 1: Find the path

[Find the WAGO endpoint on Netmap](/network/?search=wago-plc2a-vlan10).

Locate `wago-plc2a-vlan10`, its access switch `sw-acc1`, the distribution switch `sw-dist`, and gateway `gw`. Follow their connections.

[Inspect VLAN 10 on Netmap](/network/?viewMode=vlan&vlanIds=10). Compare the discovered membership with the historical names. Return to the WAGO Netmap link above for the port checks; the packet tasks will verify tagging on the wire.

Select `sw-acc1`, then open **Ports** in its details panel. Find the port facing WAGO by its description. Select `sw-dist` and open **Neighbors** to find the port facing `gw`.

**Observe:** Which sw-acc1 port faces the WAGO endpoint? Which sw-dist port faces the gateway?

??? info "Hint: identify the ports"
    In **Ports**, look for `wago-plc2a` in the description. Scroll the table sideways if that column is out of view. In **Neighbors**, find `gw` and read the local interface.

??? success "Explanation: a name is not configuration"
    In the unchanged lab, sw-acc1 `ether2` faces WAGO and sw-dist `ether12` faces `gw`. Hostnames are labels, not proof of VLAN membership: the captures will show whether a frame carries a tag.

## Task 2: Generate traffic

[Open the WAGO terminal](/network/terminal?node=wago-plc2a-vlan10). Wait for **Connected**. Keep this tab open for both captures.

In the WAGO terminal, enter:

```bash
ping -c 4 10.10.0.1
```

This sends four ping requests to the gateway.

**Observe:** Did all four requests receive replies?

??? info "Hint: terminal or ping problems"
    If the terminal does not connect, check the selected host and credentials. If ping gets no replies, confirm the lab is running and inspect the path in Netmap before continuing.

??? success "Explanation: what ping establishes"
    Replies demonstrate IP reachability. They do not prove VLAN isolation or firewall enforcement. Different IP subnets or hostname suffixes do not establish different VLANs.

## Task 3: Inspect the tagged uplink

1. [Open the gateway-uplink capture](/network/capture?node=sw-dist&interface=ether12). It selects `sw-dist`, port `ether12` facing `gw-firewall` (the `gw` shown on Netmap).
2. Choose **Start Capture** and wait for Wireshark to appear.
3. Return to the WAGO terminal and run the same four-packet ping again.
4. Return to the capture tab. In Wireshark's display-filter bar above the packet list, enter `icmp` and press Enter. Select a ping packet. In the packet details below, expand **802.1Q Virtual LAN** and read its VLAN ID.

Allow a few seconds after ping finishes for the capture display to catch up.

**Observe:** Is there an 802.1Q header? What VLAN ID does it contain?

??? info "Hint: no ping packets"
    Check that you started the capture before repeating the ping and that the ping received replies. The selected port must be sw-dist `ether12` facing `gw-firewall`. Clear the display filter to check whether any traffic was recorded.

    Packet-analysis results call this port `eth11`; the RouterOS label in the selector is `ether12`.

??? info "Hint: isolate tagged packets"
    Use `icmp && ip.addr == 10.10.0.11` if other ping traffic is present. The filter `vlan` shows tagged packets. A display filter changes what you see, not what was recorded.

??? success "Explanation: expected uplink observation"
    The ping crosses the tagged VLAN 10 uplink. Its Ethernet frame includes an 802.1Q header with VLAN ID 10.

Close the capture tab before continuing. This ends the packet capture.

## Task 4: Compare the untagged access port

1. [Open the WAGO access-port capture](/network/capture?node=sw-acc1&interface=ether2). The link opens a capture tab with `sw-acc1`, port `ether2` facing `wago-plc2a-vlan10`.
2. Choose **Start Capture** and wait for Wireshark to appear.
3. In the WAGO terminal, repeat `ping -c 4 10.10.0.1`.
4. Return to the capture tab. Apply `icmp`, select a ping packet and compare its packet details with the uplink packet.
5. Try the display filter `vlan`, then clear it or return to `icmp`.

**Observe:** How does this packet's Ethernet header differ from the uplink packet? What happened when you applied `vlan`?

??? info "Hint: an empty filtered view"
    The `vlan` filter hides untagged frames. Clear the filter to confirm whether packets were captured. Verify that you chose the endpoint-facing port rather than an uplink or management interface.

    Packet-analysis results name this sw-acc1 access port `eth1`; the RouterOS label in the selector is `ether2`.

??? success "Explanation: expected access-port observation"
    WAGO sends and receives untagged Ethernet frames on this access link. The switch associates them with VLAN 10 and adds an 802.1Q tag on the tagged uplink.

Close the capture tab and the WAGO terminal tab.

## Finish

The main practical ends after Task 4. Complete your handover below, then choose the optional maintenance checks if time permits.

You have compared the same traffic on an uplink and an access port. Check that you can explain where the VLAN tag appeared and why an empty `vlan` filter can coexist with successful ping traffic.

If you get stuck, open the existing assistant in any lab tab. It knows this exercise. Include the task number and, for capture questions, your filter or frame number. Wait for its answer before asking from another tab.

## Your handover

Record the PLC address, endpoint-facing port, uplink port and one frame from each capture. Explain to a colleague why an empty `vlan` filter at the PLC port is normal here. Identify one further check needed before declaring the PLC application operational: a successful application exchange, not only ping.

## Solution

??? success "Compare your evidence"
    The WAGO access port is sw-acc1 `ether2`; the gateway uplink is sw-dist `ether12`. Four successful pings produce four requests and four replies. On the gateway uplink, those packets carry VLAN ID 10. On the access link they have no 802.1Q header. A display filter hides packets without changing the recording.

    Keep one frame number from each capture and record the interface, filter and VLAN ID (or absence of a tag). For the same default ping, the tagged frame is four bytes longer. Neither reachability nor a hostname proves isolation.

## Extra 1: Check a replacement PLC connection

**Optional, 10 minutes.** The maintenance note says “WAGO A is in VLAN 10.” Produce the information a colleague needs to reconnect it correctly, without changing any settings.

1. Open [sw-acc1's terminal](/network/terminal?node=sw-acc1) and run:

    ```routeros
    /interface bridge port print detail where interface=ether2
    /interface bridge vlan print detail
    ```

2. On **WAGO A**, run:

    ```bash
    ip -br address show eth1
    ip route get 10.10.0.1
    ```

3. Record switch/port, **PVID** (the VLAN assigned to incoming untagged traffic), untagged VLAN membership, PLC IP/prefix and whether the PLC sends tags. Compare the configuration with Task 4's packet evidence.

**Checkpoint:** In this baseline ether2 assigns untagged traffic to VLAN 10, and WAGO uses `10.10.0.11/8` on eth1. The replacement connection needs both the intended switch-port configuration and compatible endpoint addressing. A link light or the suffix in a hostname is insufficient. The `/8` means addresses beginning `10.` are on the same IP subnet here; it is deliberately broad lab addressing, not a plant design recommendation.

Close the switch terminal. Keep your record as a commissioning checklist.

## Extra 2: Follow the next-hop MAC

**Optional, 10 minutes.** Does sending to a different IP destination always change the Ethernet destination?

1. Start a fresh WAGO access-port capture from Task 4.
2. In the WAGO terminal, run:

    ```bash
    ip route get 2.2.2.2
    ping -c 2 10.10.0.1
    ping -c 2 2.2.2.2
    ip neigh show 10.10.0.1 dev eth1
    ```

3. Filter with `icmp.type == 8 && ip.src == 10.10.0.11`. Compare **Internet Protocol → Destination Address** with **Ethernet II → Destination** in the requests to both targets.

**Checkpoint:** Both destinations reply. The IP destinations differ, but both requests use the gateway's Ethernet MAC on this access link. The route to `2.2.2.2` goes via `10.10.0.1`. Here `2.2.2.2` is the simulated DMZ jumphost inside the lab.

??? success "Why this happens"
    An Ethernet frame addresses the next hop on the local link. The IP packet addresses the final destination. The gateway forwards the jumphost packet onto another network and builds a new Ethernet frame for that link.

Close the capture and terminal tabs. Continue with [Exercise 2](ex2-arp-spoofing.md).

## Explore the topic

- [Wireshark VLAN fields](https://www.wireshark.org/docs/dfref/v/vlan.html): search for `vlan.id` and `vlan.priority`.
- [Wireshark display-filter guide](https://www.wireshark.org/docs/man-pages/wireshark-filter): try the filters from your observations.
- [MikroTik bridge VLAN table](https://help.mikrotik.com/docs/spaces/ROS/pages/28606465/Bridge+VLAN+Table): compare access-port PVID and tagged membership.
