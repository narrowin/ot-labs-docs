# Exercise 5: Assess access from the DMZ

**Time:** 30–40 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Intermediate.

Find what a DMZ source can reach and produce an evidence-based finding. Limit discovery to the four listed lab IPs and service checks to WAGO A. No broad subnet scans, password guessing or software installation are needed.

**Learning objectives:** Establish remote-access position, discover scoped lab hosts, identify reachable services, distinguish observations from vulnerabilities, and document a reproducible assessment with recommendations.

## Before you start

Open the configured [DMZ jumphost terminal](/network/terminal?node=proxmox-jumphost). All main-task commands run there. Use [Netmap's WAGO search](/network/?search=wago-plc2a-vlan10) to locate the target and its gateway path.

| Target | Role |
| --- | --- |
| `10.10.0.1` | OT gateway |
| `10.10.0.11` | WAGO A, service-check target |
| `10.10.0.12` | WAGO B |
| `10.20.0.11` | CtrlX |

These are lab addresses. Historical `vlan20` names do not establish separate VLANs in the current flat OT network.

??? info "Using standalone documentation?"
    Open your assigned live lab's **Netmap**, search **proxmox-jumphost** and use **Open terminal** in its action menu. For Extra 1 select **wago-plc2b-vlan10**. For Extra 2 select **wago-plc2a-vlan10**, **Capture packets**, interface **eth1**. Reader links retain your network and snapshot and select these targets automatically.

## Task 1: Establish your vantage point

```bash
date -Iseconds
hostname
ip -br addr show eth1
ip route get 10.10.0.11
command -v nmap curl
```

**Checkpoint:** The jumphost has data address `2.2.2.2/24`. The selected route to WAGO A uses gateway `2.2.2.1`, interface `eth1` and source `2.2.2.2`. The management interface is a separate path; record the route actually selected for the target.

## Task 2: Discover only the listed hosts

```bash
nmap -n -sn --max-retries 1 --host-timeout 10s 10.10.0.1 10.10.0.11 10.10.0.12 10.20.0.11
```

`-n` avoids DNS lookups; `-sn` performs host discovery without a port scan. Record the responding IPs.

**Checkpoint:** All four responded in the unchanged lab walkthrough. Your result is an observation from this source at this time. A missing reply is not proof that a host is absent or that a particular VLAN is blocked.

??? info "Hint: discovery and service reachability differ"
    Confirm the route first. The next task uses `-Pn` on one known lab target, so a host-discovery probe is not required before its service check. Do not expand the target range when troubleshooting.

## Task 3: Measure exposed TCP ports

```bash
nmap -n -sT -Pn -p22,502,4840,8080 --max-retries 1 --host-timeout 20s --scan-delay 100ms 10.10.0.11
```

This makes bounded TCP connection attempts. It does not authenticate or send PLC write commands. Record each port's **STATE** separately from **SERVICE**.

| Port | Common use; still needs verification |
| --- | --- |
| 22/tcp | SSH |
| 502/tcp | Modbus TCP |
| 4840/tcp | OPC UA |
| 8080/tcp | Lab WebVisu HTTP |

**Checkpoint:** All four were open in the baseline walkthrough. Without version detection, Nmap's SERVICE column is primarily a port database label; `http-proxy` on 8080 does not prove that the application is a proxy. An open TCP port does not prove a vulnerability or an unauthenticated application.

## Task 4: Confirm one application

```bash
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

This retrieves the page, prints headers and discards the body. Read the status and content type. Do not enter credentials or operate process controls.

**Checkpoint:** The walkthrough returned HTTP `200 OK`, `Content-type: text/html` and a CODESYS `3S_WebServer` banner. This is stronger HTTP evidence than the port label. A banner alone does not establish software version, patch status or exploitability.

??? info "Hint: connection works but the page fails"
    Retain the HTTP status. A 404 is a responding HTTP service with a missing path; a timeout is a different result. Confirm `/webvisu.htm`, the target and port. A temporarily restarting runtime may require waiting and repeating this one request.

## Task 5: Write a finding someone else can reproduce

In your own notes, complete:

| Field | Your evidence |
| --- | --- |
| Time and source | Timestamp, hostname, selected source IP |
| Target and scope | Exact IPs and ports checked |
| Observation | Port states plus HTTP status and banner |
| Impact | Which DMZ-to-OT access was demonstrated |
| Limit | What you did not test: login, authorization, encryption or vulnerabilities |
| Recommendation | One required business flow and one unnecessary exposure to review |
| Retest | Exact source, target, command and intended result after a future policy change |

**Checkpoint:** Another student can reproduce the observation without guessing the source or scan range. Say “reachable from the DMZ” if that is what you measured. Do not claim default credentials were accepted: this exercise never tries them. Close the terminal when done.

## Solution

??? success "Compare your finding"
    The DMZ jumphost routes through `2.2.2.1` to the flat OT network. Four specified targets respond; WAGO A accepts TCP connections on 22, 502, 4840 and 8080 in the baseline. The HTTP response confirms an application on 8080.

    A defensible finding is “The DMZ source can initiate connections to these OT services; confirm which are required and restrict unnecessary initiation.” It is not “all PLCs are vulnerable.” `open`, `closed` and `filtered` describe what this scan could establish from this source; they are not security ratings.

## Extra 1: Compare two vantage points

**Optional, 10 minutes.** Repeat this small scan from the **jumphost**, then from [WAGO B's terminal](/network/terminal?node=wago-plc2b-vlan10):

```bash
ip route get 10.10.0.11
nmap -n -sT -Pn -p22,8080 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
```

Make a two-row table of source, next hop and port states.

**Checkpoint:** The baseline exposes both ports from both sources. WAGO B's route is direct on `eth1`; the jumphost's route is via a gateway. Identical port results do not imply identical paths or that a gateway firewall can filter the direct same-VLAN flow. Close B's terminal.

## Extra 2: Prove open versus closed with packets

**Optional, 10 minutes.** [Capture WAGO A's data port](/network/capture?node=wago-plc2a-vlan10&interface=eth1), choose **Start Capture**, then run on the **jumphost**:

```bash
nmap -n -sT -Pn -p8080,65000 --max-retries 1 --host-timeout 10s --scan-delay 100ms 10.10.0.11
```

Wait a few seconds. In Wireshark apply:

```wireshark
ip.addr == 2.2.2.2 && (tcp.port == 8080 || tcp.port == 65000)
```

Find the initial SYN and the target's response for each port; inspect **TCP → Flags**.

**Checkpoint:** In this baseline 8080 returns SYN/ACK and 65000 returns RST/ACK, matching `open` and `closed`. If a later lab uses 65000, record the actual response instead. A filtered result needs different evidence; silence alone does not identify the responsible firewall rule. Close capture and terminal tabs. Continue with [Exercise 6](ex6-protocol-discovery.md).

## Explore the topic

- [Nmap port-state definitions](https://nmap.org/book/man-port-scanning-basics.html): compare `closed` and `filtered`.
- [Nmap host-discovery controls](https://nmap.org/book/host-discovery-controls.html): find `-sn` and `-Pn`.
- [Nmap TCP connect scans](https://nmap.org/book/man-port-scanning-techniques.html): find **TCP connect scan**.
