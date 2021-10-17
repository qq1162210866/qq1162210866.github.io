# 	Java中Stream了解和使用

​	还是工作中遇到的一些事情，之前因为基础不是很好或者工作中使用的东西比较局限。所以现在在慢慢补上自己之前的一些知识盲点，可能在大家看来都是比较浅显的东西，还是那句话，博客以记录为主。这一篇博客就是补充Stream知识点。

## 普通代码中操作List和集合类

​	我们写代码通常会遇到操作集合类的需求，例如找出集合中符合条件的元素，或者需要对集合中某些元素做一些操作。我之前的写法代码如下：

```java
public static void ordinary(List<Integer> arr) {
    //查找特殊的，for循环遍历
    List<Integer> result = new ArrayList<>();
    long tmp = System.currentTimeMillis();
    for (Integer integer : arr) {
        if (integer > 5) {
            result.add(integer);
        }
    }
    System.err.println(System.currentTimeMillis() - tmp);
    System.err.println(result);

    result.clear();
    //java的Lambda表达式本质是匿名内部类，在匿名的内部类中不能访问局部变量，只能访问常量。
    tmp = System.currentTimeMillis();
    arr.forEach(integer -> {
        if (integer > 5) result.add(integer);
    });
    System.err.println(System.currentTimeMillis() - tmp);
    System.err.println(result);
}
```

​	就算我们使用foreach加上lambda表达式，可以看到代码量还是有一些的，而且这样写总感觉很别扭。所以Java就提供了Stream，来方便我们的写法。

## Java Stream的操作

​	使用Stream编写和上面功能一样的代码，具体代码如下：

```java
public static void streamTrain(List<Integer> arr) {
    long tmp = System.currentTimeMillis();
    List<Integer> result = arr.stream().filter(integer -> integer > 5).collect(Collectors.toList());
    System.err.println(System.currentTimeMillis() - tmp);
    System.err.println(result);
}
```

​	可以看到代码明显的减少了一部分，而且Stream的优点不仅仅是这些。

## 简单介绍Stream

​	 Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

​	以上的介绍来自于菜鸟教程。我的理解就是将集合中的数据看成水流，使用Stream中提供的方法对水流进行一系列的处理，聚合、过滤、分流等等。这个概念和Storm中的概念类似。最后得到我们想要的数据。简化我们的代码的同时保证代码的可读性。

## 常用API的介绍

​	介绍了基本的概念，下面就说一下常用的API，这些都是看源码发现的，建议大家如果不知道如何调用的时候，看看源码差不多就知道大致怎么写了。

* ```java
  Stream<T> filter(Predicate<? super T> predicate);
  ```

  ​	返回由与给定谓词匹配的此流的元素组成的流。我理解就是过滤器，通过你的参数对流中的数据进行过滤。例子代码如下：

  ```java
  arr.stream().filter(integer -> integer > 5)
  ```

* ```java
  <R> Stream<R> map(Function<? super T, ? extends R> mapper);
  ```

  ​	返回由将给定函数应用于此流的元素的结果组成的流。我理解就是对流中元素进行一些操作，然后返回操作后的元素（这个返回的元素可以是是新的元素）。例子代码如下：

  ```java
  arr.stream().map(integer -> integer*integer+"")
  ```

* ```java
  IntStream mapToInt(ToIntFunction<? super T> mapper);
  ```

  ​	返回一个IntStream它包含将给定函数应用于此流的元素的结果。就是将你操作后的结果转换为int，例如你有一个用户集合，你想要过去所有用户年龄的int集合，使用这个方法就可以。代码我就不再写了。

* ```java
  <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
  ```

  ​	返回一个流，该流由通过将提供的映射函数应用于每个元素而生成的映射流的内容替换此流的每个元素的结果组成。 每个映射流在其内容放入该流后closed 。 （如果映射流为null ，则使用空流代替。）这个不太好理解，就是将你流中的每个元素再生成一个流，再操作这些流，使用完子流后再关闭子流。举个例子，例如遍历某个目录的文件夹，子流就相当于子文件夹，再遍历这些子文件夹就可以使用这个方法。再打个比方，就是将一个对象切片为多个对象。例子如下：

  ```java
  public static void flatMapTrain() {
      List<List<Integer>> arr = new ArrayList<>();
      List<Integer> result = arr.stream().flatMap(integers -> integers.stream()).filter(integer -> integer > 5).collect(Collectors.toList());
      System.err.println(result);
  }
  ```
  
  ​	这段代码的作用就是将arr中所有的大于5的数值合并到一个新的集合中，在开发中不仅仅可以这样用，还可以写的更加复杂。例如有一个用户类集合，用户类中嵌套了多个地址集合，需要判断这个集合中的所有用户是否属于某个地址，就可以使用这个方法。
  
* ```java
  Stream<T> distinct();
  ```

  ​	返回由该流的不同元素（根据Object.equals(Object) ）组成的流。注意调用的是equals方法，记得调用该方法时重写对象的equals方法。代码没啥可写的，就不写了。

* ```java
  Stream<T> sorted();
  ```

  ​	返回由该流的元素组成的流，按自然顺序排序。 如果此流的元素不是Comparable ，则在执行终端操作时可能会抛出java.lang.ClassCastException 。需要注意两点：排序的结果是自然排序，就是1、2，3.如果元素没有实现Comparable，可能会抛出异常。所以也就有了下面的方法。

* ```java
  Stream<T> sorted(Comparator<? super T> comparator);
  ```

  ​	返回由该流的元素组成的流，根据提供的Comparator进行排序。可以按照你的需求对流中的元素进行排序，示例代码如下：

  ```java
  List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
  List<Integer> result = numbers.stream().sorted((o1, o2) -> o2 - o1).collect(Collectors.toList());
  System.err.println(result);
  ```

  ​	这里的示例代码就是将集合倒序。

* ```java
  Stream<T> peek(Consumer<? super T> action);
  ```

  ​	返回一个由该流的元素组成的流，另外在每个元素上执行提供的操作，因为元素从结果流中被消耗。我理解就是一个窥探的方法，主要就是调试使用，例如打印当前流中元素，方便检查。示例代码也不再写了。

* ```java
  Stream<T> limit(long maxSize);
  ```

  ​	返回由该流的元素组成的流，其长度被截断为不超过maxSize 。注意：如果传入为负，会抛出异常。举例代码如下：

  ```java
  List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
  List<Integer> result = numbers.stream().sorted((o1, o2) -> o2 - o1).limit(3).collect(Collectors.toList());
  System.err.println(result);
  ```

* ```java
  Stream<T> skip(long n);
  ```

  ​	在丢弃流的前n元素后，返回由该流的其余元素组成的流。 如果此流包含少于n元素，则将返回一个空流。和上面类似，如果传入参数为负，也会有一个`IllegalArgumentException`异常的抛出。

* ```java
  void forEach(Consumer<? super T> action);
  ```

  ​	对此流的每个元素执行一个操作。这是一个终端操作。上面的方法都是中间操作，这个方法是最终操作，即调用这个方法后，所有的流全部使用完毕了，一般只有在最后才会使用。

* ```java
  Object[] toArray();
  ```

  ​	返回一个包含此流元素的数组。注意返回的格式，需要你强转一下类型。

* ```java
  <R> R collect(Supplier<R> supplier,
                BiConsumer<R, ? super T> accumulator,
                BiConsumer<R, R> combiner);
  ```

  ​	对此流的元素执行可变归约操作。 可变归约是其中归约值是可变结果容器，例如ArrayList ，并且通过更新结果的状态而不是替换结果来合并元素。我常用的就是上面示例代码的最后一步，将流转换到一个新的集合中。

* ```java
  long count();
  ```

  ​	返回此流中元素的计数。这是归约的一个特例，这是一个终端操作。就是求和，看看流中有多少元素。

* ```java
  Optional<T> min(Comparator<? super T> comparator);
  ```
  
  ​	根据提供的Comparator返回此流的最小元素。 这是一个减少的特例。我理解就是取最小的元素，然后关闭流。`Optional`这个知识点放到下一篇博客中讲解，最近也是了解了一下，现在可以简单将其理解为优雅的判断是否为null。
  
* ```java
  Optional<T> max(Comparator<? super T> comparator);
  ```
  
  ​	根据提供的Comparator返回此流的最大元素。 这是一个减少的特例。和上面的相反。
## 备注

​	当前的博客还只是初步的使用API，后面可能会了解底层的一些原理，例如底层的这个Stream是如何实现的？为什么数据小的时候for循环快？数据大的时候Stream快？这些问题都是后续慢慢补充的。

​	每个人在工作中的情况都是不一样的，可能某些情况下使用Stream会便捷一些，某些情况下使用for循环方便一些。还是要具体到自己的工作情况中，一味的追求某种写法都是不好的。多了解一些方便的写法，以后写代码也会多一种思路。

​	就这样吧，结束。