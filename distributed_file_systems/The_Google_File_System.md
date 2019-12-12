# The Google File System

This paper introduced atomic record appends to deal with large files, derives the HDFS in futher.

[link](https://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf)

## Keywords

* atomic append

## Different Situation than previous DFS

* component failures are the norm
* files are hugh (Multi-GB)
* mutated by appending than overwriting

## Assumption

* large streaming read and small random reads
* multi clients that concurrently append to the same file
* high sustained bandwidth more important than low latency

## Design

A GFS cluster consists of a single master and multiple chunkservers and is accessed by multiple clients

### single master model

* the involvement in read and write file is minimized
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

## Consistency Model, atomic record append

* a record append atomically at least once
* client submit only the data to append, offset is then returned.
* stale replicas will be garbage collected ASAP
* since clemts cache chunk location, it may be stale replicas, but it will usally a premature end of chunk rather than outdated data since most file are append-only
* regular handshakes between master and all chunkservers to dect data corruption by checksumming.