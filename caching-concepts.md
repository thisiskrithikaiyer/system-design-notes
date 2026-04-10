# Caching

When user requests data frequently, store the values in memory (redis/memcache) instead of querying through the database. Saves time.

## Scaling Issue

When you scale your server, with multiple nodes and a load balancer. Now, the load balancer needs to route the traffic to specific servers. 
Now, each server has it's own cache. This will lead in lot of cache miss, when that server is down and another server takes on the same request. 

To solve, this use Distributed Cache/ Global Cache

## Global Cache

This is used when your system is simple, and you want to store mostly stable data in the cache that is accessable by all servers. The tradeoff of 
this is that you have single point of failure. You can workaround, using master slave stratergies. 

## Distributed Cache

This is used when you want high availability, cache storing is complex, want to scale horizontally (add more cache nodes as the system grows).
The tradeoff, is complex system. It uses consistent hashing, so tradeoff here is when one node is lost data is lost.

## Redis vs Memcache
1. Redis is used when you have complex data structures which needs adding queue, lists, sets and other data structure. 
2. Memcache is simple only store key value pair, this is used when your app needs fast simple caching. 

## CDN

This is used when you want to retrive data faster over network so you store geographically nearer to the request. Large files, videos, images use this. 

A CDN is a distributed edge cache for web content, so yes, it is caching, but not all caching is a CDN

## Cache invalidations

1. Write through - used when consistency is important over availabilty. You updata database and then cache together.
2. Write Around - used when inconsistency is okay, data is written only in the database. The cache keeps reducing when outdated. Recently written data will see cache miss high latency.
3. Write Back - data is only written in cache, you write to DB later. Cons. could cause incosistency when write fails. Pros. Low latency.

### Cache eviction policy

1. LIFO
2. FIFO
3. LRU
4. MRU
5. LFU
