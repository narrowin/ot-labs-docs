# Exercise 6: Identify OT protocols from evidence

**Time:** 40–50 minutes. **Lab:** current student `ot-sec-flat`. **Difficulty:** Guided practice.

Identify actual protocol messages, make one bounded Modbus read, and separate observed application behavior from assumptions based on a port number. Do not change PLC values, download programs or operate process controls.

**Learning objectives:** Identify Modbus and HTTP messages, inspect CODESYS/OPC UA listeners, distinguish local Ethernet discovery from routed protocols, and explain which security questions these checks leave open.

## Your commissioning task

After a maintenance change, the team says: “The PLC answers ping, so the network is fine.” Build a more useful check: can the client reach the required service, send the right request and receive an interpretable answer? Also identify which discovery traffic cannot simply cross a router.

A **protocol** defines the messages applications exchange. A Modbus **register** is an addressed data item; the vendor's register map defines its meaning, type and units. A **transaction ID** helps pair a request with its reply within the same TCP conversation. An open port alone answers none of these application questions.

**Predict:** Would a successful TCP connection to port 502 prove that the requested register exists or that its value is a temperature?

## Before you start

Open [`wago-plc2b-vlan10`'s terminal](/network/terminal?node=wago-plc2b-vlan10) for the client commands and [`wago-plc2a-vlan10`'s terminal](/network/terminal?node=wago-plc2a-vlan10) for local listener inspection. The target is `wago-plc2a-vlan10` `10.10.0.11`; `wago-plc2b-vlan10` is `10.10.0.12`.

[Open `wago-plc2a-vlan10`'s data-port capture](/network/capture?node=wago-plc2a-vlan10&interface=eth1) and choose **Start Capture**. Keep it running through the main tasks. Use the exercise title to read beside the capture.

??? info "Using standalone documentation?"
    Open your assigned live lab's **Netmap** and search for `wago-plc2a-vlan10` and `wago-plc2b-vlan10`. Use **Open terminal** for the client/server checks. On `wago-plc2a-vlan10` use **Capture packets**, interface **eth1**, or **Open WebVisu** for the visualization. Reader links make those selections and retain network/snapshot; only approved responding WebVisu runtimes in the live view are available.

## Task 1: Discover before probing

Watch about 15 seconds. Clear the display filter, then open **Statistics → Protocol Hierarchy**. Note the observed protocols and close that dialog.

Try `modbus` and `pn_dcp` separately. For one message, record source, destination, protocol, time and frame number. For discovery frames without IP, record Ethernet addresses.

**Checkpoint:** The baseline has background Modbus traffic and lab-generated PROFINET DCP Identify messages. Background Modbus can include reads and writes from other lab applications; observing them does not mean you issued them. If a protocol is absent in your observation window, record “not observed”, not “not installed”.

Before generating your own traffic, find one background request and its response if present. Record the client, server and function. In the course plant, `abb-800xa-vlan40` is the supervisory system; `wago-plc2a-vlan10` and `wago-plc2b-vlan10` are the treatment-controller endpoints. Verify the actual source against Netmap; do not assume every Modbus client is `abb-800xa-vlan40`. Your `wago-plc2b-vlan10` → `wago-plc2a-vlan10` read below is a diagnostic transaction, not proof of the plant's normal control relationship. An unexplained write needs an owner and purpose, not automatic blocking.

## Task 2: Make and trace one Modbus read

On **`wago-plc2b-vlan10`**, check the installed tool and run exactly this command:

```bash
command -v mbpoll
mbpoll -m tcp -a 1 -t 4 -r 1 -c 2 -1 -o 2 10.10.0.11
```

`-t 4` selects holding registers, `-r 1` starts at the first displayed reference, `-c 2` requests two registers, `-1` polls once and `-o 2` bounds the response wait. There are no trailing write values: this is a read.

In Wireshark use `modbus && ip.addr == 10.10.0.12` to separate your request from the background. Expand **Modbus/TCP** and **Modbus**. Match request and response by transaction ID.

**Checkpoint:** The request has function **3 (Read Holding Registers)**, wire starting address **0** and quantity **2**. A response returns two register values; the walkthrough returned zeros, but process values can vary. Record the actual values without assigning engineering units that the register map has not established.

??? info "Hint: TCP connects but the read fails"
    An exception response is still Modbus protocol evidence. Check unit 1, holding-register type and the exact address/count. Record the exception; do not try write functions or sweep register ranges. If the runtime is restarting, wait and repeat this one read.

## Task 3: Verify HTTP and view the live application

On **`wago-plc2b-vlan10`**:

```bash
curl -sS --max-time 5 -D - -o /dev/null http://10.10.0.11:8080/webvisu.htm
```

In Wireshark apply `http.request && ip.src == 10.10.0.12`. Find the GET path, then inspect its response with `http && ip.addr == 10.10.0.12`.

[Open `wago-plc2a-vlan10`'s WebVisu](/network/webvisu?node=wago-plc2a-vlan10). Observe the page and record a displayed value or status twice, a few seconds apart, without pressing its controls. A steady process value is valid. Close the visualization afterwards.

**Checkpoint:** The direct request returns HTTP 200 and the trace contains GET `/webvisu.htm`. The visualization renders process controls/status and continues exchanging application data. Stable values alone do not establish a frozen page. Browser-proxied WebVisu and `wago-plc2b-vlan10`'s direct request have different source paths; do not expect the browser request to originate from `wago-plc2b-vlan10`.

## Task 4: Distinguish listeners, protocols and security

On **`wago-plc2a-vlan10`**:

```bash
ss -lnt
ss -lnu
```

Record the listening addresses and ports. This baseline includes TCP 502, 4840, 8080 and 11740, plus CODESYS UDP 1740.

On **`wago-plc2b-vlan10`**, check the TCP connection to port 4840:

```bash
nc -vz -w 3 10.10.0.11 4840
```

Filter the capture with `ip.addr == 10.10.0.12 && tcp.port == 4840`. Find the SYN and SYN/ACK. This command opens and closes a TCP connection; it sends no OPC UA application message.

| Check | What the result establishes |
| --- | --- |
| Local listener on TCP 4840 | A service is listening on that address and port |
| `wago-plc2b-vlan10`'s TCP connection succeeds | That listener is reachable from `wago-plc2b-vlan10` |
| Decoded Modbus request and response | The requested Modbus read was processed |
| HTTP request and response, plus live WebVisu | The web application responds; inspect ongoing exchanges for live updates |
| OPC UA authentication or encryption | Not checked by this TCP connection |

**Checkpoint:** Keep port availability separate from application behavior. Record OPC UA session security as **unverified**. Checking it would require an OPC UA client session and its negotiated security settings.

Close the capture and terminals, or keep them for an extra section.

## Your commissioning record

For Modbus, HTTP and OPC UA, record source → target, service, actual message/result and what remains unknown. For Modbus include the request function, address and matching response; leave units unknown without a register map. For HTTP distinguish the direct client test from the browser proxy. For OPC UA, the listener and TCP connection check leave session security unverified. This is the start of a communication matrix you can use in Exercise 7.

## Solution

??? success "Compare your evidence"
    Protocol Hierarchy classifies captured messages. Your single Modbus request uses function 3 and a zero-based address, while mbpoll's default display uses reference 1. The matching response proves a read, and the visible fields show that this exchange is readable on the capture link.

    The HTTP GET and response establish application behavior on 8080. WebVisu uses its own exchanges; loading a page does not prove that it uses Modbus. A TCP listener is weaker evidence than a decoded protocol transaction. Neither a protocol's advertised security capabilities nor an open port proves the settings of a live session.

## Extra 1: Check an HMI register mapping

**Optional, 10 minutes.** A replacement HMI labels the first register “0”; the maintenance sheet calls it “1”. Determine whether that alone means the HMI reads the wrong process value. Start a fresh data-port capture on `wago-plc2a-vlan10` if needed. On **`wago-plc2b-vlan10`**, run these two reads:

```bash
mbpoll -m tcp -a 1 -t 4 -r 1 -c 2 -1 -o 2 10.10.0.11
mbpoll -m tcp -a 1 -t 4 -0 -r 0 -c 2 -1 -o 2 10.10.0.11
```

Filter `modbus && ip.addr == 10.10.0.12`. Compare the starting addresses and quantities in the two requests, and compare mbpoll's printed labels.

**Checkpoint:** Both request wire address 0 with quantity 2. The first labels its results `[1]` and `[2]`; the second uses `[0]` and `[1]`. Process values may change between reads. Different display numbering does not imply different wire addresses. Do not append values, which would turn the command into a write.

## Extra 2: Explain why remote device discovery can fail

**Optional, 10 minutes.** A remote engineer can reach a PLC by IP but cannot find it with local PROFINET discovery. Would opening another TCP port solve that? In `wago-plc2a-vlan10`'s capture apply `pn_dcp`. Select an Identify message and inspect its Ethernet destination, EtherType and DCP service fields. Use [the DCP field reference](https://www.wireshark.org/docs/dfref/p/pn_dcp.html) to find `pn_dcp.service_id` and the station-name field.

Now try `pn_dcp && ip` and compare it with `pn_dcp` alone.

**Checkpoint:** These lab discovery frames use Ethernet EtherType `0x8892`; they contain no IP/TCP header. A TCP-port-only rule matrix cannot describe this flow. Record any advertised station name as a claim from the sender, not authenticated identity. These Identify messages do not establish that the simulator implements a complete PROFINET IO controller/device relationship.

Explain why a routed VLAN boundary and an IP firewall have different effects on this local Ethernet discovery. Close all tool tabs. Continue with [Exercise 7](ex7-segmentation-firewall.md).

## Explore the topic

- [Modbus specifications](https://www.modbus.org/modbus-specifications): locate function 03; distinguish classic Modbus TCP from Modbus Security over TLS.
- [mbpoll reference](https://github.com/epsilonrt/mbpoll): compare `-0`, `-1` and the optional write values.
- [OPC Foundation security modes](https://reference.opcfoundation.org/specs/OPC-10000-2/4.8): compare integrity with confidentiality.
- [CODESYS WebVisu execution](https://content.helpme-codesys.com/en/CODESYS%20Visualization/_visu_execute_webvisu.html): distinguish a web visualization from an engineering connection.
