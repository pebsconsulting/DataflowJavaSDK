#Sorter
This module provides the SortValues transform, which takes a `PCollection<KV<K, Iterable<KV<K2, V>>>>` and produces a `PCollection<KV<K, Iterable<KV<K2, V>>>>` where, for each primary key `K` the paired `Iterable<KV<K2, V>>` has been sorted by the byte encoding of secondary key (`K2`). It will efficiently and scalably sort the iterables, even if they are large (do not fit in memory).

##Caveats
* This transform performs value-only sorting; the iterable accompanying each key is sorted, but *there is no relationship between different keys*, as Dataflow does not support any defined relationship between different elements in a PCollection.
* Each `Iterable<KV<K2, V>>` is sorted on a single worker using local memory and disk. This means that `SortValues` may be a performance and/or scalability bottleneck when used in different pipelines. For example, users are discouraged from using `SortValues` on a `PCollection` of a single element to globally sort a large `PCollection`. A (rough) estimate of the number of bytes of disk space utilized if sorting spills to disk is `numRecords * (numSecondaryKeyBytesPerRecord + numValueBytesPerRecord + 16) * 3`.

##Options
* The user can customize the temporary location used if sorting requires spilling to disk and the maximum amount of memory to use by creating a custom instance of `BufferedExternalSorter.Options` to pass into `SortValues.create`.

##Using `SortValues`
~~~~
PCollection<KV<String, KV<String, Integer>>> input = ...

// Group by primary key, bringing <SecondaryKey, Value> pairs for the same key together.
PCollection<KV<String, Iterable<KV<String, Integer>>>> grouped =
    input.apply(GroupByKey.<String, KV<String, Integer>>create());

// For every primary key, sort the iterable of <SecondaryKey, Value> pairs by secondary key.
PCollection<KV<String, Iterable<KV<String, Integer>>>> groupedAndSorted =
    grouped.apply(
        SortValues.<String, String, Integer>create(new BufferedExternalSorter.Options()));
~~~~
