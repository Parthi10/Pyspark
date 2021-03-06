# learn RDD - Resilient Distributed Datasets

## about

RDDs are immutable, distributed (partitioned) collection of data. The partitions may be computed on different nodes of the cluster. It can be created in 3 ways, namely a) load from an external dataset b) parallelize a collection (list) c) transforming another RDD

## resources


[programming guides 1](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/rdd/RDD.html)

[programming guides 2](https://spark.apache.org/docs/1.1.1/api/python/pyspark.rdd.RDD-class.html)

[programming guides 3](http://spark.apache.org/docs/2.2.0/api/python/_modules/pyspark.html)

[paper by Matei](https://cs.stanford.edu/~matei/papers/2012/nsdi_spark.pdf)

[Advanced Apache Spark- Sameer Farooqui (Databricks)](https://www.youtube.com/watch?v=7ooZ4S7Ay6Y)

[A Deeper Understanding of Spark Internals - Aaron Davidson (Databricks)](https://www.youtube.com/watch?v=dmL0N3qfSc8)

[Introduction to AmpLab Spark Internals](https://www.youtube.com/watch?v=49Hr5xZyTEA)

[blog on DAG](https://databricks.com/blog/2015/06/22/understanding-your-spark-application-through-visualization.html)

[cloudera blog on spark tuning](https://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-1/)

[cloudera blog](https://blog.cloudera.com/blog/2015/07/how-to-do-data-quality-checks-using-apache-spark-dataframes/)
[and related talk](https://www.youtube.com/watch?v=WyfHUNnMutg)

[RDD slides](https://www.slideshare.net/satyanarayanpatel1/ibm-spark-meetup-rdd-spark-basics?from_action=save)

[sameer faruqui lab documents](https://tinyurl.com/dsesparklab) and [this](tinyurl.com/cdhsparklab)

https://legacy.gitbook.com/book/databricks/databricks-spark-knowledge-base/details

### types of RDD

Following are helpful to understand logical plan of rdd (rdd.toDebugString()). A logical plan is always generated, irrespective of whether an action is called or not. These are basically sub-classes of base RDD class.

a. HadoopRDD - returned by sc.textFile

b. mapPartitionRDD - returned by operations like map, flatmap, filter, mapPartitions.

c. shuffledRDD - result of repartition or coalesce transformations (? in case of Key-value rdds)

d. parallelCollectionRDD - returned by sc.parallelize (when used to create a rdd)

e. coalescedRDD - returned by a coalesce or repartition function

f. FilteredRDD

g. MappedRDD

h. PairRDD

i. UnionRDD

j. PythonRDD

k. DoubleRDD

l. JdbcRDD

m. JsonRDD

n. SchemaRDD

o. VertexRDD

p. EdgeRDD

[also see](https://spark.apache.org/docs/0.6.2/api/core/spark/rdd/package.html)

### create

#### create from a text file


sc.textFile('filepath',minPartitions=n,use_unicode=True)

notes:
minPartitions and use_unicode are optional parameters
filepath can be path to a single file, or multiple files provided with wildcards, or compressed files (gz)
filepath should be accessible from driver and all worker nodes [either copied to all worker nodes, or placed in network-mounted file system]
increasing the value of minParitions would increase parallelism and improve execution time.

`sc.wholeTextFiles('filepath')`

notes:
read multiple files. each row would contain file name and content of each file as pairs.

further read: rdd programming guide

reading from hdfs:
`sc.textFile('hdfs://folder/filename.csv')`

reading from s3:
`sc.textFile('s3://bucket/folder/filename.csv')`

### create from a list

`sc.parallelize(list)`

## transformations

### narrow vs wide transformations

wide transformations are those, which require a shuffle, while narrow ones does not. In wide transformations, multiple child partitions depend on each parent partition.
In case of narrow transformations, each partition in parent rdd is used by at most one partition in child rdd.
example of narrow transformations are map,filter. SortByKey, join, GroupByKey are wide transformations e.g. Narrow transformations are faster to execute.

### map

using lambda function:

using custom function:

### flatmap

### filter

### distinct

### sample

### union

### intersection

### subtract

### join

### cartesian

### 
## actions

### reduce

### treeReduce

### fold
applies an associative function and an initial value to each partition, and then the results of all partitions. The initial value should be set to zero.

`add code here`

### aggregate

### collect

### take

### first

### top

### count

### countApprox

### countByValue

### countByValueApprox

### takeOrdered

### takeSample

### treeAggregate

### countApproxDistinct

### foreach
difference with map: map accumulates all of the results into a collection, whereas foreach returns nothing. map is usually used when you want to transform a collection of elements with a function, whereas foreach simply executes an action for each element.

### foreachPartition

### zip

### zipPartitions

### cache

### persist

### checkpoint


### comparison

https://github.com/JerryLead/SparkInternals/blob/master/markdown/english/6-CacheAndCheckpoint.md

https://stackoverflow.com/questions/35127720/what-is-the-difference-between-spark-checkpoint-and-persist-to-a-disk

### getCheckpointFile

### getNumPartitions

get the number of partitions in the rdd.

`rdd.getNumPartitions()`

### glom

Return an RDD created by coalescing all elements within each partition into a list. The method does not perform any repartition or shuffle.

`rdd.glom()`

### groupBy

Returns an rdd of grouped items, based on the function supplied. The result has to be converted to a list further, using a map function.

`code goes here - refer 26`

### groupByKey

`r1 = sc.parallelize([("a",1),("b",2),("c",3),("d",4),("a",5),("c",6),("c",2)])`  
`r3 = r1.groupByKey()`  
`r3.collect()`  

r3 rdd returns a collection of objects, which needs to be converted further to a list.

`r4 = r3.map(lambda x:(x[0],list(x[1])))`  
`print(r4.toDebugString().decode("utf-8"))`  
`print(r4.collect())`

Note: performs shuffle first. We need to apply map function further to derive the data. We should use reduceByKey instead to achieve the same results. See [this](https://databricks.gitbooks.io/databricks-spark-knowledge-base/content/best_practices/prefer_reducebykey_over_groupbykey.html) and [this](https://stackoverflow.com/questions/43364432/spark-difference-between-reducebykey-vs-groupbykey-vs-aggregatebykey-vs-combineb)

### reduceByKey

`r1 = sc.parallelize([("a",1),("b",2),("c",3),("d",4),("a",5),("c",6),("c",2)])  
r2 = r1.reduceByKey(lambda x,y:x+y)  
r2.collect()`

Note: Data is combined so that at each partition there should be at least one value for each key. And then shuffle happens and it is sent over the network to some particular executor for some action such as reduce.

### combineByKey

Combine values with the same key using a different result type.

### foldByKey

similar to fold, applied by key

`code goes here`

### mapValues

Apply a function to each value of a pair RDD without changing the key.



### id
Returns an unique numeric id of the rdd within the sparkcontext of the application. This id can be seen in RDD logical map as well.

`rdd.id()`

### isCheckpointed

### isEmpty
Returns true/false, based on whether RDD is empty or not. Please note, it is applied on the whole rdd, not a particular partition.

`rdd.isEmpty()`

### mapPartitions

### mapPartitionsWithIndex

### max

### min

### name

### partitioner

### partitions

### pipe

### randomSplit

### repartition(numPartitions)

Can Increase/decrease number of partitions

Internally does shuffle

expensive due to shuffle

For decreasing partition use coalesce

#### custom partitioner

### Coalesce(numPartition,Shuffle: <ins>[True/False]</ins>

Decreases partitions

Goes for narrow dependencies

Avoids shuffle

In case of drastic reduction may trigger shuffle

### saveAsObjectFile

### saveAsTextFile

### sortBy

Returns the rdd sorted by the given key function. Ascending is default order. set ascending=False in case of descending.

`rdd.sortBy(lambda x:x[0])`


### setName

Useful to define a name for a rdd. The name would be visible in DAG in Spark UI, plus, if you cache it, it would be visible in storage.

`rdd.setName("New Name")`

### toString

### toDebugString

### zipWithIndex

### zipWithUniqueId

### unpersist

### collectAsMap
returns an RDD of key-value pairs as dictionary

### lookup

### countByKey


## Performance Tuning

a. avoid too few partitions. it would lead to less concurrency (and unused cores)  
b. avoid too many partitions. it would mean more framework overhead  
c. avoid too big partitions  
d. if no of partitions are close to 2000, bump it up to make it above 2000  
e. resolve data skew using salting technique  
f. resolve cartesian joins using methods like nested structure, windowing  
g. avoid shuffle as much as possible in your DAG (it is expensive)  
h. use reduceByKey instead of groupByKey  
i. use treeReduce over reduce  
j. keep data in serialized format, to minimize storage  
k. Commonly between 100 and 10,000 partitions  
l. Lower bound: At least ~2x number of cores in cluster  
m. Upper bound: Ensure tasks take at least 100ms
n. Don't copy all elements of a large RDD to the driver. Collect will attempt to copy every single element in the RDD onto the single driver program, and then run out of memory and crash.Instead, you can write out the RDD to files or export the RDD to a database that is large enough to hold all the data.

## Write custom RDD

http://blog.madhukaraphatak.com/extending-spark-api/

## spark cassendra connector

https://github.com/datastax/spark-cassandra-connector

## Logical Plan

`rdd.toDebugString().decode("utf-8")`

Or set logLineage to true, with the following code
`spark-submit mycode.py --conf spark.logLineage=true`

Or

`spark.conf.set("logLevel",'true')
sc.setLogLevel("INFO")`


## Python unicode support

https://docs.python.org/3/howto/unicode.html


