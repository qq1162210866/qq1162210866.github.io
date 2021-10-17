## Java反射

* 编译时的类型由声明对象来决定，运行时类型由赋值对象来决定。即编译看左边，运行看右边。
* 代理模式：为其他对象提供一种代理以控制对这个对象的访问。
* 静态代理：就是在运行前就存在代理的代码，代理类和原始类的关系在运行前就已经确定了。缺点时维护成本高，修改原始类时还要修改代理类。
* 动态代理：在程序运行期间，通过JVM反射等机制动态的生成代理类的代码，代理类和原始类的关系是在运行后才确定的。

### 反射基础概念

反射指的是在运行状态中，对于任意一个类能够知道这个类的所有属性和方法，对于任意一个对象，都能调用它的任意一个方法和属性。

代码示例：

```java
package ReflectionTrain;

/**
 * 反射小练习
 * 
 * @author yanyu
 *
 */
public class demo1 
{
   public static void main(String[] args)
   {
      try{
         //通过类的路径来获取Class
         /*
          * 1 class.forName("类的路径")
          * 2 类名.Class
          * 3 实例.getClass
          */          
         Class a=Class.forName("ReflectionTrain.Sub");
         /**
          * 1 newInstance: 弱类型。低效率。只能调用无参构造。
          * 2 new: 强类型。相对高效。能调用任何public构造。
          */
         Base b=(Base)a.newInstance();
         //为什么打印的是 Sub？
         /*
          * 子类强转为父类，向上转换
          * 通过向上转换，我们能够在编写程序时采用通用程序设计的思想，
          * 在需要使用子类对象的时候，通过把变量定义为父类型，我们可以通过一个变量，使用该父类型的所有子类型实例
          * 子类可以转换为父类，即父类引用指向子类对象。引用的属性是父类的，方法若果被子类重写则是子类的方法。
          */
         b.f();
      }catch (Exception e) {
         e.printStackTrace();
      }
   }

}
/**
 * 
 * @author yanyu
 *
 */
class Base
{
   public void f()
   {
      System.out.println("Base");
   }
}

/**
 * 
 * @author yanyu
 *
 */
class Sub extends Base
{
   public void f()
   {
      System.out.println("Sub");
   }
   
}
/**
 * 
 * @author yanyu
 * 测试类，测试子类与父类的关系
 */
class Sub2 extends Base
{
   public void f()
   {
      System.out.println("Sub2");
   }
   
}
```

反射相关的类：

* Class类：用来表示类的信息。
* Field类：表示类的成员变量。
* Method类：表示类的方法。
* Constructor类：用来表示类的构造方法。

获取Class的三种方法

​	获取Class对象有三种方法：1）通过对象方法。object.getClass();。2）调用类的Class属性，即：Object.class。3）通过Class类中的ForName方法获取，即：Class.forName("类的全路径").

​	创建对象的两种方式：调用类的newInstance()方法来创建、通过Class获取Constructor对象，再调用newInstance()方法，这种方法可以选定构造方法。

​	Java有5种方式来创建对象： 1、使用 new 关键字（最常用）： ObjectName obj = new ObjectName(); 2、使用反射的Class类的newInstance()方法： ObjectName obj = ObjectName.class.newInstance(); 3、使用反射的Constructor类的newInstance()方法： ObjectName obj = ObjectName.class.getConstructor.newInstance(); 4、使用对象克隆clone()方法： ObjectName obj = obj.clone(); 5、使用反序列化（ObjectInputStream）的readObject()方法： try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(FILE_NAME))) { ObjectName obj = ois.readObject(); }

### 反射需要注意的

* 子类对象确实拥有父类对象中所有的属性和方法，但是父类对象中的私有属性和方法，子类是无法访问到的，只是拥有，但不能使用。（可以通过反射获取父类的字段）
* 编译时的类型由声明对象来决定，运行时类型由赋值对象来决定。即编译看左边，运行看右边。
* 代理模式：为其他对象提供一种代理以控制对这个对象的访问。
* 静态代理：就是在运行前就存在代理的代码，代理类和原始类的关系在运行前就已经确定了。缺点时维护成本高，修改原始类时还要修改代理类。
* 动态代理：在程序运行期间，通过JVM反射等机制动态的生成代理类的代码，代理类和原始类的关系是在运行后才确定的。
* Spring中的IOC使用到了反射，通过反射获取到配置里面类的实例对象，存入到Spring的bean容器中。

