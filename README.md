# Designing-Data-Intensive-Apps
*The aim of this repository is to keep track of my learnings from the book "Designing Data Intensive Applications" by M.Kleppmann.*
---

# Foundations of Data Systems
# 1. Reliable, Scalable, and Maintainable Applications
---
Data-intensive applications are usually built by using
1. *Databases* to store data so it can be retrieved later
2. *Caches* to speed up the reads
3. *Search indexes* to allow users to search or filter data by keywords
4. *Steam processing* to handle data asynchronously
5. *Batch processing* to handle large amounts of data periodically

## Reliability
*Reliability* means making systems work correctly, even when faults occur. Faults can be in hardware(random&uncorrelated), software(systematic bugs), and humans(inevetable).
For software, typical expectations include:
- The application performs the function that the user expected
- It can tolerate the user making mistakes or using the software in unexpected ways
- Its performance is good enough for the required use case, under the expected load and data volume
- The system prevents any unauthorized access and abuse

## Scalability
*Scalability* is the term used to describe a system's ability to cope with increased load.
In order to discuss scalability, we first need ways of describing load and performance quantitatively.

### Describing Load
Load can be described with measurable numbers, which are called **load parameters**. </br>
For example
- Requests per second to a web server
- The ratio of reads and writes in a database
- The number of simultaneously active users in a chat room
- The hit rate on a cache

Provided example of Twitter where its scaling challenge is not primarily due to tweet volume, but due to "fan-out" - each user follows many people, and each user is followed by many people.
The distribution of followers per user (maybe weighted by how often those users tweet) is a key load parameter for discussing scalability, since it determines the fan-out load.

### Describing Performance
Once the load is described, what happens if
- Load parameters are increased and the system resources unchanged, how is the performance of the system affected?
- Load parameters are increased, how much is it needed to increase the resources if you want to keep performance unchanged?

To answer these questions, we need performance numbers.</br>
In batch processing systems we usually care about **throughput** - the number of records we can process per second or t he total time it takes to run a job on a dataset of a certain size. </br>
In online systems what's usually more important is the service's **response time** - the time between a client sending a request and receiving a response. </br>

For **response time** measurement, it's good to use **percentiles**. </br>
Most common percentiles are 95th(p95) and 99th(p99), for example if the 95th percentile response time is 1.2 seconds, that means 95 out of 100 requests take less than 1.2 seconds and 5 out of 100 take 1.2 seconds or more. </br>
When it comes to customers of a web shop, the customers with the slowest requests are often those who have the most data on their accounts, because they have made many purchases, meaning they are the most valuable customers.

*Latency and response time are often used synonymously, but they are not the same. Latency is the duration that a request is waiting to be handled (delay).*

**Head-of-Line Blocking**: When several backend calls are needed to serve a request, it takes just a single slow backend request to slow down the entire end-user request

Scalable architecture is built around assumptions of which operations will be common and which will be rare - **the load parameters**

## Maintainability
Keeping maintainability in mind reduces future blockers and bottlenecks. There are 3 things to keep in mind:
- *Operability* - Make it easy for software/operations teams to keep up the system running smoothly
- *Simplicity* - Make it easy for new engineers to understand the system
- *Evolvability/Modifiability* - Make it easy for engineers to make changes to the system in the future

Set up proper monitoring, keep software up to date, write documentation. Good abstractions can help reduce complexity.
---
# 2. Data Models & Query Languages
---
**SQL and the Relational Model**:
  - Suited for structured data with a fixed schema.
  - Emphasizes data integrity and ACID (Atomicity, Consistency, Isolation, Durability) transactions.
  - SQL is a declarative language, specifying what data is needed without detailing how to fetch it, contrasting with the imperative approach that required detailing each step of the data access process.
  - Often experiences impedance mismatch with object-oriented programming, due to the difference in data representation.
  - Normalization is key in relational databases to avoid data duplication but can impact query performance due to the need for joins.

**NoSQL and Non-Relational Models**:
  - Ideal for semi-structured or unstructured data with dynamic schemas.
  - Emerged due to demands for scalability, open-source solutions, and specialized queries not supported by SQL.
  - Document databases (a type of NoSQL) are known for schema flexibility and often align better with the data structures used in applications.
  - They typically store related data together for improved performance (due to data locality) but struggle with joins and complex many-to-many relationships.
  - In document databases, accessing parts of large documents can be inefficient; it's recommended to keep documents small to avoid performance issues.

**Polyglot Persistence**:
  - The concept of using different data storage technologies in a single application, chosen based on varying needs of the application components.
  - Addresses the limitations of a single database system by leveraging the strengths of various database models.

**Data Denormalization**:
  - Often used in NoSQL databases to improve read performance.
  - Involves storing redundant copies of data, improves performance at the cost of data integrity and increased complexity in maintaining consistency.

**Graph Databases**:
  - Utilize nodes and edges to represent and store data.
  - Each entity (node) and relationship (edge) carries its unique identifier and a set of key-value pairs (properties).
  - Highly flexible and extensible, making them ideal for data with complex relationships and evolving schemas.
---
# 3. Storage and Retrieval
---
Storage is an important component of a software system. It is crucial to understand how storage works to make better decisions when working with.
  - Different storage engines are optimized for different workloads. Storage engines are often optimized for either transactional workloads (frequent updates and reads, like in web applications) or analytics (complex queries, often reading large volumes of data)
  - Important to understand how to optimize your storage engine based on applications needs
  - Different storage engines have different scalability and fault tolerance mechanisms

## Data Structures That Power Your Database
- **Hash Indexes**:
  - *Purpose*: Fast data access using a key.
  - *How It Works*: Utilizes a hash function to calculate an index, determining where data is stored.
  - *Use Case*: Optimal for scenarios requiring quick lookups of exact matches.

- **SSTables (Sorted String Tables)**:
  - *Purpose*: Efficiently store large volumes of data in a sorted order.
  - *How It Works*: Data is organized in key-value pairs and sorted based on the key.
  - *Use Case*: Beneficial for range queries and effective in merging data segments.

- **LSM Trees (Log-Structured Merge-Trees)**:
  - *Purpose*: Designed to manage high volumes of write operations.
  - *How It Works*: New writes are initially stored in a memtable, then transferred to SSTables, which are periodically merged and compacted in the background.
  - *Use Case*: Suitable for write-intensive applications, like logging systems.

- **B-Trees**:
  - *Purpose*: Ensure efficient management of sorted data for both read and write operations.
  - *How It Works*: Data is structured in a balanced tree format, with sorted information within each node.
  - *Use Case*: Widely used in database indexing due to effective performance in both reading and writing.

- **Comparing B-Trees and LSM-Trees**:
  - *B-Trees*: Generally offer faster read operations, making them suitable for balanced read/write workloads.
  - *LSM-Trees*: Tend to be faster for write operations, advantageous in write-dominant scenarios.
  - *Note*: Each has distinct performance and storage traits, necessitating selection based on specific application requirements.

- **Other Indexing Structures**:
  - Include various types such as multi-dimensional, full-text search, and fuzzy indexes.
  - *Specializations*: Each type is tailored for specific query needs, like geospatial data, textual search queries, or handling data with potential inaccuracies.

## Transaction Processing or Analytics?
| Property               | Transaction processing systems (OLTP)       | Analytic systems (OLAP)                |
|------------------------|---------------------------------------------|----------------------------------------|
| Main read pattern      | Small number of records per query, fetched by key | Aggregate over large number of records |
| Main write pattern     | Random-access, low-latency writes from user input | Bulk import (ETL) or event stream     |
| Primarily used by      | End user/customer, via web application      | Internal analyst, for decision support |
| What data represents   | Latest state of data (current point in time)| History of events that happened over time |
| Dataset size           | Gigabytes to terabytes                      | Terabytes to petabytes                 |

## Data Warehousing
* Serves as a centralized repository for analyzing large datasets originated from multiple sources, often OLTP systems
* Optimized for read-heavy, complex queries. Stores historical data for business intelligence.
### Stars and Snowflakes: Schemas for Analytics
- *Star Schema*: Fact table at the center of the schema. Each row represents an event. Columns are foreign key references to other tables (*dimension tables*)
- *Snowflake Schema*: Variation of Star schema, dimension tables are further broken down intoo subdimensions
Snowflake Schemas are more normalized than Star Schemas but are more complex and thus harder to query which makes Star Schemas more preferred

## Column-Oriented Storage
- *Purpose*: Optimized for efficiently querying large datasets, particularly in analytics and data warehousing.
- *How It Works*: Data is stored by columns instead of rows, enabling efficient retrieval of specific attributes without needing to read entire records.
- *Use Case*: Highly suitable for operations that require scanning large datasets for a limited number of columns, including various aggregation and analytical processes.

### Column Compression
- *Purpose*: Aims to decrease storage requirements and boost read performance.
- *How It Works*: Employs compression algorithms, such as bitmap encoding, to take advantage of the repetition and pattern characteristics inherent within columnar data.
- *Use Case*: Extremely useful in data warehouses that feature columns with repetitive or predictable data patterns, improving query performance and reducing disk input/output operations.

### Sort Order in Column Storage
- *Purpose*: Improves query performance and the efficiency of data compression methods.
- *How It Works*: Although data is sorted in a row-wise fashion, it is stored column-wise, which leads to strong compression, especially in primary sort columns.
- *Use Case*: Advantageous for executing range queries and for analytical operations where sorting by particular keys (e.g., dates, categories) is a regular requirement.

### Writing to Column-Oriented Storage
- *Purpose*: Designed to handle updates and insertions of new data efficiently in columnar databases.
- *How It Works*: Uses structures like LSM-trees, data is initially gathered in memory and subsequently merged into on-disk columnar files.
- *Use Case*: Appropriate for environments where read operations are prioritized, and write operations can be accumulated and executed in batches.

### Aggregation: Data Cubes and Materialized Views
- *Purpose**: To expedite the execution of common aggregate queries.
- *How It Works*:
  - *Data Cubes*: Aggregate data is precalculated and stored in multi-dimensional structures, which leads to swift query responses.
  - *Materialized Views*: Results from aggregate queries are persisted on disk, allowing quicker retrieval.
- *Use Case*: Particularly effective for repetitive aggregate computations, commonly found in business intelligence and reporting workflows.


