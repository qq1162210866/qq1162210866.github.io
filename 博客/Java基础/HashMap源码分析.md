## HashMap源码分析

​	今天来看一下HashMap的源码，学习一下相关的知识点。不多啰嗦了，直接开始吧。

​	先看一段代码，是使用HashMap的代码，比较简单，先从代码入手。

```java
        Map<String, Date> test = new HashMap<>();
        test.put("test1", new Date());
        test.put("test2", new Date());
        test.put("test1", new Date());
        Date date = test.get("test1");
        System.err.println(date);
```

​	代码比较简单，建立一个map，存放date，先后放了三次，代表三个场景，第一次当map为空时放入、第二次放入不同的key，第三次放入相同key，但是value不同。最后取出第一次放入的key的value并且打印出来。下面就来具体看看源码，看看HashMap到底做了什么。

​	先看第一行代码，看到声明了一个map，执行了key和value的类型。

![image-20210228215831632](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210228215831632.png)

​	构造函数只有一行，指定了负载系数，下面看一下这个负载系数是什么东西。`负载因子是在自动增加其哈希表容量之前允许哈希表获得的满度的度量。`不难理解，这个负载系数是map自动扩增的一个阈值。好像这个构造函数比较简单，只是简单的赋值了，也没有什么初始化的操作，别急，看看第二行代码吧。

​	第二行代码就是放入一个key为test，value为当前时间的操作。来看看`put(K key, V value)`做了什么。`put(K key, V value)`的代码只有一行。

![image-20210301161338805](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210301161338805.png)

​	`putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict)`是一个实现，看一下传参的含义，需要注意的参数不多，就一个hash值，下面看一下hash方法。
![image-20210301161814463](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210301161814463.png)

​	方法注释如下：**计算key.hashCode（）并将哈希的较高位（XOR）扩展为较低。 由于该表使用2的幂次掩码，因此仅在当前掩码上方的位中发生变化的哈希集将始终发生冲突。 （众所周知的示例是在小表中包含连续整数的Float键集。）因此，我们应用了一种变换，将向下扩展较高位的影响。 在速度，实用性和位扩展质量之间需要权衡。 由于许多常见的哈希集已经合理分布（因此无法从扩展中受益），并且由于我们使用树来处理容器中的大量冲突集，因此我们仅以最便宜的方式对某些移位后的位进行XOR，以减少系统损失，以及合并最高位的影响，否则由于表范围的限制，这些位将永远不会在索引计算中使用。**这边说一下自己的理解：这个方法返回的是int类型，四个字节，后面是用来确定key在数组中的下标，但是map中数组长度是有限制的，一般小于2^16，即65536。获取key下标的代码如下：`tab[i = (n - 1) & hash]`（hash%n等价于(n - 1) & hash）,也就是说只可能会和四个字节的低两个字节进行位运算，所以这里的hash方法作用也就不难判断出来，将hash值变得更加散乱，减少哈希冲突。这里的实现就是让hashCode和高16位进行异或运算（异或的运算不偏向于0或者1）。`hashCode()`方法调用底层C的代码，这里能力有限，就不再列出来叙述了。到这里，明白了HashMap并不是将hashCode直接拿来用，还是有一定的运算。这里参考了一片博客：[讲解hash方法](https://blog.csdn.net/qq_42034205/article/details/90384772)

​	回到`putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict)`方法，代码比较多，这里直接粘贴出来：

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

​	这里初看代码有些不知道看哪里，可以按照最开始编写的代码去跑这个代码逻辑。下面就开始按照第二行代码梳理。

​	代码先是判断数组存不存在，数组每个位置存放的是一个节点链表。如果数组为空，就调用初始化方法`resize()`,这个方法可以放在后面讲，现在暂时略过，方法作用很简单，就是初始化或者增加表大小。下面就是根据哈希值获取key所在的下标，因为这里第一次放入，代码肯定位空，所以直接新建节点并且放入，next节点置为空。接着就来到了最后，判断当前负载系数是否超过最开始设置的值（默认0.75），如果超过，就再执行一次`resize()`.最后返回一个空。以上解析就是HashMap在放入第一个值时做的事，看着还是挺多的，下面接着看最开始编写的第三行代码。

​	第三行代码放入了一个不同的key，不同的value。按照上面的代码再走一遍，不过这个时候数组已经有了，不需要再次创建。但是因为key不相同，所以哈希值不相同（假设没有哈希冲突），所以后面的步骤就和上面的一样了：新建一个节点，放入到数组中。同时在最后也会对数组长度做判断，达到负载系数就扩容。那就来看第四行代码。

​	第四行代码是继续放入一个相同的key，但是value不同。仍然走上面的代码，发现数组已经创建，但是通过计算下标发现该下标已经有节点，执行了else部分。先是判断key是否相同，这里判断方法为：`hash值是否相等&&(两个key相等||两个key的eques方法返回true)`。看到这里，是不是也清楚了：重写eques方法一定要重写hashCode方法。因为如果这里没有重写hashCode方法，即哈希值不一定相同，就会造成相同的key，却有两个下标，导致后续的put无法覆盖掉原有的key。回到源码，看到else最后将原有的value取出，将新的放进去，并且将原有value返回，结束本次放入。（这里可能理解不太对，欢迎大佬们指正）

​	到了最后一行，将放入到值取出。看看源码。

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

​	代码可以比较清晰的看到HashMap中`节点`这个概念的存储结构，先是找到哈希值所在的下标，再去判断节点的第一个到key是否相同（和上面所到判断方法一致），到这里，也就理解了：哈希值相同，eques方法不一定返回true。如果不是第一个元素，在开始遍历节点中的链表，一直到找到为止。到这里，最开始编写的代码也就全部解析完毕。中间的一些逻辑分支没有细说，东西都是大差不差，没有什么需要特别说的点。下面根据这个方法我找到了一个流程图。和下面的图片来源一样。

![image-20210302141253511](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210302141253511.png)

​	最后开始填上面的坑，resize方法。这个方法是比较重要的，属于map的扩容机制。先提一点，通过上面的解析，现在也大致清楚了HashMap中的结构，不仅仅的是一个数组，是将链表和数组的形式结合了起来。类似于下面这个形式。

![image-20210302110441809](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210302110441809.png)

​	这个图片来源于:[Java8的HashMap详解（存储结构，功能实现，扩容优化，线程安全，遍历方法）](https://blog.csdn.net/login_sonata/article/details/76598675),后面的解析也参考了这个博客，讲解的比较清晰。

​	当数组到了一定的容量后，需要对数组进行扩容，扩容后也需要对元素的下标进行重新计算，resize方法完成的就是这些内容。源码如下：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

​	这里按照讲解put方法的方式分析，按照两种情况进行解析。1.数组没有初始化的时候扩容。2.数组负载系数达到阈值时，对数组进行扩容。

* 当数组没有初始化，先是将数组长度和长度阈值设置好，然后直接创建数组，将数组给全局变量就完成这一步骤。初始化比较简单，主要就是各个参数的设置。

* 当数组需要扩容时就比较复杂了，判断是否需要扩容这一部分就不再细说了，就是各个逻辑的梳理，按照代码来即可，没有什么可说的点。只需要记得扩容后就是将原有数组长度乘2，所以HashMap的数组长度只能是2的幂次方（包括你自己设置为13，也会自动寻找最近的2的幂次方）需要重点说的就是扩容后，将原有数组移植到新数组中。这里有两个分支不需要讲：1.链表长度超过8时转换为红黑树的实现（这部分包括put方法里面涉及红黑树的会单独写一个博客，留坑）。2.链表只有一个节点时的情况，比较简单，重新计算下标即可。重点中的重点就是，链表节点树为多个的情况。就是最后一个else里面的代码。

  先看一个解析。

  ![image-20210302113927205](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210302113927205.png)

  

  这个解析比较清晰，所以判断一个链表中改位置需不需要移动，就可以直接用`(e.hash & oldCap) == 0`来判断，即直接将数组长度和哈希值与一下就能判断出高位是否会变化。如果等于零就不需要移动，不等于零，就需要移动到新的位置，新的数组下标为多少，上面的解析也能看出来，就是加8，即加上旧数组长度。这样的话，看这段代码就比较容易看懂了。循环遍历链表，再判断该节点是否需要移动，需要移动就放到高位节点链表中，不需要移动就放到低位节点链表中。创建四个节点也是为了方便放入，最后遍历完毕，将高低位的头节点放入到指定下标即可，完成了数组的迁移。

​	至此，关于最开始编写的代码的解析都已经完毕了，剩下的红黑树实现和哈希表的原理因为能力有限，放到后面讲，要不然博客就太长了。在最后，欢迎各位大佬指正和批评。

​	就这样吧，结束。

