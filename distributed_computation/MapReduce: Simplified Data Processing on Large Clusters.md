# MapReduce: Simplified Data Processing on Large Clusters
[Link](https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf)

MapReduce is a programming model and a associated implementation for processing and generating large data sets.
Programs written in this functional style are automatically parallized and executed on a large cluster of commodity machines.

## Assumption
* input data is large
* computation have to be distributed across hundreds or thousandes of machines in order to finish in a reasonable amount of time
* large cluster of commodity computers

## Contribution
* Abstract the complexity of parallize the computation, distribute the data,handle the failures and load balancing.
* Large variety of problems are easily expressible as MapReduce problem
* The implementation scales well to large cluster of machines (thousands)

## Findings
* restricting programming model makes it easy to parallize, and make distribute computaions fault tolerant
* network bandwidth is a scare resource. 


## Conception
### Map
A map function that processes a key/value pair to generate a set of itermediate key/value pairs

### Reduce
A reduce function that merge all intermediate values associated with the same intermediate key.

## Implementation
Google's implementation is based on a large cluster of commodity computers, with commodity networking hardware and storage.

* Input files =>  splits
* master picks idles workers and assign each one a map task or reduce task
* workers who assigned map tasks buffer the intermediate pairs produce by map function in memory, which periodically written to local disk, passed back to the master.
* master forwards these location to reduce worker, which use remote procedure calls to read the buffered data from the local disks of the map workers. When it reads all such intermediate data, it sort the data. It iterate over the sorted data and fore each unique intermediate key, run reduce function. The output of reduce is appened to a final output file.
* after sucessful completion, the output is available in output files(one per reduce task), which often pass to another MapReduce call, or directly used by another distributed application. (Therefore no need to combine these output files into one file)

## Failure Tolerance
### Worker Failure
* Completed map tasks are re-executed on a failure because their output is stored on the local disk of the failed machine and is therefore inaccessible, completed reduce tasks do not need to be re-executed since their output is tored in a global file system.
* When a map worker failed, all reduce workers are notified to read from another available location when they do not already touch the failed worker.
### Master Failure
Make periodic checkpoints if multiple masters. For single master, it absort the MapReduce.


## Refinements
* **Partitioning Function**: user can specify partitioning function, default is hash(key) mod R.
* **Combiner Function**: user coudld specify combiner to conduct partial merging of data before go to reduce.
* **Skip Bad Records**: sequence numbers is stored before each process start, if it failed it will send "last gasp" UDP to master, which enable master to determine sequence that failed many times as "bad records" to skip
* **Side Effects**: additional outputs will be deterministic
* **Status Information**: master run an internal HTTP server to provide status informations
* **Counters**: user can also define counters and get count result from user code
* **Enable Backup**: backup, when a MapReduce is close to complete to avoid "straggler" issue, avoid long-tail before the whole task is finished. Disable backup takes 44% longer to finish

## Practice
M(Map tasks) and R(R tasks) should be much larger than the workers. Google: M = 200,000, R = 5000, 2000 worker machines.

