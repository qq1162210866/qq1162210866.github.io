## Java代码实现ping命令

​	在一个项目中，遇到了一个问题，需要查看某个IP是否能够ping通，这里就直接使用Java代码实现了，记录一下。

​	先说一下实现的几个方法。

* Jdk1.5的InetAddresss方式

* 调用本机CMD

* Java调用控制台执行ping命令
  

​	下面也就不不废话了，直接上代码吧。

```java
package network_train;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.InetAddress;

/**
 * NetWorkTrain.java
 * Description: 网络测试demo
 *
 * @author Peng Shiquan
 * @date 2020/6/14
 */
public class NetWorkTrain {
    /**
     * Description: 主方法
     *
     * @param args
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/6/15
     */
    public static void main(String[] args) {
        String ipAddress = "192.168.16.127";


        try {
            /**
             * Jdk1.5的InetAddresss方式
             */
            System.err.println("Jdk1.5的InetAddresss方式");
            System.err.println("主机的状态，" + pingforInetAddresss(ipAddress));

            /**
             * 直接调用CMD
             */
            System.err.println("直接调用CMD");
            //pingForCMD(ipAddress);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.err.println("更加完善的调用CMD方法");
        System.err.println("主机的状态" + pingForExec(ipAddress, 5, 1000));
    }

    /**
     * Description: Jdk1.5的InetAddresss方式，返回值是true时，说明host是可用的
     *
     * @param ipAddress
     * @return boolean
     * @Author: Peng Shiquan
     * @Date: 2020/6/15
     */
    public static boolean pingforInetAddresss(String ipAddress) throws Exception {
        //超时应该在3秒以上
        int timeOut = 3000;
        // 当返回值是true时，说明host是可用的，false则不可。
        boolean status = InetAddress.getByName(ipAddress).isReachable(timeOut);
        return status;
    }

    /**
     * Description: 直接调用CMD,方法直接将CMD窗口的信息打印出来
     *
     * @param ipAddress
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/6/15
     */
    public static void pingForCMD(String ipAddress) throws Exception {
        String line = null;
        try {
            Process pro = Runtime.getRuntime().exec("ping " + ipAddress);
            BufferedReader buf = new BufferedReader(new InputStreamReader(
                    pro.getInputStream()));
            while ((line = buf.readLine()) != null)
                System.out.println(line);
        } catch (Exception ex) {
            System.out.println(ex.getMessage());
        }
    }

    /**
     * Description: 更加完善的调用CMD方法
     *
     * @param ipAddress
     * @param pingTimes
     * @param timeOut
     * @return boolean
     * @Author: Peng Shiquan
     * @Date: 2020/6/15
     */
    public static boolean pingForExec(String ipAddress, int pingTimes, int timeOut) {
        BufferedReader in = null;
        // 将要执行的ping命令,此命令是windows格式的命令
        Runtime r = Runtime.getRuntime();

        String pingCommand = "ping " + "-c " + pingTimes + " " + ipAddress;

        try {   // 执行命令并获取输出
            System.out.println(pingCommand);
            Process p = r.exec(pingCommand);
            if (p == null) {
                return false;
            }
            // 逐行检查输出,计算类似出现=23ms TTL=62字样的次数
            in = new BufferedReader(new InputStreamReader(p.getInputStream()));
            int connectedCount = 0;
            String line = null;
            while ((line = in.readLine()) != null) {
                connectedCount += getCheckResult(line);
                // 如果出现类似=23ms TTL=62这样的字样,出现的次数=测试次数则返回真
            }
            return connectedCount == pingTimes;
        } catch (Exception ex) {
            ex.printStackTrace();   // 出现异常则返回假
            return false;
        } finally {
            try {
                if (null != in) {
                    in.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * Description: 若line含有=18ms TTL=16字样,说明已经ping通,返回1,否則返回0.
     *
     * @param line
     * @return int
     * @Author: Peng Shiquan
     * @Date: 2020/6/15
     */
    private static int getCheckResult(String line) {
        //System.out.println("控制台输出的结果为:" + line);
        String trueZF = "time=";
        if (line.contains(trueZF)) {
            return 1;
        } else {
            return 0;
        }
    }
}
```

​	运行的截图如下：

​	成功的：

![image-20200615154620793](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200615154620793.png)

​	失败的：

![image-20200615154550305](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200615154550305.png)

​	下面就说说中间的问题。

​	一个是不同的电脑系统，代码中的命令是不一样的，我这里的是Mac os，所以命令和Windows上面的有所不同。

* Mac os（或者Linux）:ping -c 5 127.0.0.1 

* Windows： ping 127.0.0.1 -n 5 -w 1000

​	所以不同的系统要拼接的命令也是不一样的。然后就是需要注意最后结果的分析，我这里的代码是直接判断有没有`time=`,有就是成功了。大家也可以根据自己的需求对延迟做一定对要求，我这里比较简单，就没有写的太复杂。到这里就基本完成了。

​	就这样吧，结束。