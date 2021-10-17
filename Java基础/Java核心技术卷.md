# 卷二
## 第1章 Java SE 8的流库
* 流和集合的区别
	* 流并不存储元素
	* 流的操作不会修改其数据源
	* 流的操作是尽可能惰性执行		

~~~java
    public static void main(String[] args) throws Exception {
        String wordsString = new String(Files.readAllBytes(Paths.get("/Users/Desktop/dev/aaa.txt")), StandardCharsets.UTF_8);
        //以非字母分隔符
        List<String> wordsList = Arrays.asList(wordsString.split("\\PL+"));
        /**
         * 使用迭代的方式查询大于12的单词
         */
        long count = 0;
        for (String word : wordsList) {
            if (word.length() > 12) count++;
        }
        System.err.println("使用迭代的方式来统计大于十二的单词" + count);
        /**
         * 使用流的方式查询大于12的单词
         */
        //单线程运行
        count = wordsList.stream().filter(w -> w.length() > 12).count();
        System.err.println("使用单线程的流的方式来统计大于十二的单词" + count);
        //多线程运行
        count = wordsList.parallelStream().filter(w -> w.length() > 12).count();
        System.err.println("使用多线程的流的方式来统计大于十二的单词" + count);
        /**
         * 流是使用stream和parallelStream方法来创建的，filter方法用来对流进行转化，count
         * 方法用来终结操作
         */
    }
~~~

* 流的创建方法	
	JavaAPI中有大量的方法可以产生流，这里不再全部叙述，也不太现实。就直接以一个demo的形式叙述一下我了解的创建流的方式。
	
~~~java
package streamtrain;

import java.math.BigInteger;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Arrays;
import java.util.List;
import java.util.regex.Pattern;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * SetUpStream.java
 * Description:
 *
 * @author Peng Shiquan
 * @date 2020/4/27
 */
public class SetUpStream {
    public static void main(String[] args) throws Exception {
        Path path = Paths.get("/Users/pengshiquan/Desktop/dev/aaa.txt");
        String wordsString = new String(Files.readAllBytes(path), StandardCharsets.UTF_8);
        String[] arrayString = {"aaaa", "bbbb", "ccc", "ddd"};

        /**
         * Stream方法创建流
         */
        Stream<String> stringStream = Stream.of(wordsString);

        Stream<String> stringStream1 = Stream.of("hahah", "aaaa", "bbb", "dddd");
        show("Stream.of", stringStream1);
        /**
         * Array.stream(array,from,to)可以创建一个流，从数组中的from到to
         */
        Stream<String> stringStream2 = Arrays.stream(arrayString, 1, 3);
        show("Arrays.stream", stringStream2);
        /**
         * Stream.empty()用来创建一个不包含任何元素的流
         */
        Stream<String> stringStream3 = Stream.empty();
        show("Arrays.stream", stringStream3);
        /**
         * Stream两个创建无限流的方法
         */
        //使用lambda表达式重写Supplier<T>函数表达式
        Stream<String> stringStream4 = Stream.generate(() -> "HELLO");
        show("Stream.generate", stringStream4);
        //lambda表达式中的实例方法引用
        Stream<Double> stringStream5 = Stream.generate(Math::random);
        show("Stream.generate", stringStream5);
        // 前面是种子，反复调用函数，应用到之前到结果上    顺序如下： 0，1，2，3，4
        Stream<BigInteger> integerStream = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
        show("Stream.iterate", integerStream);

        /**
         * pattern中的splitAsStream方法将字符串分割为一个一个单词
         */
        Stream<String> stringStream6 = Pattern.compile("a").splitAsStream(wordsString);
        show("splitAsStream", stringStream6);

        /**
         * 返回一个包含了文件所有行的stream
         */
        try (Stream<String> stringStream7 = Files.lines(path, StandardCharsets.UTF_8)) {
            show("Files.lines", stringStream7);
        }
    }

    /**
     * Description: 添加了<T>的展示方法，添加<T>可以使得方法参数类型不受对象范型类限制
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/4/4
     */
    public static <T> void show(String title, Stream<T> stream) {
        final int SIZE = 10;
        List<T> headList = stream.limit(SIZE + 1).collect(Collectors.toList());
        System.err.print(title + ": ");
        for (T t : headList) {
            System.err.print(t);
        }
        System.err.println();
    }
}
~~~
* 对流的操作的方法
* 

