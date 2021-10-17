## Java中的参数传递类型

​	在阅读HashMap中的源码过程中，阅读到一些方法之间的调用涉及到参数传递，有些情况下感觉最后的结果和我想的不一样，所以特地了解了一下参数传递，在这里也记录一下，防止自己忘记。

[TOC]

### 了解一些基础概念

​	在学习参数传递之前需要先了解一些基础概念，这里也会给出实例代码。方便理解。

#### 形式参数和实际参数

​	参数传递在程序中是比较常见的。参数传递涉及到两个概念。形式参数和实际参数。下面就说一下两者的区别。

> 形式参数：是在定义函数名和函数体的时候使用的参数，目的是用来接收调用该函数时传入的参数，简称“形参”。
> 实际参数：在主调函数中调用一个函数时，函数名后面括号中的参数称为“实际参数”，简称“实参”。

​	Java也不例外，也存在形式参数和实际参数，在这里分别举个例子。

#### 形参和实参的举例

​	形式参数：

```java
public static void test(StringBuffer s, int a)//形式参数s和a
```

​	实际参数：

```java
StringBuffer s = new StringBuffer("main");
int a = 1;
test(s, a);//实际参数s和a
```

​	看了上面的例子，是不是了解了形式参数和实际参数的区别。

#### 值传递和引用传递

​	在程序语言中的参数传递类型有两种，分别为值传递和引用传递。两者的概念和区别如下：

>值传递是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。
>
>引用传递是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

​	这里的举例用C来举例，用C的代码分别举例值传递和引用传递。

#### 值传递和引用传递的举例

​	值传递：

```c
#include <stdio.h>
void swap(int a,int b);

void main(){
    int a = 3;
    int b = 4;
    printf("bef swap, add of a = %d\n",&a);
    printf("aft swap, val of a = %d\n",a);
    swap(a,b);
    printf("aft swap, add of a = %d\n",&a);
    printf("aft swap, val of a = %d\n",a);
}
// pass by value
void swap(int a,int b){
    int temp = a;
    a = b;
    b = temp;
}
```

​	引用传递：

```c
#include <stdio.h>
void swap2(int *a,int *b);

void main(){
    int a = 3;
    int b = 4;
    printf("bef swap, add of a = %d\n",&a);
    printf("aft swap, val of a = %d\n",a);
    swap2(&a,&b);
    printf("aft swap, add of a = %d\n",&a);
    printf("aft swap, val of a = %d\n",a);
}

// pass by reference
void swap2(int *a,int *b){
    int temp = *a;
    *a = *b;
    *b = temp;
}
```

​	上面两个例子就是值传递和引用传递的区别，运行后也是不同的效果，还是比较明显的。c这一块不是很熟悉，代码可能有误，但是思想我认为传达到了。

### 了解Java中的传递类型

​	终于到了本文的正题了，开始讲解一下Java中的参数传递的机制和原理。在讲解之前，需要先了解一下JVM中的堆和栈的区别。

* 栈内存。栈内存首先是一片内存区域，存储的都是局部变量，凡是定义在方法中的都是局部变量（方法外的是全局变量），for循环内部定义的也是局部变量，是先加载函数才能进行局部变量的定义，所以方法先进栈，然后再定义变量，变量有自己的作用域，一旦离开作用域，变量就会被释放。栈内存的更新速度很快，因为局部变量的生命周期都很短。
* 堆内存。存储的是数组和对象（其实数组就是对象），凡是new建立的都是在堆中，堆中存放的都是实体（对象），实体用于封装数据，而且是封装多个（实体的多个属性），如果一个数据消失，这个实体也没有消失，还可以用，所以堆是不会随时释放的，但是栈不一样，栈里存放的都是单个变量，变量被释放了，那就没有了。堆里的实体虽然不会被释放，但是会被当成垃圾，Java有垃圾回收机制不定时的收取。

​	可以简单的理解为方法中的变量都存储在栈中，实际的对象都存储在堆中。方法之间是不能够互相修改变量的。所以也就有了这种说法。**Java中只存在值传递，不存在引用传递**，因为本质上都是复制了一份副本，所以都是值传递。下面就开始讲解。

#### 对于传递类型的解析

​	Java中的参数分为两种，基本类型参数和引用数据类型，基本数据类型基本上没有什么分歧，都认为是值传递。主要分歧都在引用数据类型上。

​	在方法之间传递一个引用数据类型是，类似于这种`foo(User user)`,变量user相当于一个指针，指向了堆中实际的存储对象。在传递过程中，将指针进行复制，传递到foo方法中，两个变量是有区别的，但是指想的都是堆中同一个对象。

​	说的可能不太好理解，举一些例子来说明一下。

#### 举一些小例子

* 例子一：

```java
package nativetrain;

/**
 * TransferHKTrain.java
 * Description: 调用海康SDK的练习
 *
 * @author Peng Shiquan
 * @date 2021/3/9
 */
public class TransferHKTrain {

    public static void main(String[] args) {
        User user = new User("main");
        test(user);
        System.err.println(user.name);
    }

    public static void test(User user) {
        user.name = "test";
        System.err.println(user.name);
    }
}

class User {
    public String name;

    User(String name) {
        this.name = name;
    }
}

```

​	运行结果如下图：

![image-20210317153823967](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210317153823967.png)

​	这个运行结果好像和我们说的不一致，其实细想一下就明白了，方法传递的是一个指针的副本，最后指向的都是堆中的对象。修改这个对象后在主方法中当然可以起效。但是和引用传递还是有区别的，区别就是引用传递能修改真正的参数，但是Java中的传递并不行。可以再看一下下面一个代码示例。

* 例子二：

```java
package nativetrain;

/**
 * TransferHKTrain.java
 * Description: 调用海康SDK的练习
 *
 * @author Peng Shiquan
 * @date 2021/3/9
 */
public class TransferHKTrain {

    public static void main(String[] args) {
        User user = new User("main");
        test(user);
        System.err.println(user.name);
    }

    public static void test(User user) {
        user = new User("test");
        System.err.println(user.name);
    }
}

class User {
    public String name;

    User(String name) {
        this.name = name;
    }
}
```

​	截图如下：

![image-20210317154636836](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210317154636836.png)	

​	只是简单的修改了一下，最后的结果就不一样了。这里将参数传递的变量进行初始化，相当于在堆中又创建了一个User对象，和主方法中的user对象所指向的对象是两个完全不同的对象，所以这里的修改也就没有起效。两次的打印也就不一样。

​	到这里，是不是对于Java中参数传递类型就更加清楚了一些。能力有限，如果文中有些错误，欢迎大佬指正。

​	本文也借鉴了一些博客，博客地址：[深入理解Java中方法的参数传递机制](https://www.cnblogs.com/sum-41/p/10799555.html)

​	就这样吧，结束。



​	

