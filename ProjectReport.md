#Telecoms 2 - Assignment 2
####An Implementation of Distance Vector and Link State Routing Protocols	
Group: PestoPasta
##How to Start the Program Examples
####Distance Vector
(Defaults to topology 1)

Clients are started in a new terminal window to receive proper prompts. It will take 2 - 3 update cycles for messages to send
correctly.
######Topology 1
```
	java Server

	(Each client should start in a new Terminal window)
	java Client 4000 5004
	java Client 4001 5002
	java Client 4002 5001

```

######Topology 2:
```
	java Server

	(Each client should start in a new Terminal window)
	java Client 4000 5000
	java Client 4001 5002

```



####Link State
(Defaults to topology 2)
Clients are started in a new terminal window and *started first*
######Topology 2:
```
	(Each client should start in a new Terminal window)
	java Client 4000 5000
	java Client 4001 5002

	java SuperServer

```



######Topology 1:
```
	(Each client should start in a new Terminal window)
	java Client 4000 5004
	java Client 4001 5002
	java Client 4002 5001
	
	java SuperServer

```



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

2. Every node then computes their own full Routing Table or Forwarding Table by running Dijkstra's shortest path algorithm (or similar).

We were hampered by the virtualisation process here also, as we used the same base network setting for both of our implementations. However this does mean we got around the problem in the same way as distance vector. The network does flood, but it begins the flood already knowing who their neighbours are and the cost to these neighbours.  The details of our implementation of flooding and Dijkstra's algorithm will be explored below.


##General Network Setting Implementation (What is the same between both approaches)
###Virtualise all the things!

To implement both routing algorithms we used a virtuallised approach. This allowed us to focus on actually implementing valid forms of distance vector and link state protocols. To virtualise the network we used predefined topologies that are read from a text file in the same directory as the Java binary. All packets are sent to localhost on different ports. Routers and client have a list of user defined ports that they are allowed to talk to.

We used a Java class called SuperServer to read in the data from the text file containing the network setup. It will then setup each router on it's own thread and pass it the list of it's connections. Each router uses two threads, one to receive packets and one process tables and forward messages. We needed to use two threads to ensure that if a message is received when the router is busy it will not be dropped.

###Packet Headers

Both of our implementations make use of a custom routing protocol and so we had to build our own packet header. Both protocols use the same header for messages sent from clients.

#####Client Header

Client headers contain only the absolute necessary data required to keep the size small. Currently the header is only 10 bytes in size which works out quite well considering the way that Java converts strings to byte arrays and the maximum safe size of a UDP packet. Despite the small size the header is still reasonably expendable in functionality.

Byte 0 is used to mark the byte type. This leaves 256 values that can be used for different type (a Java byte is -128 to 127 including zero).

Bytes 1-4 are used for the final destination port of the message. The standard UDP header contains the port of the router/client that the message is immediately being send to. Currently these bytes are used as an integer as we use ports to communicate for virtualization. It could easily be reused as an IP address where each byte stores one octet of the address.

Byte 5 is used to store the message length. Because of the way the datagram packet interprets packet size we had to add our own size field to the header.

Bytes 6-9 are used to store the origin port of the message. This allows clients to know where the message originated from. As with the destination this could easily be changed to an IP address.

#####Routing Table Header

Routing tables only have a full header in the Link State protocol. They were required as routers rebroadcast the tables of other routers to each other and they need to know who originally created the table.

Byte 0 is used to mark the byte type. This tells the router whether it should treat the packet as a table to be processed or a message to be forwarded.

Byte 1-4 are used for the table length. This is again because of the way Java's DatagramPacket's size function misinterprets the size of our packets. It treats the array size as the message size even if array elements are null.

Byte 5-8 are used to store the original table owner (currently for ports could be changed to an ip address).

Byte 9 is reserved for future use. This was done to keep the header length consistent.

###Routing Tables

Both protocols use the same routing table format. Although the Link State version has two extra methods, clear table and equals. Otherwise the two classes are identical. Routing tables store the routes that each router has available. It has many useful internal functions such as add, update and getNextHop. This allows them to be used and edited easily with minimal impact to the main application.

All of its variables are private and can only be accessed via the available methods. This means that we could later change the internal structure of the class without affecting any other code. Currently destination, nextHop and cost are stored in separate array lists. This allows them to be expanded easily. 

Routing tables can be constructed from either an integer and two arrays (owner, connections, initialCost) or from a byte array received in a packet. Routing tables can be converted to a byte array for sending.

###Client

Our client is a simple but efficient. It has a text only userface. The user is asked to type a message and then asked for a destination port. The client will continue to ask the user until they give a valid port (above 1000 below 50000). The client in link state is slightly different as it will reply to a flood message confirming that it is in fact online by reply with a table of size one, containing only the router it's connected to.


###Distance Vector Implementation
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
If not, the server will send a packet back to the Client letting them know there was no route to host found, it will also print to it's own console.




###Link State

Our link state implementation has two main stages, the flooding stage and the processing stage. In our virtualised environment these both only run once. In the real world they would run every time a router is added or removed.

######Stage 1 - Flooding

After all the servers are initialised on their independent threads they will send a copy of their routing table to all of their connected neighbours, hence flooding. They will also send this to any clients they are connected to.

When a router receives a table it checks if it has received it before. If the table is new to it it will add it to an ArrayList of RoutingTables and then again 'flood' this new table to all of it's connected neighbours. If it already had received the table it won't store or send it to anyone.

Because tables are only flooded once; either when it is received, or when the router is first started, the flooding stage will end relatively quickly. In this regard it's better than distance vector as it wastes less time sending packets.

######Stage 2.0 - Processing

When a router has received all the RoutingTables of all the other routers in the network it will enter the processing stage. In our virtuallised environment this happens a predefined time for ease of use. In the real world this would be run after a new table is added to the ArrayList of received tables.

We use a timer task to schedule this to happen two seconds after the routers are first started.

The timer task calls the createGraph function. This function creates a graph of the entire network from the ArrayList of received RoutingTables. First of all it creates a vertex for each router with an ID which is the router's port. The vertex itself has a cost(initialised to the maximum integer value), a visited boolean, an ArrayList of edges and a pointer to the previous vertex. These are all used by Dijkstra's algorithm later.

After creating all the vertexes the createGraph function will iterate through each vertex and find it's matching routing table in the ArrayList of received tables. When it finds the corresponding table for a vertex it creates an edge for each connection the router has (it's immediate neighbors). When the edge is being created createGraph loops through all the vertexes to find the destination of the edge and add it (instead of just the port number), it's cost is also added from the routing table.

After all the edges have been added to the vertexes a graph is created. Dijkstra's algorithm is now run on the graph.

######Stage 2.1 - Dijkstra's algorithm

Dijkstra's algorithm is used in our server to find the shortest route to every node(client or server) in the network. It is slightly altered to suit our needs.

When algo is first called it finds it's own vertex in the graph. It sets it's cost to 0 and marks it as visited. The currentVertex is then set to home.

The graph loops through the following code while there are still unvisited vertexes 

*start of loop*

* It gets all edges of the current vertex.

* Then iterates through each all of the edges and calculates a temporary cost to that vertex from this vertex. If this cost is lower the the current cost of the destination vertex it replaces it. The destination vertex's previous field is also set to this vertex.

* The current vertex is then marked as visited.

* The unvisited vertex with the lowest cost is then selected from the graph and currentVertex is set to it.

*end of loop*

The router then creates another ArrayList of it's immediately connected vertexes (it's neighbors).

Afterward the RoutingTable is cleared for new data.

The router then iterates through each vertex in the graph(except itself). If current vertex in the loop is in the ArrayList of immediately connected vertexes it adds it to the RoutingTable with its associate cost and itself as the next hop and destination.

If it is not an immediately connected vertex it will check the previous value of the vertex, if that is also not in the ArrayList of immediately connected vertexes it will again check previous value of the vertex. 

Eventually it will get to a neighbor vertex that is directly connected to the router. It will then add the destination vertex as the destination and the neighbor vertex as the next hop. The cost is set to the cost to the neighbor vertex.


###What We Learned
#####Communication 
This was a group project and because of this we decided that communication would be very important throughout the project. We thought that Git would be the the best way to organise the code and eliminate conflicts or duplication of work. We chose GitHub as our Git vendor of choice and all of the projects code was committed to there on a regular basis.
This was our first time using Git in a formal fashion and had the opportunity to learn how branches can be used effectively to work on different sections of the code. Our Link State approach was branched from the Distance Vector code at the stage where the General Network Setting was just about in place. We then used other branches to test code that would otherwise stop the other team members from working, merging them back to master when all major bugs were ironed out.

In the end we decided to create different Repositories for Distance Vector and Link State as there began to be very large differences between the 2 sets of source files.
The repositories are currently private but we are attaching a temporary login to a guest account that has read access to both of the repos.

We also found Slack to be a very important tool for communication. It was alert us to commits made to the Git Repository and allowed us to chat about changes or ideas we had. When text communication wasn't enough we used a Mumble server as a VOIP alternative. Overall we felt the communication channels we established were effective and helped us to create a finished project in a coherent fashion.

#####Academic
By doing this project we learned a lot about networking, both it's support in Java and its history. Much research was done before we could even start the project. We had to learn exactly what distance vector and link state routing protocols are. Wikipedia was an invaluable resource for leaning about things such as Dijkstra's algorithm as it contains visualisations of the algorithm at work.


###Closing
Overall we found the project to be a lot of fun. It was an interesting project and we decided to model our workflow to be as close to production quality as we could within our skill-level. All of the methods are documents in the source and the commits over time graphs on Github show a persistent effort toward the project overall.


Dylan Hobbs

Owen Mooney

