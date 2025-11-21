# OT Security Lab Exercises

## Overview

These hands-on exercises provide practical experience with OT network security concepts. Each exercise focuses on fundamental network security principles commonly encountered in industrial control system environments.

**Total Time**: 3-4 hours for all exercises

**Lab Requirements**: You need a running lab environment (GitHub Codespaces or DevPod). See the [Lab Environment](../lab.md) page for setup instructions.

**Exercise Approach**: Exercises are independent and can be completed in any order, though following the sequence provides a logical progression of concepts.

## Prerequisites

### Required Knowledge

- Basic Linux command line
- Network fundamentals (IP addressing, VLANs, routing)
- SSH basics
- Wireshark basics

### Lab Access

Before starting, ensure you can:

1. Access your lab environment (Codespaces or DevPod)
2. Deploy a lab topology using Containerlab extension
3. SSH into lab devices
4. Capture traffic using Wireshark via Edgeshark

### Topology Selection

These exercises use two lab topologies:

- `ot-sec-flat.clab.yml` - Flat network with VLANs but minimal segmentation
- `ot-sec-segmented.clab.yml` - Segmented network with firewall rules

Each exercise specifies which topology to use.

## Exercise List

| # | Exercise | Time | Topology | Difficulty | Link |
|---|----------|------|----------|------------|------|
| 1 | VLAN Discovery | 20-30 min | Both | Beginner | [Start :material-arrow-right:](ex1-vlan-discovery.md) |
| 2 | ARP Spoofing Attack | 30-40 min | Both (flat easier) | Intermediate | [Start :material-arrow-right:](ex2-arp-spoofing.md) |
| 3 | Spanning Tree Analysis | 25-35 min | Both | Intermediate | [Start :material-arrow-right:](ex3-spanning-tree.md) |
| 4 | Link Aggregation | 20-30 min | Segmented only | Intermediate | [Start :material-arrow-right:](ex4-link-aggregation.md) |
| 5 | Remote Access Assessment | 35-45 min | Both | Intermediate | [Start :material-arrow-right:](ex5-reconnaissance.md) |
| 6 | OT Protocol Discovery | 30-35 min | Both (flat easier) | Intermediate | [Start :material-arrow-right:](ex6-protocol-discovery.md) |
| 7 | Segmentation & Firewall Design | 35-45 min | Segmented (primary) | Intermediate/Advanced | [Start :material-arrow-right:](ex7-segmentation-firewall.md) |

## Additional Resources

- [Appendix: Wireshark Filters](appendix.md#wireshark-filters-quick-reference)
- [Troubleshooting Guide](appendix.md#troubleshooting-common-issues)
- [External Resources](appendix.md#additional-resources)
