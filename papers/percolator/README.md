# Large-scale Incremental Processing Using Distributed Transactions and Notifications

https://ai.google/research/pubs/pub36726

## Notes

* Percolator: A system for incrementally processing updates to a large data set.
* Problem pattern: Data processing tasks that transform a large repository of data via small, independent mutations.
* Databases do not meet the storage or throughput requirements of these tasks. Google's indexing system stores tens of petabytes of data and processes billions of updates per day on thousands of machines.
* MapReduce and other batch-processing systems cannot process small updates individually as they rely on creating large batches for efficiency.
* By replacing a batch-based indexing system with an indexing system based on incremental processing using Percolator, we process the same number of documents per day, while reducing the average age of documents by 50%.

* Programmers of an incremental system need to keep track of the state of the incremental computation. To assist in this task, Percolator provides observers, which are pieces of code that are invoked by the system whenever a user-specified column changes.
* Percolator applications are structured as a series of **observers**; each observer completes a task and creates more work for "downstream" observers by writing to the table. An external process triggers the first observer int he chain by writing initial data into the table.
* Evaluating whether Percolator is a good fit:
    * Comptuations where the result can't be broken down into small updates (e.g. sorting) are better handled by MapReduce
    * Computation should have strong consistency requirements; otherwise, Bigtable is sufficient
    * The comptuation should be very large in some dimension (total data size, CPU required for transformation, etc).
    * Smaller computations not suited to MapReduce or Bigtable can be handled by traditional DBMS.
* The primary application of Percolator at Google is preparing web pages for inclusion in the live web search index. By converting the indexing system to an incremental system, we are able to process individual documents as they are crawled, reducing the average document processing latency by a factor of 100, and the average age of a document appearing in a search result dropped by nearly 50%.

* Percolator provides two main abstractions: ACID transactions over a random-access repository, and observers, a way to organize an incremental computation.
* A Percolator system consists of three binaries that run on every machine in the cluster:
    * a worker
    * a Bigtable tablet server
    * a GFS chunkserver
* All observers are linked into the worker, which scans Bigtable for changed columns ("notifications") and invokes the corresponding observers as a function call in the worker process.
* The observers perform transactions by sending read/write RPCs to Bigtable tablet servers, which in turn send read/write RPCs to GFS chunkservers.
* The system also depends on two small services:
    * the timestamp oracle, which provides strictly increasing timestamps: a property required for correct operation of the snapshot isolation protocol
    * the lightweight lock service, to make the search for dirty notifications more efficient
* Percolator has requirement to run at massive scales, but can accept a lack of requirement for extremely low latency. RElaxed latency requirements let us take a lazy approach to cleaning up locks left behind by transactions running on failed machines. This delays transaction commits by tens of seconds; which would not be acceptable in a DBMS running OLTP tasks, but is tolerable in an incremental processing system building an index of the web.
* Percolator has no central location for transaction management; in particular, it lacks a global deadlock detector. This increases the latency of conflicting transactions but allows the system to scale to thousands of machines.

* Percolator is built on top of the Bigtable distributed storage system. The decision to build on Bigtable defined the overall shape of Percoaltor, which maintains the gist of Bigtable's interface: data is organized into Bigtable rows and columns, with Percolator metadata stored alongside in special columns. The challenge is providing features that Bigtable does not: multirow transactions and the observer framework.
* Bigtable presents a multi-dimensional sorted map to users. Keys are (row, column, timestamp) tuples. Bigtable provides lookup and update operations on each row, and Bigtable row transactions enable atomic read-modify-write operations on individual rows. Bigtable handles petabytes of data and runs reliably on large numbers of (unreliable) machines.
* Bigtable consists of a collection of tablet servers, each of which is responsible for serving several tablets (contiguous regions of the key space). A master coordinates the operation of tablet servers by, for example, directing them to load or unload tablets.
* A tablet is stored as a collection of read-only files in the Google SSTable format. Bigtable allows users to control the performance characteristics of the table by grouping a set of columns into a locality group.
* Bigtable relies on GFS to preserve data in the event of disk loss.

## Questions

* What is SSTable format? Columnar binary?


CONTINUE AT "2.2 TRANSACTIONS"

