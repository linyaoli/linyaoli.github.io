# Comparing Layer 4 and Layer 7 Load Balancers


I stole the following from Nginx website.

## L4

Layer 4 load balancing operates at the intermediate transport layer, which deals with delivery of messages with no regard to the content of the messages. Transmission Control Protocol (TCP) is the Layer 4 protocol for Hypertext Transfer Protocol (HTTP) traffic on the Internet. Layer 4 load balancers simply forward network packets to and from the upstream server without inspecting the content of the packets. They can make limited routing decisions by inspecting the first few packets in the TCP stream.

Layer 4 load balancers make their routing decisions based on address information extracted from the first few packets in the TCP stream, and do not inspect packet content. A Layer 4 load balancer is often a dedicated hardware device supplied by a vendor and runs proprietary load-balancing software, and the NAT operations might be performed by specialized chips rather than in software.

Layer 4 load balancing was a popular architectural approach to traffic handling when commodity hardware was not as powerful as it is now, and the interaction between clients and application servers was much less complex. It requires less computation than more sophisticated load balancing methods (such as Layer 7), but CPU and memory are now sufficiently fast and cheap that the performance advantage for Layer 4 load balancing has become negligible or irrelevant in most situations.

## L7

Layer 7 load balancing operates at the high-level application layer, which deals with the actual content of each message. HTTP is the predominant Layer 7 protocol for website traffic on the Internet. Layer 7 load balancers route network traffic in a much more sophisticated way than Layer 4 load balancers, particularly applicable to TCP-based traffic such as HTTP. A Layer 7 load balancer terminates the network traffic and reads the message within. It can make a load-balancing decision based on the content of the message (the URL or cookie, for example). It then makes a new TCP connection to the selected upstream server (or reuses an existing one, by means of HTTP keepalives) and writes the request to the server.

Layer 7 load balancing is more CPU-intensive than packet-based Layer 4 load balancing, but rarely causes degraded performance on a modern server. Layer 7 load balancing enables the load balancer to make smarter load-balancing decisions, and to apply optimizations and changes to the content (such as compression and encryption). It uses buffering to offload slow connections from the upstream servers, which improves performance.

A device that performs Layer 7 load balancing is often referred to as a reverse-proxy server.
