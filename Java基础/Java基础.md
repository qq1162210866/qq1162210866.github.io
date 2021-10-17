## 范型

​	简单范型的例子

```java
package paradigmtrain;

import java.time.LocalDate;

/**
 * ParadigmTrain1.java
 * Description:  范型练习类
 *
 * @author Peng Shiquan
 * @date 2020/5/15
 */
public class ParadigmTrain1 {
    public static void main(String[] args) {
        LocalDate[] dates = {LocalDate.of(1996, 12, 9), LocalDate.of(1997, 5, 15), LocalDate.of(1998, 5, 15)};
        Pair<LocalDate> result = minmax(dates);
        System.err.println(result.getFirst());
        System.err.println(result.getSecond());
      	/**
         * 这里可以通过参数的形式来推断出范型的类型，所以不需要输入范型的类型参数
         */
        new Pair<>("haha", "hehe");	
    }

    /**
     * Description: 输入的参数必须要实现Comparable。
     *
     * @param a
     * @return paradigmtrain.Pair<T>
     * @Author: Peng Shiquan
     * @Date: 2020/5/15
     */
    public static <T extends Comparable> Pair<T> minmax(T[] a) {
        /**
         * 第一个范型是对参数进行限制，第二个是返回的类型
         */
        if ((a == null) || a.length == 0) return null;
        T min = a[0];
        T max = a[0];
        int aLength = a.length;
        for (int i = 0; i < aLength; i++) {
            /**
             * 比较大小，通过compareTo方法
             */
            if (min.compareTo(a[i]) > 0) min = a[i];
            if (max.compareTo(a[i]) < 0) max = a[i];
        }
        return new Pair<>(min, max);
    }
}

/**
 * ParadigmTrain1.java
 * Description:  范型类
 *
 * @author Peng Shiquan
 * @date 2020/5/15
 */
class Pair<T> {

    private T first;
    private T second;

    public Pair() {
        first = null;
        second = null;
    }

    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() {
        return first;
    }

    public T getSecond() {
        return second;
    }
}
```

* 范型方法类型变量要放在方法修饰符后面，返回类型前面。
* 范型里面，尖括号里面是类代表一个点，如果带有问号，代表一个范围。
* 限定列表中如果存在类，则必须为限定列表中的第一个。（接口需要放在类后面）
* 基本数据类型不能用于范型。
* 范型的查询只能判断原始类型，例如: `a instanceof Pair<String>` 只能判断a是否是任意一个类型的Pair。
* 编译器会将范型类型转换为原始类型（类型擦除），有限定类型就使用第一个限定类型，没有就用object代替。返回类型时，由于返回的为限定类型或者为Object，会将限定类型或者Object强制转换为对应的类型。
* 不允许创建参数化类型的数组。例如`Pair<String>[] table = new Pair<String>[10]` 这样是错误的。（可以通过注解消除这个警告）因为范型擦除的存在，所以会将类型进行擦除，导致存储的元素可能不一致。（存储Integer和String都是可以的）
* 推荐通过功能性接口（Supplier）来实例化类型变量，反射也可以实现，但是比较麻烦。
* 不能构造范型数组，例如`public static <T> T[] minmax(T[] t)`这种写法是错误的。这里也是可以利用函数式接口实现。
* 范型类中，静态代码中的类型变量无效。（因为类型擦除的原因，返回的Onject不知道转换为那种类型）
* 不能抛出或者捕获范型类的实例。
* 范型类之间不存在继承关系，无论S和T什么关系，`Pair<S>和Pair<T>不存在继承关系`
* 如上面所说，`Pair<Object>和Pair<String>`不存在继承关系，但是方法中想要使用这样的关系如何写呢？这个时候就要用到通配符类型，即？,代码如下：`public static void test(Pair<? extends Object> p)`这样就可以传String和Integer.通配符类型上届使用extends，下届用super。上届一般用于读取，下届一般用于写入。
* 桥方法是为了保持多态，具体的原因如下代码。

```java
package paradigmtrain;

import java.util.Date;

/**
 * paradigmTrain3.java
 * Description: 范型方法桥方法练习
 *
 * @author Peng Shiquan
 * @date 2021/5/13
 */
public class paradigmTrain3 {
    public static void main(String[] args) {
        Base<Date> base = new Test();
        //这里调用的应该是setFirst(Date date),但是实际会调用setFirst(Object date)
        base.setFirst(new Date());
    }
}

class Test extends Base<Date> {
    @Override
    public void setFirst(Date date) {
        System.err.println("这是子类的覆盖方法");
    }
    //这里编译后会生成一个桥方法，为了维持多态
//    public void setFirst(Object date){
//        setFirst((Date) date);
//    }
}

class Base<T> {
    public void setFirst(T t) {
        System.err.println("这是父类的方法");
    }
}
```

## Java特点

* 封装：封装是把过程和数据包围起来，对数据的访问只能通过已定义的接口。
* 继承是面向对象最显著的一个特性。继承是从已有的类中派生出新的类，新的类能吸收已有类的数据属性和行为，并能扩展新的能力。
* 多态：按字面的意思就是“多种状态”。在面向对象语言中，接口的多种不同的实现方式即为多态。即父对象可以根据赋值的不同子对象调用不同的方法。即调用子对象重写的方法。
* 抽象：抽象是通过分析与综合的途径，运用概念在人脑中再现对象的质和本质的方法。即将生活中的万物形容成一个个类。



* 类与类之间最常见的关系主要有三种：依赖(uses-a)、聚合(has-a)和继承(is-a)。依赖就是屠夫使用刀、聚合就是大雁组成雁群，大雁相对于雁群是聚合关系（两个不同的类）、继承就是儿子继承父亲的家业。

## 内部类

简单的内部类代码：



* 内部类就是定义在一个类中的类，最开始的作用就是简化代码，比较便捷。还有就是内部类完善了多重继承，每个内部类都可以独立继承一个接口的实现，但是感觉相对于它所带来的复杂程度而言，带来的便捷性其实不是很好，也可能是我学的比较浅。
* 内部类对象有一个隐式的引用，指向了实例化该内部类对象的外部类对象。
* 不需要内部类引用外部类的时候，可以使用静态内部类。
* 静态内部类创建实例的形式和普通内部类不太一样 

```java
//普通内部类
Out out=new Out();
out.in in=out.new in();
//静态内部类
Out.In in=new Out.In();
```

* 双括号初始化

```java
        /**
         * 双括号的使用方法，匿名列表，适用于只使用一次的数组列表。
         */
        ArrayList<String> test = new ArrayList<String>() {{
            add("hah");
            add("test");
        }};
```

### 内部类的分类

* 成员内部类
  * 成员内部类不能使用static方法和变量
  * 只有创建了外部类才能创建内部类

* 局部内部类
  * 可以引用局部变量，但是局部变量必须声明为final（jdk1.8之前）

* 匿名内部类
  * 匿名内部类不能有构造方法（构造方法必须和类名相同）
  * 匿名内部类没有访问修饰符

* 静态内部类
  * 静态内部类只能访问外部类的静态成员和方法
  * 静态内部类的创建不需要依赖于外部类的对象

## Java语法基础

* `==`当两边的是基本类型时，对比的是值，当两边是引用类型时，比较的内存地址。当两边一个基本一个引用时，因为Java自动拆包，所以也是判断值是否相等。
* count=count++就是先把局部变量表中count的值0放入操作数栈中，然后直接对局部变量表中的count加1，然后再把操作数栈中的0出栈赋值给局部变量表中的count，最终局部变量表中的count值仍为0。
* 非new生成的Integer和new生成的Integer结果只能为false，两个非new比较时，都在-128-127内，比较值，否则比较地址。
* +=会自动装箱，将运算的结果转换为相应的类型。`byte a = 127;byte b = 126;b = a + b;(错误) b+=a;(正确)`
* 自动装箱只能装相应的类型，没法自动转换。
* 三元运算符会自动做类型提升。
* &是逻辑与，会计算两边的等式，&&是短路与，某些情况下只计算左边的等式。｜和｜｜的区别类似。
* instanceof 严格来说是Java中的一个双目运算符，用来测试一个对象是否为一个类的实例。
* while循环就是当条件不满足时，跳出，无论while还是dowhile，只不过dowhile第一次无论如何都会执行。
* 如果不加break，是从case语句匹配的位置往下一直执行
* 取余取头，取模取尾。java中的%是取余运算，看头。即最后结果的符号看运算符的前面还是后面。
* 同名方法，不同参数是重载，父类和子类之间是重写。
* 重写方法时，访问权限不能比父类中被重写的方法的访问权限更低，父类为public，子类不能为private。
* 编译看左边，运行看右边。 ClassA a = new ClassB();编译时为ClassA，运行时为ClassB。
* finally是在return执行之后，语句返回之前执行的。
* 二维数组定义，一维长度必须定义，二维可以后续定义。
* 类中的变量可以不用初始化，但是方法中的变量声明后一定要初始化。
* try块后面可以不加catch块，但是必须要有finally和catch之一。
* Math类中提供了三个与取整有关的方法：ceil,floor,round,这些方法的作用于它们的英文名称的含义相对应，例如：ceil的英文意义是天花板，该方法就表示向上取整，Math.ceil（11.3）的结果为12，Math.ceil(-11.6)的结果为-11；floor的英文是地板，该方法就表示向下取整，Math.floor(11.6)的结果是11，Math.floor(-11.4)的结果-12；最难掌握的是round方法，他表示“四舍五入”，算法为Math.floor(x+0.5),即将原来的数字加上0.5后再向下取整。
* str.split(&quot;,&quot;)方法是把str字符串根据分割符&quot;,&quot;划分成一个字符串数组，如果str字符串中找不到分隔符&quot;,&quot;，则把整个str字符串放入字符
* replaceAll方法的第一个参数是一个正则表达式，.代表所有字符。
* 如果字符串长度没有初始化长度大，capacity返回初始化的长度，如果append后的字符串长度超过初始化长度，capacity返回增长后的长度。
* JSP中：application可以被web应用程序访问，session可以被同一对话访问，request可以被同一请求访问，pageContext可以被当前页面访问。
* 子类抛出的异常不能比父类的更加广泛，更加广泛的异常类的catch块要放在下面。
* 所有的异常都是继承Throwable，下层分为Error和Exception，Error描述了Java运行时系统的内部错误和资源耗尽错误。Exception分为运行时异常和其他异常。
* Error和Exception都是集成Throwable,其中Exception又被IOException和RuntimeException继承。
* 枚举类默认的toString方法会将字符串打印出来。
* 串数组的第一个元素。
* 静态代码块可以分开写。
* 导包只可以导到当前层。
* 包装类的值都是不可变的。

## 抽象类和接口

* 抽象类是类的抽象化，接口则是抽象方法的集合。抽象类可以被实现类继承，接口则只能被实现类实现。抽象类是自下而上的设计理念，接口则是自上而下的设计理念。两个都不能被实例化。
* 接口就是在不同层面对一类事物的抽象描述。例如：在生物层面，人和猪都是会吃东西的。但是不同的层面，人和猪可能就不是同一类事物。
* 抽象类可以存在普通成员变量和静态成员变量，也可以存在静态方法和构造方法、普通方法。接口可以存在成员变量，但都是静态的。接口在1.8之后可以定义静态方法。

## 继承相关的

* 在构造方法中如果使用关键字 `this` 调用其他构造方法，则 `this(参数列表)` 语句必须出现在其他语句之前。super是调用父类的某个构造函数，也必须在第一行。两者不能同时出现在一个函数里面。两者都不能在static中使用。
* 不会初始化子类的几种、调用的是父类的static方法或者字段、调用的是父类的final方法或者字段、通过数组来引用。
* 如果构造方法没有显式地调用同一个类中其他的构造方法或父类的构造方法，将隐性地调用父类的无参数构造方法，即编译器会把 `super()` 作为构造方法的第一个语句。
* 静态代码块只执行一次，构造代码块只要创建对象就会执行。
* java语言是静态多分派，动态单分派的。 如果是重载方法之间的选择，则是使用静态类型。 如果是父类与子类之间的重写方法的选择，则是使用动态类型。 如A a = new B(); 会使用类型B去查找重写的方法，使用类型A去查找重载的方法。

## 计算机基础

byte的数值为-128-127，因为-0表示为128

* 原码：计算机中将一个数字转换为二进制，并在其最高位加上符号的一种表示方法。

* 反码：根据表示规定，正数的反码就是本身，而负数的反码，除符号位外，其余位依次取反。

* 补码：根据表示规定，正数的补码就是本身，而负数的补码，是在其反码的末位加1。

