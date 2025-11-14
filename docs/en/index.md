# Understanding OT Networks & Network Security Monitoring

**Workshop**

**Instructors:**

- [Martin Scheu](mailto:martin.scheu@switch.ch) - OT Security Engineer, [SWITCH](https://switch.ch)
- [Mischa Diehm](mailto:mischa.diehm@narrowin.ch) - CTO and Network Engineer, [Narrowin](https://narrowin.ch)

---

## Workshop Materials

<a href="slides/CPH2025%20-%20Unfold%20the%20OT%20Network%20Jungle%20-%20Diehm%20-%20Scheu.pdf" class="md-button md-button--primary">Download Workshop Slides (PDF)</a>

---

## Overview

### About This Workshop

Operational Technology (OT) networks are fundamentally different from traditional IT networks. They connect industrial control systems, PLCs, HMIs, and field devices that control critical infrastructure and manufacturing processes. Yet many organizations struggle to gain visibility into these environments, leaving them vulnerable to security threats and compliance gaps.

This hands-on workshop addresses the challenge of understanding and securing OT networks using practical, open-source tools. You will learn how to navigate the complexity of OT protocols, identify devices and data flows, discover blind spots, and implement effective network security monitoring tailored to industrial environments.

Unlike vendor-centric approaches, this workshop emphasizes pragmatic solutions that can be deployed immediately in real-world OT environments.

### What Makes OT Different

OT networks present unique challenges:

* Legacy protocols designed without security in mind (Modbus, S7, CIP, BACnet)
* Long lifecycles with devices that cannot be easily patched or replaced
* Real-time requirements where availability trumps confidentiality
* Limited network visibility due to flat architectures and unmanaged switches
* Heterogeneous environments mixing multiple vendors and protocols
* IT/OT convergence creating new attack surfaces

Understanding these differences is critical for effective network segmentation and security monitoring.

### Learning Objectives

By the end of this workshop, you will be able to:

* Identify and characterize common OT protocols and their security implications
* Select monitoring points based on risk assessment and network architecture
* Configure traffic collection and forwarding for inter-zone visibility
* Discover assets and map communication patterns in OT networks
* Recognize the limits of different approaches to network security monitoring in industrial settings
* Apply segmentation techniques at IT/OT boundaries and edge devices

---

## Course Requirements

**Level:** Intermediate, hands-on workshop

**What You Need to Know:**

* Networking fundamentals (IP addressing, VLANs, basic routing)
* Command line basics (ssh, ping, basic file operations)
* Container concepts helpful but not required (we explain what you need)

**Lab Environment:**

The lab runs in pre-configured cloud or local environments. See the [Lab Guide](lab.md#getting-started) for setup instructions.

---

## Pre-Workshop Preparation

No software installation required. The lab runs in the cloud via GitHub Codespaces or locally via DevPod.

**Before the workshop:**

1. Ensure you have a GitHub account
2. Review the [Lab Guide](lab.md#getting-started) for environment setup options
3. **Important:** If possible, start your Codespace before the workshop. Initial setup takes 5-10 minutes to download and configure container images. Subsequent starts are under 1 minute.
4. Optional: Familiarize yourself with basic Linux command line and networking concepts

---

## What You Will Take Away

* Practical experience with open-source OT lab environments
* Working knowledge of OT protocol analysis
* Detection use cases applicable to your environment
* Hands-on lab environment you can continue using after the workshop
* Network with peers facing similar OT security challenges

---

## Questions?

For technical questions or special requirements, contact the instructors before the workshop:

* Martin Scheu: martin.scheu@switch.ch
* Mischa Diehm: mischa.diehm@narrowin.com

We look forward to working with you during the workshop.
