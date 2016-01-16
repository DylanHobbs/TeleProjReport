#Telecoms 2 - Assignment 2

Group: PestoPasta
####An Implementation of Distance Vector and Link State Routing Protocols


#####Distance Vector

Both of these routing implementations operate in a distributed network setting. That being, nodes only know the costs to their neighbours and can only talk to these neighbours.
Distance Vector itself can be thought of as a distributed version of the Bellman-Ford algorithm. It was used in early days of the Internet (even back when it was the ARPANET) and can be seen in the protocol RIP. 

In the very basic sense Distance Vector Routing can be broken down to 3 steps

1. Init Routing Table vector with a cost of 0 to itself and infinity (or some representation of infinity) to all other destinations.

2. Periodically send your Table to neighbours.

3. Update your own Table for each destination by selecting the shortest distance received (taking into account cost to neighbour link).

In practice, the limitations imposed by virtualisation made staying true to this very difficult. At router initialisation any given router is made aware of all its neighbours and the cost to these routers / clients immediately. This is defined by a formatted text file that is passed into the program that we will go into more detail in below. The limitation that virtualisation has imposed on the project means that we start at step 2, with the first round of transfers already complete.



#####Link State

As stated before this routing implementation also works in a distributed setting. This is widely used in practice as it was more fault tolerant than distance vector. On the downside it is a good bit more computationally intensive when compared to Distance Vector. Where in Distance Vector we spread the work of computing routes out among all the routers in a network, in Link State we give everyone a copy of the topology and let everyone compute their own routes. 
It has been in use in the Internet/ARPANET since 1979 and a number of modern routing protocols are based off Link State such as OSPF and IS-IS, which are widely used.

In a similar fashion to Distance Vector we can break down the Link State routing approach into a couple steps.

1. Nodes flood the network with their own Link State Packets and through this each node learns the full topology.

2. Every node then computes their own full Routing Table or Forwarding Table by running Dijkstra's algorithm (or similar).

We were hampered by the virtualisation process here also, as we used the same base network setting for both of our implementations. However this does mean we got around the problem in the same way as distance vector. The network does flood, but it begins the flood already knowing who their neighbours are and the cost to these neighbours.  The details of our implementation of flooding and Dijkstra's algorithm will be explored below.


###General Network Setting Implementation 


###Distance Vector Implementation


###Link State Implementation 
