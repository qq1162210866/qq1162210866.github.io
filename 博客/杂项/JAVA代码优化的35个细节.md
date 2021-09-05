# JAVA代码优化的35个细节
* 1、尽量指定类、方法的final修饰符  
	为类指定final修饰符可以让类不可以被继承，为方法指定final修饰符可以让方法不可以被重写。如果指定了一个类为final，则该类所有的方法都是final的。如果一个方法没有被覆盖并且很短，编译器就能够对它进行优化处理，这个过程就是内联。
* 2、尽量重用对象  
新建对象不仅仅要花时间去生成，还要花时间去回收。对于字符串的操作最好是使用StringBuilder/StringBuffer。

        /**
         * 不正确的方式，会新建多个String对象
         */
        String s = "a" + "b";
        System.out.println(s);
        /**
         * 使用StringBuilder/StringBuffer的方法来拼接字符串
         */
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("a").append("b");
        System.out.println(stringBuilder);
        
* 3、尽可能使用局部变量   
	调用方法时传递的参数以及在调用中创建的临时变量都保存在栈中速度较快，其他变量，如静态变量、实例变量等，都在堆中创建，速度较慢。另外，栈中创建的变量，随着方法的运行结束，这些内容就没了，不需要额外的垃圾回收。
	* 栈内存:栈内存首先是一片内存区域，存储的都是局部变量，凡是定义在方法中的都是局部变量（方法外的是全局变量），for循环内部定义的也是局部变量，是先加载函数才能进行局部变量的定义，所以方法先进栈，然后再定义变量，变量有自己的作用域，一旦离开作用域，变量就会被释放。栈内存的更新速度很快，因为局部变量的生命周期都很短。  
	*  堆内存:存储的是数组和对象（其实数组就是对象），凡是new建立的都是在堆中，堆中存放的都是实体（对象），实体用于封装数据，而且是封装多个（实体的多个属性），如果一个数据消失，这个实体也没有消失，还可以用，所以堆是不会随时释放的，但是栈不一样，栈里存放的都是单个变量，变量被释放了，那就没有了。堆里的实体虽然不会被释放，但是会被当成垃圾，Java有垃圾回收机制不定时的收取。  

	```
	 int [] arr=new int [3];
	```  
	这样先在栈中创建一个arr变量，再给这个变量赋值，发现右边是一个实体，所以在堆中创建一个实体空间。
* 4、及时关闭流  
	对流的操作结束了，要及时关闭以释放资源。
* 5、尽量减少对变量的重复计算	
	调用方法，哪怕只有一行，也是有消耗的。
		
		for (int i = 0; i < arr.length; i++) {
            //频繁调用了arr.length方法
        }
        int length = arr.length;
        for (int i = 0; i < length; i++) {
            //比上一个少调用了length方法
        }
* 6、尽量采用懒加载的策略，即在需要的时候才创建	
	判断创建对象时，尽量在if语句里面创建对象，采用懒加载方式。
* 7、慎用异常	
	异常只能用于错误处理，不应该用来控制程序流程。
* 8、不要在循环中使用try…catch…，应该把其放在最外层
* 9、如果能估计到待添加的内容长度，为底层以数组方式实现的集合、工具类指定初始长度	如果初始容量为5000，默认初始长度为16（StringBuilder），这样就会一直在扩展和复制数组，造成资源浪费。
* 10、当复制大量数据时，使用System.arraycopy命令
* 11、乘法和除法使用移位操作		
	最好加上注释，方便别人理解
	
		for (int var = 0; var < 100; var += 5) {
            int a = var * 8;
            int b = var / 2;
            System.err.println("a="+a+": b="+b);
        }
        for (int var = 0; var < 100; var += 5) {
            int a = var << 3;
            int b = var >> 1;
            System.err.println("a="+a+": b="+b);
        }
        
		<<      :    左移运算符，num <<1,相当于num乘以2
		>>      :    右移运算符，num >>1,相当于num除以2
		>>>    :    无符号右移，忽略符号位，空位都以0补齐，（计算机中数字以补码存储，首位为符号位）。
* 12、循环内不要不断创建对象引用		
	这样导致内存中有循环次数个对象。建议在循环外声明，在循环内重新赋值。
* 13、基于效率和类型检查的考虑，应该尽可能使用array，无法确定数组大小时才使用ArrayList
* 14、尽量使用HashMap、ArrayList、StringBuilder，除非线程安全需要，否则不推荐使用Hashtable、Vector、StringBuffer，后三者由于使用同步机制而导致了性能开销
* 15、不要将数组声明为public static final	
	将数组声明为static和final没有意义，因为数组的内容还是可以改变。
* 16、尽量在合适的场合使用单例	
单例适用的范围。
	* 控制资源的使用，通过线程同步来控制资源的并发访问
	* 控制实例的产生，以达到节约资源的目的
	* 控制数据的共享，在不建立直接关联的条件下，让多个不相关的进程或线程之间实现通信
* 17、尽量避免随意使用静态变量	
	如果某个对象被静态变量引用，除非静态变量所在的类被卸载，否则这个对象是不会被gc回收的。
* 18、及时清除不再需要的会话        
	当会话不再需要时，应当及时调用HttpSession的invalidate方法清除会话。
* 19、实现RandomAccess接口的集合比如ArrayList，应当使用最普通的for循环而不是foreach循环来遍历	
	实现RandomAccess接口用来表明其支持快速随机访问，此接口的主要目的是允许一般的算法更改其行为，从而将其应用到随机或连续访问列表时能提供良好的性能。实际经验表明，实现RandomAccess接口的类实例，假如是随机访问的，使用普通for循环效率将高于使用foreach循环；
		
		ArrayList arrayList = new ArrayList<String>(100);
        if (arrayList instanceof RandomAccess) {
            /**
             *  先判断list是不是RandomAccess的实现类，如果是使用for循环，否则使用迭代器
             */
            int listLength = arrayList.size();
            for (int i = 0; i < listLength; i++) {
            }
        } else {
            Iterator<String> stringIterator = arrayList.iterator();
            while (stringIterator.hasNext()) {
                stringIterator.next();
            }
        }
        
* 20、使用同步代码块替代同步方法	
	除非能确定一整个方法都是需要进行同步的，否则尽量使用同步代码块，避免对那些不需要进行同步的代码也进行了同步，影响了代码执行效率。
* 21、将常量声明为static final，并以大写命名
* 22、不要创建一些不使用的对象，不要导入一些不使用的类
* 23、程序运行过程中避免使用反射	
	将那些需要通过反射加载的类在项目启动的时候通过反射实例化出一个对象并放入内存
* 24、使用数据库连接池和线程池
* 25、使用带缓冲的输入输出流进行IO操作
* 26、顺序插入和随机访问比较多的场景使用ArrayList，元素删除和中间插入比较多的场景使用LinkedList		
ArrayList从名字上来讲是数组列表，表面上是动态大小，其底层实现原理其实还是一个数组。LinkedList实际上是用双向循环链表实现的。因为ArrayList有索引，所以顺序插入使用ArrayList比较快。中间插入使用LinkedList比较快，因为只需要修改两个引用就可以了。
* 27、不要让public方法中有太多的形参	
	形参太多不符合java万物皆对象的理念，多个形参可以封装为一个实体对象。
* 28、字符串变量和字符串常量equals的时候将字符串常量写在前面		
避免空指针异常。
* 29、在java中if (i == 1)和if (1 == i)是没有区别的，但从阅读习惯上讲，建议使用前者
* 30、不要对数组使用toString方法	
数组的toString方法会打印出数组引用对象的地址，有可能产生空指针异常。集合的toString可以打印所有元素，因为它重写了父类的toString方法。
* 31、不要对超出范围的基本数据类型做向下强制转型	
超出范围的基本数据强转的1话会导致数据变化，并且很难得到想要的数据。

		Long aLong = 123456789012345L;
        int i = (int) aLong;
        System.out.println(i);
* 32、公用的集合类中不使用的数据一定要及时remove掉	
公用集合（也就是说不是方法里面的属性）中的数据不及时remove掉会有内存泄漏的危险。
* 33、把一个基本数据类型转为字符串，基本数据类型.toString是最快的方式、String.valueOf次之、数据+””最慢		
	* 1、String.valueOf方法底层调用了Integer.toString方法，但是会在调用前做空判断
	* 2、Integer.toString方法就不说了，直接调用了
	* 3、i + “”底层使用了StringBuilder实现，先用append方法拼接，再用toString方法获取字符
   
   	这个暂时不知道原因，现在也没法得出结论，有大佬知道的可以指点一下。	
* 34、使用最有效率的方式去遍历Map	

		HashMap<String, String> stringStringHashMap = new HashMap<>();
        stringStringHashMap.put("aaa", "111");
        /**
         * 先获取map中各个键值对映射关系的集合，再使用迭代器来遍历这个集合达到遍历map的作用
         * 如果只是想遍历一下这个Map的key值，可以使用 Set keySet = hm.keySet;
         */
        Set<Map.Entry<String, String>> entrySet = stringStringHashMap.entrySet();
        Iterator<Map.Entry<String, String>> iterator = entrySet.iterator();
        while (iterator.hasNext()) {
            Map.Entry<String, String> stringStringEntry = iterator.next();
            System.out.println(stringStringEntry.getKey() + "\t" + stringStringEntry.getValue());
        }
* 35、对资源的close建议分开操作 	
这样操作是为了避免一个资源关闭异常导致下一个资源无法关闭的情况。