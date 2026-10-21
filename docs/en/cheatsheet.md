# Workshop quick reference

Use your assigned, already-running student LNE. Select the live network. Tool links below work in the exercise reader; in standalone documentation use the [manual navigation](lab.md#first-five-minutes). The [exercise reference](exercises/appendix.md) covers packet filters and troubleshooting.

## Open a tool

| Action | Destination |
| --- | --- |
| [terminal on `wago-plc2a-vlan10`](/network/terminal?node=wago-plc2a-vlan10) | Main observation source, `10.10.0.11/8` |
| [terminal on `wago-plc2b-vlan10`](/network/terminal?node=wago-plc2b-vlan10) | Second source, `10.10.0.12/8` |
| [`proxmox-jumphost` terminal](/network/terminal?node=proxmox-jumphost) | Routed source, `2.2.2.2/24` |
| [Gateway-uplink capture](/network/capture?node=sw-dist&interface=ether12) | Tagged VLAN 10 traffic |
| [access-port capture for `wago-plc2a-vlan10`](/network/capture?node=sw-acc1&interface=ether2) | Untagged endpoint traffic |
| [VLAN view](/network/?viewMode=vlan&vlanIds=10) | VLAN 10 in the selected network/snapshot |
| [Spanning-tree view](/network/?viewMode=spanning-tree&stpInstanceId=0&dataType=All) | Instance 0, all data |

Wait for **Connected** in the terminal. Start a capture before generating traffic. Stop it before changing targets. Use `exit` and close terminal/capture tabs after work. Configured actions select existing credentials.

## PLC web interfaces

| Runtime | Reader link |
| --- | --- |
| `wago-plc2a-vlan10` | [WebVisu](/network/webvisu?node=wago-plc2a-vlan10) |
| `wago-plc2b-vlan10` | Not currently offered by the lab's approved browser actions; use `wago-plc2a-vlan10` for the visualization task |
| `ctrlx-plc3-vlan20` | [WebVisu](/network/webvisu?node=ctrlx-plc3-vlan20) |
| `schneider-plc4-vlan30` | [WebVisu](/network/webvisu?node=schneider-plc4-vlan30) |
| `abb-800xa-vlan40` | [WebVisu](/network/webvisu?node=abb-800xa-vlan40) |

Only approved, responding runtimes in the live network are offered. In standalone documentation open the PLC's WebVisu action in LNE. Direct `http://<data-IP>:8080/webvisu.htm` is an inside-lab address; it is not a laptop port-forwarding instruction.

## Bounded Linux checks

Run on the node named in the lesson:

```bash
ip -br addr show eth1
ip route get 10.10.0.11
ip neigh show dev eth1
ping -c 4 10.10.0.1
```

From the **`proxmox-jumphost`**, the scoped service check is:

```bash
nmap -n -sT -Pn -p22,502,4840,8080 --max-retries 1 --host-timeout 20s --scan-delay 100ms 10.10.0.11
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

These observations do not change a PLC program. Use the exact read-only Modbus command in [Exercise 6](exercises/ex6-protocol-discovery.md). Follow [Exercise 2](exercises/ex2-arp-spoofing.md) for the bounded ARP task and its mandatory cleanup.

## RouterOS observations

```routeros
/interface bridge monitor bridge once
/interface bridge port print
/interface bridge vlan print
/ip firewall filter print detail where chain=forward
```

The baseline is OT VLAN 10 with `/8` addresses and DMZ VLAN 60. A hostname, open port or final drop rule alone does not prove segmentation. Work in your own lab. [Exercise 7](exercises/ex7-segmentation-firewall.md) creates VLAN 70 there and includes cleanup to restore the flat baseline. Continue in [exercise order](exercises/index.md).
