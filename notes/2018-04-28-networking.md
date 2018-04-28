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
