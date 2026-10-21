# Container101

The workshop appliance runs simulated PLCs, switches and monitoring tools. Containers package these components so the instructor can provide a repeatable lab. You use the running lab through your browser.

## Your workshop workflow

1. Open the assigned LNE URL and sign in.
2. Follow the [Lab Guide](lab.md) to select the live student network.
3. Open [Exercise 1](exercises/ex1-vlan-discovery.md). Its links select the named terminal, capture or visualization.
4. Finish each exercise's cleanup, then close its tool tabs.

The instructor has already deployed the appliance and its lab network. Your laptop needs a browser and access to the assigned URL; no local container installation or GitHub account is required. Deployment and appliance resets are instructor operations.

## Images, containers and persistent data

| Object | Meaning | Lab example |
| --- | --- | --- |
| Image | Packaged application files and defaults used to create a container | Software for a PLC simulator |
| Container | A running instance with its own processes, filesystem view and network interfaces | The `wago-plc2a` lab node |
| Persistent data | Files kept in a volume or mounted host directory, outside the container's writable layer | Saved device or application state |

An image is a starting point. A running container also has current processes, connections and configuration. Restarting it does not automatically restore the exercise's starting state. Persistent data can survive even when a container is recreated. [Docker explains volume persistence](https://docs.docker.com/engine/storage/volumes/).

## Containers and virtual machines

A virtual machine runs a guest operating system and kernel. Linux containers normally share their host's kernel while isolating processes and resources. The workshop can use both: the appliance can be a VM, with lab components running inside it. Some network-device containers start a virtualized network OS internally.

| Question | Container | Virtual machine |
| --- | --- | --- |
| What starts? | Isolated application processes | A guest operating system |
| What is shared? | Host kernel | Physical hardware through a hypervisor |
| Does restart mean a clean lab? | Depends on stored state and startup configuration | Depends on disks, snapshots and startup configuration |

Startup time and resource use depend on the component. A running container alone does not prove that its application is ready or its simulated protocol is complete. The exercises check replies, decoded packets and application behavior.

## What Containerlab contributes

Containerlab reads a topology definition describing nodes, their software images and their links. It creates the connections used by the lab. This makes a topology easier to reproduce than manually wiring each virtual device.

Use the [Lab Guide's network table](lab.md#understand-the-two-network-planes) to find each node's data address and VLAN. The exercise links select the required node and interface.

## Know which terminal you opened

| Terminal | Where commands run |
| --- | --- |
| A node's **Open terminal** action | Inside that named device: Linux on `wago-plc2a`, RouterOS on a switch or gateway |
| Instructor's appliance console | On the host that runs Docker, the lab nodes and platform services |

The exercise commands name their target above each block. Keep that node name visible while working. If a terminal disconnects, record the node and error before restarting anything: a restart can erase diagnostic evidence and temporary configuration in your lab.

## Optional instructor demonstration: inspect the running lab

Ask the instructor to demonstrate these read-only commands on the **assigned appliance's console**. They use its existing Docker installation. A PLC or switch terminal is a different context.

List running lab containers:

```bash
docker ps --filter label=containerlab=ot-sec-flat --format 'table {{.Names}}\t{{.Status}}'
```

Find `clab-ot-sec-flat-wago-plc2a-vlan10` and match it to `wago-plc2a` on Netmap. Then inspect only its name and image:

```bash
docker inspect --format '{{.Name}} {{.Config.Image}}' clab-ot-sec-flat-wago-plc2a-vlan10
```

**Checkpoint:** You can distinguish the container's running status, its packaged image and the application's behavior. A successful ping from `wago-plc2a` or WebVisu response provides different evidence from Docker's `Up` status.

## What happens to state?

Closing a browser tab ends that tool session; it does not reset the appliance. Device configuration and application data may remain between sessions. Follow each exercise's cleanup and verify its checkpoint.

Exercises 1–6 use the flat baseline in your own lab. Exercise 7 temporarily builds a routed VLAN in that lab, then restores the baseline. Its instructor preparation includes a [private checkpoint and restore procedure](lab-practical.md). A historical snapshot in Explorer displays collected network data; it is not an appliance restore operation.

## What this lab teaches

Virtual links carry real Ethernet frames and protocol messages that your capture tools can inspect. Simulators make it practical to repeat networking and security observations without operating production equipment. Their timing, protocol coverage and firmware behavior still depend on the installed component; they do not establish hard real-time or safety behavior for a physical plant.

## Explore the concepts

- [Docker: what is a container?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/): compare a process container with a VM.
- [Docker volumes](https://docs.docker.com/engine/storage/volumes/): explain why persistent files can outlive a container.
- [Containerlab topology definition](https://containerlab.dev/manual/topo-def-file/): find the `nodes` and `links` sections that define a lab's devices and connections.

Continue with the [Lab Guide](lab.md) or [Exercise 1](exercises/ex1-vlan-discovery.md).
