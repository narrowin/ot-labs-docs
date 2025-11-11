# Workshop Cheatsheet

## Credentials

```
Username: admin
Password: admin
```

## Start and Stop Lab

Use the VS Code Containerlab extension with the Graph TopoViewer. Right click on `ot-sec-segmented.clab.yml` (or `ot-sec-flat.clab.yml`) and select `Graph Topoviewer`.

To start the lab press the play button in the TopoViewer navigation.


## Connect to Devices

Quick SSH (use device name):

```bash
ssh clab-ot-sec-flat-proxmox-jumphost
ssh clab-ot-sec-flat-wago-plc2a-vlan10
ssh clab-ot-sec-flat-wago-plc2b-vlan10
ssh clab-ot-sec-flat-abb-800xa-vlan40
```

Or SSH by IP address:

```bash
ssh admin@192.168.100.52   # jumphost
ssh admin@192.168.100.53   # WAGO PLC 2A
ssh admin@192.168.100.54   # WAGO PLC 2B
ssh admin@192.168.100.57   # ABB 800XA
```

## Web Access

HMI Desktop:

<http://192.168.100.60:5800>

ABB System:

<http://192.168.100.57:8080>

## Commands on PLC

Check if PLC runtime is running:

Ping test:

```bash
ping 10.10.0.1
```

Capture network traffic:

```bash
sudo tcpdump -lnei eth1
```

## Scan Network from Jumphost

Find devices:

```bash
nmap 10.10.0.0/24
```

Check if PLC port is open:

```bash
nc -zv 10.10.0.11 1217
```
