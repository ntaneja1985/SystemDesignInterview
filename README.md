# System Design Interview Preparation
System Design Interview Questions

# Choosing the Right Database
- Choice of database depends on the kind of data that we have: whether it is structured data or unstructured data or semi-structured data.
- It also depends on the type of query pattern that we have
- Scale that we need to handle

## Caching Solutions
- If we are making a call to another system and that takes lot of time, we can consider caching its response.
- Rather than query a database everytime, we can store value inside the cache
- We have a key-value store.
- Common solutions are Redis, Memcache

## File Storage Solutions
- Blob Storage (Not really a database)
- Amazon S3 (Images,Videos)
- Need Content Delivery Network
- CDN: Individual servers across the globe.
- Provide good availability and redundancy but data could be lost.
- Efficient at Search(Remember Blob Metadata)
  
## Text Search
- We may require ability to search like Name, Title, Description with support of Fuzzy Search
- So we require a Text Search Engine like Elastic Search or Solr. Built on top of Apache Lucene.
- **Fuzzy Search**: If the user searches for 'Airport' and in the searchbox we write: 'Airprot', We cannot return 0 results. This is bad user experience.  So our search engine should be able to calculate edit distance between the correct word and its wrong spelling and allow it to search for 'Airport'.

## Supporting Logging and Instrumentation (Metrics Tracking System)
- Suppose we have applications that have lot of instrumentation data like garbage collector state, throughput,CPU, latencies, logs that are being generated.
- Use **Time Series Database**. Extension of relational databases. Good thing about relational databases is that they allow us to update random records. In a time series database, however, we dont really need random updates. We only have sequential updates in append-only mode. So if we have entry at time T1, next entry is at time T2.
- Also our queries are bulk read queries
- Time series database are optimized for this kind of query pattern.
- Time series databases are designed to handle large volumes of data with high write and query performance.
- Examples are Influx DB, Timescale DB, Graphite, Prometheus, Open TSDB(Open Time series database).
- These databases are particularly useful in scenarios like monitoring server health, application performance, financial trading, sensor data in IoT, and much more.
- They provide specialized features such as data compression, downsampling, and aggregation to efficiently handle large datasets over time.

## Databases optimized for Analytics
- Need data warehouse
- We can dump all the data and have very good querying capabilities to serve lot of reports
- However, data warehouses are not used for transaction systems, they are used for offline reporting.
- Hadoop is a good option for transactional data, where we can put that in and have various systems that can provide reporting on top of that data.

## How to decide where to use relational database vs non-relational database
- If we have structured data and if we need ACID guarantee, we can use RDBMS(MySql, Oracle, Sql Server, Postgres). Useful in payment systems or inventory management system.
- If we have structured data but we dont need ACID guarantee, we can still go with relational database or go with non-relational database. Would not make much of a difference
- What if we have non-structured data
- Consider we are trying to build a catalog type of system or e-commerce system like Amazon. Different products have different attributes. Tshirt is different from refrigerator.
- We can just store data on JSON, but relational database make it harder to query such kind of data.
- We need database optimized for these kind of queries
- If we have large number of data-types and there are lot of queries that can come in, then we can use a document database like MongoDb, CouchBase.
- Even ElasticSearch, Solr are special cases of document databases.
- Let us consider another scenario for non-structured data, what if we have ever-increasing data but we have finite set of queries. For e.g Uber( thousands of drivers sending their location constantly.(X number of location records per day). Data increases enormously.Data increases in an exponential fashion. Most of queries related to working on such data are common like Find me locations of the drivers by Driver Id.
- If we have less number of queries, but exponentiallly increasing data, we need to use Columnar database like Apache Cassandra or HBase. Cassandra is not very heavy to deploy.

## Using combination of databases
- Now let's take an example of us building an e-commerce platform something like an Amazon. Now when you are managing inventory on that
side you want to make sure that you are not over selling items. So let's say there is just one quantity of a particular item left and there are 10 users who want to buy that you want to have this ACID properties there to make sure that only one of the users should be able to commit the transaction other users should not be able to commit the transaction, right. You can put constraints and all of that on there. It would make sense to use a RDBMS database for the inventory management system of a system like Amazon or maybe an order management system.

- But if you look at the data of Amazon it is an ever interesting data. Why? because the number of orders are additive each day new orders are coming in they cannot purge the data due to lot of legal reasons plus the number of orders are also increasing so it naturally fits into this modelso we need to use Cassandra. We could use a combination of both of these databases. You could use MySQL or any other RDBMS alternate for storing data about the orders it has just placed and not yet deliver to the customer. Once the item is delivered to the customer you can then remove from this RDBMS, and put it into Cassandra as a permanent store.

- Let's say we want to build a reporting kind of a thing which lets you query something like
  'Get me all the users who have bought sugar in last five days'
- sugar is not just a product. There are a lot of sellers selling different sugar alternates of different companies maybe, of different qualities maybe.
- Sugar would like lot of ItemIDs and on top of these ItemIDs there would be lot of orders.
- Orders can be in either RDBMS or Cassandra
- What we can do is store the querying part(subset of data) inside a Document Database and let that reference to more data inside RDBMS db or Cassandra db using a key like OrderId.
- We can do a query to fetch list of orders using MongoDb, fetch the OrderIds and then query RDBMS or Cassandra database.
- Sometimes one database cannot fit all of our requirements. We may need to use a combination of databases.

