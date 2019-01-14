
# Resilient Distributed Datasets (RDDs)

## Introduction

Resilient Distributed Datasets (RDD) are fundamental data structures of Spark. An RDD is, essentially, the Spark representation of a set of data, spread across multiple machines, with APIs to let you act on it. An RDD could come from any datasource, e.g. text files, a database, a JSON file etc.

> RDDs Spark specific structures that let users explicitly __persist__ intermediate results in memory, control their __partitioning__ to optimize data placement, and manipulate them using a rich set of operators.

This is quite a handful. In this lesson, we will try to demistify RDDs with some hands on pratice. 

*__Note:__ Your are advised to visit all the included external links to see more examples of stated concepts.* 

## Objectives

You will be able to:

* Describe RDDs as fundamental storage units in Spark computing environment
* Create RDDs from Python collections
* Set number of partitions for parallelizing RDDs
* Review an RDD's dependancy graph at different stages of processing. 

## What Are RDDs ?

Resilient Distributed Datasets (RDDs) are the core concept in Spark. RDDs can be thought of as __immutable distributed collections of data objects__. Each dataset in RDD is divided into logical partitions, which may be computed on different nodes of the cluster. In order to understand how spark works, we should know what RDD’s are and how they work.

<img src="split.png" width=500>

__The Spark RDD is a fault tolerant, distributed collection of data that can be operated in parallel__. With the RDD split into partitions as shown above, spark runs one task for each partition. The Spark RDDs can contain any type of Python, Java or Scala objects, including user-defined classes. 

>RDDs are not actual data, but they are Objects, which contains information about data residing on the cluster. 

RDDs try to solve computational problems by enabling fault tolerant, distributed In-memory computations. RDDs have the following properties –

__Immutability and partitioning__: RDDs composed of collection of records which are partitioned. Partition is basic unit of parallelism in a RDD, and each partition is one logical division of data which is immutable and created through some transformations on existing partitions.Immutability helps to achieve consistency in computations.Users can define their own criteria for partitioning based on keys on which they want to join multiple datasets if needed.

__Coarse grained operations__: Coarse grained operations are operations which are applied to all elements in datasets. For example – a map, or filter or groupBy operation which will be performed on all elements in a partition of RDD.

__Fault Tolerance__: Since RDDs are created over a set of transformations , it logs those transformations, rather than actual data.Graph of these transformations to produce one RDD is called as Lineage Graph.

__Resiliance__: RDDs load the data for us and are resilient, which means they can be recomputed when needed.

__Lazy evaluated__: The data inside RDD is not available or transformed until an action is executed that triggers the execution.

### RDD Transformations vs Actions

In Spark, we first create a __base RDD__ and then apply one or more transformations to that base RDD following our processing needs. Being immutable means, **once an RDD is created, it cannot be changed**. As a result, **each transformation on an RDD creates a new RDD**. Finally, we can apply one or more **actions** to the RDDs. Spark uses lazy evaluation, so transformations are not actually executed until an action occurs. 

>*RDDs perform two types of operations: Transformations, which creates a new dataset from the previous RDD and Actions, which return a value to the driver program after performing the computation on the dataset.*

<img src="rdd1.png" width=500>



Let's see this behaviour through a simple example. In this example, we shall perform several actions and transformations on RDDs in order to obrain a better understanding of Spark processing. 

### Create a Python collection 

We need some data to start experimenting with RDDs. Let's create some sample data and see how RDDs handle it. 

- Create a Python list `data` of integers between 1 and 1000. 
- Use the `range()` function to create this list of a length = 1000 (1 - 1000 inclusive). 
- Sanity check : confirm the length of the list 


```python
# Create a list (data) as data = [1,2,3..,1000]

data = range(1, 1001)
print(data[0], data[999])
print (len(data))

# (1, 1000)
# 1000
```

    1 1000
    1000


### Create RDD from Python collections

In the spark computation model, datasets are treated as list of entries. Such lists are split into a number of different logical partitions that may be stored on different cores on the same computer or across the cluster. Each split partition holds a unique subset of original data. RDDs and are used to split, transform and combine the data during processing. 

Unlike Hadoop which uses physical disks to process data and store intermediate results, __Spark performs in-memory processing__ which allows high speed processing, that can be performed at a much lower computational cost. Following figure shows an overview of Spark processing model. For a list like the one we created above, all entries are distributed over different workers as shown:

![](partitions.png)

RDDs can be created in two different ways:

1. __Referencing__ an external data set in an external storage system, such as a shared file system, HDFS, HBase, or any data source offering a Hadoop Input Format.

2. __Parallelizing__ a collection of objects (a list or a set) in the driver program.

[Visit the documentation](https://spark.apache.org/docs/2.1.1/programming-guide.html) to read more on these methods. For this lab, we shall use the second approach with the collection we created earlier. 

### `SparkContext.parallelize()` for creating RDDs

In order to create RDDs in Spark, we will use `sc.parallelize()` method. This tells spark to create a new RDD based on the data that is passed in. A second argument tells Spark to set a number of paritions for this data as highlighted in the figure above. The elements of the collection are copied to form a distributed data set that can be operated on in parallel.

__Refer to the documentation and use this method to create an RDD (call it `intRDD`) with 5 partitions for the list of integers in `data`, calculated earlier. Remember to intialize a SparkContext first.__ 


```python
# Import spark and initiate a local SparkContext (execute only once)
import pyspark
sc = pyspark.SparkContext('local[*]')
```


```python
# Create an RDD with list created earlier, splitting it into 5 partitions
intRDD = sc.parallelize(data,5)
```

Remember due to lazy evaluation used by Spark, no Spark computation (a.k.a a Spark job) is performed at this state. We can see more options for creating new RDDs by studying built in help on `sc.parallelize()`.


```python
# Get help on sc.parallelize() method
help(sc.parallelize)
```

    Help on method parallelize in module pyspark.context:
    
    parallelize(c, numSlices=None) method of pyspark.context.SparkContext instance
        Distribute a local Python collection to form an RDD. Using xrange
        is recommended if the input represents a range for performance.
        
        >>> sc.parallelize([0, 2, 3, 4, 6], 5).glom().collect()
        [[0], [2], [3], [4], [6]]
        >>> sc.parallelize(xrange(0, 6, 2), 5).glom().collect()
        [[], [0], [], [2], [4]]
    


Note: `glom()` method allows treating a partition as a multidimensional array rather as single row at time allowing to speed up some operations with some increased memory usage. More information on `glom()` can be found [here](http://blog.madhukaraphatak.com/glom-in-spark/). We shall also see how `collect()` action works later in this lab. 

### Types of RDDs

There are many different types of RDDs in Spark. The base class for RDDs is `pyspark.RDD`. Since the other RDD types inherit from pyspark.RDD they have the same APIs and are functionally identical. [Visit this document](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-rdd.html) to see a list of Common RDD types.



Initializing `sc.parallelize()` with a list generates a `pyspark.rdd.PipelinedRDD`. Let's check the type of our RDD to confirm this.   


```python
# Check the type of RDD created earlier, i.e. intRDD

type(intRDD)

# pyspark.rdd.PipelinedRDD
```




    pyspark.rdd.PipelinedRDD



### RDD `id` and `setName()`
Each RDD created this way has a unique ID which is used by the Spark system to refer to the RDD. this can be viewed by using `RDD.id()` function. Let's check the id of RDD we have just created. 

*Note that this value may be different depending upon the sequence of RDD creation and code execution.*


```python
# Each RDD gets a unique ID
print ('intRDD id: {0}'.format(intRDD.id()))

# intRDD id: 3
```

    intRDD id: 1


Spark also allows naming of RDDs after they've been created using the `setname()` method. Let's use this method to set the name of our RDD as "My First RDD""


```python
# We can name each newly created RDD using the setName() method
intRDD.setName('My first RDD')

# My first RDD PythonRDD[3] at RDD at PythonRDD.scala:49
```




    My first RDD PythonRDD[1] at RDD at PythonRDD.scala:53



### RDD Lineage
RDD Lineage (aka RDD operator graph or RDD dependency graph) is a graph of all the parent RDDs of a RDD. It is built as a result of applying transformations to the RDD and creates a logical execution plan. 

<img src="lineage.JPG" width=400>


We can view the lineage (set of transformations) using the `RDD.toDebugString()` method. 

Let's check the lineage of our RDD using this method. 


```python
# Get RDD Dependancy Graph
intRDD.toDebugString()

# b'(4) PythonRDD[2] at RDD at PythonRDD.scala:49 []\n |  
# ParallelCollectionRDD[1] at parallelize at PythonRDD.scala:184 []'
```




    b'(4) PythonRDD[2] at RDD at PythonRDD.scala:49 []\n |  ParallelCollectionRDD[1] at parallelize at PythonRDD.scala:184 []'



You are encouraged to call this property at different stages of data processing to view the dependancy between RDDs. We will see this in effect in our upcoming labs with transformations. 

In order to check the methods and transformations that can be applied to RDDs, we can simply use the `help()` function.


```python
help(intRDD)
```

    Help on PipelinedRDD in module pyspark.rdd object:
    
    class PipelinedRDD(RDD)
     |  Pipelined maps:
     |  
     |  >>> rdd = sc.parallelize([1, 2, 3, 4])
     |  >>> rdd.map(lambda x: 2 * x).cache().map(lambda x: 2 * x).collect()
     |  [4, 8, 12, 16]
     |  >>> rdd.map(lambda x: 2 * x).map(lambda x: 2 * x).collect()
     |  [4, 8, 12, 16]
     |  
     |  Pipelined reduces:
     |  >>> from operator import add
     |  >>> rdd.map(lambda x: 2 * x).reduce(add)
     |  20
     |  >>> rdd.flatMap(lambda x: [x, x]).reduce(add)
     |  20
     |  
     |  Method resolution order:
     |      PipelinedRDD
     |      RDD
     |      builtins.object
     |  
     |  Methods defined here:
     |  
     |  __init__(self, prev, func, preservesPartitioning=False)
     |      Initialize self.  See help(type(self)) for accurate signature.
     |  
     |  getNumPartitions(self)
     |      Returns the number of partitions in RDD
     |      
     |      >>> rdd = sc.parallelize([1, 2, 3, 4], 2)
     |      >>> rdd.getNumPartitions()
     |      2
     |  
     |  id(self)
     |      A unique ID for this RDD (within its SparkContext).
     |  
     |  ----------------------------------------------------------------------
     |  Methods inherited from RDD:
     |  
     |  __add__(self, other)
     |      Return the union of this RDD and another one.
     |      
     |      >>> rdd = sc.parallelize([1, 1, 2, 3])
     |      >>> (rdd + rdd).collect()
     |      [1, 1, 2, 3, 1, 1, 2, 3]
     |  
     |  __getnewargs__(self)
     |  
     |  __repr__(self)
     |      Return repr(self).
     |  
     |  aggregate(self, zeroValue, seqOp, combOp)
     |      Aggregate the elements of each partition, and then the results for all
     |      the partitions, using a given combine functions and a neutral "zero
     |      value."
     |      
     |      The functions C{op(t1, t2)} is allowed to modify C{t1} and return it
     |      as its result value to avoid object allocation; however, it should not
     |      modify C{t2}.
     |      
     |      The first function (seqOp) can return a different result type, U, than
     |      the type of this RDD. Thus, we need one operation for merging a T into
     |      an U and one operation for merging two U
     |      
     |      >>> seqOp = (lambda x, y: (x[0] + y, x[1] + 1))
     |      >>> combOp = (lambda x, y: (x[0] + y[0], x[1] + y[1]))
     |      >>> sc.parallelize([1, 2, 3, 4]).aggregate((0, 0), seqOp, combOp)
     |      (10, 4)
     |      >>> sc.parallelize([]).aggregate((0, 0), seqOp, combOp)
     |      (0, 0)
     |  
     |  aggregateByKey(self, zeroValue, seqFunc, combFunc, numPartitions=None, partitionFunc=<function portable_hash at 0x7ff918446c80>)
     |      Aggregate the values of each key, using given combine functions and a neutral
     |      "zero value". This function can return a different result type, U, than the type
     |      of the values in this RDD, V. Thus, we need one operation for merging a V into
     |      a U and one operation for merging two U's, The former operation is used for merging
     |      values within a partition, and the latter is used for merging values between
     |      partitions. To avoid memory allocation, both of these functions are
     |      allowed to modify and return their first argument instead of creating a new U.
     |  
     |  cache(self)
     |      Persist this RDD with the default storage level (C{MEMORY_ONLY}).
     |  
     |  cartesian(self, other)
     |      Return the Cartesian product of this RDD and another one, that is, the
     |      RDD of all pairs of elements C{(a, b)} where C{a} is in C{self} and
     |      C{b} is in C{other}.
     |      
     |      >>> rdd = sc.parallelize([1, 2])
     |      >>> sorted(rdd.cartesian(rdd).collect())
     |      [(1, 1), (1, 2), (2, 1), (2, 2)]
     |  
     |  checkpoint(self)
     |      Mark this RDD for checkpointing. It will be saved to a file inside the
     |      checkpoint directory set with L{SparkContext.setCheckpointDir()} and
     |      all references to its parent RDDs will be removed. This function must
     |      be called before any job has been executed on this RDD. It is strongly
     |      recommended that this RDD is persisted in memory, otherwise saving it
     |      on a file will require recomputation.
     |  
     |  coalesce(self, numPartitions, shuffle=False)
     |      Return a new RDD that is reduced into `numPartitions` partitions.
     |      
     |      >>> sc.parallelize([1, 2, 3, 4, 5], 3).glom().collect()
     |      [[1], [2, 3], [4, 5]]
     |      >>> sc.parallelize([1, 2, 3, 4, 5], 3).coalesce(1).glom().collect()
     |      [[1, 2, 3, 4, 5]]
     |  
     |  cogroup(self, other, numPartitions=None)
     |      For each key k in C{self} or C{other}, return a resulting RDD that
     |      contains a tuple with the list of values for that key in C{self} as
     |      well as C{other}.
     |      
     |      >>> x = sc.parallelize([("a", 1), ("b", 4)])
     |      >>> y = sc.parallelize([("a", 2)])
     |      >>> [(x, tuple(map(list, y))) for x, y in sorted(list(x.cogroup(y).collect()))]
     |      [('a', ([1], [2])), ('b', ([4], []))]
     |  
     |  collect(self)
     |      Return a list that contains all of the elements in this RDD.
     |      
     |      .. note:: This method should only be used if the resulting array is expected
     |          to be small, as all the data is loaded into the driver's memory.
     |  
     |  collectAsMap(self)
     |      Return the key-value pairs in this RDD to the master as a dictionary.
     |      
     |      .. note:: this method should only be used if the resulting data is expected
     |          to be small, as all the data is loaded into the driver's memory.
     |      
     |      >>> m = sc.parallelize([(1, 2), (3, 4)]).collectAsMap()
     |      >>> m[1]
     |      2
     |      >>> m[3]
     |      4
     |  
     |  combineByKey(self, createCombiner, mergeValue, mergeCombiners, numPartitions=None, partitionFunc=<function portable_hash at 0x7ff918446c80>)
     |      Generic function to combine the elements for each key using a custom
     |      set of aggregation functions.
     |      
     |      Turns an RDD[(K, V)] into a result of type RDD[(K, C)], for a "combined
     |      type" C.
     |      
     |      Users provide three functions:
     |      
     |          - C{createCombiner}, which turns a V into a C (e.g., creates
     |            a one-element list)
     |          - C{mergeValue}, to merge a V into a C (e.g., adds it to the end of
     |            a list)
     |          - C{mergeCombiners}, to combine two C's into a single one (e.g., merges
     |            the lists)
     |      
     |      To avoid memory allocation, both mergeValue and mergeCombiners are allowed to
     |      modify and return their first argument instead of creating a new C.
     |      
     |      In addition, users can control the partitioning of the output RDD.
     |      
     |      .. note:: V and C can be different -- for example, one might group an RDD of type
     |          (Int, Int) into an RDD of type (Int, List[Int]).
     |      
     |      >>> x = sc.parallelize([("a", 1), ("b", 1), ("a", 2)])
     |      >>> def to_list(a):
     |      ...     return [a]
     |      ...
     |      >>> def append(a, b):
     |      ...     a.append(b)
     |      ...     return a
     |      ...
     |      >>> def extend(a, b):
     |      ...     a.extend(b)
     |      ...     return a
     |      ...
     |      >>> sorted(x.combineByKey(to_list, append, extend).collect())
     |      [('a', [1, 2]), ('b', [1])]
     |  
     |  count(self)
     |      Return the number of elements in this RDD.
     |      
     |      >>> sc.parallelize([2, 3, 4]).count()
     |      3
     |  
     |  countApprox(self, timeout, confidence=0.95)
     |      .. note:: Experimental
     |      
     |      Approximate version of count() that returns a potentially incomplete
     |      result within a timeout, even if not all tasks have finished.
     |      
     |      >>> rdd = sc.parallelize(range(1000), 10)
     |      >>> rdd.countApprox(1000, 1.0)
     |      1000
     |  
     |  countApproxDistinct(self, relativeSD=0.05)
     |      .. note:: Experimental
     |      
     |      Return approximate number of distinct elements in the RDD.
     |      
     |      The algorithm used is based on streamlib's implementation of
     |      `"HyperLogLog in Practice: Algorithmic Engineering of a State
     |      of The Art Cardinality Estimation Algorithm", available here
     |      <http://dx.doi.org/10.1145/2452376.2452456>`_.
     |      
     |      :param relativeSD: Relative accuracy. Smaller values create
     |                         counters that require more space.
     |                         It must be greater than 0.000017.
     |      
     |      >>> n = sc.parallelize(range(1000)).map(str).countApproxDistinct()
     |      >>> 900 < n < 1100
     |      True
     |      >>> n = sc.parallelize([i % 20 for i in range(1000)]).countApproxDistinct()
     |      >>> 16 < n < 24
     |      True
     |  
     |  countByKey(self)
     |      Count the number of elements for each key, and return the result to the
     |      master as a dictionary.
     |      
     |      >>> rdd = sc.parallelize([("a", 1), ("b", 1), ("a", 1)])
     |      >>> sorted(rdd.countByKey().items())
     |      [('a', 2), ('b', 1)]
     |  
     |  countByValue(self)
     |      Return the count of each unique value in this RDD as a dictionary of
     |      (value, count) pairs.
     |      
     |      >>> sorted(sc.parallelize([1, 2, 1, 2, 2], 2).countByValue().items())
     |      [(1, 2), (2, 3)]
     |  
     |  distinct(self, numPartitions=None)
     |      Return a new RDD containing the distinct elements in this RDD.
     |      
     |      >>> sorted(sc.parallelize([1, 1, 2, 3]).distinct().collect())
     |      [1, 2, 3]
     |  
     |  filter(self, f)
     |      Return a new RDD containing only the elements that satisfy a predicate.
     |      
     |      >>> rdd = sc.parallelize([1, 2, 3, 4, 5])
     |      >>> rdd.filter(lambda x: x % 2 == 0).collect()
     |      [2, 4]
     |  
     |  first(self)
     |      Return the first element in this RDD.
     |      
     |      >>> sc.parallelize([2, 3, 4]).first()
     |      2
     |      >>> sc.parallelize([]).first()
     |      Traceback (most recent call last):
     |          ...
     |      ValueError: RDD is empty
     |  
     |  flatMap(self, f, preservesPartitioning=False)
     |      Return a new RDD by first applying a function to all elements of this
     |      RDD, and then flattening the results.
     |      
     |      >>> rdd = sc.parallelize([2, 3, 4])
     |      >>> sorted(rdd.flatMap(lambda x: range(1, x)).collect())
     |      [1, 1, 1, 2, 2, 3]
     |      >>> sorted(rdd.flatMap(lambda x: [(x, x), (x, x)]).collect())
     |      [(2, 2), (2, 2), (3, 3), (3, 3), (4, 4), (4, 4)]
     |  
     |  flatMapValues(self, f)
     |      Pass each value in the key-value pair RDD through a flatMap function
     |      without changing the keys; this also retains the original RDD's
     |      partitioning.
     |      
     |      >>> x = sc.parallelize([("a", ["x", "y", "z"]), ("b", ["p", "r"])])
     |      >>> def f(x): return x
     |      >>> x.flatMapValues(f).collect()
     |      [('a', 'x'), ('a', 'y'), ('a', 'z'), ('b', 'p'), ('b', 'r')]
     |  
     |  fold(self, zeroValue, op)
     |      Aggregate the elements of each partition, and then the results for all
     |      the partitions, using a given associative function and a neutral "zero value."
     |      
     |      The function C{op(t1, t2)} is allowed to modify C{t1} and return it
     |      as its result value to avoid object allocation; however, it should not
     |      modify C{t2}.
     |      
     |      This behaves somewhat differently from fold operations implemented
     |      for non-distributed collections in functional languages like Scala.
     |      This fold operation may be applied to partitions individually, and then
     |      fold those results into the final result, rather than apply the fold
     |      to each element sequentially in some defined ordering. For functions
     |      that are not commutative, the result may differ from that of a fold
     |      applied to a non-distributed collection.
     |      
     |      >>> from operator import add
     |      >>> sc.parallelize([1, 2, 3, 4, 5]).fold(0, add)
     |      15
     |  
     |  foldByKey(self, zeroValue, func, numPartitions=None, partitionFunc=<function portable_hash at 0x7ff918446c80>)
     |      Merge the values for each key using an associative function "func"
     |      and a neutral "zeroValue" which may be added to the result an
     |      arbitrary number of times, and must not change the result
     |      (e.g., 0 for addition, or 1 for multiplication.).
     |      
     |      >>> rdd = sc.parallelize([("a", 1), ("b", 1), ("a", 1)])
     |      >>> from operator import add
     |      >>> sorted(rdd.foldByKey(0, add).collect())
     |      [('a', 2), ('b', 1)]
     |  
     |  foreach(self, f)
     |      Applies a function to all elements of this RDD.
     |      
     |      >>> def f(x): print(x)
     |      >>> sc.parallelize([1, 2, 3, 4, 5]).foreach(f)
     |  
     |  foreachPartition(self, f)
     |      Applies a function to each partition of this RDD.
     |      
     |      >>> def f(iterator):
     |      ...     for x in iterator:
     |      ...          print(x)
     |      >>> sc.parallelize([1, 2, 3, 4, 5]).foreachPartition(f)
     |  
     |  fullOuterJoin(self, other, numPartitions=None)
     |      Perform a right outer join of C{self} and C{other}.
     |      
     |      For each element (k, v) in C{self}, the resulting RDD will either
     |      contain all pairs (k, (v, w)) for w in C{other}, or the pair
     |      (k, (v, None)) if no elements in C{other} have key k.
     |      
     |      Similarly, for each element (k, w) in C{other}, the resulting RDD will
     |      either contain all pairs (k, (v, w)) for v in C{self}, or the pair
     |      (k, (None, w)) if no elements in C{self} have key k.
     |      
     |      Hash-partitions the resulting RDD into the given number of partitions.
     |      
     |      >>> x = sc.parallelize([("a", 1), ("b", 4)])
     |      >>> y = sc.parallelize([("a", 2), ("c", 8)])
     |      >>> sorted(x.fullOuterJoin(y).collect())
     |      [('a', (1, 2)), ('b', (4, None)), ('c', (None, 8))]
     |  
     |  getCheckpointFile(self)
     |      Gets the name of the file to which this RDD was checkpointed
     |      
     |      Not defined if RDD is checkpointed locally.
     |  
     |  getStorageLevel(self)
     |      Get the RDD's current storage level.
     |      
     |      >>> rdd1 = sc.parallelize([1,2])
     |      >>> rdd1.getStorageLevel()
     |      StorageLevel(False, False, False, False, 1)
     |      >>> print(rdd1.getStorageLevel())
     |      Serialized 1x Replicated
     |  
     |  glom(self)
     |      Return an RDD created by coalescing all elements within each partition
     |      into a list.
     |      
     |      >>> rdd = sc.parallelize([1, 2, 3, 4], 2)
     |      >>> sorted(rdd.glom().collect())
     |      [[1, 2], [3, 4]]
     |  
     |  groupBy(self, f, numPartitions=None, partitionFunc=<function portable_hash at 0x7ff918446c80>)
     |      Return an RDD of grouped items.
     |      
     |      >>> rdd = sc.parallelize([1, 1, 2, 3, 5, 8])
     |      >>> result = rdd.groupBy(lambda x: x % 2).collect()
     |      >>> sorted([(x, sorted(y)) for (x, y) in result])
     |      [(0, [2, 8]), (1, [1, 1, 3, 5])]
     |  
     |  groupByKey(self, numPartitions=None, partitionFunc=<function portable_hash at 0x7ff918446c80>)
     |      Group the values for each key in the RDD into a single sequence.
     |      Hash-partitions the resulting RDD with numPartitions partitions.
     |      
     |      .. note:: If you are grouping in order to perform an aggregation (such as a
     |          sum or average) over each key, using reduceByKey or aggregateByKey will
     |          provide much better performance.
     |      
     |      >>> rdd = sc.parallelize([("a", 1), ("b", 1), ("a", 1)])
     |      >>> sorted(rdd.groupByKey().mapValues(len).collect())
     |      [('a', 2), ('b', 1)]
     |      >>> sorted(rdd.groupByKey().mapValues(list).collect())
     |      [('a', [1, 1]), ('b', [1])]
     |  
     |  groupWith(self, other, *others)
     |      Alias for cogroup but with support for multiple RDDs.
     |      
     |      >>> w = sc.parallelize([("a", 5), ("b", 6)])
     |      >>> x = sc.parallelize([("a", 1), ("b", 4)])
     |      >>> y = sc.parallelize([("a", 2)])
     |      >>> z = sc.parallelize([("b", 42)])
     |      >>> [(x, tuple(map(list, y))) for x, y in sorted(list(w.groupWith(x, y, z).collect()))]
     |      [('a', ([5], [1], [2], [])), ('b', ([6], [4], [], [42]))]
     |  
     |  histogram(self, buckets)
     |      Compute a histogram using the provided buckets. The buckets
     |      are all open to the right except for the last which is closed.
     |      e.g. [1,10,20,50] means the buckets are [1,10) [10,20) [20,50],
     |      which means 1<=x<10, 10<=x<20, 20<=x<=50. And on the input of 1
     |      and 50 we would have a histogram of 1,0,1.
     |      
     |      If your histogram is evenly spaced (e.g. [0, 10, 20, 30]),
     |      this can be switched from an O(log n) inseration to O(1) per
     |      element (where n is the number of buckets).
     |      
     |      Buckets must be sorted, not contain any duplicates, and have
     |      at least two elements.
     |      
     |      If `buckets` is a number, it will generate buckets which are
     |      evenly spaced between the minimum and maximum of the RDD. For
     |      example, if the min value is 0 and the max is 100, given `buckets`
     |      as 2, the resulting buckets will be [0,50) [50,100]. `buckets` must
     |      be at least 1. An exception is raised if the RDD contains infinity.
     |      If the elements in the RDD do not vary (max == min), a single bucket
     |      will be used.
     |      
     |      The return value is a tuple of buckets and histogram.
     |      
     |      >>> rdd = sc.parallelize(range(51))
     |      >>> rdd.histogram(2)
     |      ([0, 25, 50], [25, 26])
     |      >>> rdd.histogram([0, 5, 25, 50])
     |      ([0, 5, 25, 50], [5, 20, 26])
     |      >>> rdd.histogram([0, 15, 30, 45, 60])  # evenly spaced buckets
     |      ([0, 15, 30, 45, 60], [15, 15, 15, 6])
     |      >>> rdd = sc.parallelize(["ab", "ac", "b", "bd", "ef"])
     |      >>> rdd.histogram(("a", "b", "c"))
     |      (('a', 'b', 'c'), [2, 2])
     |  
     |  intersection(self, other)
     |      Return the intersection of this RDD and another one. The output will
     |      not contain any duplicate elements, even if the input RDDs did.
     |      
     |      .. note:: This method performs a shuffle internally.
     |      
     |      >>> rdd1 = sc.parallelize([1, 10, 2, 3, 4, 5])
     |      >>> rdd2 = sc.parallelize([1, 6, 2, 3, 7, 8])
     |      >>> rdd1.intersection(rdd2).collect()
     |      [1, 2, 3]
     |  
     |  isCheckpointed(self)
     |      Return whether this RDD is checkpointed and materialized, either reliably or locally.
     |  
     |  isEmpty(self)
     |      Returns true if and only if the RDD contains no elements at all.
     |      
     |      .. note:: an RDD may be empty even when it has at least 1 partition.
     |      
     |      >>> sc.parallelize([]).isEmpty()
     |      True
     |      >>> sc.parallelize([1]).isEmpty()
     |      False
     |  
     |  isLocallyCheckpointed(self)
     |      Return whether this RDD is marked for local checkpointing.
     |      
     |      Exposed for testing.
     |  
     |  join(self, other, numPartitions=None)
     |      Return an RDD containing all pairs of elements with matching keys in
     |      C{self} and C{other}.
     |      
     |      Each pair of elements will be returned as a (k, (v1, v2)) tuple, where
     |      (k, v1) is in C{self} and (k, v2) is in C{other}.
     |      
     |      Performs a hash join across the cluster.
     |      
     |      >>> x = sc.parallelize([("a", 1), ("b", 4)])
     |      >>> y = sc.parallelize([("a", 2), ("a", 3)])
     |      >>> sorted(x.join(y).collect())
     |      [('a', (1, 2)), ('a', (1, 3))]
     |  
     |  keyBy(self, f)
     |      Creates tuples of the elements in this RDD by applying C{f}.
     |      
     |      >>> x = sc.parallelize(range(0,3)).keyBy(lambda x: x*x)
     |      >>> y = sc.parallelize(zip(range(0,5), range(0,5)))
     |      >>> [(x, list(map(list, y))) for x, y in sorted(x.cogroup(y).collect())]
     |      [(0, [[0], [0]]), (1, [[1], [1]]), (2, [[], [2]]), (3, [[], [3]]), (4, [[2], [4]])]
     |  
     |  keys(self)
     |      Return an RDD with the keys of each tuple.
     |      
     |      >>> m = sc.parallelize([(1, 2), (3, 4)]).keys()
     |      >>> m.collect()
     |      [1, 3]
     |  
     |  leftOuterJoin(self, other, numPartitions=None)
     |      Perform a left outer join of C{self} and C{other}.
     |      
     |      For each element (k, v) in C{self}, the resulting RDD will either
     |      contain all pairs (k, (v, w)) for w in C{other}, or the pair
     |      (k, (v, None)) if no elements in C{other} have key k.
     |      
     |      Hash-partitions the resulting RDD into the given number of partitions.
     |      
     |      >>> x = sc.parallelize([("a", 1), ("b", 4)])
     |      >>> y = sc.parallelize([("a", 2)])
     |      >>> sorted(x.leftOuterJoin(y).collect())
     |      [('a', (1, 2)), ('b', (4, None))]
     |  
     |  localCheckpoint(self)
     |      Mark this RDD for local checkpointing using Spark's existing caching layer.
     |      
     |      This method is for users who wish to truncate RDD lineages while skipping the expensive
     |      step of replicating the materialized data in a reliable distributed file system. This is
     |      useful for RDDs with long lineages that need to be truncated periodically (e.g. GraphX).
     |      
     |      Local checkpointing sacrifices fault-tolerance for performance. In particular, checkpointed
     |      data is written to ephemeral local storage in the executors instead of to a reliable,
     |      fault-tolerant storage. The effect is that if an executor fails during the computation,
     |      the checkpointed data may no longer be accessible, causing an irrecoverable job failure.
     |      
     |      This is NOT safe to use with dynamic allocation, which removes executors along
     |      with their cached blocks. If you must use both features, you are advised to set
     |      L{spark.dynamicAllocation.cachedExecutorIdleTimeout} to a high value.
     |      
     |      The checkpoint directory set through L{SparkContext.setCheckpointDir()} is not used.
     |  
     |  lookup(self, key)
     |      Return the list of values in the RDD for key `key`. This operation
     |      is done efficiently if the RDD has a known partitioner by only
     |      searching the partition that the key maps to.
     |      
     |      >>> l = range(1000)
     |      >>> rdd = sc.parallelize(zip(l, l), 10)
     |      >>> rdd.lookup(42)  # slow
     |      [42]
     |      >>> sorted = rdd.sortByKey()
     |      >>> sorted.lookup(42)  # fast
     |      [42]
     |      >>> sorted.lookup(1024)
     |      []
     |      >>> rdd2 = sc.parallelize([(('a', 'b'), 'c')]).groupByKey()
     |      >>> list(rdd2.lookup(('a', 'b'))[0])
     |      ['c']
     |  
     |  map(self, f, preservesPartitioning=False)
     |      Return a new RDD by applying a function to each element of this RDD.
     |      
     |      >>> rdd = sc.parallelize(["b", "a", "c"])
     |      >>> sorted(rdd.map(lambda x: (x, 1)).collect())
     |      [('a', 1), ('b', 1), ('c', 1)]
     |  
     |  mapPartitions(self, f, preservesPartitioning=False)
     |      Return a new RDD by applying a function to each partition of this RDD.
     |      
     |      >>> rdd = sc.parallelize([1, 2, 3, 4], 2)
     |      >>> def f(iterator): yield sum(iterator)
     |      >>> rdd.mapPartitions(f).collect()
     |      [3, 7]
     |  
     |  mapPartitionsWithIndex(self, f, preservesPartitioning=False)
     |      Return a new RDD by applying a function to each partition of this RDD,
     |      while tracking the index of the original partition.
     |      
     |      >>> rdd = sc.parallelize([1, 2, 3, 4], 4)
     |      >>> def f(splitIndex, iterator): yield splitIndex
     |      >>> rdd.mapPartitionsWithIndex(f).sum()
     |      6
     |  
     |  mapPartitionsWithSplit(self, f, preservesPartitioning=False)
     |      Deprecated: use mapPartitionsWithIndex instead.
     |      
     |      Return a new RDD by applying a function to each partition of this RDD,
     |      while tracking the index of the original partition.
     |      
     |      >>> rdd = sc.parallelize([1, 2, 3, 4], 4)
     |      >>> def f(splitIndex, iterator): yield splitIndex
     |      >>> rdd.mapPartitionsWithSplit(f).sum()
     |      6
     |  
     |  mapValues(self, f)
     |      Pass each value in the key-value pair RDD through a map function
     |      without changing the keys; this also retains the original RDD's
     |      partitioning.
     |      
     |      >>> x = sc.parallelize([("a", ["apple", "banana", "lemon"]), ("b", ["grapes"])])
     |      >>> def f(x): return len(x)
     |      >>> x.mapValues(f).collect()
     |      [('a', 3), ('b', 1)]
     |  
     |  max(self, key=None)
     |      Find the maximum item in this RDD.
     |      
     |      :param key: A function used to generate key for comparing
     |      
     |      >>> rdd = sc.parallelize([1.0, 5.0, 43.0, 10.0])
     |      >>> rdd.max()
     |      43.0
     |      >>> rdd.max(key=str)
     |      5.0
     |  
     |  mean(self)
     |      Compute the mean of this RDD's elements.
     |      
     |      >>> sc.parallelize([1, 2, 3]).mean()
     |      2.0
     |  
     |  meanApprox(self, timeout, confidence=0.95)
     |      .. note:: Experimental
     |      
     |      Approximate operation to return the mean within a timeout
     |      or meet the confidence.
     |      
     |      >>> rdd = sc.parallelize(range(1000), 10)
     |      >>> r = sum(range(1000)) / 1000.0
     |      >>> abs(rdd.meanApprox(1000) - r) / r < 0.05
     |      True
     |  
     |  min(self, key=None)
     |      Find the minimum item in this RDD.
     |      
     |      :param key: A function used to generate key for comparing
     |      
     |      >>> rdd = sc.parallelize([2.0, 5.0, 43.0, 10.0])
     |      >>> rdd.min()
     |      2.0
     |      >>> rdd.min(key=str)
     |      10.0
     |  
     |  name(self)
     |      Return the name of this RDD.
     |  
     |  partitionBy(self, numPartitions, partitionFunc=<function portable_hash at 0x7ff918446c80>)
     |      Return a copy of the RDD partitioned using the specified partitioner.
     |      
     |      >>> pairs = sc.parallelize([1, 2, 3, 4, 2, 4, 1]).map(lambda x: (x, x))
     |      >>> sets = pairs.partitionBy(2).glom().collect()
     |      >>> len(set(sets[0]).intersection(set(sets[1])))
     |      0
     |  
     |  persist(self, storageLevel=StorageLevel(False, True, False, False, 1))
     |      Set this RDD's storage level to persist its values across operations
     |      after the first time it is computed. This can only be used to assign
     |      a new storage level if the RDD does not have a storage level set yet.
     |      If no storage level is specified defaults to (C{MEMORY_ONLY}).
     |      
     |      >>> rdd = sc.parallelize(["b", "a", "c"])
     |      >>> rdd.persist().is_cached
     |      True
     |  
     |  pipe(self, command, env=None, checkCode=False)
     |      Return an RDD created by piping elements to a forked external process.
     |      
     |      >>> sc.parallelize(['1', '2', '', '3']).pipe('cat').collect()
     |      ['1', '2', '', '3']
     |      
     |      :param checkCode: whether or not to check the return value of the shell command.
     |  
     |  randomSplit(self, weights, seed=None)
     |      Randomly splits this RDD with the provided weights.
     |      
     |      :param weights: weights for splits, will be normalized if they don't sum to 1
     |      :param seed: random seed
     |      :return: split RDDs in a list
     |      
     |      >>> rdd = sc.parallelize(range(500), 1)
     |      >>> rdd1, rdd2 = rdd.randomSplit([2, 3], 17)
     |      >>> len(rdd1.collect() + rdd2.collect())
     |      500
     |      >>> 150 < rdd1.count() < 250
     |      True
     |      >>> 250 < rdd2.count() < 350
     |      True
     |  
     |  reduce(self, f)
     |      Reduces the elements of this RDD using the specified commutative and
     |      associative binary operator. Currently reduces partitions locally.
     |      
     |      >>> from operator import add
     |      >>> sc.parallelize([1, 2, 3, 4, 5]).reduce(add)
     |      15
     |      >>> sc.parallelize((2 for _ in range(10))).map(lambda x: 1).cache().reduce(add)
     |      10
     |      >>> sc.parallelize([]).reduce(add)
     |      Traceback (most recent call last):
     |          ...
     |      ValueError: Can not reduce() empty RDD
     |  
     |  reduceByKey(self, func, numPartitions=None, partitionFunc=<function portable_hash at 0x7ff918446c80>)
     |      Merge the values for each key using an associative and commutative reduce function.
     |      
     |      This will also perform the merging locally on each mapper before
     |      sending results to a reducer, similarly to a "combiner" in MapReduce.
     |      
     |      Output will be partitioned with C{numPartitions} partitions, or
     |      the default parallelism level if C{numPartitions} is not specified.
     |      Default partitioner is hash-partition.
     |      
     |      >>> from operator import add
     |      >>> rdd = sc.parallelize([("a", 1), ("b", 1), ("a", 1)])
     |      >>> sorted(rdd.reduceByKey(add).collect())
     |      [('a', 2), ('b', 1)]
     |  
     |  reduceByKeyLocally(self, func)
     |      Merge the values for each key using an associative and commutative reduce function, but
     |      return the results immediately to the master as a dictionary.
     |      
     |      This will also perform the merging locally on each mapper before
     |      sending results to a reducer, similarly to a "combiner" in MapReduce.
     |      
     |      >>> from operator import add
     |      >>> rdd = sc.parallelize([("a", 1), ("b", 1), ("a", 1)])
     |      >>> sorted(rdd.reduceByKeyLocally(add).items())
     |      [('a', 2), ('b', 1)]
     |  
     |  repartition(self, numPartitions)
     |      Return a new RDD that has exactly numPartitions partitions.
     |      
     |      Can increase or decrease the level of parallelism in this RDD.
     |      Internally, this uses a shuffle to redistribute data.
     |      If you are decreasing the number of partitions in this RDD, consider
     |      using `coalesce`, which can avoid performing a shuffle.
     |      
     |      >>> rdd = sc.parallelize([1,2,3,4,5,6,7], 4)
     |      >>> sorted(rdd.glom().collect())
     |      [[1], [2, 3], [4, 5], [6, 7]]
     |      >>> len(rdd.repartition(2).glom().collect())
     |      2
     |      >>> len(rdd.repartition(10).glom().collect())
     |      10
     |  
     |  repartitionAndSortWithinPartitions(self, numPartitions=None, partitionFunc=<function portable_hash at 0x7ff918446c80>, ascending=True, keyfunc=<function RDD.<lambda> at 0x7ff9180dc378>)
     |      Repartition the RDD according to the given partitioner and, within each resulting partition,
     |      sort records by their keys.
     |      
     |      >>> rdd = sc.parallelize([(0, 5), (3, 8), (2, 6), (0, 8), (3, 8), (1, 3)])
     |      >>> rdd2 = rdd.repartitionAndSortWithinPartitions(2, lambda x: x % 2, True)
     |      >>> rdd2.glom().collect()
     |      [[(0, 5), (0, 8), (2, 6)], [(1, 3), (3, 8), (3, 8)]]
     |  
     |  rightOuterJoin(self, other, numPartitions=None)
     |      Perform a right outer join of C{self} and C{other}.
     |      
     |      For each element (k, w) in C{other}, the resulting RDD will either
     |      contain all pairs (k, (v, w)) for v in this, or the pair (k, (None, w))
     |      if no elements in C{self} have key k.
     |      
     |      Hash-partitions the resulting RDD into the given number of partitions.
     |      
     |      >>> x = sc.parallelize([("a", 1), ("b", 4)])
     |      >>> y = sc.parallelize([("a", 2)])
     |      >>> sorted(y.rightOuterJoin(x).collect())
     |      [('a', (2, 1)), ('b', (None, 4))]
     |  
     |  sample(self, withReplacement, fraction, seed=None)
     |      Return a sampled subset of this RDD.
     |      
     |      :param withReplacement: can elements be sampled multiple times (replaced when sampled out)
     |      :param fraction: expected size of the sample as a fraction of this RDD's size
     |          without replacement: probability that each element is chosen; fraction must be [0, 1]
     |          with replacement: expected number of times each element is chosen; fraction must be >= 0
     |      :param seed: seed for the random number generator
     |      
     |      .. note:: This is not guaranteed to provide exactly the fraction specified of the total
     |          count of the given :class:`DataFrame`.
     |      
     |      >>> rdd = sc.parallelize(range(100), 4)
     |      >>> 6 <= rdd.sample(False, 0.1, 81).count() <= 14
     |      True
     |  
     |  sampleByKey(self, withReplacement, fractions, seed=None)
     |      Return a subset of this RDD sampled by key (via stratified sampling).
     |      Create a sample of this RDD using variable sampling rates for
     |      different keys as specified by fractions, a key to sampling rate map.
     |      
     |      >>> fractions = {"a": 0.2, "b": 0.1}
     |      >>> rdd = sc.parallelize(fractions.keys()).cartesian(sc.parallelize(range(0, 1000)))
     |      >>> sample = dict(rdd.sampleByKey(False, fractions, 2).groupByKey().collect())
     |      >>> 100 < len(sample["a"]) < 300 and 50 < len(sample["b"]) < 150
     |      True
     |      >>> max(sample["a"]) <= 999 and min(sample["a"]) >= 0
     |      True
     |      >>> max(sample["b"]) <= 999 and min(sample["b"]) >= 0
     |      True
     |  
     |  sampleStdev(self)
     |      Compute the sample standard deviation of this RDD's elements (which
     |      corrects for bias in estimating the standard deviation by dividing by
     |      N-1 instead of N).
     |      
     |      >>> sc.parallelize([1, 2, 3]).sampleStdev()
     |      1.0
     |  
     |  sampleVariance(self)
     |      Compute the sample variance of this RDD's elements (which corrects
     |      for bias in estimating the variance by dividing by N-1 instead of N).
     |      
     |      >>> sc.parallelize([1, 2, 3]).sampleVariance()
     |      1.0
     |  
     |  saveAsHadoopDataset(self, conf, keyConverter=None, valueConverter=None)
     |      Output a Python RDD of key-value pairs (of form C{RDD[(K, V)]}) to any Hadoop file
     |      system, using the old Hadoop OutputFormat API (mapred package). Keys/values are
     |      converted for output using either user specified converters or, by default,
     |      L{org.apache.spark.api.python.JavaToWritableConverter}.
     |      
     |      :param conf: Hadoop job configuration, passed in as a dict
     |      :param keyConverter: (None by default)
     |      :param valueConverter: (None by default)
     |  
     |  saveAsHadoopFile(self, path, outputFormatClass, keyClass=None, valueClass=None, keyConverter=None, valueConverter=None, conf=None, compressionCodecClass=None)
     |      Output a Python RDD of key-value pairs (of form C{RDD[(K, V)]}) to any Hadoop file
     |      system, using the old Hadoop OutputFormat API (mapred package). Key and value types
     |      will be inferred if not specified. Keys and values are converted for output using either
     |      user specified converters or L{org.apache.spark.api.python.JavaToWritableConverter}. The
     |      C{conf} is applied on top of the base Hadoop conf associated with the SparkContext
     |      of this RDD to create a merged Hadoop MapReduce job configuration for saving the data.
     |      
     |      :param path: path to Hadoop file
     |      :param outputFormatClass: fully qualified classname of Hadoop OutputFormat
     |             (e.g. "org.apache.hadoop.mapred.SequenceFileOutputFormat")
     |      :param keyClass: fully qualified classname of key Writable class
     |             (e.g. "org.apache.hadoop.io.IntWritable", None by default)
     |      :param valueClass: fully qualified classname of value Writable class
     |             (e.g. "org.apache.hadoop.io.Text", None by default)
     |      :param keyConverter: (None by default)
     |      :param valueConverter: (None by default)
     |      :param conf: (None by default)
     |      :param compressionCodecClass: (None by default)
     |  
     |  saveAsNewAPIHadoopDataset(self, conf, keyConverter=None, valueConverter=None)
     |      Output a Python RDD of key-value pairs (of form C{RDD[(K, V)]}) to any Hadoop file
     |      system, using the new Hadoop OutputFormat API (mapreduce package). Keys/values are
     |      converted for output using either user specified converters or, by default,
     |      L{org.apache.spark.api.python.JavaToWritableConverter}.
     |      
     |      :param conf: Hadoop job configuration, passed in as a dict
     |      :param keyConverter: (None by default)
     |      :param valueConverter: (None by default)
     |  
     |  saveAsNewAPIHadoopFile(self, path, outputFormatClass, keyClass=None, valueClass=None, keyConverter=None, valueConverter=None, conf=None)
     |      Output a Python RDD of key-value pairs (of form C{RDD[(K, V)]}) to any Hadoop file
     |      system, using the new Hadoop OutputFormat API (mapreduce package). Key and value types
     |      will be inferred if not specified. Keys and values are converted for output using either
     |      user specified converters or L{org.apache.spark.api.python.JavaToWritableConverter}. The
     |      C{conf} is applied on top of the base Hadoop conf associated with the SparkContext
     |      of this RDD to create a merged Hadoop MapReduce job configuration for saving the data.
     |      
     |      :param path: path to Hadoop file
     |      :param outputFormatClass: fully qualified classname of Hadoop OutputFormat
     |             (e.g. "org.apache.hadoop.mapreduce.lib.output.SequenceFileOutputFormat")
     |      :param keyClass: fully qualified classname of key Writable class
     |             (e.g. "org.apache.hadoop.io.IntWritable", None by default)
     |      :param valueClass: fully qualified classname of value Writable class
     |             (e.g. "org.apache.hadoop.io.Text", None by default)
     |      :param keyConverter: (None by default)
     |      :param valueConverter: (None by default)
     |      :param conf: Hadoop job configuration, passed in as a dict (None by default)
     |  
     |  saveAsPickleFile(self, path, batchSize=10)
     |      Save this RDD as a SequenceFile of serialized objects. The serializer
     |      used is L{pyspark.serializers.PickleSerializer}, default batch size
     |      is 10.
     |      
     |      >>> tmpFile = NamedTemporaryFile(delete=True)
     |      >>> tmpFile.close()
     |      >>> sc.parallelize([1, 2, 'spark', 'rdd']).saveAsPickleFile(tmpFile.name, 3)
     |      >>> sorted(sc.pickleFile(tmpFile.name, 5).map(str).collect())
     |      ['1', '2', 'rdd', 'spark']
     |  
     |  saveAsSequenceFile(self, path, compressionCodecClass=None)
     |      Output a Python RDD of key-value pairs (of form C{RDD[(K, V)]}) to any Hadoop file
     |      system, using the L{org.apache.hadoop.io.Writable} types that we convert from the
     |      RDD's key and value types. The mechanism is as follows:
     |      
     |          1. Pyrolite is used to convert pickled Python RDD into RDD of Java objects.
     |          2. Keys and values of this Java RDD are converted to Writables and written out.
     |      
     |      :param path: path to sequence file
     |      :param compressionCodecClass: (None by default)
     |  
     |  saveAsTextFile(self, path, compressionCodecClass=None)
     |      Save this RDD as a text file, using string representations of elements.
     |      
     |      @param path: path to text file
     |      @param compressionCodecClass: (None by default) string i.e.
     |          "org.apache.hadoop.io.compress.GzipCodec"
     |      
     |      >>> tempFile = NamedTemporaryFile(delete=True)
     |      >>> tempFile.close()
     |      >>> sc.parallelize(range(10)).saveAsTextFile(tempFile.name)
     |      >>> from fileinput import input
     |      >>> from glob import glob
     |      >>> ''.join(sorted(input(glob(tempFile.name + "/part-0000*"))))
     |      '0\n1\n2\n3\n4\n5\n6\n7\n8\n9\n'
     |      
     |      Empty lines are tolerated when saving to text files.
     |      
     |      >>> tempFile2 = NamedTemporaryFile(delete=True)
     |      >>> tempFile2.close()
     |      >>> sc.parallelize(['', 'foo', '', 'bar', '']).saveAsTextFile(tempFile2.name)
     |      >>> ''.join(sorted(input(glob(tempFile2.name + "/part-0000*"))))
     |      '\n\n\nbar\nfoo\n'
     |      
     |      Using compressionCodecClass
     |      
     |      >>> tempFile3 = NamedTemporaryFile(delete=True)
     |      >>> tempFile3.close()
     |      >>> codec = "org.apache.hadoop.io.compress.GzipCodec"
     |      >>> sc.parallelize(['foo', 'bar']).saveAsTextFile(tempFile3.name, codec)
     |      >>> from fileinput import input, hook_compressed
     |      >>> result = sorted(input(glob(tempFile3.name + "/part*.gz"), openhook=hook_compressed))
     |      >>> b''.join(result).decode('utf-8')
     |      'bar\nfoo\n'
     |  
     |  setName(self, name)
     |      Assign a name to this RDD.
     |      
     |      >>> rdd1 = sc.parallelize([1, 2])
     |      >>> rdd1.setName('RDD1').name()
     |      'RDD1'
     |  
     |  sortBy(self, keyfunc, ascending=True, numPartitions=None)
     |      Sorts this RDD by the given keyfunc
     |      
     |      >>> tmp = [('a', 1), ('b', 2), ('1', 3), ('d', 4), ('2', 5)]
     |      >>> sc.parallelize(tmp).sortBy(lambda x: x[0]).collect()
     |      [('1', 3), ('2', 5), ('a', 1), ('b', 2), ('d', 4)]
     |      >>> sc.parallelize(tmp).sortBy(lambda x: x[1]).collect()
     |      [('a', 1), ('b', 2), ('1', 3), ('d', 4), ('2', 5)]
     |  
     |  sortByKey(self, ascending=True, numPartitions=None, keyfunc=<function RDD.<lambda> at 0x7ff9180dc488>)
     |      Sorts this RDD, which is assumed to consist of (key, value) pairs.
     |      
     |      >>> tmp = [('a', 1), ('b', 2), ('1', 3), ('d', 4), ('2', 5)]
     |      >>> sc.parallelize(tmp).sortByKey().first()
     |      ('1', 3)
     |      >>> sc.parallelize(tmp).sortByKey(True, 1).collect()
     |      [('1', 3), ('2', 5), ('a', 1), ('b', 2), ('d', 4)]
     |      >>> sc.parallelize(tmp).sortByKey(True, 2).collect()
     |      [('1', 3), ('2', 5), ('a', 1), ('b', 2), ('d', 4)]
     |      >>> tmp2 = [('Mary', 1), ('had', 2), ('a', 3), ('little', 4), ('lamb', 5)]
     |      >>> tmp2.extend([('whose', 6), ('fleece', 7), ('was', 8), ('white', 9)])
     |      >>> sc.parallelize(tmp2).sortByKey(True, 3, keyfunc=lambda k: k.lower()).collect()
     |      [('a', 3), ('fleece', 7), ('had', 2), ('lamb', 5),...('white', 9), ('whose', 6)]
     |  
     |  stats(self)
     |      Return a L{StatCounter} object that captures the mean, variance
     |      and count of the RDD's elements in one operation.
     |  
     |  stdev(self)
     |      Compute the standard deviation of this RDD's elements.
     |      
     |      >>> sc.parallelize([1, 2, 3]).stdev()
     |      0.816...
     |  
     |  subtract(self, other, numPartitions=None)
     |      Return each value in C{self} that is not contained in C{other}.
     |      
     |      >>> x = sc.parallelize([("a", 1), ("b", 4), ("b", 5), ("a", 3)])
     |      >>> y = sc.parallelize([("a", 3), ("c", None)])
     |      >>> sorted(x.subtract(y).collect())
     |      [('a', 1), ('b', 4), ('b', 5)]
     |  
     |  subtractByKey(self, other, numPartitions=None)
     |      Return each (key, value) pair in C{self} that has no pair with matching
     |      key in C{other}.
     |      
     |      >>> x = sc.parallelize([("a", 1), ("b", 4), ("b", 5), ("a", 2)])
     |      >>> y = sc.parallelize([("a", 3), ("c", None)])
     |      >>> sorted(x.subtractByKey(y).collect())
     |      [('b', 4), ('b', 5)]
     |  
     |  sum(self)
     |      Add up the elements in this RDD.
     |      
     |      >>> sc.parallelize([1.0, 2.0, 3.0]).sum()
     |      6.0
     |  
     |  sumApprox(self, timeout, confidence=0.95)
     |      .. note:: Experimental
     |      
     |      Approximate operation to return the sum within a timeout
     |      or meet the confidence.
     |      
     |      >>> rdd = sc.parallelize(range(1000), 10)
     |      >>> r = sum(range(1000))
     |      >>> abs(rdd.sumApprox(1000) - r) / r < 0.05
     |      True
     |  
     |  take(self, num)
     |      Take the first num elements of the RDD.
     |      
     |      It works by first scanning one partition, and use the results from
     |      that partition to estimate the number of additional partitions needed
     |      to satisfy the limit.
     |      
     |      Translated from the Scala implementation in RDD#take().
     |      
     |      .. note:: this method should only be used if the resulting array is expected
     |          to be small, as all the data is loaded into the driver's memory.
     |      
     |      >>> sc.parallelize([2, 3, 4, 5, 6]).cache().take(2)
     |      [2, 3]
     |      >>> sc.parallelize([2, 3, 4, 5, 6]).take(10)
     |      [2, 3, 4, 5, 6]
     |      >>> sc.parallelize(range(100), 100).filter(lambda x: x > 90).take(3)
     |      [91, 92, 93]
     |  
     |  takeOrdered(self, num, key=None)
     |      Get the N elements from an RDD ordered in ascending order or as
     |      specified by the optional key function.
     |      
     |      .. note:: this method should only be used if the resulting array is expected
     |          to be small, as all the data is loaded into the driver's memory.
     |      
     |      >>> sc.parallelize([10, 1, 2, 9, 3, 4, 5, 6, 7]).takeOrdered(6)
     |      [1, 2, 3, 4, 5, 6]
     |      >>> sc.parallelize([10, 1, 2, 9, 3, 4, 5, 6, 7], 2).takeOrdered(6, key=lambda x: -x)
     |      [10, 9, 7, 6, 5, 4]
     |  
     |  takeSample(self, withReplacement, num, seed=None)
     |      Return a fixed-size sampled subset of this RDD.
     |      
     |      .. note:: This method should only be used if the resulting array is expected
     |          to be small, as all the data is loaded into the driver's memory.
     |      
     |      >>> rdd = sc.parallelize(range(0, 10))
     |      >>> len(rdd.takeSample(True, 20, 1))
     |      20
     |      >>> len(rdd.takeSample(False, 5, 2))
     |      5
     |      >>> len(rdd.takeSample(False, 15, 3))
     |      10
     |  
     |  toDebugString(self)
     |      A description of this RDD and its recursive dependencies for debugging.
     |  
     |  toLocalIterator(self)
     |      Return an iterator that contains all of the elements in this RDD.
     |      The iterator will consume as much memory as the largest partition in this RDD.
     |      
     |      >>> rdd = sc.parallelize(range(10))
     |      >>> [x for x in rdd.toLocalIterator()]
     |      [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
     |  
     |  top(self, num, key=None)
     |      Get the top N elements from an RDD.
     |      
     |      .. note:: This method should only be used if the resulting array is expected
     |          to be small, as all the data is loaded into the driver's memory.
     |      
     |      .. note:: It returns the list sorted in descending order.
     |      
     |      >>> sc.parallelize([10, 4, 2, 12, 3]).top(1)
     |      [12]
     |      >>> sc.parallelize([2, 3, 4, 5, 6], 2).top(2)
     |      [6, 5]
     |      >>> sc.parallelize([10, 4, 2, 12, 3]).top(3, key=str)
     |      [4, 3, 2]
     |  
     |  treeAggregate(self, zeroValue, seqOp, combOp, depth=2)
     |      Aggregates the elements of this RDD in a multi-level tree
     |      pattern.
     |      
     |      :param depth: suggested depth of the tree (default: 2)
     |      
     |      >>> add = lambda x, y: x + y
     |      >>> rdd = sc.parallelize([-5, -4, -3, -2, -1, 1, 2, 3, 4], 10)
     |      >>> rdd.treeAggregate(0, add, add)
     |      -5
     |      >>> rdd.treeAggregate(0, add, add, 1)
     |      -5
     |      >>> rdd.treeAggregate(0, add, add, 2)
     |      -5
     |      >>> rdd.treeAggregate(0, add, add, 5)
     |      -5
     |      >>> rdd.treeAggregate(0, add, add, 10)
     |      -5
     |  
     |  treeReduce(self, f, depth=2)
     |      Reduces the elements of this RDD in a multi-level tree pattern.
     |      
     |      :param depth: suggested depth of the tree (default: 2)
     |      
     |      >>> add = lambda x, y: x + y
     |      >>> rdd = sc.parallelize([-5, -4, -3, -2, -1, 1, 2, 3, 4], 10)
     |      >>> rdd.treeReduce(add)
     |      -5
     |      >>> rdd.treeReduce(add, 1)
     |      -5
     |      >>> rdd.treeReduce(add, 2)
     |      -5
     |      >>> rdd.treeReduce(add, 5)
     |      -5
     |      >>> rdd.treeReduce(add, 10)
     |      -5
     |  
     |  union(self, other)
     |      Return the union of this RDD and another one.
     |      
     |      >>> rdd = sc.parallelize([1, 1, 2, 3])
     |      >>> rdd.union(rdd).collect()
     |      [1, 1, 2, 3, 1, 1, 2, 3]
     |  
     |  unpersist(self)
     |      Mark the RDD as non-persistent, and remove all blocks for it from
     |      memory and disk.
     |  
     |  values(self)
     |      Return an RDD with the values of each tuple.
     |      
     |      >>> m = sc.parallelize([(1, 2), (3, 4)]).values()
     |      >>> m.collect()
     |      [2, 4]
     |  
     |  variance(self)
     |      Compute the variance of this RDD's elements.
     |      
     |      >>> sc.parallelize([1, 2, 3]).variance()
     |      0.666...
     |  
     |  zip(self, other)
     |      Zips this RDD with another one, returning key-value pairs with the
     |      first element in each RDD second element in each RDD, etc. Assumes
     |      that the two RDDs have the same number of partitions and the same
     |      number of elements in each partition (e.g. one was made through
     |      a map on the other).
     |      
     |      >>> x = sc.parallelize(range(0,5))
     |      >>> y = sc.parallelize(range(1000, 1005))
     |      >>> x.zip(y).collect()
     |      [(0, 1000), (1, 1001), (2, 1002), (3, 1003), (4, 1004)]
     |  
     |  zipWithIndex(self)
     |      Zips this RDD with its element indices.
     |      
     |      The ordering is first based on the partition index and then the
     |      ordering of items within each partition. So the first item in
     |      the first partition gets index 0, and the last item in the last
     |      partition receives the largest index.
     |      
     |      This method needs to trigger a spark job when this RDD contains
     |      more than one partitions.
     |      
     |      >>> sc.parallelize(["a", "b", "c", "d"], 3).zipWithIndex().collect()
     |      [('a', 0), ('b', 1), ('c', 2), ('d', 3)]
     |  
     |  zipWithUniqueId(self)
     |      Zips this RDD with generated unique Long ids.
     |      
     |      Items in the kth partition will get ids k, n+k, 2*n+k, ..., where
     |      n is the number of partitions. So there may exist gaps, but this
     |      method won't trigger a spark job, which is different from
     |      L{zipWithIndex}
     |      
     |      >>> sc.parallelize(["a", "b", "c", "d", "e"], 3).zipWithUniqueId().collect()
     |      [('a', 0), ('b', 1), ('c', 4), ('d', 2), ('e', 5)]
     |  
     |  ----------------------------------------------------------------------
     |  Data descriptors inherited from RDD:
     |  
     |  __dict__
     |      dictionary for instance variables (if defined)
     |  
     |  __weakref__
     |      list of weak references to the object (if defined)
     |  
     |  context
     |      The L{SparkContext} that this RDD was created on.
    


We shall cover some of these methods and transformations in upcoming lessons and labs. As a quick example, Let's look at the `getNumPartitions()` function which resuts the total partitions of an RDD.

    getNumPartitions(self)
           Returns the number of partitions in RDD

__Get number of partitions of the `intRDD`__



```python
# Get number of partitions
intRDD.getNumPartitions()
```




    5



## Types of RDD operations

Earlier we mentioned that RDD operations can be distonguished as Actions of Transformations. Let's quickly look what these entail.

### Transformations

Transformations create a new data set from an existing one by passing each dataset element through a function and returns a new RDD representing the results. In short, creating an RDD from an existing RDD is ‘transformation’.
All transformations in Spark are lazy. They do not compute their results right away. Instead, they just remember the transformations applied to some base data set (e.g. a file). The transformations are only computed when an action requires a result that needs to be returned to the driver program.
A transformationon a RDD that returns another RDD, like map, flatMap, filter, reduceByKey, join, cogroup, etc.

### Actions
Actions return final results of RDD computations. Actions trigger execution using lineage graph to load the data into original RDD and carry out all intermediate transformations and return the final results to the Driver program or writes it ou
t to the file system. An action returns a value (to a Spark driver - the user program).

Here are some of key transformations and actions that we will explore later.
<img src="rdd2.png" width=400>
          

In the following labs. we shall practice the key transformations and actions in order to split, process and analyze datasetes as RDDs. 

## Additional Reading 

- [RDDs in Apache Spark](https://data-flair.training/blogs/create-rdds-in-apache-spark/)
- [Programming with RDDs](https://runawayhorse001.github.io/LearningApacheSpark/rdd.html)

## Summary

In this lesson we went through a brief introduction to RDD creation from a Python collection, setting number of logical partitions for an RDD and extracting lineage and of an RDD in a spark application. We also looked at checking an RDD's id used by spark and setting names for RDDs as an indication of content. Following labs will build upon this knowledge and we shall see how transformations and actions can be applied to RDDs in a distributed setup. 

