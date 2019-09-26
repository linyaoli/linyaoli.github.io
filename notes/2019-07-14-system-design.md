## System Design
### Understanding the problem
Clarify the use case, interface and data of the system, double check parts that remain unknown. Read this https://linyaoli.github.io/#/notes/2017-04-06-scalability-rules to understand what scalable system should have.

### Devising a plan
There are a few cornerstones that need to be taken into consideration:

- patterns of the traffic: e.g. request rate, R/W-heavy, I/O rate, storage volume etc. <b>Make calculations!</b>
- scale of the system: e.g. geologically is it distributed across different locations.
- concurrency: i.e. is it possible that high concurrent requests hit on a single data entry.
- scalability, reliability, availability, performance, serviceability/manageability and cost.

With these, break down the problem into components.

### Carrying out the plan
Major components of the system that must be included:
1. business logic and its abstraction
2. data storage and caching
3. networking and load balancing
4. interface
5. real-world performance

Some are optional but great to have:

1. service-oriented / microservice architecture, avoid monolithic applications.
2. service-to-service communication method (RESTful, message bus and etc.)
3. service discovery
4. scalable infrastructure
5. monitoring and logging
6. database sharding
7. proxies
8. redundancy
9. SQL/NoSQL


### Looking back
<b>There are no perfect systems</b>, every system design is about making tradeoffs.

Now look back and try to find bottlenecks and drawbacks of the system:
1. There are a few principles you can use here (Occam's razer, long-tail, 80/20, CAP)
2. Is there any component of the system can be optimized?
3. Is there any component which is single point of failure? can it recover from failure?
4. Can the system easily scale horizontally?
5. Is the system easy to maintain?


## Example Problems
The following are some system design problem examples and key points to discuss:

### URL Shortener
- Components: url generator and disk/cache storage.
- The system has to be highly available (> 99.999%) and low-latency.
- It is extremely read-heavy, this is a typical 80/20 issue. Number of requests are massive.
- Relatively small storage and cache are needed.


### Instagram
- Components: image storage, image processing service, uploading/downloading service, CDN, search service.
- Requests are usually with multipart-file images, therefore networking bandwidth must meet the traffic.
- Images require massive blob storage. Hot images need to be cached.
- Downloading requests are more than uploading.
- A search service is needed to find images.
- Images need to be consistent and immutable.
- side services for downloading/uploading, compression, security and transformation.

### Google Docs
- Components: file storage, online editing/formatting, collaboration, access control.
- realtime collaboration requires long-live connection, i.e. websocket.
- major problem to be addressed: concurrent editing. How can we ensure consistency while every user is editing the same document?
- Every change user makes must be shown in other users' browser as quick as possible, therefore low latency is a must-have.
- fun things to think about: while updating the file, the browser only receives the updated content rather than downloading the whole file. Refer to <b>Operational Transformation</b>.

### Twttier
### Dropbox
### Youtube
### API Rate Limiter
### Search
### Web crawler
### Facebook newsfeed
### Yelp
### Uber Backend
### Ticketmaster
