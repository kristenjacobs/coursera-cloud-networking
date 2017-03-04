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
then set up a tunnel to the destination router based on the current traffic and the
shortest pass info. The tunnel uses labels, and only the ingress and egress
routers actually read the actual packet info. MPLS works over multiple protocols.

**Problem 1**: This can be inefficient, as these networks are provisioned for peak
bandwidth. Therefore you are better off provisioning for the peak latency 
sensitive traffic, an fill the gaps with the background traffic (i.e. backups).
However, this is had to do with MPLS, as it doesn't work at the application level.

**Problem 2**: Inflexible sharing. Only link level fairness is obtained. At the 
flow level, it can be unfair.

# 4.1.2 Inter-Data Center Networking: Cutting-edge Solutions
