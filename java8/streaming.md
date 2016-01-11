# Streaming API Introduction

Stream interface definition: A sequence of elements supporting sequential and parallel aggregate operations.

## Using Stream API

Example: Count non-null string from a string array.

*Old Style*
```java
String[] strs = new String[] { "a", null, "b", "c", null, "d" };
int numNonNull = 0;
for (String str : strs) {
    if (str != null) {
        numNonNull += 1;
    }
}
System.out.println("The number of non-null string is: " + numNonNull);

```
*Using Java 8 Streaming API*
```java
String[] strs = new String[] { "a", null, "b", "c", null, "d" };
long numNonNull = Stream.of(strs).filter(str -> str != null).count();
System.out.println("The number of non-null string is: " + numNonNull);
```
## Parse Stream
* Create Stream
* Convert Stream
* Reduce Stream

## Benefits from Streaming API
* Easy to use
* High performance
* Tell what to do not how to do

## Create Stream
### Using Stream Interface
```java
Stream<String> stream = Stream.of("a", "b", "c");
```
### Implement Supplier Interface
```java
public class RandomStream {

    public static void main(String[] args) {
        Stream<String> stream = Stream.generate(new Supplier<String>() {
            @Override
            public String get() {
                return String.valueOf(Math.random());
            }
        });
        stream.limit(10).forEach(System.out::println);
    }
}
```
Less code version
```java
Stream<String> stream = Stream.generate(() -> String.valueOf(Math.random()));
stream.limit(10).forEach(System.out::println);
```
Using generate method will generate a unlimited Stream, if we need use the Stream we need limit its length.

### Get Stream from Collection
```java
long numNonNull = strings.stream().filter(str -> str != null).count();
System.out.println("The number of non-null string is: " + numNonNull);
```

## Intermediate Operation
转换Stream其实就是把一个Stream通过某些行为转换成一个新的Stream。

1. distinct: Returns a stream consisting of the distinct elements (according to Object.equals(Object)) of this stream.
![Alt Stream Distinct](stream_distinct.jpg)

1. filter: Returns a stream consisting of the elements of this stream that match the given predicate.
![Alt Stream Filter](stream_filter.jpg)

1. map: Returns a stream consisting of the results of applying the given function to the elements of this stream.
![Alt Stream Map](stream_map.jpg)

1. flatMap：
Returns a stream consisting of the results of replacing each element of this stream with the contents of a mapped stream produced by applying the provided mapping function to each element. Each mapped stream is closed after its contents have been placed into this stream. (If a mapped stream is null an empty stream is used, instead.)
FlatMap transforms each element of the stream into a stream of other objects. So each object will be transformed into zero, one or multiple other objects backed by streams. The contents of those streams will then be placed into the returned stream of the flatMap operation.
![Alt Stream FlatMap](stream_flatmap.jpg)
Example Code
```java
public class FlatMapStream {
    public static void main(String[] args) throws Exception {
        Stream<String> lines = Files.lines(
                new File("test.txt").toPath(), StandardCharsets.UTF_8);
        lines.flatMap(line -> Stream.of(line.split("=")))
                .forEach(System.out::println);
        lines.close();
    }
}
```

1. peek
Returns a stream consisting of the elements of this stream, additionally performing the provided action on each element as elements are consumed from the resulting stream.
![Alt Stream Peek](stream_peek.jpg)
Example: Intermediate operation vs. Terminal operation
Using peek Only: 
```java
List<String> list = Arrays.asList("Bender", "Fry", "Leela");
list.stream().peek(System.out::println);
```
vs. Plus foreach:
```java
List<String> list = Arrays.asList("Bender", "Fry", "Leela");
list.stream()
    .peek(System.out::println)
    .forEach(System.out::println);
```

1. limit: 对一个Stream进行截断操作，获取其前N个元素，如果原Stream中包含的元素个数小于N，那就获取其所有的元素
![Alt Stream Limit](stream_limit.jpg)

1. skip: 返回一个丢弃原Stream的前N个元素后剩下元素组成的新Stream，如果原Stream中包含的元素个数小于N，那么返回空Stream
![Alt Stream Skip](stream_skip.jpg)

## Terminal operation
A reduction operation (also called a fold) takes a sequence of input elements and combines them into a single summary result by repeated application of a combining operation, such as finding the sum or maximum of a set of numbers, or accumulating elements into a list. The streams classes have multiple forms of general reduction operations, called reduce() and collect(), as well as multiple specialized reduction forms such as sum(), max(), or count().

## Parallel Streams

Streams can be executed in parallel to increase runtime performance on large amount of input elements. Parallel streams use a common ForkJoinPool available via the static ForkJoinPool.commonPool() method. The size of the underlying thread-pool uses up to five threads - depending on the amount of available physical CPU cores:
```java
ForkJoinPool commonPool = ForkJoinPool.commonPool();
System.out.println(commonPool.getParallelism());
```
This value can be decreased or increased by setting the following JVM parameter:
```java
-Djava.util.concurrent.ForkJoinPool.common.parallelism=5
```


## 可变汇聚

Reference: http://ifeve.com/stream/
http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/


*Java 7 style*
```java
String[] kpiNames = trackable.getKpis();
for (String kpiName : kpiNames) {
    double kpiValue = trackable.getAndResetKpiValue(kpiName);
    buffer.append(kpiName).append(" = ").append(kpiValue).append(", ");
}
logger.info(buffer.toString());
```

*Java 8 style*
```java
Stream.of(trackable.getKpis()).
    forEach(kpiName -> {
        double kpiValue = trackable.getAndResetKpiValue(kpiName);
        buffer.append(kpiName).append(" = ").append(kpiValue).append(", ");
    });
logger.info(buffer.toString());
```
