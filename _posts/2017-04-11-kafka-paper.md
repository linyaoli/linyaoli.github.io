---
layout: post
title: Kafka paper Q&A
tags: programming
---

This post briefly discusses some key aspects in Kafka paper.

------
Q: What's the correspondence between partition and topic?

A: Each topic can have one or many partitions, each partition has only one topic.

------
Q: Why does message use offset instead of id?

A: Offset essentially has the same functionality of an id. It is unique and increasing(not consecutive). The offset of next message is to add the length of current message to its current offset.

------
Q: What's the correspondence between a consumer and partition?

A: A consumer always consumes messages from a particular partition sequentially.

------

Q: Why does Kafka not use fancy caching system to cache messages?

A: Kafka relies on file system page cache. Because page cache is by no doubt extremely effective and independent from Kafka application process, Kafka messages can stay alive in cache even when a broker process is restarted. It also immensely reduces load for Java GC.

------
Q: Why is broker stateless?

A: Broker does not maintain how much each consumer has consumed. This makes broker less complicated and more efficient. Broker removes messages from queue on time-basis.

------
Q: What's the correspondence among consumer groups, consumers and topics?

A: A conumer group consists of one or many consumers, which consume a set of topics. However each message can only be consumed by one consumer.

------

Q: What does "a partition is the smallest unit of parallelism" mean?

A: At a given time, all messages within a parition can only be consumed by a single consumer within each consumer group. If a topic's messages need more consumers, we simply overpartition the topic.

------
Q: What does Zookeeper do in Kafka?

A: 1) detecting the addition and removal of brokers and consumers. 2) triggering a rebalance process in each consumer when above events happen. 3) maintaining the current offset in each partition. 4) stores brokers' registry information, e.g. host and port, set of topics and corresponding parititon keys.

------
Q: What does rebalance process do?

A: Say consumer C in group G is deleted, Kafka will start a rebalance, it first remove C's registry on topics/partiitons, then for each topic of C, the system uniformly distributes its corresponding partitions to other consumers in group G which also subscribes this topic.

------
Q: Why does Kafka use at-least-once delivery instead of exactly-once delivery?

A: Exactly-once delivery requires two-phase commit protocol, it's simply less cost-effective. In addition, because there is no guarentee to avoid duplication, one must implement its own strategy.

------

These are simply extractions from Kafka paper, there are many new stuffs after that, esp. replication, firewall and more.

------
Appendix

1. Each topic has an ever-growing log, with a default 7-day retention policy.
2. Kafka uses zero-copy file-to-socket transfer.
3. New features

  - Compression
    - Producer will compress messages by blocks, and consumer is responsible for decompressing. This is useful when mirroring data across data centers.
    - Compression is time-consuming, so don't use it for real-time required scenario.

  - Replication
    - Strong durability and high availability.
    - Use primary-backup replication strategy, leader waits until writes finish on replicas before ack client. A f replicas tolerate f-1 failures.
      - Writes: write to leader -> write to log -> slave read from log -> update replica.
      - Reads: always read from leader.

    - Upon leader failure
      - Surviving replicas register themselves in Zookeeper.
      - First registered replica gets elected as leader.
      - Each replica register a listener in Zookeeper for leader change.
      - Leader waits until all other replicas catch up with leader's data.
      - Leader enables R/W.

  - Stream processing (WIP)
