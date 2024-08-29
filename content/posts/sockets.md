---
title: "C++ Sockets"
summary: "C++ Sockets Notes"
date: 2024-08-25
tags: ["c++", "sockets", "networking"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---

### Introduction
This is my notes for c++ sockets. A socket is a entrance/exit of the transport layer in the 7 layers of network(OSI layers). It is used to transmit data using different transmission protocols eg. TCP, UDP.

### Libs required for sockets
For windows you will require the winsock from windows (ws2_32.lib)
```c++{linenos=true}
#include <winsock2.h>
#include <Ws2tcpip.h>
#pragma comment(lib, "Ws2_32.lib")
```

For unix/linux you will require the standard C socket libraries
```c++{linenos=true}
#include <sys/socket.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
```

### Common Function/Data Types
1. socket()
```c++ {linenos=true}
socket(domain, service, protocol)
/*domain parameter specifies the communication domain. some examples:
AF_INET - IPv4 Internet Protocols
AF_INET6 - IPv6 Internet Protocols
AF_UNIX/AF_LOCAL - Local Communication
AF_BLUETOOTH - bluetooth low-level socket protocol


service parameter specifies the type of transmission, some examples:
SOCK_STREAM - TCP
SOCK_DGRAM - UDP

protocol specifies the protocol for the specific protocol family, usually only a single protocol exists which can be specified as 0. HOwever it is possible that multiple protocol exists and is used to specify the "communication domain" which communication is to take place.

On success, the socket or file descriptor is returned, on error, -1 is returned.
*/

//EXAMPLE
socket(AF_INET, SOCK_STREAM, 0);
socket(AF_INET6, SOCK_DGRAM, 0);
```

2. sockaddr & sockaddr_in (linux/unix) / SOCKADDR & SOCKADDR_IN (WIN)
