### What Makes Kafka High In Throughput

* Maximized use of sequential disk reads and writes
* Zero-copy processing of messages
* Use of Linux OS page cache rather than Java heap for caching
* Partitioning of topics across multiple brokers in a cluster
* Smart client libraries that offload certain functions from the brokers
* Batching of multiple published messages to yield less frequent network round trips to the broker
* Support for multiple in-flight messages
* Prefetching data into client buffers for faster subsequent requests.
