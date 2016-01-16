tables
Link state
Client
Things we learned



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

###Routing Tables

Both protocols use the same routing table format. Although the Link State version has two extra methods, clear table and equals. Otherwise the two classes are identical. Routing tables store the routes that each router has available. It has many useful internal functions such as add, update and getNextHop. This allows them to be used and edited easily with minimal impact to the main application.

All of its variables are private and can only be accessed via the available methods. This means that we could later change the internal structure of the class without affecting any other code. Currently destination, nextHop and cost are stored in seperate array lists. This allows them to be expanded easily. 

Routing tables can be constructed from either an integer and two arrays (owner, connctions, initialCost) or from a byte array received in a packet. Routing tables can be converted to a byte array for sending.



