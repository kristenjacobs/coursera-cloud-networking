# Cloud Networking Week 3

## 3.1.1 Software-Defined Networking Architecture - Part 1

The problem (that SDN tries to solve):

- Networks are complicated, and distributed, thus hard to manage.

- Network equipment is proprietary, and are each configured differently.

- => Hard to inovate and change/update networks (due to lots of devices).

The solution: **Software Defined Networking**

- Hardware switch/router provides an API into the data plane of the network.

- Have a logically centralised controller which communicates to each devices API.

- Policy/intelligence is held in the controller, thus can be updated in a single place 
  and pushed out to each device.

- SND controllers typically provide the ability to match packets, execute actions (rewrite, forward, etc).

- Also usually provide some sort of monitoring and topology discovery functionality.

## 3.1.2 Software-Defined Networking Architecture - Part 2

The evolution of SDN

  1. **MPLS (1997)** (or label switching). This scheme allows us to attach labels to packets,
  and base routing decisions on these labels. This allow other scheme than just shortest path,
  i.e. packets of a certain type will always be routed on certain paths (i.e. provides options
  for more traffic engineering).

  2. **Active Networks (1999)**. Allows the embedding of a program (or pointer to) 
  within a packet header! 

  3. **Routing Control Platform (2005)**

  4. **4D architecture (2005)**

  5. **Ethane (2007)**. Allow the definition of access control in a central location
  and have this pushed out to the network.

  6. **OpenFlow (2008)**. Provides a API and centralised view of the network
  (NOX was the first implementation of OpenFlow).

*Opportunities*

  1. Open data plane interface
    - HW: with standard API, easier to change HW.
    - SW: can more directly access device behaviour.

  2. Centralised controller
    - Control via software (algorithms)
    - Languages for controlling the high level policy of the network (DSLs?)

*Challenges*

- Performance and scalability.
- Resilience of the centralised controller.
- Need to reach a standard for the data plane protocol.

Killer Apps for SDN

  1. Cloud virtualisation
    - Can create a separate virtual network for each tenant.
    - Allow flexible placement and movement of VMs.

  2. Inter DC traffic engineering
    - Want 100% utilisation.

## 3.2.1 Multi-Tenant Data Centers: The Challenges

### Key needs

The ability to use any hardware for any VM at any time (i.e. **agility**). In
traditional DCs, each rack might be allocated to a particular service or
tenant. This leads to under utilisation, and lack of scalability when a
particular service out grows its allocated space.  Also, each rack might have
its own subnet, thus moving a service will involve changing the IP address of a
running service, which can be problematic. **We need to ability for the tenant
IP address to be portable.** Also, cross rack apps might not get full line rate
(in a tree based STP DC).  Therefore, we **want the same network performance
between any 2 VMs in the network**.  Finally, need **security to protect
services/tenants from each other** (i.e. **Micro Segmentation**). We also might
need to **support layer 2 services** (broadcast, etc), as some legacy
applications might be relying on this environment.

## 3.3.1 Network Virtualisation Case Study: VL2 - Part 1

- Increasing traffic within DC is a bottleneck (4x that of the external traffic)

- Unpredictable, rapidly changing traffic patterns.

=> Design Result: Require a non-blocking fabric (where we have line card rates
between all the servers, thus the netwrok fabric will not be the bottleneck.

VL2 choose a CLOS topology. They choose a routing algorithnm called 
**oblivious routing**, which means that the path a packet travels along
does not depend on the current traffic matrix (as we have rapidly changing 
unpredictable traffic patterns).

Used **valiant load balencing**. Aims to evenly distribute the flows across 
the network. Therefore each flow is load balanced evenly though one of the 
top level switches.

3 tier design. TOR switch -> Aggregate switch -> Intermediate switch.
The intermediate switches are all assigned the same anycast IP address, thus
the TOR switch can send to a random one using the single address, and using ECMP
will ensure that we use all possible paths to the intermediate switches.

The outer anycast address is wrapping an inner header which contains the 
actual destination IP, thus the intermediate switch will then forward it on to the 
dest TOR, again the ECMP causes all paths to be used evenly.

Note: This design has a similar effect to just using ECMP, but the 
anycast mechanism will cause the routing tables to be smaller at 
each switch which might be significant once we scale up.

## 3.3.1 Network Virtualisation Case Study: VL2 - Part 2

TODO

## 3.4.1 Network Virtualisation Case Study: NVP

TODO
