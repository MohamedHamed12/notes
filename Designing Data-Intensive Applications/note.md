## sources
1. [github](https://github.com/keyvanakbary/learning-notes/blob/master/books/designing-data-intensive-applications.md)
##  **Reliable , Scalable and Maintainable**
1. **Reliability**: This refers to the ability of the system to consistently perform its intended functions correctly, even under adverse conditions or in the presence of faults. Key aspects of reliability include:
    
    - **Functionality**: The application should perform the tasks expected by the user.
    - **Fault Tolerance**: The system should be resilient to failures or faults in individual components, ensuring that the overall service remains available.
    - **Performance**: The application should maintain good performance levels, even during high loads or unexpected events.
    - **User Error Handling**: The system should be designed to handle user errors gracefully, providing feedback and recovering from errors when possible.
    - **Abuse Prevention**: Measures should be in place to prevent abuse or malicious actions from impacting the system's operation.
2. **Scalability**: Scalability refers to the system's ability to handle increasing workloads or growing amounts of data without sacrificing performance or reliability. Key aspects of scalability include:
    
    - **Vertical Scalability**: Adding more resources (e.g., increasing CPU, memory) to individual components to handle increased loads.
    - **Horizontal Scalability**: Distributing the workload across multiple instances or nodes, allowing the system to scale out by adding more servers or instances.
    - **Elasticity**: The system should be able to dynamically adapt to changing demand, automatically scaling resources up or down as needed.
    - **Load Balancing**: Distributing incoming requests evenly across multiple servers to avoid overloading any single component.
3. **Maintainability**: Maintainability refers to the ease with which a system can be modified, updated, or repaired over time. Key aspects of maintainability include:
    
    - **Modularity**: The system should be modular, with well-defined components that can be easily understood and modified independently.
    - **Documentation**: Clear and comprehensive documentation should be provided to aid developers in understanding the system's architecture, design principles, and codebase.
    - **Testing**: Robust testing practices, including unit tests, integration tests, and system tests, should be implemented to ensure that changes do not introduce regressions or unforeseen issues.
    - **Monitoring and Logging**: Effective monitoring and logging mechanisms should be in place to track system performance, detect errors, and facilitate troubleshooting.
    - **Version Control**: The use of version control systems allows for tracking changes to the codebase and facilitates collaboration among developers.
## **Data models and query language**
### data models
   of database systems  relational  and  document model.

1. **Layered Data Models:**
    
    - Most applications are constructed by layering one data model on top of another.
    - Each layer hides the complexities of the layers below, providing a cleaner data model.
    - Abstractions in these layers enable different groups of people to work effectively.
2. **Relational Model vs. Document Model:**
    
    - Relational databases have roots in business data processing, transaction processing, and batch processing.
    - The relational model aimed to hide implementation details behind a cleaner interface.
    - NoSQL databases emerged with forces like greater scalability, preference for open source software, specialized query optimizations, and a desire for a more dynamic data model.
    - SQL models may face an impedance mismatch, but JSON models reduce this mismatch and offer better data locality.
3. **JSON Model Advantages:**
    
    - JSON representation provides better locality compared to multi-table SQL schemas.
    - Relevant information is stored in one place, reducing the need for multiple queries.
    - Document databases don't require joins for one-to-many tree structures.
    - example 
		```json
		[
		  {
		    "id": 1,
		    "title": "Post 1",
		    "comments": [
		      {"id": 1, "content": "Comment 1"},
		      {"id": 2, "content": "Comment 2"}
		    ]
		  },
		  {
		    "id": 2,
		    "title": "Post 2",
		    "comments": [
		      {"id": 3, "content": "Comment 3"}
		    ]
		  }
		]
		
		```
1. **Challenges with Joins:**
    
    - In relational databases, joins are easy, but in document databases, support for joins can be weak.
    - Emulating joins in application code through multiple queries might be necessary if the database doesn't support joins.
5. **Historical Database Models:**
    
    - IBM's Information Management System (IMS) was a popular database in the 1970s.
    - IMS used a hierarchical model, similar to document databases, working well for one-to-many relationships but struggled with many-to-many relationships and lacked support for joins.
6. **Network Model:**
    
    - The Conference on Data Systems Languages (CODASYL) standardized the network model as a generalization of the hierarchical model.
    - In the network model, records could have multiple parents, and links between records were akin to pointers in a programming language.
    - Accessing a record required following a path from a root record called an access path.
7. **Challenges with CODASYL:**
    
    - Queries in CODASYL involved moving a cursor through the database by iterating over a list of records.
    - Difficulty arose if a path to the desired data was not available, making changes to an application's data model challenging.
8. **The relational model**
	
	By contrast, the relational model was a way to lay out all the data in the open" a relation (table) is simply a collection of tuples (rows), and that's it.
	
	The query optimiser automatically decides which parts of the query to execute in which order, and which indexes to use (the access path).
	
	The relational model thus made it much easier to add new features to applications.
9. **relation or document**
	The main arguments in favour of the document data model are schema flexibility, better performance due to locality, and sometimes closer data structures to the ones used by the applications. The relation model counters by providing better support for joins, and many-to-one and many-to-many relationships.
	
	If the data in your application has a document-like structure, then it's probably a good idea to use a document model. The relational technique of _shredding_ can lead unnecessary complicated application code.
	
	The poor support for joins in document databases may or may not be a problem.
	
	If you application does use many-to-many relationships, the document model becomes less appealing. Joins can be emulated in application code by making multiple requests. Using the document model can lead to significantly more complex application code and worse performance.
10. **Schema flexibility**
	
	Most document databases do not enforce any schema on the data in documents. Arbitrary keys and values can be added to a document, when reading, **clients have no guarantees as to what fields the documents may contain**
11. **ata locality for queries**

	If your application often needs to access the entire document, there is a performance advantage to this _storage locality_.
	
	The database typically needs to load the entire document, even if you access only a small portion of it. On updates, the entire document usually needs to be rewritten, it is recommended that you keep documents fairly small.
### Query languages for data
1. _declarative_ vs  _imperative
	SQL is a _declarative_ query language. In an _imperative language_, you tell the computer to perform certain operations in order.

	In a declarative query language you just specify the pattern of the data you want, but not _how_ to achieve that goal.
	
	A declarative query language hides implementation details of the database engine, making it possible for the database system to introduce performance improvements without requiring any changes to queries.
	
	Declarative languages often lend themselves to parallel execution while imperative code is very hard to parallelise across multiple cores because it specifies instructions that must be performed in a particular order. Declarative languages specify only the pattern of the results, not the algorithm that is used to determine results.
	
	**Declarative queries on the web**

	In a web browser, using declarative CSS styling is much better than manipulating styles imperatively in JavaScript. Declarative languages like SQL turned out to be much better than imperative query APIs.
2. **MapReduce querying**

	_MapReduce_ is a programming model for processing large amounts of data in bulk across many machines, popularized by Google.

	Mongo offers a MapReduce solution.

	```js
	db.observations.mapReduce(
	    function map() { 2
	        var year  = this.observationTimestamp.getFullYear();
	        var month = this.observationTimestamp.getMonth() + 1;
	        emit(year + "-" + month, this.numAnimals); 3
	    },
	    function reduce(key, values) { 4
	        return Array.sum(values); 5
	    },
	    {
	        query: { family: "Sharks" }, 1
	        out: "monthlySharkReport" 6
	    }
	);
	```

	The `map` and `reduce` functions must be _pure_ functions, they cannot perform additional database queries and they must not have any side effects. These restrictions allow the database to run the functions anywhere, in any order, and rerun them on failure.
	
	A usability problem with MapReduce is that you have to write two carefully coordinated functions. A declarative language offers more opportunities for a query optimiser to improve the performance of a query. For there reasons, MongoDB 2.2 added support for a declarative query language called _aggregation pipeline_

	```js
	db.observations.aggregate([
	    { $match: { family: "Sharks" } },
	    { $group: {
	        _id: {
	            year:  { $year:  "$observationTimestamp" },
	            month: { $month: "$observationTimestamp" }
	        },
	        totalAnimals: { $sum: "$numAnimals" }
	    } }
	]);
	```
3. Graph-like data models
	
	If many-to-many relationships are very common in your application, it becomes more natural to start modelling your data as a graph.
	
	A graph consists of _vertices_ (_nodes_ or _entities_) and _edges_ (_relationships_ or _arcs_).
	
	Well-known algorithms can operate on these graphs, like the shortest path between two points, or popularity of a web page.
	
	There are several ways of structuring and querying the data. The _property graph_ model (implemented by ==Neo4j==, ==Titan==, and ==Infinite Graph==) and the _triple-store_ model (implemented by Datomic, AllegroGraph, and others). There are also three declarative query languages for graphs: ==Cypher, SPARQL, and Datalog.==
	
	**Property graphs**
	
	Each vertex consists of:
	
	- Unique identifier
	- Outgoing edges
	- Incoming edges
	- Collection of properties (key-value pairs)
	
	Each edge consists of:
	
	- Unique identifier
	- Vertex at which the edge starts (_tail vertex_)
	- Vertex at which the edge ends (_head vertex_)
	- Label to describe the kind of relationship between the two vertices
	- A collection of properties (key-value pairs)
	
	Graphs provide a great deal of flexibility for data modelling. Graphs are good for evolvability.
	 _Cypher_ is a declarative language for property graphs created by Neo4j
	- Graph queries in SQL. In a relational database, you usually know in advance which joins you need in your query. In a graph query, the number if joins is not fixed in advance. In Cypher `:WITHIN*0...` expresses "follow a `WITHIN` edge, zero or more times" (like the `*` operator in a regular expression). This idea of variable-length traversal paths in a query can be expressed using something called _recursive common table expressions_ (the `WITH RECURSIVE` syntax).
	4.  **Triple-Store Concept:**
    - In a triple-store, all information is stored as simple three-part statements known as triples: subject, predicate, object. For example, "Jim likes bananas" could be represented as the triple (Jim, likes, bananas).
    - Triples are equivalent to vertices in a graph, forming the basic building blocks of the data model.



2. **SPARQL and RDF:**
    
    - SPARQL is a query language designed for querying triple-stores that use the RDF (Resource Description Framework) data model.
    - RDF is a model for representing information about resources in the form of subject-predicate-object triples.
3. **Query Example:**
    
    - SPARQL queries allow you to retrieve and manipulate data stored in triple-stores. For instance, you could use SPARQL to retrieve all the fruits Jim likes.

4. **Datalog as a Foundation:**
    
    - Datalog serves as the foundational model that later query languages, including SPARQL, build upon.
    - Datalog's model is similar to the triple-store model but is more generalized. Instead of writing triples, it uses a predicate notation: `predicate(subject, object)`.
5. **Rules in Datalog:**
    
    - Datalog allows the definition of rules that inform the database about new predicates. Rules can refer to other rules, similar to how functions can call other functions or even recursively call themselves.
6. **Combining and Reusing Rules:**
    
    - Rules in Datalog can be combined and reused in different queries. This feature enhances flexibility and scalability, especially in handling complex data structures.
7. **Adaptability for Complex Data:**
    
    - Datalog may be less convenient for simple, one-off queries compared to direct triple representations. However, its strength lies in its ability to cope better with complex data structures and relationships.

## **Storage and Retrieval in Databases**

1. **Core Functions:**
    
    - Databases have two primary functions: storing data and retrieving data.
2. **Data Structures:**
    
    - Many databases use a log, which is an append-only data file.
    - A log is a sequence of records and can raise issues like concurrency control, disk space reclamation, and error handling.

### Indexing for Efficient Retrieval:

1. **Index as a Data Structure:**
    
    - To efficiently find the value for a particular key, databases use an additional data structure called an index.
    - An index is derived from the primary data and helps speed up read queries.
2. **Trade-off between Read and Write Performance:**
    
    - Well-chosen indexes can enhance read query performance, but every index slows down write operations.
    - Databases do not index everything by default, and manual index selection is often required based on typical query patterns.

### Hash Indexes:

1. **Key-Value Stores and Hash Indexes:**
    
    - Key-value stores are similar to the dictionary type and often use hash maps or hash tables.
    - In-memory hash maps map keys to byte offsets in the data file for efficient retrieval.
2. **Bitcask Storage Engine:**
    
    - Bitcask, the default storage engine in Riak, uses in-memory hash maps where each key is mapped to a byte offset in the data file.
    - It is well-suited for scenarios where values for keys are frequently updated.

### Managing Disk Space and Compaction:

1. **Segmenting the Log:**
    
    - To avoid running out of disk space, the log is broken into segments of a certain size.
    - When a segment reaches a certain size, subsequent writes go to a new segment file.
2. **Compaction:**
    
    - Compaction involves discarding duplicate keys in the log and keeping only the most recent update for each key.
    - Merging segments and compaction are done in the background to optimize disk space usage.

### Implementation Considerations:

1. **File Format:**
    
    - Binary format is simpler to use.
2. **Deleting Records:**
    
    - Deletion is managed by appending a special deletion record (tombstone) to the data file, indicating that the merging process should discard previous values.
3. **Crash Recovery:**
    
    - In-memory hash maps are lost on restart, but recovery is sped up by storing a snapshot of each segment hash map on disk.
4. **Partially Written Records:**
    
    - Checksums are included to detect and ignore corrupted parts of the log.
5. **Concurrency Control:**
    
    - A single writer thread appends writes to the log in a strictly sequential order.
    - Segments are immutable, allowing concurrent reads by multiple threads.

### Advantages of Append-Only Design:

1. **Sequential Write Operations:**
    
    - Sequential writes (appending and segment merging) are faster than random writes, especially on spinning-disk storage.
2. **Simplicity in Concurrency and Crash Recovery:**
    
    - Concurrency control and crash recovery are simplified.
3. **Avoiding Fragmentation:**
    
    - Merging old segments prevents files from getting fragmented over time.

### Limitations of Hash Table:

1. **Memory Dependency:**
    
    - The hash table must fit in memory, making on-disk hash maps challenging to perform well.
2. **Inefficiency in Range Queries:**
    
    - Range queries are not efficient with hash tables.
### SSTables (Sorted String Tables):

1. **Sorting Requirement:**
    
    - SSTables store key-value pairs, and the sequence of these pairs is required to be sorted by the key.
2. **Advantages Over Hash Indexes:**
    
    - SSTables offer advantages over log segments with hash indexes.
    - Merging segments is efficient using algorithms like mergesort.
    - SSTables eliminate the need to keep an index of all keys in memory.
3. **Merging and Compaction:**
    
    - Merging segments is simple, and compaction ensures that each key appears only once.
    - When multiple segments contain the same key, the value from the most recent segment is retained, discarding values in older segments.
4. **In-Memory Index:**
    
    - An in-memory balanced tree structure, called a memtable, is used for writes.
    - When the memtable exceeds a certain size threshold, it is written out to disk as an SSTable file.
    - Read requests involve searching the memtable, the most recent on-disk segment, and older segments if needed.
5. **Crash Recovery:**
    
    - A separate log on disk is maintained to which every write is immediately appended.
    - In the event of a database crash, the log is used to restore the memtable.
6. **LSM Structure Engines:**
    
    - Storage engines following the merging and compacting sorted files principle are referred to as LSM structure engines or LSM-trees.
    - Lucene, used by Elasticsearch and Solr, employs a similar method for storing its term dictionary.

### LSM-Tree Algorithm:

1. **Write Process:**
    
    - When a write occurs, it is added to an in-memory balanced tree structure (memtable).
    - Once the memtable exceeds a threshold, it is written out to disk as an SSTable file, and new writes continue to a new memtable.
2. **Read Process:**
    
    - Read requests search the memtable, followed by on-disk segments (recent to older).
    - Periodic merging and compaction are performed in the background to discard overwritten and deleted values.
3. **Crash Handling:**
    
    - In the event of a crash, the separate log on disk helps restore the memtable.
4. **Optimizations:**
    
    - LSM-tree algorithms may be slow when looking up keys that don't exist. Bloom filters are often used to optimize this process.

### Compaction Strategies:

1. **Size-Tiered and Leveled Compaction:**
    - There are different strategies for determining the order and timing of how SSTables are compacted and merged.
    - Size-tiered compaction involves merging newer and smaller SSTables into older and larger ones.
    - Leveled compaction splits the key range into smaller SSTables and organizes older data into separate "levels," optimizing disk space usage.
### B-trees

	![[Pasted image 20240128071348.png]]
1. **Key-Value Pair Sorting**: B-trees keep key-value pairs sorted by key, facilitating efficient lookups and range queries.
    
2. **Page Structure**: The database is divided into fixed-size blocks or pages, usually around 4KB in size. One of these pages is designated as the root, serving as the starting point for accessing data.
    
3. **Hierarchy**: Each page contains several keys and references to child pages. This hierarchical structure allows for efficient navigation through the data.
    
4. **Updating Values**: When updating the value for an existing key, the leaf page containing that key is located, the value is changed, and the page is written back to disk.
    
5. **Adding New Keys**: To add a new key, the appropriate page is found, and if there isn't enough space, the page is split into two half-full pages. The parent page is then updated to accommodate the new key ranges, ensuring that the tree remains balanced.
    
6. **Balanced Trees**: B-trees maintain balance, meaning that a B-tree with n keys always has a depth of O(log n). This balance ensures efficient operations even as the size of the database grows.
    
7. **Write Operations**: The basic write operation in a B-tree involves overwriting a page on disk with new data. Unlike log-structured indexes, which only append to files, B-trees may require overwriting multiple pages when performing certain operations like splitting a page.
    
8. **Write-Ahead Log (WAL)**: To enhance reliability, databases often include a write-ahead log (WAL or redo log) on disk. This log records changes before they are applied to the actual data, ensuring that the database can recover in case of a crash.
    
9. **Concurrency Control**: When multiple threads are accessing the B-tree simultaneously, careful concurrency control is necessary. This is typically achieved by protecting the tree's internal data structures with latches, which are lightweight locks.
    

10. **B-trees and LSM-trees** are two different indexing structures used in databases, each with its own set of advantages and disadvantages.

11. **B-trees**:

- Advantages:

1. Efficient for reads: B-trees are often faster for read operations due to their structure, which allows for quick key-value lookups and range queries.
2. Strong transactional semantics: Each key exists in exactly one place in the index, offering strong transactional semantics. Transaction isolation can be implemented using locks attached directly to the tree.
3. Predictable performance: B-trees tend to offer more predictable performance compared to LSM-trees, especially as the database size grows.

- Disadvantages:

1. Lower write throughput: B-trees typically have lower write throughput compared to LSM-trees because of their overwrite-based write operations. This can result in higher write amplification, where a single write to the database results in multiple writes to disk.
2. Fragmentation: B-trees may leave disk space unused due to fragmentation, as they do not compress data as efficiently as LSM-trees.

12. **LSM-trees**:

Advantages:

1. Higher write throughput: LSM-trees are generally faster for write operations, as they append data to disk rather than overwriting existing data. This can lead to higher write throughput, especially in high-write workloads.
2. Better compression: LSM-trees can often compress data more effectively, resulting in smaller files on disk compared to B-trees.
3. Efficient use of disk space: Due to their append-only nature, LSM-trees tend to use disk space more efficiently and experience less fragmentation.

Disadvantages:

1. Slower reads: Reads can be slower on LSM-trees compared to B-trees because reads may need to check multiple data structures and SSTables at different stages of compaction.
2. Compaction overhead: The compaction process in LSM-trees can interfere with ongoing reads and writes, potentially impacting performance. Additionally, as the database grows, the compaction process may require significant disk bandwidth, and if not configured properly, it can lead to disk space exhaustion.

**Other indexing structures **
	beyond key-value indexes offer different ways to organize and access data efficiently. 

1. **Secondary Indexes**: These indexes are derived from a primary key index but differ in that the indexed values are not necessarily unique. There are two common approaches to constructing secondary indexes:
	    image
		    ![[Pasted image 20240129084221.png]]
    - **List of Row Identifiers**: Each value in the index corresponds to a list of matching row identifiers.
    - **Appending Row Identifiers**: Each entry in the index is made unique by appending a row identifier to it.
2. **Full-Text Search and Fuzzy Indexes**: Traditional indexes are not designed for searching similar or misspelled keys. Fuzzy querying requires specialized techniques:
    
    - **Full-Text Search Engines**: Allow for synonyms, grammatical variations, and proximity searches (words occurring near each other). Lucene is an example, using a structure similar to SSTables and finite state automata.
3. **In-Memory Databases**: These databases reside entirely in RAM, offering faster access compared to disk-based databases. Key points include:
    
    - **Durability**: While disks are durable and cost-effective, in-memory databases sacrifice durability for speed.
    - **Examples**: Products like VoltDB, MemSQL, and Oracle TimesTen are in-memory databases. Redis and Couchbase offer weaker durability.
4. **Transaction Processing vs. Analytics**:
    
    - **OLTP (Online Transaction Processing)**: Involves transactions with reads and writes forming logical units.
    - **OLAP (Online Analytics Processing)**: Focuses on querying and analyzing large volumes of data, typically for business intelligence purposes.
    
1. **Data Warehousing**: A separate database optimized for analytics, populated by extracting, transforming, and loading (ETL) data from OLTP systems. It often follows a star schema, with facts (events) surrounded by dimension tables.
	image
		![[Pasted image 20240129084957.png]]

1. **Column-Oriented Storage**: Unlike row-oriented storage, this stores all values of each column together, facilitating efficient querying and compression. It's particularly useful for data warehouses, where read-heavy queries are common.  A technique that is particularly effective in data warehouses is _bitmap encoding_. Bitmap indexes are well suited for all kinds of queries that are common in a data warehouse.Cassandra and HBase have a concept of _column families_, which they inherited from Bigtable.
    
7. **Materialized Views and Data Cubes**: These are caches of frequently used query results, often aggregated along different dimensions. Materialized views are copies of query results stored on disk, while data cubes organize aggregates in a grid format.
## **Encoding and Evolution:**

- **Change Management:** Any alteration in an application's features typically requires a corresponding adjustment in the data it stores. This means that as the application evolves, so too must the structure and format of its stored data.
- **Schema Management:** In relational databases, there's typically a predefined schema that outlines the structure of the data. While this schema can be modified, at any given time, there's only one schema in effect. On the other hand, "schema-on-read" or "schemaless" systems allow for flexibility, accommodating a mix of older and newer data formats.
- **Rolling Upgrades:** In large-scale applications, updates don't occur instantaneously across all nodes. Instead, a rolling upgrade strategy is often employed, where new versions are gradually deployed to subsets of nodes without causing service downtime. This means that during the upgrade process, both old and new versions of the application code, as well as old and new data formats, may coexist.
- **Compatibility:** To ensure smooth transitions during upgrades, compatibility in both backward and forward directions is crucial. Backward compatibility ensures that newer code can still understand and process data written by older versions, while forward compatibility ensures that older code can interpret data written by newer versions.

**Formats for Encoding Data:**

- **In-Memory Representation:** Data within the application is typically stored and manipulated in memory using a specific representation that aligns with the programming language's constructs.
- **Serialization:** When data needs to be stored persistently (e.g., in a file) or transmitted over a network, it must be converted into a byte sequence. This process of converting in-memory representation to byte sequences is called encoding or serialization.
- **Decoding:** Conversely, when data is retrieved from storage or received over the network, it must be transformed back into the in-memory representation. This process is known as decoding or deserialization.

**Versioning:**

- **Efficiency:** Efficient versioning mechanisms are crucial for managing changes in data formats and ensuring compatibility across different versions of the application. This involves strategies for efficiently encoding and decoding data, as well as managing the evolution of schemas and compatibility between different versions of the software.
**Drawbacks of  JSON, XML, and CSV: **
1. **Ambiguity and Limitations:**
    
    - These formats may have ambiguity around the encoding of numbers, especially when dealing with large numbers.
    - While they support Unicode character strings, they lack support for binary strings. To handle binary data, developers often resort to encoding it as Base64, which increases data size by about 33%.
2. **Schema Support:**
    
    - XML and JSON offer optional schema support, which allows developers to define the structure and constraints of the data. However, CSV lacks any built-in schema support.
3. **Binary Encoding:**
    
    - JSON and XML, while human-readable, are not space-efficient compared to binary formats. Several binary encodings like MessagePack, BSON, and others have been developed to address this issue.
    - Apache Thrift and Protocol Buffers (protobuf) are libraries specifically designed for efficient binary encoding.
4. **Schema Evolution:**
    
    - Thrift and Protocol Buffers handle schema changes while ensuring backward and forward compatibility through various mechanisms:
        - **Forward Compatible Support:** New fields are added with new tag numbers, allowing old code to ignore unrecognized tags when reading data from newer versions.
        - **Backward Compatible Support:** Each field is assigned a unique tag number, ensuring that new code can read old data. New fields added after the initial deployment of the schema must be optional or have default values.
        - **Removing Fields:** Removing fields follows a similar principle as adding them, but in reverse. Only optional fields can be removed, and the same tag cannot be reused.
        - **Changing Data Types:** Changing the data type of a field carries the risk of losing precision or truncating values, which developers need to consider carefully during schema evolution.

