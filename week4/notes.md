## Cloud Networking Week 4

# 4.1.1 Inter-Data Center Networking: The Problem

Cloud providers have multiple data centers spread over 
different regions for the following reasons:

- data avalibility
- load balancing
- latency
- local data laws
- hybrid public/private cloud

Therefore, inter data-center traffic/bandwidth is important
as there is now more traffic between DCs than between DCs and clients.

The goal is inter DC networks is to provide persistent (dedicated) connectivity
bewtten the end-points (~100s of Gbps)

These WANs also have higher network design flexibility as they are typically
operated/controlled by a single provider.

However, WAN bandwidth is a very expensive resource!

Traditional WAN approach is to use MPLS. Use a link state protocol (OSPF) to
flood the topology to all nodes (DCs) in the WAN. Also pass the current
bandwidth usage of links in the network to each node. Each ingress router can
then set up a tunnel to the destination router based on the current traffic and
the shortest pass info. The tunnel uses labels, and only the ingress and egress
routers actually read the actual packet info. MPLS works over multiple
protocols.

**Problem 1**: This can be inefficient, as these networks are provisioned for peak
bandwidth. Therefore you are better off provisioning for the peak latency 
sensitive traffic, an fill the gaps with the background traffic (i.e. backups).
However, this is had to do with MPLS, as it doesn't work at the application level.

**Problem 2**: Inflexible sharing. Only link level fairness is obtained. At the 
flow level, it can be unfair.

# 4.1.2 Inter-Data Center Networking: Cutting-edge Solutions

Common themes to solving the above problems:

1. Leverage service diversity, i.e. latency sensitive (web search), or backup.
   Note: This can only be done on propriatry networks, i.e. google/aws, as normal 
   ISPs dont have access to the endpoints so dont know the service types of each flow.
2. Centralize the traffic engineering using a SDN approach.
3. Traditionally we would use linear programming to solve this kind of constraint based
   problem. However, this could be too slow to do in real time.
4. Dynamically change how traffic is routed based on the current utilisation
   (maybe a few hundred times a day?)
5. Edge rate limiting.

**Case study 1: Googles B4**
 
- On the edge of each Google DC it has cluster border routers.

- Traditionally we would then connect the border routers to WAN routers that
  would then use iBGP/IS-IS. However, we want more control thus wont leave the 
  routing to the hardware.

- Uses a system called *Quagga* which runs on the boarder routers, decides the 
  routing then updates an openflow controller who then updates all the WAN routers
  with the new forwarding rules. A traffic engineering server can then make decisions
  for the whole WAN.

- As a fallback option, they keep the BGP forwarding state (tables) available i
  thus they can at any time fall back to using standard BGP routing.

- Google also made their own WAN SDN switches. These are simple, with smaller forwarding 
  tables (only 12 DCs are connected), and don't require large buffers as they rely on
  edge based rate limiting. Each switch has an embedded Linux chip which is what the 
  openflow controller talks to.

**Case study 2: Microsofts SWAN**

- Similar to above in many ways. Also aims to minimise the link sharing of different
  flows even during the changing of routes for existing flows.




    





