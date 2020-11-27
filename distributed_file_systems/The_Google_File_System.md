# The Google File System


[Link](https://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf)



## Contribution
* GFS illustrates how to handle files in the environment of large-scale data processing workloads on commodity hardware.
* It introduced atomic record appends to deal with large files, differentiate from previous DFS.
* It derives the HDFS in futher.


## Assumption
* component failures are the norm
* files are huge (Multi-GB)
* large streaming (sequential) read and small random reads
* multi clients that concurrently append to the same file
* high sustained bandwidth more important than low latency

* single datacenter
* internal usage in Google
* performance over consistency (if some data is wrong order in search engine, who cares?)
* Let application decide how to handle wrong data

## Implementation

A GFS cluster consists of a single master and multiple chunkservers and is accessed by multiple clients.
 File system control(via master) and data transfer (directly between chunck servers and client) is separated.

### single master model

* Less master involvement: the involvement in read and write file is minimized
* Clients never read and write file data through the master
* Clients ask master which chunkservers it should contact and cache it, then TCP to chunkservers

### metadata in master: all in memory

* file and chunk namespaces (also kept in operation log)
* mapping from files to chunks (also kept in operation log)
* locations of each chunk's replicas (not persistent, poll from each chunkservers when startup and whenever a chunkserver joins the cluster)

#### Operation log

* compatct B-Tree like form
* file system states can be recovered by using operation log to replay
* create checkpoint to backup than delete old operation log

### large chunk size: 64MB

* pro: redude interaction frequency, reduce memory consumption by metadata
* contra: single hotspot (fix: higher replication factor, maybe consider allow client to read data from other client)

### Consistency Model, atomic record append

* a record append atomically at least once
* client submit only the data to append, offset is then returned.
* stale replicas will be garbage collected ASAP
* since clemts cache chunk location, it may be stale replicas, but it will usally a premature end of chunk rather than outdated data since most file are append-only
* regular handshakes between master and all chunkservers to dect data corruption by checksumming.
* does not guarantee all replicas are bytewise identical
* guarantee that data is written at least once as an atomic unit

## Performance
High aggregate throuput to many concurrent readers and wirters.

### Environment
GFS: 1 master, 2 master replicas, 16 chunkservers, 16 clients, all the machines are dual 1.4 GHZ PIII, 2GB RAM, 80GB 5400 rpm disks * 2, 100 Mbps full-duplex Ethernet. All 19 GFS server machines are connected to one switch, all 16 client machines to the other switch. The two switches connected with a 1 Gbps link.

### Read
* 125 MB/s is the limit of the switch
* One client read. 10 MB/s
* All clients randomly read simutaneously 4mb area. Aggregate at 94 MB/s for 16 readers (6MB/s per client).

### Write
* 67 MB/s is the limit of the switch, need to write to 3 of chunkservers(12.5 MB/s each).
* one client write, 6.3 MB/s
* All clients randomly write simutaneously to distinct files. Aggreagte at 35 MB/s (2.2 MB/s per client)

### Append
* 6.0 MB/s for one client
* 4.8 MB/s when 16 clients append simultaneously to a single file (congestion and variances in network transfer rate seen by different clients)

## Recovery
kill two chunkservers each with 16000 chunks and 660 GB data => 266 chunks only have single replica => restore to at least 2x replica within 2 minutes

