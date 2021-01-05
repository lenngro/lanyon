---
layout: post
title: "A simple TCP/IP server in C++"
author: Lennart Grosser
date: 2021-01-05
category: "how-to"
description: "A short description of how TCP/IP servers work and how one can program a basic one in C++."
tags: ["C", "C++", "tcp", "ip", "cpp", "server", "socket"]
---

> This post will be a brief introduction to TCP/IP servers, what they are, how they work and how one can program a fairly simple server in C++.

## TCP/IP General

### TCP/IP
TCP/IP is a combination of two protocols that define how computers can communicate over the internet. TCP/IP servers are servers that communicate with clients following the TCP/IP protocols.

#### TCP
The **Transmission Control Protocol (TCP)** is a protocol that guarantees reliable end-to-end connections between two communication partners in a computer network. It's responsible for splitting the data that is supposed to be exchanged into smaller packages, merging it in the correct order and preventing as well as correcting package loss and [network congestion](https://en.wikipedia.org/wiki/Network_congestion). The TCP protocol can be understood as an equivalent to the transport layer in the [OSI model](https://en.wikipedia.org/wiki/OSI_model).
TCP was originally developed by Robert E. Kahn and Vinton G. Cerf in 1973 and standardized in 1981.

#### IP
The **Internet Protocol (IP)** is reponsible for routing, e.g. finding the way through the network to the correct receiver. The IP assigns addresses (the IP addresses) to the network's participants following a specific pattern (see IPv4 vs. IPv6). By sending data to the respective IP address, the devices in the network can communicate with each other.

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

In the first step, the client sends the server a SYN package (SYN=synchronize) with a random sequence number. The sequence number is a fundamental tool to ensure the complete and correctly ordered transmission of packages.

Once the server has received the SYN package, it accepts the connection by replying with a SYN-ACK package (ACK=acknowledgement) that contains the sequence number increased by one to indicate the correctly received synchronization request (the increased sequence number may also be called acknowledgement number). Additionally, it includes another (again random) sequence number to express the wish for a converse connection.

The client receives the SYN-ACK package. It returns the so-called ACK package that contains the its own sequence number aswell as the server's sequence number increased by one. Note how after this package both sequence numbers were increased one time by the opposite side to express a successful connection establishment.

After the server receives the ACK package the connection is established. No actual payload data was exchanged until to point.

#### Payload Exchange
The fourth package is the first package that contains payload data and sent by the client to the server (typically containing the HTTP request). The two sequence numbers are unchanged. Let's assume the payload has a length of 104 bytes.

The server receives the package and returns the first package containing payload itself. In the package he changes the client's sequence number to 105 (+104 bytes) to indicate how much data he already has received. The server's sequence number remains unchanged. Let's assume the package has a size of 1200 bytes.

The client receives the package with it's sequence number set to 105. He now knows the server has already received 105 bytes of data. He builds the next package with payload data and changes the server's sequence number to 1201 (+1200 bytes) and sends it.

If the client does not need to send any more data and only wants tqo receive data, it's sequence number will remain unchanged while the server's sequence number increases with every package.

This process is executed until the client received the complete payload data from the server.

#### Connection Tear-down
Once the client has acknowledged the last package sent by the server it decides to terminate the connection. Therefore it sends the FIN package that doesn't contain payload data (the client's sequence number remains unchanged at 105 while the server's sequence number equals the initial number + the sum of all the bytes the client received). Let's assume the server's sequence number is 10004.

The server accepts the termination and returns another package with the client's sequence number increased by one (to 106). The server's sequence number remains unchanged.

The client ultimately finishes the connection by sending the final package containing it's unchanged sequence number and the server's sequence number increased by one (10005). 

## Simple Implementation (C++)
tbd

## Further resources
1. [A nice review of the most important terms in TCP](https://www.cs.miami.edu/home/burt/learning/Csc524.032/notes/tcp_nutshell.html)
2. [A great post describing package exchange in detail by monitoring a TCP connection in Wireshark](https://packetlife.net/blog/2010/jun/7/understanding-tcp-sequence-acknowledgment-numbers/)