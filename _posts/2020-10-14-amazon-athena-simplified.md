---
date: 2020-10-14
title: Amazon Athena, simplified
categories:
author_staff_member: dave
---

Last time I wrote about [why small startups might want to collect and use their data](https://www.tabbydata.com/2020/09/12/the-case-for-small-data/). If you buy into the _why_, the next question you might ask is _how_?

At first glance, Amazon Athena sounds like it should be the holy grail of data warehouses for scrappy startups: dump any lightly-structured data (e.g. JSON, CSV) into S3 and then query it with SQL anytime. Unfortunately, that promise is rarely realized for small startups, because Athena and the ecosystem it belongs to basically don't prioritize that audience. However, it _is_ possible for tiny 1-, 2-, or 3-person startups to live the Athena dream! Here I will try to outline how.

If we break down the problem somewhat, there are actually a few separate problems:

1. Data ingestion
2. Data storage
3. Data usage

I'll address these in reverse order, because the requirements build on top of each other that way.

## Data usage

Athena is basically just a (barely) dressed-up version of [Presto](https://prestodb.io/). Presto is a standards-compliant SQL query engine, so you can more-or-less query it with anything that can query SQL. That makes using the data in Athena really easy, but there are a few gotchas around performance and (very relatedly) cost.

 - The most important consideration when querying large tables in Presto is minimizing the data that needs to be scanned by the system, and the most straightforward way of doing this is by making sure your queries always include partition values. This leads to a storage constraint: **tables must have partitions!**
 - Another important consideration, which Amazon does not document very well at all, is the _number of files_ that will be scanned by the system. This is very different from the _size of the data_ that will be scanned; after all, 1GB of data might be one file or it might be 1 million; reading the former from S3 costs $0.0000004 while reading the latter costs $0.40. There is no way to control this at query-time, so this introduces another storage constraint: **tables must be distributed amongst fewer files where possible.**
   - This is at odds with the first constraint, and is part of the reason why complicated file formats like Parquet exist. More on this in the storage section below.
 - Even when partitions are referenced, it's possible to construct queries in Presto that run very inefficiently. In practice, I have not found this to be much of a problem, but from time to time it is necessary to optimize. Searching Google for "optimizing Presto queries" returns lots of useful goodies like [this](https://www.doorda.com/how-to-guides/query-optimization-for-presto-on-doordahost/).

## Data storage

Ok, so we want to keep costs low and queries performant by... partitioning data into separate files as much as we can while also... not having more files than we need? Basically, yes. In practice, it helps to not overthink this. The rules of thumb that I recommend are:

 - All data entries should be represented as either events (things that happened at a specific time) or objects (usually mapping to domain objects in your application, like a user or an inventory item, and having an ID).
 - All events should be partitioned based on the time that they occurred, bucketed into days.
 - All objects should be bucketed into ~200 partitions based on a hash of the ID.
 - Files in a partition should be allowed to grow to about 100MB before adding a new file.

Athena is very flexible about file formats (it would even allow you to define your own serialization logic) but in my opinion there are three mainstream options:

1. **CSV.** Generally fine, but I'm inclined to avoid because it doesn't work well with more complicated data types (e.g. Presto supports columns with array values, but I don't want to think about how to encode that in CSV).
2. **JSON.** Tried and true.
3. **Apache ORC or Apache Parquet.** These are two separate file formats, but I'm grouping them together as "fancy binary formats that solve a lot of problems, but introduce a few of their own." Specifically, these are _column-oriented_ formats, which is neat because it means `SELECT foo FROM bar WHERE baz=3` will only scane the `foo` and `baz` columns in the `bar` table, instead of scanning all the columns, saving both time and money. (This is basically like another dimension of partitioning.) The main downside, however, is that generating these files is less straightforward and usually requires using things like Apache Spark, which (a) represent too much of a distraction for tiny startups, if they're not already skilled in it, and (b) don't support real-time updates very well.

Speaking of real-time updates: ideally, data would be added to the data warehouse almost immediately after it's produced, so that you can use it right away. This is non-trivial, but easier for most programmers to reason about and implement using CSV/JSON vs the binary formats. At scale, the binary formats end up being really worthwhile, but early on the costs to be concerned about are dominated by people-hours rather than infrastructure expenses. The CSV/JSON approach will easily scale up to several terabytes, especially if they're compressed (Athena supports a variety of compression formats, including gzip).

## Data ingestion

Collecting data and adding it to our Athena tables is the complicated part, informed by the requirements discovered above. We want to create tables containing events and objects, partitioned by date or ID, which update in close to real-time. At a very high level, it will look something like this:

 - Integration points (e.g. parts of application code that wish to emit an event) place new data entries into a queue. The queue is sharded by event or object type (this is implemented as a "message group ID" in SQS FIFO queues).
 - Workers (e.g. Lambda functions) process the queue. Unfortunately, S3 does not expose any way of simply appending to a file, so we need to overwrite files instead. The worker determines the file path based on event/object type and partition ID, then inserts a file if it does not exist or modifies it (adding a row) if it does. If the last file is larger than 100 MB, it starts a new file.
   - I've also seen and used an approach which uses SQL `INSERT INTO` statements at this point, via Athena, rather than mutating S3 files directly. This works, but will end up violating our file-size requirement and result in unnecessarily high query costs.

That's it! I should be clear that this is not the "normal" way of using Athena; it's customized based on what I've seen work well for very small startups. The "normal" way involves things like AWS Glue and Hive/Hadoop/Spark/Flink/etc; most existing resources for using Athena are centered on that approach. I've focused on the approach described above because I think it's more approachable (in terms of total cost) than anything else.

## Cost modeling

I've referenced cost a few times above. In a future post I'll present a cost model based on each aspect—ingestion, storage, and usage.
