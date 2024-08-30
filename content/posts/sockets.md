---
title: "C++ Sockets"
summary: "C++ Sockets Notes"
date: 2024-08-25
tags: ["c++", "sockets", "networking"]
author: ["JC"]
draft: true
weight: 0
ShowToc: true
---

### Introduction
This is my notes for c++ sockets. 

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
A socket is a entrance/exit of the transport layer in the 7 layers of network(OSI layers). It is used to transmit data using different transmission protocols eg. TCP, UDP.
```c++ {linenos=true}
socket(domain, service, protocol)
/*domain/Adress family parameter specifies the communication domain. some examples:
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
m_Sock = socket(AF_INET, SOCK_STREAM, 0);
m_Sock = socket(AF_INET6, SOCK_DGRAM, 0);
```

2. sockaddr & sockaddr_in (linux/unix) / SOCKADDR & SOCKADDR_IN (WIN)
Socket addresses are what allows sockets to identify the address your socket is sending/receiving to or from 
```c++ {linenos=true}
//sockaddr is used to specify the address of the socket (target or even native)

//Example for creating a sockaddr_in for a client
SOCKADDR_IN BlankNetSocket::CreateSOCKAddrForClient(ADDRESS_FAMILY family, str ipAddr, uint port)
{
	SOCKADDR_IN addr;
	addr.sin_family = family;
	addr.sin_port = htons(port);
	inet_pton(family, ipAddr.c_str(), &addr.sin_addr);
	return addr;
}

//Example for creating a sockaddr_in for server
SOCKADDR_IN BlankNetSocket::CreateSOCKAddrForServer(ADDRESS_FAMILY family, uint port)
{
	SOCKADDR_IN addr;
	addr.sin_family = family;
	addr.sin_port = htons(port);
	addr.sin_addr.s_addr = htonl(INADDR_ANY);
	return addr;
}
```

3. connection and binding functions
These are the functions that allow you to connect your client to the server and allow your server to accept your client
```c++ {linenos=true}
//connect() connects the client to the server at some ipaddr via the sockaddr
int ret = connect(m_Sock, (SOCKADDR*)&m_Address, sizeof(m_Address));

//bind() binds the server to the stated ipaddr
int ret = bind(m_Sock, (SOCKADDR*)&m_Address, SOCKETADDRLEN);

//listen() marks the socket as a passive socket that only receives incoming connection requests using accept() [ie. servers]
int ret = listen(m_Sock, MAX_LISTEN_SOCKETS_QUEUE);

//accept() gets the socket of the requesting client and get its data
SOCKET clientSocket = accept(m_Sock, (SOCKADDR*)&clientAddr, &SOCKETADDRLEN);
//note that ret in these cases are the success/failure returns which tells us if there are errors with the function
```

4. communication functions
some common functions to send and recv data
```c++ {linenos=true}
//send() sends a message through a socket
int ret = send(m_Sock, message.c_str(), message.length(), 0);

//recv() reads the data gotten from the socket which send() a message
int ret = recv(m_Sock, buf, MAX_BUF_SIZE, 0);

//TODO: ADD RECVFROM()

//note that ret in these cases are the success/failure returns which tells us if there are errors with the function
```

5. IO multiplexing
we will be talking about select(), epoll(), fd_set and how to use them
```c++ {linenos=true}


```