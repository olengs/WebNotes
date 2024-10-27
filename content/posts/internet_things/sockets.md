---
title: "C++ Sockets"
summary: "C++ Sockets Notes"
date: 2024-08-25
tags: ["C++", "Sockets", "Networking"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---

### Introduction
This is my notes for c++ sockets. There will be differences when using sockets in windows and linux/unix/OSX. This is due to the different socket types where windows uses [WinSock sockets](https://en.wikipedia.org/wiki/Winsock) and unix/OSX uses [Berkeley sockets](https://en.wikipedia.org/wiki/Berkeley_sockets)

### Libs required for sockets
For windows you will require the winsock from windows (ws2_32.lib)
```c++{linenos=true}
#include <winsock2.h>
#include <Ws2tcpip.h>
#pragma comment(lib, "Ws2_32.lib")

//you are also required to initialise and de-initialise WSA
//startup example
if (WSAStartup(MAKEWORD(2, 2), &WsaData) != 0) {
    printf("WSAStartup() error!\n");
    return;
}
//destroy example
WSACleanup();
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

//to close the socket in windows you call closesocket()
closesocket(m_Sock);

//in linux/unix/OSX you call close()
close(m_Sock);
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

5. I/O multiplexing
we will be talking about select(), epoll(), fd_set and how to use them

i. we will first look at select(), select() is used differently in WinSock (windows) and Berkeley sockets (OSX/unix) and I will go through the windows one here (Berkeley sockets below)
```c++ {linenos=true}
/*int select(int nfds, fd_set *_Nullable restrict readfds,
                  fd_set *_Nullable restrict writefds,
                  fd_set *_Nullable restrict exceptfds,
                  struct timeval *_Nullable restrict timeout);*/
int ret = select(0, &m_TempFds, 0, 0, &m_Timeout);
```
Lets run through the arguments:
int nfds -> only used for Berkeley sockets
readfds -> set of sockets to be checked for readability
writefds -> set of sockets to be checked for writability
exceptfds -> set of sockets to be checked for errors
timeout -> the maximum time for select to wait (a blocking function until timeout)

select() is normally called in a loop
```c++ {linenos=true}
while (true) {
	TIMEVAL m_Timeout;
	m_Timeout.tv_sec = 1;
	m_Timeout.tv_usec = 0;
	m_TempFds = m_ReadFds;

#ifdef WIN32
	int ret = select(0, &m_TempFds, 0, 0, &m_Timeout);
	if (0 == ret) {
		printf("select() returned timeout\n");
		return;
	}
	else if (0 > ret) {
		printf("select() returned error\n");
		return;
	}
	for (int Index = 0; Index < m_TempFds.fd_count; Index++)
	{
		//if (!FD_ISSET(Index, &m_TempFds)) continue;
		if (m_TempFds.fd_array[Index] == m_Sock)
		{ // New connection requested by new client.
			InitNewConnection();
		}
		else {
			// Something to read from socket.
			ReadFromConnection(m_TempFds.fd_array[Index]);
		}
	}
#endif
}
```
select() will return 0 if timeout, -1 if error and 1 if successful
when successful, readable sockets will be returned.
if the readable sockets include the server, it means a new connection has been found which you then call accept() to accept the connection.
any other readable sockets returned is the message sent by the client sockets connected.

For Berkeley sockets, I've only tested this on my M2 mac. But this is what I have. Note that for some reason, when I use select() in a while loop on my M2 Mac, the function gets stuck, and only fixes when I call select() via another function.
```c++ {linenos=true}
int ret = select(m_MaxFD + 1, &m_TempFds, NULL, NULL, &m_Timeout);
if (0 == ret) {
	printf("select() returned timeout\n");
	return;
}
else if (0 > ret) {
	printf("select() returned error\n");
	m_IsListening = false;
	return;
}
if (FD_ISSET(m_Sock, &m_TempFds)) {
	//Handle new connection
	InitNewConnection();
}

std::vector<SOCKET> readable;
for (auto it : m_Users) {
	if (FD_ISSET(it.first, &m_TempFds)) {
		readable.push_back(it.first);
	}
}
for (auto it : readable) {
	ReadFromConnection(it);
}
```
For Berkeley scokets, you have to do the following:
- Keep track of the total number of file descriptors (sockets)
- Use FD_ISSET to check all connected sockets for available file descriptors


### Additional Notes
Why did I use FD_ISSET instead of fd_set.fd_array[]? It is because Berkeley sockets doesn't allow access to fd_array. The socket.h and winsock2.h lib have different designs for fd_set.

For Berkeley sockets there is an alternative for message watching, the epoll() function. I personally haven't tried it. Maybe when I try it one day. I will update this. For now, this is all for sockets.
