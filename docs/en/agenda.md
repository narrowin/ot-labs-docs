# Two-day course lab sequence

Use the **OT Network Security – Design and Defend** handout for the teaching sessions and these exercises for the practical work. Allow about 5–6½ hours for main lab work and discussion across both days; breaks and lecture time are additional. Optional extensions can be completed later.

## Day 1: connections, boundaries and availability

1. Introduce the water-treatment plant, consequences of loss of control, and the difference between the physical process and the network diagram. Complete the [lab access check](lab.md#first-five-minutes).
2. Teach Ethernet, IP addressing and VLANs, then [Exercise 1](exercises/ex1-vlan-discovery.md): establish a PLC's actual network connection.
3. Teach ARP and Layer 2 trust, then complete [Exercise 2](exercises/ex2-arp-spoofing.md) in each student's lab. Verify cleanup before the next activity.
4. Introduce link aggregation and RSTP, then [Exercise 3](exercises/ex3-spanning-tree.md) and [Exercise 4](exercises/ex4-link-aggregation.md). Discuss standby links, capture coverage and shared failure points without creating an outage.

## Day 2: remote access, evidence and zone design

1. Teach the remote-access lifecycle, then [Exercise 5](exercises/ex5-reconnaissance.md): compare supplier requirements with actual DMZ exposure.
2. Discuss monitoring placement and protocol evidence, then [Exercise 6](exercises/ex6-protocol-discovery.md): record observed plant communication and one controlled read.
3. Use [Exercise 7](exercises/ex7-segmentation-firewall.md) for the water-treatment zone design, individually or with discussion. Check the mixed controller responsibilities before drawing zone boundaries. Each student implements and verifies one small conduit in their own lab, then restores the baseline.
4. With the routed diode installed, use [Exercise 8: Microsegment a legacy endpoint](exercises/ex8-diode-microsegmentation.md) (35–45 minutes). Preserve one engineering source, block another and contain new endpoint-initiated connections; compare with the VLAN boundary and restore the baseline.
5. Relate results to the handout's logging, roadmap and [automation outlook](automation.md). Each student selects one improvement, an owner, a dependency and an acceptance check.

## Short workshop selection

For a shorter event, choose a complete outcome rather than rushing through all eight lessons: Exercise 1's paired captures, Exercise 4's monitoring blind spot, or Exercise 6's single Modbus transaction. Exercise 7's group design can stand alone; do not start its configuration build without time and access for verification and cleanup.
