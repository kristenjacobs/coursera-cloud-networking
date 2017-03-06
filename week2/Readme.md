# Cloud Networking Week 2

## 2.1.1 Host Virtualisation

Required to provide greater utilisation.

Hypervisor manages the physical NIC. Each VM has a virtual NIC which talks to
the hypervisor. The hypervisor contains a virtual switch.

The CPU has to do the work to forward packets from the NIC to each VM, i.e.
simulate the switch in software => Flexible but expensive in terms of CPU
workload.

2 approaches to the above problem:

  1. SR-IOV: Single Root I/O Virtualisation.
  In this approach, the physical NIC supports the VM NIC virtualisation in hardware.
  It supports a number of packet queues, each one of which can be mapped to
  a VM. There is a L2 switch in the NIC hardware that places the packets on the queues, and
  the queue state is DMA'ed directly into the VM memory, thus the hypervisor doesn't have
  to do anything (other than to initially assign the queues to VMS). This approach is quick,
  but not too flexible (i.e. forwarding rules cant be changed), and live VM migration
  is tricky due to the fact that the state of the packet queues for a given VM is held in the
  physical NIC on the host.

  2. Open VSwitch.
  Goals: Flexible and fast forwarding.
  Forwarding logic is done in user space in the first instance. However,
  the forwarding rule is then installed in the kernel, thus making it quicker for
  any subsequent packets that match (i.e. in the same flow).
  To gain speed, a hash table in the kernel is used to index into the forwarding table,
  thus we get a constant time lookup for each forwarding route.

## 2.2.1 Routing and Traffic Engineering

- 2 general classes of routing protocols:

  1. **Link state routing**
  Each node builds a graph of the entire network, and uses this to populate its routing table.
  Nodes tell their nearest neighbours about thier own connections.

  2. **distance vector routing**. Each node shares its routing table with its nearest
  neighbours, i.e. passes a vector of hops to a distination IP range.
  These are then used by each switch to populte it's routing table out of
  the most suitable option (i.e. shortest paths (no. of hops)) for each destination IP.

- Traditional DC networks use a **Spanning tree protocol** for layer 2 network routing.
  The main benefit is that it achieves loop free routing. It will pick a subset of the network links, which forms
  a tree, and use these to route packets, thus there will be no loops, and there is only a single path, i.e.
  no choices have to be made. The disadvantage of this approach is that some links are not used
  thus the network is under utilised in general. However, the other links can be used in case of failure.

  - Works well for small enterprise DCs. However, for large DCs which have lots
  of east/west traffic at the leaves, this puts lots of load on the core switches.
  - Not a good fit for CLOS networks though, as the utilisation goes down.
  - The network is open to partitions (if a switch goes down)

- **TRILL: Transparent Interconnection of Lots of Links**
  - Provides an alternative to STP.
  - Works at layer 2.
  - Is a link state protocol, i.e. all the swicthes in the network
  learn about the entire network topology, and use this to wrap packets with
  thier next hop destination switch.
  - Utilises all paths in the network.
  - Requires special hardware.

- **OSPF: Open shortest path first**
  - Provides an alternative to STP.
  - Works at layer 3.
  - Is a link state protocol, i.e. all the swicthes in the network
  learn about the entire network topology, and use this to wrap packets with
  thier next hop destination switch.
  - Utilises all paths in the network.

- **BGP: Border Gateway Protocol**
  - Provides an alternative to STP.
  - Works at layer 3.
  - Is a path vector protocol. Each router passes info on the
  path (list of routers) from a destination IP to itself to each of its neighbours.
  (and as its a path we can tell that there is no loops in there!)
  Each router can then choose the next hop for a IP packet based on the path options,
  i.e. each router will discard longer paths to dest IPs if it already has knowledge
  of a shorter path in its routing table.

## 2.2.2 Routing and Traffic Engineering: Packet Forwarding on Multiple Paths

- Goal: Minimise network congestion.

- *ECMP: Equal cost multi-path*
  If there are multiple paths to the destination in the routing table of
  a switch, each with the same cost (no. of hops), then ECMP basically chooses
  one at random.

  - ECMP works well for both layer 2 and layer 3 routing.

  - However, issue with packets in a single flow travelling along different paths,
  i.e. they can arrive out of order (thus TCP can consider this a loss if the time
  difference was to large and the TCP buffer has become full). In this case, the
  packet need to be retransmitted, thus decreasing throughput.

  - Solution: Use flow level hashing, i.e. use the hash of the packet header
  to decide which path to take. Howver, for long lasting dataflows, this can cause
  congestion on some links.

  - Solution 2: Use **flowlets**. Break the flow into chunks, and if the gap between
] chunks is bigger than the largest latency difference between the 2 paths, then the
  flowlets can travel different paths and all is still ok (i.e. no reordering).

  - ECMP only makes local choices, thus failures later in the path (i.e. downed switches)
  are not known about so the wrong path is chosen (even though a valid path exists).

  - Solution: CONGA. Here we keep track of congestion for each path between a switch and
  each of its potential destination, thus we can choose the least congested path.
  Done on a flowlet granularity.

## 2.3.1 Congestion Control: Part 1

Problem: Multiple flows need to share network links. We need to choose the sending rate
such that it is not so high as packets are lost (thus need retransmission), but not too low
such that the netwrok is being underutilised.

- TCP acks give us some indication of congestion, i.e. if we dont get an ack back
  in the expected round-trip-time, then we can consider the link to be congested.
  The sender can then slow the rate of packet sending.
  Likewise, if all of the packets are being ack'ed, then we can increase the sending rate.
  This is the way TCP works. It rapidly increases (doubling) until we hit a packet loss,
  then we go into a additive increase phase. If too much loss occurs, then we backoff and
  halve the transmission rate. If we miss enough, then TCP will timeout the connection
  and start from the beginning again.

## 2.3.2 Congestion Control: Part 2

Problems with the TCP congestion control scheme:

- Multiplicative decrease (halving) is quite drastic.

- A packet can get across a data center on a fibre cable in roughly 1us.
  However, going though the network devices, it'll take roughly 10us.
  TCP queuing delay drastically lengthens this.

- **TCP Incast** Is a pathological condition that occurs when queues get too big.
  Can occur with burst traffic (i.e. scatter-gather).

## 2.3.3 Congestion Control: Part 3

Solution to the above TCP problems is DCTCP (data center TCP).
Here we don't wait until the buffer is full before decreasing the rate,
because at that point, it is too late as packet loss has already occurred.

**Explicit Congestion Notification ECN**. When a packet arrives at a switch,
the switch looks at the average queue length over time window and if this is
above a certain threshold, then it will set a congestion bit in the packet. If
it is below the low threshold, it is not marked. In-between, it is marked
probabilistically.  The bit gets ciopied back in the ack, thus the sender can
then react in the same way as TCP does but without the loss.

With DCTCP, we don't use a window, i.e. it uses an instantaneous queue length.
Also, DCTCP doesn't just use a single bit, and senders look at the frequency of
these bits over a time window and scales the transmission rate proportionally.

DCTCP doesn't require any specific hardware, as the ECN bits are already
supported in switches.
