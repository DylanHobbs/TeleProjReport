#Telecoms 2 - Assignment 2

Group: PestoPasta
####An Implementation of Distance Vector and Link State Routing Protocols


#####Distance Vector

Both of these routing implementations operate in a distributed network setting. That being, nodes only know the costs to their neighbours and can only talk to these neighbours.
Distance Vector itself can be thought of as a distributed version of the Bellman-Ford algorithm. It was used in early days of the internet (even back when it was the ARPANET) and can be seen in the protocol RIP. 

In the very basic sense Distance Vector Routing can be broken down to 3 steps

1. Init Routing Table vector with a cost of 0 to itself and infinity (or some representation of infinity) to all other destinations.

2. Periodically send your Table to neighbours.

3. Update your own Table for each destination by selecting the shortest distance received (taking into account cost to neighbour link).

In practice, the limitations imposed by virtualisation made staying true to this very difficult. At router initialisation any given router is made aware of all it’s neighbours and the cost to these routers / clients immediately. This is defined by a formatted text file that is passed into the program that we will go into more detail in below. The limitation that virtualisation has imposed on the project means that we start at step 2, with the first round of transfers already complete.