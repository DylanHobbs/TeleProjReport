##Impletmentation of Distance Vector Routing


###Virtualise all the things!

To implement both routing algorithms we used a virtualised approach. This allowed us to focus on actually implementing valid forms of distance vector and link state protocols. To virtualise the network we used predefined topologies that are read from a text file in the same directory as the java binary. All packets are sent to localhost on different ports. Routers and client have a list of user defined ports that they are allowed to talk to.

We used a java class called SuperServer to read in the data from the text file containing the network setup. It will then setup each router on it's own thread and pass it the list of it's connections. Each router uses two threads, one to receive packets and one process tables and forward messages. We needed to use two threads to ensure that if a message is received when the router is busy it will not be dropped.

###Packet Headers

Both of our implementations make use of a custom routing protocol and so we had to build our own packet header. Both protocols use the same header for messages sent from clients.

#####Client Header

Client headers contain only the absolute necessary data required to keep the size small. Currently the header is only 10 bytes in size which works out quite well considering the way that java converts strings to byte arrays and the maximum safe size of a UDP packet. Despite the small size the header is still resonably extendable in functionality.

Byte 0 is used to mark the byte type. This leaves 256 values that can be used for different type (a java byte is -128 to 127 including zero).

Bytes 1-4 are used for the final destination port of the message. The standard UDP header contains the port of the router/client that the message is immediately being send to. Currently these bytes are used as an integer as we use ports to communicate for virtualization. It could easly be reused as an ip address where each byte stores one octet of the address.

Byte 5 is used to store the message length. Because of the way the datagram packet interprets packet size we had to add our own size field to the header.

Bytes 6-9 are used to store the origin port of the message. This allows clients to know where the message originated from. As with the destination this could easily be changed to an ip address.

#####Routing Table Header

Routing tables only have a full header in the Link State protocol. They were required as routers rebroadcast the tables of other routers to each other and they need to know who originally created the table.

Byte 0 is used to mark the byte type. This tells the router whether it should treat the packet as a table to be processed or a message to be forwarded.

Byte 1-4 are used for the table length. This is again because of the way java's DatagramPacket's size function misinterprets the size of our packets. It treats the array size as the message size even if array elements are null.

Byte 5-8 are used to store the original table owner (currently for ports could be changed to an ip address).

Byte 9 is reserved for future use. This was done to keep the header length consistent.

###Routing Tables

Both protocols use the same routing table format. Although the Link State version has two extra methods, clear table and equals. Otherwise the two classes are identical. Routing tables store the routes that each router has available. It has many useful internal functions such as add, update and getNextHop. This allows them to be used and edited easily with minimal impact to the main application.

All of its variables are private and can only be accessed via the available methods. This means that we could later change the internal structure of the class without affecting any other code. Currently destination, nextHop and cost are stored in seperate array lists. This allows them to be expanded easily. 

Routing tables can be constructed from either an integer and two arrays (owner, connctions, initialCost) or from a byte array received in a packet. Routing tables can be converted to a byte array for sending.

###Link State

Our link state implmentation has two main stages, the flooding stage and the processing stage. In our virtualised envirnment these both only run once. In the real world they would run every time a router is added or removed.

####Stage 1 - Flooding

After all the servers are initialised on their independant threads they will send a copy of their routing table to all of their connected neighbours, hence flooding. They will also send this to any clients they are connected to.

When a router receives a table it checks if it has received it before. If the table is new to it it will add it to an ArrayList of RoutingTables and then again 'flood' this new table to all of it's connected neighbours. If it already had received the table it won't store or send it to anyone.

Because tables are only flooded once; either when it is received, or when the router is first started, the flooding stage will end relatively quickly. In this regard it's better than distance vector as it wastes less time sending packets.

####Stage 2.0 - Processing

When a router has received all the RoutingTables of all the other routers in the network it will enter the processing stage. In our virtuallised environment this happens a predefined time for ease of use. In the real world this would be run after a new table is added to the ArrayList of received tables.

We use a timer task to schedule this to happen two seconds after the routers are first started.

The timer task calls the createGraph function. This function creates a graph of the entire network from the ArrayList of received RoutingTables. First of all it creates a vertex for each router with an ID which is the router's port. The vertex itself has a cost(initialised to the maximum integer value), a visited boolean, an ArrayList of edges and a pointer to the previous vertex. These are all used by dijkstra's algorithm later.

After creating all the vertexes the createGraph function will iterate through each vertex and find it's matching routing table in the ArrayList of received tables. When it finds the corrasponding table for a vertex it creates an edge for each connection the router has (it's immediate neighbors). When the edge is being created createGraph loops through all the vertexes to find the destination of the edge and add it (instead of just the port number), it's cost is also added from the routing table.

After all the edges have been added to the vertexes a graph is created. Dijkstra's algorithm is now run on the graph.

####Stage 2.1 - Dijkstra's algorithm

Dijkstra's algorithm is used in our server to find the shortest route to every node(client or server) in the network. It is slightly altered to suit our needs.

When algo is first called it finds it's own vertex in the graph. It sets it's cost to 0 and marks it as visited. The currentVertex is then set to home.

The graph loops through the following code while there are still unvisited vertexes 

*start of loop*

* It gets all edges of the current vertex.

* Then iterates through each all of the edges and calculates a temporery cost to that vertex from this vertex. If this cost is lower the the current cost of the destination vertex it replaces it. The destination vertex's previous field is also set to this vertex.

* The current vertex is then marked as visited.

* The unvisited vertex with the lowest cost is then selected from the graph and currentVertex is set to it.

*end of loop*

The router then creates another ArrayList of it's immediately connected vertexes (it's neighbors).

Afterword the RoutingTable is cleared for new data.

The router then iterates through each vertex in the graph(except itself). If current vertex in the loop is in the ArrayList of immediately connected vertexes it adds it to the RoutingTable with its associate cost and itself as the next hop and destination.

If it is not an immediately connected vertex it will check the previous value of the vertex, if that is also not in the ArrayList of immediately connected vertexes it will again check previous value of the vertex. 

Eventually it will get to a neighbor vertex that is directly connected to the router. It will then add the destination vertex as the destination and the neighbor vertex as the next hop. The cost is set to the cost to the neighbor vertex.

###Client

Our client is a simple but efficient. It has a text only userface. The user is asked to type a message and then asked for a destination port. The client will continue to ask the user until they give a valid port (above 1000 below 50000). The client in link state is slightly different as it will reply to a flood message confirming that it is infact online by reply with a table of size one, containing only the router it's connected to.

####What We Learned(temporary)

By doing this project we learned a lot about networking, both it's support in java and its history. Much research was done before we could even start the project. We had to learn exactly what distance vector and link state routing protocols are. Wikipedia was an invaluable resource for leaning about things such as Dijkstra's algorithm as it contains visualisations of the algorithm at work.