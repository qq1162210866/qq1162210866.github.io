## 集合

### 集合的框架图

集合框架中继承关系：

![IMG_0ABE1119FACC-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_0ABE1119FACC-1.jpeg)

常用的各个集合的作用和特点：

![IMG_62E779E2DD25-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_62E779E2DD25-1.jpeg)

### LIst相关知识点

* List是有序的Collection（指的是插入顺序）
* ArrayList是常用的List，通过数组实现。允许对元素快速访问。扩容是当前容量*1.5+1,当元素满的时候扩容。线程不安全。
* Vector也是通过数组实现的，但是它支持多线程访问。速度比ArrayList慢。初始化为10，同步的实现原理是synchronized.
* LinkedList通过双向循环链表数据结构，插入和删除比较方便，随机访问比较慢。

### Set相关知识点

* Set主要存放的无序的元素（指的是插入的顺序），值不能重复，只允许插入一个空元素。
* HashSet是按照Hash值来存取元素的，只能通过迭代器访问元素。
* TreeSet是使用二叉树的原理对元素进行顺序排序。
* LinkedHashSet继承HashSet又基于LinkedHashMap实现。

### Map相关知识点

* HashMap就是键值映射，能够存储null值和一个null的key，线程不安全。其他可以看看博客。线程安全问题存在扩容情况，就是多个线程扩容时，会造成环链的情况，导致无限循环。
* Hashtable 是一个哈希表，该类继承自Dictionary类，实现了 Map 接口。HashMap是基于哈希表实现的，该类继承AbstractMap，实现Map接口。Hashtable 线程安全的，而 HashMap 是线程不安全的。HashMap允许将 null 作为一个 entry 的 key 或者 value，而 Hashtable 不允许。
* ConcurrentHashMap支持并发操作，采用的是分段锁的形式。Segment代表一个段，一个Segment又代表了一个小型的HashMap，默认并发度为16即16个Segment。
* TreeMap是排序map，能够将存储的元素进行排序，默认是升序，需要插入的元素实现Comparable接口。
* LinkHashMap继承HashMap，保存了插入的顺序。
* 重写equals时一定要重写hashcode，因为不重写会导致，map中判断相同的对象无法达到覆盖的目的。equals判断为true，hashcode可能为false。
* WeakHashMap使用的是弱引用保存键。

### 迭代器相关知识点

* 使用迭代器的原因就是可以在迭代过程中删除元素。还有就是解耦合，将遍历Collection和具体的实现抽离出来。
* 具体使用代码：

```java
List<String> abc = new ArrayList<String>() {{
            add("1");
            add("2");
        }};
        Iterator<String> it  = abc.iterator();
        while (it.hasNext()){
            System.err.println(it.next());
            it.remove();
        }
```

* ListIteratort只能遍历List，Iterator不仅可以遍历List也可以遍历Set。Iterator只能单向遍历，ListIterator可以双向遍历。ListIterator也添加了其他的功能，添加一个元素、替换一个元素、获取前面的元素或者后面的元素。

### 集合的其他知识点

* foreach循环内部实现是使用迭代器的，但是最好不要在里面执行元素的删除
* 迭代器的remove方法会删除指针的前一个元素。调用这个方法时一定要调用next方法控制指针
* 可以通过instanceof RandomAccess接口来判断当前集合是否支持高效的随机访问。
* 链表添加元素一般都是在末尾，使用迭代器可以在特定位置添加元素。（ListIterator）
* 优先级队列可以按任意顺序插入，但是最后删除时是排序的，会获取最小的元素。
* 映射视图和MySQL中的视图类似，将键、值、键值组成不同的视图。如果在视图上调用remove方法，会删除，但是不能增加元素。
* Hashtable和HashMap作用一样，不过前者是同步的，后者是不同步的。
* java中的sub一般都是第一个包含在内，第二个参数不包含在内。
* ThreadLocalMap中使用开放地址法来处理散列冲突，而HashMap中使用的是分离链表法。
* 队列先进先出，栈先进后出