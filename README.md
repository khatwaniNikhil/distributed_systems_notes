# References
https://martinfowler.com/articles/patterns-of-distributed-systems

# Scenario: Process crash - How to handle data durability considering applying every change via disk IO is expensive in terms of application throughput
## Solution: Extra step of capture change log via append only writeahead log to disk(WAL)
Examples: 
	1) Kafka storage impl
	2) almost all db(sql and nosql) use WAL for data durability
	3) log implementation in all Consensus algorithms like Zookeeper and RAFT

# Scenario: Network delays leading to 
a) inconsistencies in state/data across nodes within the distributed system 
b) split brain leading to two sets of servers, each considering another set to have failed, and therefore continuing to serve different sets of clients and lead to data conflicts

## Solution: 
1. Regular heartbeat messages,  If a heartbeat is missed, the server sending the heartbeat is considered crashed. cluster as a group can move ahead considering the server to be failing.
2. Quorum makes sure that we have enough copies of data to survive some server failures. Quoram based majority consensus is required to proceess any client request to avoid any conflicts. In general, if we want to tolerate f failures we need a cluster size of 2f + 1. 
3. For strong data consistency/latest data guarantess - only send values to clients which are guaranteed to be available on all the servers using "Leader and Followers". One of the servers is elected a leader and the other servers act as followers. leader controls and coordinates the replication on the followers. The leader now needs to decide, which changes should be made visible to the clients.  A "High-Water Mark" is used to track the entry in the write ahead log that is known to have successfully replicated to a quorum of followers. All the entries upto the high-water mark are made visible to the clients.  leader also propagates the high-water mark to the followers. So in case the leader fails and one of the followers becomes the new leader, there are no inconsistencies in what a client sees. 

# Scenario: Process Pauses leading to outdated states
Process Pauses(like gc pauses) leading a leader disconnected from the followers and out of date updates from older leaders. 

## solution:
1. We need to order the events/messages across the distributed system. Out of date leader can try to overwrite latest updates by other servers. we can not use use wall clock. Wall clocks are driven by measuring oscillations of quartz crystal is error prone and also synchronized by a service called NTP is also impacted by network delay. 
2. Use Generation clock. Maintain a monotonically increasing number indicating the generation of the server. Every time a new leader election happens, it should be marked by increasing the generation. Make it non volatile by making it part of WAL. in Java, System.currentTimeMillis() is based on the wall-clock, so subject to variations and even negative durations. In comparison, System.nanoTime() is monotonic so we should have used this one.
3. even monotonic clocks of two different servers are not synchronized and cannot be compared






