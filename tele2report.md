#Telecoms 2 - Assignment 2

Group: PestoPasta
####An Implementation of Distance Vector and Link State Routing Protocols


###Distance Vector

Both of these routing implementations operate in a distributed network setting. That being, nodes only know the costs to their neighbours and can only talk to these neighbours.
Distance Vector itself can be thought of as a distributed version of the Bellman-Ford algorithm. It was used in early days of the Internet (even back when it was the ARPANET) and can be seen in the protocol RIP. 

In the very basic sense Distance Vector Routing can be broken down to 3 steps

1. Init Routing Table vector with a cost of 0 to itself and infinity (or some representation of infinity) to all other destinations.

2. Periodically send your Table to neighbours.

3. Update your own Table for each destination by selecting the shortest distance received (taking into account cost to neighbour link).

In practice, the limitations imposed by virtualisation made staying true to this very difficult. At router initialisation any given router is made aware of all its neighbours and the cost to these routers / clients immediately. This is defined by a formatted text file that is passed into the program that we will go into more detail in below. The limitation that virtualisation has imposed on the project means that we start at step 2, with the first round of transfers already complete.



###Link State

As stated before this routing implementation also works in a distributed setting. This is widely used in practice as it was more fault tolerant than distance vector. On the downside it is a good bit more computationally intensive when compared to Distance Vector. Where in Distance Vector we spread the work of computing routes out among all the routers in a network, in Link State we give everyone a copy of the topology and let everyone compute their own routes. 
It has been in use in the Internet/ARPANET since 1979 and a number of modern routing protocols are based off Link State such as OSPF and IS-IS, which are widely used.

In a similar fashion to Distance Vector we can break down the Link State routing approach into a couple steps.

1. Nodes flood the network with their own Link State Packets and through this each node learns the full topology.

2. Every node then computes their own full Routing Table or Forwarding Table by running Dijkstra's algorithm (or similar).

We were hampered by the virtualisation process here also, as we used the same base network setting for both of our implementations. However this does mean we got around the problem in the same way as distance vector. The network does flood, but it begins the flood already knowing who their neighbours are and the cost to these neighbours.  The details of our implementation of flooding and Dijkstra's algorithm will be explored below.


###General Network Setting Implementation (What is the same between both approaches)
	- How routers are setup. eg Config text file.
	- Limitations imposed by virtualisation and getting around it.
	- Threaded Servers and Clients
	- No GUI, minimal interface. Terminal clients and servers.
	- Packet headers for client. (Different to table headers)



###Distance Vector Implementation (What makes DV different)
	-TODO Add a section at the end of every phase evaluating why you chose this
Our implementation of the Distance Vector approach sits on top of general network setting. 
######Phase 1 - Initialisation
Once a topology is loaded, the main method will create and start an array of servers to the specifications described in the .txt file. During this phase each server is made aware of it's neighbours and the cost to these connections. These then get added to the Server's Routing Table. Starting the server threads while using Distance Vector will begin a timer for each server that executes the update code at a given time interval. This timer is controlled by a constant and is set to 10 seconds for the demonstration.

######Phase 2 - updateTimer()
Every time the timer ticks over, it creates a new timer task. This timer task has the Servers current Routing Table, and a list of it's neighbours.
We chose to create a new TimerTask every interval as this updates the Routing Table that needs to be sent. The run() method in this Timer Task will distribute the tables current routing table to everyone it's connected to.

######Phase 3 - onReciept()
ServerNode's listener will receive packets for a given Server. These packets may contain messages or could be an Update Packet. Firstly the packet is stripped to get the actual data. This was a problem we ran into during development as DatagramPacket's getData() method would return the full data buffer, which usually contained a lot of empty data. Stripping the data to it's actual size eliminates us running through an empty section at the end of the data when it is being examined.
If this data is a table we work to process the given table. (See 4.1)
If the data is a message we work to forward the message to the correct destination. (See 4.2)

######Phase 4.1 - processTable()
This function is the heart of the Distance Vector Approach. It has access to the Servers own Routing Table and the one passed in. We go through each of the entries in the new table.
If we come across a destination we don't have, we add it to our table signifying the nextHop field to be the neighbour we received the Routing Table from. The cost for this becomes the neighbours cost, plus the cost to our neighbours.

If we have this destination we check which one would be the shorter path. This is the one that will be in our Routing Table.

Once the table is fully processed this function and the onReceipt() function end.

######Phase 4.2 - forwarding Packets (onReciept)
This is the communication handling section of the Protocol. First it will strip information needed from the header. Using this information we look up the current Routing Table of the server for a destination to send it to. If this is found we forward the packet and print to console. 
If not, the packet is discarded and an error is printed to the console.



###Link State Implementation (What makes LS different)
	- Flooding description
	- Dijkstra description



###What We Learned
#####Communication 
	- TODO Add in Login for Github guest account
This was a group project and because of this we decided that communication would be very important throughout the project. We thought that Git would be the the best way to organise the code and eliminate conflicts or duplication of work. We chose GitHub as our Git vendor of choice and all of the projects code was committed to there on a regular basis.
This was our first time using Git in a formal fashion and had the opportunity to learn how branches can be used effectively to work on different sections of the code. Our Link State approach was branched from the Distance Vector code at the stage where the General Network Setting was just about in place. We then used other branches to test code that would otherwise stop the other team members from working, merging them back to master when all major bugs were ironed out.

In the end we decided to create different Repositories for Distance Vector and Link State as there began to be very large differences between the 2 sets of source files.
The repositories are currently private but we are attaching a temporary login to a guest account that has read access to both of the repos.

We also found Slack to be a very important tool for communication. It was alert us to commits made to the Git Repository and allowed us to chat about changes or ideas we had. When text communication wasn't enough we used a Mumble server as a VOIP alternative. Overall we felt the communication channels we established were effective and helped us to create a finished project in a coherent fashion.


###Closing (temp)
Overall we found the project to be a lot of fun. It was an interesting project and we decided to model our workflow to be as close to production quality as we could within our skill-level. All of the methods are documents in the source and the commits over time grpahs on Github show a persistent effort toward the project overall.

Dylan Hobbs