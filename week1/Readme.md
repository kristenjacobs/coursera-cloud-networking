# Cloud Networking Week 1

## 1.1.1 Applications and Traffic Patterns

- Typical traffic pattern in data centres follow the scatter-gather pattern,
  i.e. web-search.

- However, data centre traffic depends entirely on the application type.
  One thing is for sure though, is that traffic volume is growing.

- 10x more traffic within DC than between the DC and the internet.

- Also, the proportion of traffic within the DC is growing.

- More and more traffic is cross rack but still within the cluster/DC.

## 1.2.1 Physical Structure

- Servers in racks.

- Each rack has a top of rack (TOR) switch, which supports full line rate 
  between each of its ports, i.e. all ports can talk to all other ports without
  blocking due to switch capacity (i.e. the TOR switch is not the bottleneck).

- Ideal network between the TOR switches would be a full crossbar, i.e. all of the 
  ports are connected at full line rate. 

- This is possible for small numbers of servers. However, becomes impractical
  as the DC grows.

### Tree Network Architecture 

- Could organise the network as a (redundant) tree instead. 

- Traditional DC's use a 3-tier tree based (big switch) approach.
  This has core routers at the top, aggregation (or distribution) routers
  in the middle tier, and access (TOR) switches on the 3rd tier.

- Simple architecture, can use a spanning tree for routing which ensures that 
  there are no loops. There can still be redundancy. However, only one primary route
  is used at any one time, the redundant routes only being used in the case of failure.

- Suffers from congestion at the top of the network.

- However, the big switch approach has the problem that the bandwidth within the 
  rack is much greater than the bandwidth across racks. Therefore, it makes sense
  to optimise for local traffic. This however, means the problem of placement 
  becomes an issue.
  
- Big switches are expensive.
  
- Only scales vertically, i.e. you have to buy a bigger switch, and doesn't scale well horizontally, 
  i.e. there are maximum sizes to switches. Also, the latest/fastest switch models might not be the biggest,
  thus upgrading your network hardware becomes slow/expensive.

### CLOS (Spine/Leaf) Network Architecture

- Modern approach is a CLOS topology. This uses smaller switches in layers to 
  simulate the operation of a big single switch.

- This ensures that all devices are the same distance from each other, thus the 
  latency between 2 devices is fixed and predictable.

- In fact, internally big switches use CLOS networks between the cards inside the switch, but this
  is hidden from the user. The CLOS approach is then a bit like exploding the big switch, and creating 
  one distributed big switch from lots of little commodity switches.

- Cables are bundled together to aid physical routing. 

- Long cables have to be optical.

- Other network topologies are possible, and can be application specific.
