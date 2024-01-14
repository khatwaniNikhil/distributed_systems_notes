# References
https://martinfowler.com/articles/patterns-of-distributed-systems

## Challenge: 
Guarantee Data Durability during server crash when every change first persist to disk is expensive
## Solution: 
WAL(append only write ahead log) written to disk
Examples: 
	1) Kafka storage impl
	2) almost all db(sql and nosql) use WAL for data durability
	3) log implementation in all Consensus algorithms like Zookeeper and RAFT

## Challenge: 
Distributed system's needs need to have replication of actions across multiple nodes to have consistency/avoid data loss in case any node crashes. How to decide on quorom without leading to slowness as well as maintain data consistency?
a) inconsistencies in state/data across nodes within the distributed system 
b) split brain leading to two sets of servers, each considering another set to have failed, and therefore continuing to serve different sets of clients and lead to data conflicts
## Solution: 
Majority Quorom
For a cluster of n nodes, the quorum is n / 2 + 1. To tolerate f failures we need a cluster size of 2f + 1.
https://martinfowler.com/articles/patterns-of-distributed-systems/majority-quorum.html

## Challenge3: 
WAL can be used to recover state post server restart but no availability during crash. Even with leader and followers based replication of WAL with majority quorom:
1. The leader can fail before sending its log entries to any followers.
2. The leader can fail after sending log entries to some followers, but before sending it to the majority of followers.
How each follower come to know what part of the log is safe to be made available to the clients

## Solution
A "High-Water Mark" is used to track the entry in the write ahead log that is known to have successfully replicated to a quorum of followers. All the entries upto the high-water mark are made visible to the clients.  leader also propagates the high-water mark to the followers. So in case the leader fails and one of the followers becomes the new leader, there are no inconsistencies in what a client sees. 

## Scenario: 
Process Pauses leading to outdated states
Process Pauses(like gc pauses) leading a leader disconnected from the followers and out of date updates from older leaders. 

## solution:
1. We need to order the events/messages across the distributed system. Out of date leader can try to overwrite latest updates by other servers. we can not use use wall clock. Wall clocks are driven by measuring oscillations of quartz crystal is error prone and also synchronized by a service called NTP is also impacted by network delay. 
2. Use Generation clock. Maintain a monotonically increasing number indicating the generation of the server. Every time a new leader election happens, it should be marked by increasing the generation. Make it non volatile by making it part of WAL. in Java, System.currentTimeMillis() is based on the wall-clock, so subject to variations and even negative durations. In comparison, System.nanoTime() is monotonic so we should have used this one.
3. even monotonic clocks of two different servers are not synchronized and cannot be compared






