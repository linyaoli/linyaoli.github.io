---
layout: post
title: Scalability Rules 
tags: scalability 
---

This post is a note/headline for reading book Scalability Rules, so in the future I can look back
to this rather than finding the book. If you find this interesting, go search [Scalability Rules](https://www.amazon.com/s/ref=nb_sb_noss_2?url=search-alias%3Daps&field-keywords=scalability+rules). Great book.

Before all of these below, please devote to the bible of scalable system [CAP](https://en.wikipedia.org/wiki/CAP_theorem).

---
* TOC
{:toc}

I. Reduce the equation
---------------------
1. Don't overengineer the solution. Complex systems result in costly effort and maintaince, decompose complex requiments into smaller systems, and make them sharable and reusable.
2. Design scale into solution. Design to 20x capacity, implemenet for 3x capacity, deploy for ~1.5x capacity. 
3. Simply the solution 3 times over(Pareto Principle). This falls into the Rule(1), but remember simplification is not necessarily considered as always good. 
4. Reduce DNS lookups. 
5. Reduce objects where possible. Frontend stuffs.
6. Use homogenous networks. Hardware.

II. Distribute the work
----------------------
![three axes of scale](https://ranjithabalaraman.files.wordpress.com/2014/10/scaledb.png)
7. Design to clone things(x-axis). Or horizontal scalability, essentially the duplication of services. This is the most important ideology in scalability I'd say. 
  - For services, ensure it is logically simple(use standardized interface e.g. RESTful/RPC) and easy to clone, use load balancer.  
  - For databases, normally used when Read >> Write. Master-Slave / Consistent Hashing etc. note [ACID](https://en.wikipedia.org/wiki/ACID)
8. Design to split different things(y-axis). Which I don't care much. 
9. Design to split similar things(z-axis). ..Either.

III. Design to scale out horizontally
----------------------
10. Design your solution to scale out, not just up.
  - Scaling out means the replication of services.
  - Scaling up means upgrade of computing resources.
  This is really just a duplicate from Rule(7). meh.
11. Use commodiity systems. Be cost-effective.
12. Scale out your data centers. This is more of a concern on availability and failover, so always prepare for multiple data centers. Models may vary.
13. Design to leverage the cloud. e.g store unimportant files in S3 (I will never save important data in S3).  

IV. Use the right tools
----------------------
14. Use databases appropriately.
  - Use RDBMS for cross-table lookup. e.g. MySQL.
  - Use NoSQL for simple R/W key-value queries. e.g. for great scalability, Cassandra; for grete [CRDT](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) support, Riak; for light weight, LevelDB. There are tons of options, the world of database is heaven. 

15. Don't abusively use firewalls, it makes network slow. Put firewall only in critical path.
16. It is never too less for logging. Use reliable and fast tool such as [Maxwell](https://github.com/zendesk/maxwell) for aggregation, and make it well rotated.

V. Don't duplicate your work
----------------------
17. Don't check your work. I'd say avoid duplicated data validation, act upon failures.
18. Reduce redirecting traffic. This only applies for HTML level, per se. Traffic redirection is key in server configuration and load balancing, especially for inter-service traffic. 
19. Alleviate temporal constraints.

VI. Use caching aggressively. Damn I love it.
----------------------
20. Leverage CDN.
21. Use `Expires` headers. HTML [headers](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields) offer more than you could ever imagine.
22. Cache Ajax calls. Use `Last-Modified`, `Cache-Control` and `Expires`.
23. Leverage page caches. [Reverse proxy](https://www.nginx.com/resources/glossary/reverse-proxy-server/) 
24. Utilize application caches. Cache whatever users mostly request, find the best balance between performance and cost.
25. Make use of object caches(aka in-memory cache).
26. Put object caches on their own tier. Though I regretly did this mutiple times, caching should never be embedded with crtical path. i.e. whenever caching fails, request should always be properly forwarded to database. Generally, a caching service should sit between api layer and service layer(which operates database commands). 

VII. Learn from mistakes
----------------------
1. Don't rely on QA, period.
2. Design for rollback-ready.

VIII. Database rules 
----------------------
1. Be aware of costly relationships. Cross-table queries are expensive.
2. Use the right type of database lock.
  - Implicit locks. Created automatically by system while performing a transaction.
  - Explicit locks. Locks defined by user during the interaction between user and db.
  - Row lock locks a row from update, read and write.
  - Page locks a row or a group of row.
  - Extent locks a group of pages.
  - Table lock.
  - Database lock.
3. Don't use multi-phase commits protocols. [2-PC protocol](https://en.wikipedia.org/wiki/Two-phase_commit_protocol) is mainly introduced to distributed process, but before the server returns ack,it hangs PENDING until the transaction is done.
4. Try not to use `SELECT FOR UPDATE`. `SELECT` is supposed to be fast, but `UPDATE` locks row and is slow, so it's drinking soda with a fine goblet. 
5. Don't `SELECT` everthing. Avoid `SELECT *`, and ask only for what you need.

IX. Design for fault tolerance and graceful failure
----------------------
36. Design using fault isolative swimlanes, basically be careful with any comm between swimlanes.
  type of splits:
  - Pod. Self-contained sets of functionality containing app servers and persistent storage. Essentially it is equal to a data center.
  - Cluster / Pool. A group of servers which have similar functionalities and serve similar customer groups. 
  - Shard. Horizontal partitions of databases or search engines.
  - Swimlane. A fault isolation domain.

  fault isolation principles
  - Share nothing. However, router, load balancer are acceptable. But database and servers should never be shared.
  - No sync calls between swimlanes.
  - Limit async calls between swimlanes.
  - Use timeouts and wire-on/off with async calls.

37. Never trust single points of failure. Maximize availability through multiple instances.
38. Avoid putting systems in series. 
    - ![circuit of series](http://www.physics247.com/physics-tutorial/images/seriescircuit.jpg) 

39. Ensure you can wire-on/off functions. For risky services, it should be able to be switched off during bad moment. Introducing [Zendesk Arturo](https://github.com/zendesk/arturo) .

X. Avoid or distribute state.
----------------------
40. Strive for statelessness. Use stateless implementation ([what is statelessness?](https://en.wikipedia.org/wiki/Stateless_protocol)).
41. Maintain sessions(cookies) in browser whenever possible. That is, avoid keeping those in servers.
42. Make use of a distributed cache for state. Use when cannot store session data in browser. This is because commonly session data or metadata are frequently requested.

XI. Asynchronous communication and message buses.
----------------------
43. Communicate asynchronously as much as possible

