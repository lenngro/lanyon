---
layout: post
title: "A brief intro to TCP/IP and a basic client-server setup in C++ that uses TCP/IP"
author: Lennart Grosser
date: 2021-01-05
category: "how-to"
description: "A brief intro to TCP and IP: what are they and how do they work. Additionally, how can one program a basic client-server setup in C++. that uses TCP/IP under the hood." 
tags: ["C", "C++", "tcp", "ip", "cpp", "server", "socket", "internet", "protocol", "transmission", "control"]
---

> This post will be a brief introduction to TCP/IP servers, what they are, how they work and how one can program a fairly simple server in C++.

## TCP/IP General

### TCP/IP
TCP/IP is a combination of two protocols that define how computers can communicate over the internet. TCP/IP servers are servers that communicate with clients following the TCP/IP protocols.

#### TCP
The **Transmission Control Protocol (TCP)** is a protocol that guarantees reliable end-to-end connections between two communication partners in a computer network. It's responsible for splitting the data that is supposed to be exchanged into smaller packages, merging it in the correct order and preventing/correcting package loss and [network congestion](https://en.wikipedia.org/wiki/Network_congestion). TCP can be understood as an equivalent to the transport layer in the [OSI model](https://en.wikipedia.org/wiki/OSI_model) and was originally developed by Robert E. Kahn and Vinton G. Cerf in 1973.

#### IP
The **Internet Protocol (IP)** is responsible for routing, e.g. finding the way through the network to the correct receiver. The IP assigns addresses (the IP addresses) to the network's participants following a specific pattern (see IPv4 vs. IPv6). By sending data to the respective IP address, the devices in the network can communicate with each other. The IP protocol was first published in 1974 by the Institute of Electrical and Electronics Engineers (IEEE).

##### TCP/IP
TCP and IP were combined and standardized in 1981.

### What are they used for?
TCP/IP can be used to deliver all kinds of data across a network from one device to another and back. Popular applications are:
* internet browsing
* remote login
* remote server access
* network file transfer
* e-Mail

### How do they work?
### TCP
TCP has two requirements for a successful connection:
1. both communication partners need to have a unique IP address
2. the desired port needs to be enabled. The port allows associating a specific TCP/IP connection with the corresponding application running on the server.

#### Connection Establishment
The connection establishment works as follows:

In the first step, the client sends the server a SYN package (SYN=synchronize) with a random **sequence number**. The sequence number is a fundamental tool to ensure the complete and correctly ordered transmission of packages.

Once the server has received the SYN package, it accepts the connection by replying with a SYN-ACK package (ACK=acknowledgement) that contains the sequence number increased by one to indicate the correctly received synchronization request (the increased sequence number may also be called **acknowledgement number**). Additionally, it includes another (again random) sequence number to express the wish for a converse connection.

The client receives the SYN-ACK package. It returns the so-called ACK package that contains its own sequence number as well as the server's sequence number increased by one. Note how after this package both sequence numbers were increased one time by the opposite side to express a successful connection establishment.

After the server receives the ACK package the connection is established. No actual payload data was exchanged until now.

#### Payload Exchange
The fourth package is the first package that contains payload data and is sent by the client to the server (typically containing the HTTP request). The two sequence numbers are unchanged. Let's assume the payload has a length of 104 bytes.

The server receives the package and returns the first payload-containing package itself. In the package he changes the client's sequence number to 105 (+104 bytes) to indicate how much data he has already received. The server's sequence number remains unchanged. Let's assume the package has a size of 1200 bytes.

The client receives the package with it's sequence number set to 105. He now knows the server has already received 105 bytes of data. He builds the next package with payload data and changes the server's sequence number to 1201 (+1200 bytes) and sends it.

If the client does not need to send any more data and only wants to keep receiving data, it's sequence number will remain unchanged while the server's sequence number increases with every package.

This process is executed until the client has received the complete payload data from the server.

#### Connection Tear-down
Once the client has acknowledged the last package sent by the server, it decides to terminate the connection. Therefore it sends the FIN package that doesn't contain payload data (the client's sequence number remains unchanged at 105 while the server's sequence number equals the initial number + the sum of all the bytes the client received). Let's assume the last server's sequence number is 10004.

The server accepts the termination and returns another package with the client's sequence number increased by one (to 106). The server's sequence number remains unchanged.

The client ultimately finishes the connection by sending the final package containing it's unchanged sequence number and the server's sequence number increased by one (10005). 

#### TCP header
TCP uses so-called headers to transport it's information. A TCP header is a binary number of variable length separated into 32-bit blocks. A TCP header consists of at least 5 blocks, yielding a minimum size of 20 bytes. It can be separated into different fields that describe different information. See the fields of a TCP header below along with their sizes and a short description:

| Name | Length (in bits) | Description |
|-|-|-|
| Source Port | 16 | Source port. |
| Destination Port | 16 | Destination Port. |
| Sequence Number | 32 | The client's sequence number used to confirm a successful transmission establishment / tear-down or the last received package. |
| Acknowledgement Number | 32 | The server's sequence number (acknowledgement number) used to confirm a successful transmission establishment / tear-down or the last received package. |
| Data Offset | 4 | Number of 32-bit blocks that the header is divided into. At least 5. |
| Reserved | 6 | Set to 000000 and used for extensions of the TCP header. |
| Flags | 6 | Transmission flags, e.g. SYN, ACK, FIN, URG, PSH, RST. |
| Window Size | 16 | Through the window size the receiver defines the maximum size of data that the sender is allowed to send (so input buffer overflow is prevented --> flow control). |
| Checksum | 16 | A checksum that can be used to determine transmission errors. |
| Urgent-Pointer | 16 | Combined with the sequence number used to determine the current position of the payload data in the stream of transferred data. Only valid when URG flag is set. |
| Options | variable | Variable field containing optional information |
| Padding | variable | Variable field to ensure the header's field size. |

### IP
#### IP Address
As previously mentioned, the IP address is a unique address within a computer network that allows sending data to the intended receiver. They are not bound to a specific physical location but rather to a specific device. The IP defines a package structure that summarizes the payload data: the protocol determines how the source and destination of the data is described. It separates these meta information from the actual payload by adding the 'IP header' to data packages. IP header combined with the payload data may also be called IP datagram.

So what exactly is an IP address?

It's a 32-bit number that uniquely identifies a host within a TCP/IP computer network. A host can be any device connected to the network such as a computer, a phone or a printer. The IP address is described in the ***dotted-decimal format***:

> 192.168.123.32

The IP address can be distinguished into two separate parts:
1. the network part describing the destination network of the package
2. the host part describing the destination host within the network.

#### Subnet Masks
Which part of the IP address describes the network and which part describes the host is not fixed in TCP/IP. However a router can use an additional 32-bit number to infer whether the destination host is part of the local subnet. This number is called the **subnet mask**. The subnet masks may look as follows:

> 255.255.255.0

**So how to separate the IP address into network and host by using the subnet mask?**

To answer that question we need to consider the binary representations of the IP address and subnet mask:

> 11000000.10101000.01111011.10000100 (192.168.123.32)
> 11111111.11111111.11111111.00000000 (255.255.255.0)

The network part of the IP address can now be inferred by only considering the positions where the subnet mask is set to 1. Similarly, the host part is defined by the positions of the IP address where the subnet mask is set to 0.

In this example, the first 24 bits of the IP address (11000000.10101000.01111011 = 192.168.123) refer to the network while the last 8 bits (10000100 = 32) refers to the host.

#### Routing
As a data package flows through a network (e.g. the internet) it is forwarded to it's correct destination by routers. The router determines the network that the host is part of by considering the network part of the IP address. It uses it's routing table to determine the best way to forward the package to it's destination network. Once the destination network is reached, the local router sends the package to the host by considering the host part of the IP address.

[Click here for further information on how IP routing works.](https://en.wikipedia.org/wiki/IP_routing)

#### IPv4 and IPv6
When researching or working on network communication, one will probably encounter the terms **IPv4** and **IPv6**. Despite the name they don't describe the fourth and sixth generation of the IP but the first and the second: IPv4 is the first official version of the Internet Protocol that uses the fourth version of the TCP. IPv6 is the direct successor of IPv4. IPv5 was skipped as the project was discontinued. To prevent confusion, the next version was called IPv6.

IPv4 and IPv6 use different address schemes to specify a device within a network. IPv4 features up to about 4 billion (4294967296 = 2^32) different addresses while IPv6 can define up to a sextillion (10^36 = 1 with 37 zeros) addresses.

#### IPv4 Header
As previously mentioned, the Internet Protocol uses a IP header to depict the necessary information needed for successful routing. The IP header is a binary 32 bytes number. It can be separated into different fields that describe different information. See the fields of an IPv4 header below along with their sizes and a short description:

| Name | Length (in Bits) | Description 
|-|-|
| Version | 4 | Determines the IP version (4 or 6).
| IHL | 4 | Determines the total length of the header. Needed due to the Options field. 
| Type Of Service | 8 | Defines the service of a IP data package, e.g. higher priority, type of throughput or resource allocation in routers. 
| Total Length | 16 | Determines the total length of the entire IP datagram. 
| Identification | 16 | Contains a value needed for merging fragmented packages. 
| Flags | 3 | Control flag: DF (Don't Fragment ==> no more packages following), MF (More Fragment ==> more packages following) 
| Fragmentation Offset | 13 | Contains information needed for merging fragmented packages. 
| Time To Live | 8 | Defines the time to live of the datagram. Can range from 0 to 255. Every router that forwards the package decreases the value by at least 1. 
| Protocol | 8 | Determines which protocol is supposed to further process the package after receiving (e.g. 6 for TCP or 17 for UDP). 
| Header Checksum | 16 | A checksum that can be used to determine transmission errors. 
| Source Address | 32 | Source IP address. 
| Destination Address | 32 | Destination IP address. 
| Options | variable (up to 96) | Variable field containing optional information (e.g. security info). 
| Padding | variable | Variable field to ensure the IP header's total length of 32 bytes (due to the variable Options field).                                 

[Click here to see how an IPv6 header looks like.](https://en.wikipedia.org/wiki/IPv6_packet)

## Simple Usage (C++)
Find a simple implementation of a client-server setting that uses TCP/IP under the hood below. The code will create a simple TCP/IP connection between a server and a client. The client can send messages to the server whose going to display and return the message once received.

The full script can be found at the bottom of the page.

This code follows the great tutorial by [Sloan Kelly](https://www.youtube.com/watch?v=cNdlrbZSkyQ).

### Create A Socket
Sockets are the 'receiving endpoints' of a connection within a network. Generally, client-server architectures are used: a server socket communicates with on or more client sockets.

{% highlight cpp %}
// main.cpp
std::cout << "Creating server socket..." << std::endl;
int listening = socket(AF_INET, SOCK_STREAM, 0);
    if (listening == -1)
    {
        std::cerr << "Can't create a socket!";
        return -1;
    }
{% endhighlight %}

**AF_INET** and **SOCK_STREAM** are constants that define what kind of connection is supposed to be established. **AF_INET** represents the IPv4 address family while **SOCK_STREAM** represents a TCP connection.

### Bind The Socket
The socket needs to be told to which address and port it should connect. This process is called 'binding' the socket. Therefore we are creating a socket address object **hint**. 

{% highlight cpp %}
// main.cpp
struct sockaddr_in hint;
{% endhighlight %}

No we need to configure the socket address object correctly. We need to set the correct IP address family (**AF_INET**), we need to specify the port and the IP address. 

{% highlight cpp %}
// main.cpp
hint.sin_family = AF_INET;
hint.sin_port = htons(54000);
inet_pton(AF_INET, "0.0.0.0", &hint.sin_addr);
{% endhighlight %}

Here we are using "0.0.0.0" as IP address which isn't really a specification but let's the server itself decide on which address it's going to listen to incoming connections.

[htons](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.2.0/com.ibm.zos.v2r2.bpxbd00/htons.htm) is a function that translates an unsigned integer into a network byte order and is used here to translate the human readable port into a format that the network can work with.

[inet_pton](https://man7.org/linux/man-pages/man3/inet_pton.3.html) is a function that transforms an IP address from text form into binary form.

{% highlight cpp %}
// main.cpp
std::cout << "Binding socket to sockaddr..." << std::endl;
if (bind(listening, (struct sockaddr *)&hint, sizeof(hint)) == -1) 
{
    std::cerr << "Can't bind to IP/port";
    return -2;
}
{% endhighlight %}

### Let The Socket Listen
Now that we have our socket configured and bound to an IP address and port, we need to tell it that it should listen to any incoming connections:

{% highlight cpp %}
std::cout << "Mark the socket for listening..." << std::endl;
    if (listen(listening, SOMAXCONN) == -1)
    {
        std::cerr << "Can't listen !";
        return -3;
    }
{% endhighlight %}

**SOMAXCONN** defines the maximum number of incoming connections that the socket should listen to.

### Create A Client Socket And Start A Call

We're going to create a client that will accept the server's connection:

{% highlight cpp %}
// main.cpp
sockaddr_in client;
socklen_t clientSize = sizeof(client);

std::cout << "Accept client call..." << std::endl;
int clientSocket = accept(listening, (struct sockaddr *)&client, &clientSize);

std::cout << "Received call..." << std::endl;
if (clientSocket == -1)
{
    std::cerr << "Problem with client connecting!";
    return -4;
}
{% endhighlight %}

Here we are creating a client socket through the use of [accept](https://man7.org/linux/man-pages/man2/accept.2.html). This function needs to know the listening socket that the client should connect to and an empty socket address object. It will return a newly created client socket that represents the client's connection side. It fills the socket address object 'client' with the IP family, address and port that client's connection is configured to.

We can monitor the client's connection info through the use of the function [inet_ntoa](https://linux.die.net/man/3/inet_ntoa) that converts an IP address to a human readable format.

{% highlight cpp %}
// main.cpp
std::cout << "Client address: " << inet_ntoa(client.sin_addr) << " and port: " << client.sin_port << std::endl;
{% endhighlight %}

Now that we successfully established a connection between server and client, we can close the listening socket:

{% highlight cpp %}
// main.cpp
close(listening);
{% endhighlight %}

### Exchange Messages
Let's communicate! To actually exchange messages we're going to create a reading buffer and waiting for the client to send a message. Once the client has send a message, the server is using the buffer to store the message, printing it and returning it to the client.

{% highlight cpp %}
// main.cpp
char buf[4096];
while (true) {
    // clear buffer
    memset(buf, 0, 4096);

    // wait for a message
    int bytesRecv = recv(clientSocket, buf, 4096, 0);
    if (bytesRecv == -1)
    {
        std::cerr << "There was a connection issue." << std::endl;
    }
    if (bytesRecv == 0)
    {
        std::cout << "The client disconnected" << std::endl;
    }
    
    // display message
    std::cout << "Received: " << std::string(buf, 0, bytesRecv);

    // return message
    send(clientSocket, buf, bytesRecv+1, 0);
}
// close socket
close(clientSocket);
{% endhighlight %}

And that's it. We can now exchange messages from the client to the server and back using a TCP/IP connection under the hood.

### Full Script / Usage

Find the full script below. To use it, compile and run it with the C++ compiler of your choice.

Once the script is running, open a console and type:

> telnet localhost 54000

This will create a connection from the client (your console) to the server script. Type a message and submit with enter to see it get sent to the server and back!

{% highlight cpp %}
// main.cpp
#include <iostream>
#include <sys/types.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netdb.h>
#include <arpa/inet.h>
#include <string.h>
#include <string>

int main()
{
    std::cout << "Creating server socket..." << std::endl;
    int listening = socket(AF_INET, SOCK_STREAM, 0);
    if (listening == -1)
    {
        std::cerr << "Can't create a socket!";
        return -1;
    }

    struct sockaddr_in hint;
    hint.sin_family = AF_INET;
    hint.sin_port = htons(54000);
    inet_pton(AF_INET, "0.0.0.0", &hint.sin_addr);

    std::cout << "Binding socket to sockaddr..." << std::endl;
    if (bind(listening, (struct sockaddr *)&hint, sizeof(hint)) == -1) 
    {
        std::cerr << "Can't bind to IP/port";
        return -2;
    }

    std::cout << "Mark the socket for listening..." << std::endl;
    if (listen(listening, SOMAXCONN) == -1)
    {
        std::cerr << "Can't listen !";
        return -3;
    }

    sockaddr_in client;
    socklen_t clientSize = sizeof(client);

    std::cout << "Accept client call..." << std::endl;
    int clientSocket = accept(listening, (struct sockaddr *)&client, &clientSize);

    std::cout << "Received call..." << std::endl;
    if (clientSocket == -1)
    {
        std::cerr << "Problem with client connecting!";
        return -4;
    }

    std::cout << "Client address: " << inet_ntoa(client.sin_addr) << " and port: " << client.sin_port << std::endl;

    close(listening);

    char buf[4096];
    while (true) {
        // clear buffer
        memset(buf, 0, 4096);

        // wait for a message
        int bytesRecv = recv(clientSocket, buf, 4096, 0);
        if (bytesRecv == -1)
        {
            std::cerr << "There was a connection issue." << std::endl;
        }
        if (bytesRecv == 0)
        {
            std::cout << "The client disconnected" << std::endl;
        }
        
        // display message
        std::cout << "Received: " << std::string(buf, 0, bytesRecv);

        // return message
        send(clientSocket, buf, bytesRecv+1, 0);
    }
    // close socket
    close(clientSocket);

    return 0;

}
{% endhighlight %}


## Sources and further resources
1. [A nice summary of the most important terms in TCP](https://www.cs.miami.edu/home/burt/learning/Csc524.032/notes/tcp_nutshell.html)
2. [A great post describing package exchange in detail by monitoring a TCP connection in Wireshark](https://packetlife.net/blog/2010/jun/7/understanding-tcp-sequence-acknowledgment-numbers/)
3. [A concise description of the IP datagram](http://mars.netanya.ac.il/~unesco/cdrom/booklet/HTML/NETWORKING/node020.html)
4. [An explanation of IP  addresses, subnet masks and network classes](https://docs.microsoft.com/en-us/troubleshoot/windows-client/networking/tcpip-addressing-and-subnetting)
5. [Further information on how IP routing works](https://en.wikipedia.org/wiki/IP_routing)
