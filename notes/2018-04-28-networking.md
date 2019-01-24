# Networking

## TCP vs. UDP
### TCP
1. connection oriented stream over IP network.
2. it ensures that all packets will reach the destination in the correct order.
3. less efficient that udp.
4. if a TCP packet is lost, it will be resent.

### UDP
1. connection-less, datagram oriented
2. the integrity is only guaranteed on a single datagram.
3. more efficient because there is no ACK.
4. generally used for real-time communication (Video or audio streaming).
5. allows broadcast transmission (DHCP) to multiple hosts.

However UDP is not always faster than TCP. TCP will try to buffer the data and fill a full network segment to optimize the transmission. while UDP puts the data on the wire immediately.

TCP uses ~200ms to retransmit, thus if you are behind a horrible network, then it will be much slower. but normally TCP and UDP differs very little on performance of throughput and latency.

### 3-way handshake
`SYN, SYN-ACK, ACK`

### 4-way goodbye
`FIN (a>b), ACK(a<b), FIN(a<b), ACK(a>b)`

Reason of separating one response to ACK and FIN: wait for the transmission to end.

### Slow start
To avoid sending more data than the network is capable of transmitting. the size of the congestion window will be doubled with each ACK received (1,2,4, or 10 MSS).

The transmission rate will be increased with slow-start algorithm until a loss is detected.

### 5 I/O models
1. blocking
The system call does not return until data has arrived and been copied into the application.
2. non-blocking
Instead of blocking, the system calls can immediately return EWOULDBLOCK if data is not available. This means we can now loop through all connections and serve them as needed, but if nothing is available we end up polling (busy waiting) and burning through CPU.
3. I/O multiplexing: select/poll/epoll/kqueue 
- select lets you pass in a list of file descriptors and it returns the status on each FD. It has some pretty big performance limitations (e.g. returns status of all FDs and you have to loop through to find what you’re interested in. Limited by FD_SETSIZE which is 1024 in Linux).
- poll select minus the big problems, works well up to a few thousand connections and might be good enough for small use cases.
- epoll (linux), kqueue (bsd) the recommended polling option. A bit of extra implementation complexity but scales much better.
4. Signals (sigio)
Instead of checking on the status of the file, we can ask the kernel to send a signal when data is available. Real-world implementation - Nginx.
5. POSIX AIO
Applications initiate I/O operations, and get notified when they are complete (through signals). Like SIGIO but it’s once the data has been moved to the application buffer.
