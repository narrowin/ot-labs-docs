# Container101

## Why This Page Exists

You're an OT professional working with PLCs, HMIs, industrial networks, and protocols like Modbus or PROFINET. You might have heard about Docker and containers but never really needed them. Now, for this workshop, we're using Containerlab, which is built on container technology.

This page explains what containers are, why they're perfect for our OT security and networking workshop, and why there's nothing to fear.

## The Simple Truth About Containers

**Containers are just a way to package and run software that makes it easy to start, stop, and repeat the same setup every time.**

That's it. No magic. No hidden complexity. Everything the software needs is packaged together, and it runs the same way on any system. Like having a ready-to-go appliance instead of assembling components each time.

## Why Containers Matter for This Workshop

We're building network security labs with PLCs, HMIs, switches, and security monitoring tools. Here's what containers give us:

**Fast and Repeatable**  
Instead of installing software manually on each machine, we can start a complete lab in seconds. Need to restart? It takes seconds, not minutes or hours.

**Clean and Isolated**  
Each component (PLC simulator, HMI, network tool) runs separately. If something breaks, you just restart that one piece.

**Easy to Share**  
We can give you the exact same lab setup as everyone else. No "it works on my machine" problems.

**Lightweight**  
You can run multiple PLCs, switches, and tools on your laptop without it melting. Containers use much less memory and CPU than traditional virtual machines.

## Containers vs Virtual Machines: What You Need to Know

You might be familiar with virtual machines (VMs). Containers are similar but much lighter.

### Virtual Machines

- Complete operating system for each VM
- Takes minutes to start
- Uses lots of memory and disk space
- Good isolation but heavy

### Containers

- Share the underlying operating system
- Start in seconds
- Use much less memory and disk
- Light but still isolated enough for labs

**For this workshop:** Think of containers as "lightweight VMs that start instantly." That's close enough.

### Simple Comparison Table

| What You Need | Virtual Machine | Container |
|---------------|-----------------|-----------|
| Start a PLC simulator | Wait 2-3 minutes | Ready in 2-3 seconds |
| Run 10 different devices | Heavy on your laptop | Comfortable on your laptop |
| Reset to clean state | Slow shutdown and restart | Instant |
| Share exact setup with colleagues | Large VM images to transfer | Small container images |

## What Happens in This Workshop

You will use Containerlab through a VS Code extension. No complex commands to type. No deep technical knowledge required.

### What You'll Actually Do

1. **Open a topology file** - A simple text file describing your lab network (which PLCs, switches, HMIs you want)
2. **Click "Deploy"** in VS Code - The extension starts your entire lab
3. **Use your lab** - Connect to devices, run security tests, capture traffic
4. **Click "Destroy"** when done - Everything is removed cleanly

That's the complete workflow. Four steps. Simple.

### What's Happening Behind the Scenes (But You Don't Need to Worry About It)

When you click "Deploy", Containerlab:

- Reads your topology file
- Downloads or uses pre-built container images for your devices
- Starts each device as a container
- Connects them together with virtual network cables
- Gives you access to each device

When you're done and click "Destroy", everything is removed cleanly. No leftover files or configurations.

## Common Questions from OT Professionals

**Q: Is this secure enough for our lab?**  
Yes. Containers provide good isolation for training and testing. They're used by millions of developers and companies worldwide. For our workshop, they're more than adequate.

**Q: Do I need to learn Docker commands?**  
No. We're using the Containerlab VS Code extension, which gives you buttons to click instead of commands to type.

**Q: Can containers handle industrial protocols like Modbus, S7, or PROFINET?**  
Yes. Containers can run any software, including industrial protocol stacks and simulators. The network traffic works exactly as you'd expect.

**Q: What if something crashes?**  
Just redeploy that component. It's fast and gets you back to a clean state instantly.

**Q: Is this like the "cloud"?**  
No. Everything runs locally on your laptop. No internet required (after initial setup). You have full control.

**Q: Can I use this after the workshop for my own testing?**  
Absolutely. That's one of the goals - give you a tool you can use to build your own OT security and networking test labs.

**Q: Will this mess up my computer?**  
No. Containers are isolated. When you remove them, they're completely gone. Nothing is left behind on your system.

## Why Containerlab Specifically?

Containerlab was designed by network engineers for network engineers. It solves the problem of quickly testing network configurations and topologies.

**Why it's perfect for OT work:**

- Define your entire network topology in a simple text file
- Supports both network devices (switches, routers) and end devices (PLCs, HMIs)
- Deploy in seconds, tear down just as fast
- Works seamlessly with VS Code
- Share labs easily - just send the topology file to colleagues
- Free and open source

Network engineers at major vendors like Nokia, Cisco, and Juniper use Containerlab daily. It's battle-tested and reliable.

## Real-World Perspective: Old Way vs New Way

**Traditional approach:**  
Set up physical lab or configure multiple VMs. Takes hours or days. Breaks easily. Hard to share. Painful to reset.

**Container approach:**  
Write a text file describing your network. Click deploy. Lab ready in 30 seconds. Break something? Redeploy in 30 seconds. Share the file with colleagues who get the identical setup instantly.

**The difference:**  
You spend time learning OT security and networking instead of fighting with infrastructure.

## The Truth About Learning Curves

**You don't need to become a container expert.**

What you need to know:

- How to deploy a lab (click a button)
- How to connect to devices in the lab (same as always)
- How to destroy the lab when done (click another button)

That's it. The Containerlab VS Code extension handles everything else.

**After this workshop**, if you want to dive deeper, excellent resources exist. But for now, focus on OT security and networking. Container technology is just the tool that makes the labs work.

## What Makes Containers Work for OT Labs

Containers excel at simulating OT environments:

**Network Behavior is Realistic**  
Virtual network connections behave exactly like real Ethernet cables. Industrial protocols work identically. Network traffic looks the same to your monitoring tools.

**Performance is Sufficient**  
A modern laptop comfortably runs 10-20 containerized devices. That's enough for realistic OT network topologies with multiple zones.

**State Can Be Preserved**  
Keep your PLC configuration between sessions or start fresh each time. Your choice.

**Timing is Predictable**  
Not suitable for hard real-time control, but perfect for testing network protocols, security monitoring, and attack scenarios.

## Important Limitations (Being Honest)

Let's be clear about what this is and isn't:

**Not for production**  
These are simulations and test environments. Never put this in your actual plant network.

**Not real-time control**  
Perfect for testing protocols and security. Not suitable for safety-critical control systems that require hard real-time guarantees.

**Not identical to real devices**  
We use simulators that behave like your PLCs and HMIs, but they're not running your exact production firmware.

**Bottom line:**  
For training and testing - perfect.  
For production control - use real, certified hardware.

## Your Workshop Learning Path

**Before the workshop:**

- Don't stress about containers. They're just a tool.
- Ensure Docker Desktop and VS Code are installed (we'll guide you).
- Trust that millions of people use this daily.

**During the workshop:**

- Focus on OT security and networking, not container technology.
- Use the VS Code extension - point and click interface.
- Ask questions anytime something's unclear.

**After the workshop:**

- You'll confidently build your own test labs.
- Explore more advanced features if interested.
- Understand why containers are popular in OT security and networking testing.

## Want to Learn More? (Optional)

If you want to dive deeper after the workshop, these resources have helped millions of people understand containers. All are beginner-friendly:

### Highly Recommended Videos (Start Here)

**Docker in 100 Seconds** by Fireship  
[https://www.youtube.com/watch?v=Gjnup-PuquQ](https://www.youtube.com/watch?v=Gjnup-PuquQ)  
Quick, visual, and perfect for understanding the basics.

**you need to learn Docker RIGHT NOW** by NetworkChuck  
[https://www.youtube.com/watch?v=eGz9DS-aIeY](https://www.youtube.com/watch?v=eGz9DS-aIeY)  
Fun, practical, and beginner-friendly introduction.

### Official Docker Resources

**Docker Get Started Guide**  
[https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)  
Official Docker documentation, well-structured and clear.

**Docker 101 Interactive Tutorial**  
[https://www.docker.com/101-tutorial](https://www.docker.com/101-tutorial)  
Hands-on tutorial you can do in your browser or on your machine.

**What is a Container? (Docker's explanation)**  
[https://www.docker.com/resources/what-container/](https://www.docker.com/resources/what-container/)  
Docker's own beginner-friendly explanation.

### Containerlab Specific

**Containerlab Quick Start**  
[https://containerlab.dev/quickstart/](https://containerlab.dev/quickstart/)  
The official quick start guide - very accessible.

**Containerlab VS Code Extension**  
[https://containerlab.dev/manual/vsc-extension/](https://containerlab.dev/manual/vsc-extension/)  
Documentation for the extension we'll use in the workshop.

**Containerlab YouTube Channel**  
[https://www.youtube.com/@containerlab](https://www.youtube.com/@containerlab)  
Video tutorials and demonstrations.

### For Visual Learners

**Play with Docker**  
[https://labs.play-with-docker.com/](https://labs.play-with-docker.com/)  
Free online environment to try Docker commands without installing anything.

**Docker Curriculum by Prakhar Srivastav**  
[https://docker-curriculum.com/](https://docker-curriculum.com/)  
Beginner-friendly tutorial with clear explanations and examples.

### If You Want to Compare with VMs

**Containers vs Virtual Machines (IBM Technology)**  
[https://www.youtube.com/watch?v=cjXI-yxqGTI](https://www.youtube.com/watch?v=cjXI-yxqGTI)  
Clear video explanation of the differences.

## Bottom Line for This Workshop

You don't need to be a container expert. You just need to know that containers are mature, proven technology used by millions of developers and companies worldwide.

The VS Code extension does the heavy lifting. You focus on OT security and networking.

Containers are simply the tool that lets us build fast, repeatable, shareable OT security and networking labs. That's their purpose here. They do it well.

Ready to dive into OT security and networking? Let's go.
