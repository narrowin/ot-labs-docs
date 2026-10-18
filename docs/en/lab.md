# Student lab guide

Use the instructor-provided appliance in your browser. It contains the running `ot-sec-flat` network, configured terminals, packet capture, PLC visualizations and the seven exercises. You need your assigned LNE URL and login; no local tool installation is required.

## Getting started

1. Sign in to your assigned LNE. Select the student network and the **live** snapshot.
2. Open **Exercises** in the product launcher and choose Exercise 1. Select **English / Deutsch** for the instructions.
3. Open Explorer from the lesson. Its links carry your selected network and snapshot. In terminal and capture tabs, choose the **Active exercise** title to reopen the instructions beside your work.
4. Use the checks below, then work through the [seven exercises](exercises/index.md) in order.

The appliance is already deployed. Starting another topology, restarting runtimes or resetting shared devices would interrupt other students. Use the configured terminal action rather than assuming a device password.

## First five minutes

| Action in the reader | Observable checkpoint |
| --- | --- |
| [Find WAGO A](/network/?search=wago-plc2a-vlan10) | Netmap highlights the endpoint; follow its path through sw-acc1 and sw-dist to gw |
| [Open WAGO A terminal](/network/terminal?node=wago-plc2a-vlan10) | Status becomes Connected; the commands below show the data address and four gateway replies |
| [Inspect VLAN 10](/network/?viewMode=vlan&vlanIds=10) | Netmap is in VLAN view with VLAN 10 selected |
| [Open WAGO A WebVisu](/network/webvisu?node=wago-plc2a-vlan10) | A separate tab renders the native CODESYS visualization; steady values are possible |
| [Open the gateway-uplink capture](/network/capture?node=sw-dist&interface=ether12) | sw-dist ether12 is selected; choose Start Capture, repeat the ping and filter `icmp` |

Run on **WAGO A**:

```bash
ip -br addr show eth1
ping -c 4 10.10.0.1
```

Expect `10.10.0.11/8` and four replies in the unchanged lab. On the uplink, select a ping packet and expand **802.1Q Virtual LAN**: its VLAN ID is 10. Close the capture tab when finished. [Exercise 1](exercises/ex1-vlan-discovery.md) explains the comparison with the untagged access port.

**Standalone documentation:** These relative app links are interpreted by the LNE exercise reader. In a separately hosted copy, open your assigned LNE manually, select network/live snapshot and use Explorer → Netmap. Search the node name, then its action menu → **Open terminal** or **Capture packets** and select the stated interface. For WebVisu, open the responding PLC's details and choose its WebVisu browser action. In the Netmap view selector choose **VLAN**, then **10**; for [spanning tree](/network/?viewMode=spanning-tree&stpInstanceId=0&dataType=All), choose **Spanning tree**, instance **0**, data type **All**. Simulated data addresses are reached from inside the lab, not directly from your laptop.

## Understand the two network planes

The management plane connects the appliance to the nodes for SSH and collection. Linux nodes use `eth0` for this independent path. Their **data-plane** interface is `eth1`; captures and exercise probes use the network being studied. A working management terminal does not establish data-plane connectivity.

The current baseline has two OT/DMZ broadcast domains:

| Segment | VLAN | Addressing | Gateway |
| --- | --- | --- | --- |
| Flat OT | 10 | `10.0.0.0/8` | `10.10.0.1` |
| DMZ | 60 | `2.2.2.0/24` | `2.2.2.1` |

Names ending in `vlan20`, `vlan30`, `vlan40` or `vlan50` describe intended roles; those endpoints are currently in VLAN 10 with `/8` addresses. They are not already separated into `/24` security zones. The simulated external pair is `1.1.1.1` / `1.1.1.2` inside this appliance.

| Stable node name | Data address / role |
| --- | --- |
| `wago-plc2a-vlan10` | `10.10.0.11/8`, WAGO A PLC |
| `wago-plc2b-vlan10` | `10.10.0.12/8`, WAGO B PLC |
| `ctrlx-plc3-vlan20` | `10.20.0.11/8`, process PLC |
| `schneider-plc4-vlan30` | `10.30.0.11/8`, disposal PLC |
| `abb-800xa-vlan40` | `10.40.0.11/8`, control-room application |
| `pilz-hmi01-vlan40` | `10.40.0.12/8`, HMI browser |
| `dell-ews1-vlan50` | `10.50.0.11/8`, engineering workstation |
| `hirschmann-edge1-vlan50` | `10.50.0.12/8`, edge endpoint |
| `proxmox-jumphost` | `2.2.2.2/24`, DMZ test source |
| `panasonic-toughbook-internet` | `1.1.1.1/24`, external simulation |

RouterOS devices are `gw-firewall`, `sw-dist`, `sw-acc1` and `sw-acc2`. sw-dist and sw-acc1 use a two-member LACP bond. RSTP prevents the redundant switched path from looping. The lessons use the exact port labels and inspect operational state before drawing conclusions.

## Observe the simulated process

The CODESYS applications model water-treatment functions. Use WebVisu to observe the process and [Exercise 6](exercises/ex6-protocol-discovery.md) to correlate the GUI with HTTP and Modbus traffic. A vendor name is a role label in this container lab, not evidence that it runs that vendor's physical controller firmware. Not every endpoint has a PLC runtime or every OT protocol enabled.

Observe controls without operating them during the networking tasks. A steady value can be valid; ongoing successful application exchanges provide a better availability check than forcing a process change.

## Use the right tool

| Tool | Use |
| --- | --- |
| Netmap | Paths, neighbors, ports, VLAN and spanning-tree views |
| Configured terminal | Commands on the named node, with the existing credential selection |
| Packet capture | Start on the stated port before generating bounded traffic; use Wireshark display filters |
| WebVisu | Native visualization for an approved, responding runtime |
| Toolbox | Draft zones and directional policies; saving does not configure network devices |
| Assistant | Help with the active lesson, command output and packet evidence |

Toolbox is opened from the product launcher. Its saved network workspace is shared. Exercise 7 inspects it and drafts a policy in your own notes. Building VLAN 70 is currently an end-to-end design challenge; an applied version needs an isolated lab and restoration plan.

## If a checkpoint fails

Check the selected live network, node and interface first. A historical snapshot can display topology but cannot run a live terminal, capture or WebVisu. If a tool action is unavailable, ask the instructor to check bindings, credentials, runtime status and permissions. Record the specific node/action/error instead of changing shared configuration.

For capture problems, clear the display filter, confirm capture has started, repeat the four-packet ping and allow the viewer a few seconds to catch up. See the [filter and troubleshooting reference](exercises/appendix.md).

Close captures and terminals after each lesson. Exercise 2 additionally restores the victim's gateway mapping and verifies its helper has stopped. Continue with [Exercise 1](exercises/ex1-vlan-discovery.md).
