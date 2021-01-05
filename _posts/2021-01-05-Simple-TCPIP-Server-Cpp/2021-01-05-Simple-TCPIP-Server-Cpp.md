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
The **Transmission Control Protocol (TCP)** is a protocol that guarantees reliable end-to-end connections between communication partners, e.g. splitting the data that is supposed to be exchanged into smaller packages, merging it in the correct order and preventing package loss and [network congestion](https://en.wikipedia.org/wiki/Network_congestion). The TCP protocol can be understood as an equivalent to the transport layer in the [OSI model](https://en.wikipedia.org/wiki/OSI_model).

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
something

## Simple Implementation (C++)