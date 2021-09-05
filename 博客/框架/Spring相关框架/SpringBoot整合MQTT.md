## SpringBoot整合MQTT

​	公司的项目重新进行了优化，换成了SpringBoot的架构。之前写的一个SSM整合MQTT的demo不能使用了，其实之前的代码写的也是有点烂，所以就来了解一下SpringBoot整合MQTT，顺便写一个比较好的代码。下面直接开始。

​	关于MQTT协议的一些介绍之前已经讲过了，之前对于协议的记录博客还是可以的。博客地址：`https://blog.csdn.net/qq_38533859/article/details/81872622`。所以这里不再介绍，直接开整。

​	需要添加pom依赖，这个是官方整合的依赖，官方也有例子（虽然不怎么样）。依赖如下：

```xml
        <!--  MQTT依赖 -->
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-mqtt</artifactId>
            <version>5.3.1.RELEASE</version>
        </dependency>
```

​	剩下就是配置入站适配器和出站适配器了。这个是借鉴了一下官方的文档，文档地址在这里：`https://docs.spring.io/spring-integration/reference/html/mqtt.html#mqtt`

​	因为之前是使用xml文件对mqtt适配器进行配置，所以这个新项目非常不想这样做，最后采用JavaConfig的形式配置。官方的例子写的很清楚了。我的入站适配器和出站适配器写在一起了。MQTT的配置文件如下：

```java
package com.psq.train.config;

import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.MessagingGateway;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.core.MessageProducer;
import org.springframework.integration.mqtt.core.DefaultMqttPahoClientFactory;
import org.springframework.integration.mqtt.core.MqttPahoClientFactory;
import org.springframework.integration.mqtt.inbound.MqttPahoMessageDrivenChannelAdapter;
import org.springframework.integration.mqtt.outbound.MqttPahoMessageHandler;
import org.springframework.integration.mqtt.support.DefaultPahoMessageConverter;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHandler;
import org.springframework.messaging.MessagingException;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

/**
 * MQTTConfig.java
 * Description:  MQTT的配置类，主要配置出入站适配器
 *
 * @author Peng Shiquan
 * @date 2020/7/13
 */
@Configuration
public class MQTTConfig {


    /**
     * =====================================入站适配器1=====================================
     */

    @Bean
    public MessageChannel mqttInputChannel() {
        return new DirectChannel();
    }

    @Bean
    public MessageProducer inbound() {
        MqttPahoMessageDrivenChannelAdapter adapter =
                new MqttPahoMessageDrivenChannelAdapter("tcp://127.0.0.1:1883", "testClient1",
                        "testTopic2");
        adapter.setCompletionTimeout(5000);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(1);
        adapter.setOutputChannel(mqttInputChannel());
        return adapter;
    }

    @Bean
    @ServiceActivator(inputChannel = "mqttInputChannel")
    public MessageHandler handler() {
        return new MessageHandler() {
            @Override
            public void handleMessage(Message<?> message) throws MessagingException {
                System.out.println(message.getPayload() + "=====" + message.getHeaders().get("mqtt_receivedTopic"));

            }

        };
    }


    /**
     * =====================================入站适配器2=====================================
     */
    @Bean
    public MessageChannel mqttInputChannelTwo() {
        return new DirectChannel();
    }

    @Bean
    public MessageProducer inboundTwo() {
        MqttPahoMessageDrivenChannelAdapter adapter =
                new MqttPahoMessageDrivenChannelAdapter("tcp://127.0.0.1:1883", "testClient3",
                        "testTopic2");
        adapter.setCompletionTimeout(5000);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(1);
        adapter.setOutputChannel(mqttInputChannelTwo());
        return adapter;
    }

    @Bean
    @ServiceActivator(inputChannel = "mqttInputChannelTwo")
    public MessageHandler handlerTwo() {
        return new MessageHandler() {
            @Override
            public void handleMessage(Message<?> message) throws MessagingException {
                System.out.println(message.getPayload() + "=====" + message.getHeaders().get("mqtt_receivedTopic"));
            }

        };
    }

    /**
     * =====================================出站适配器=====================================
     */

    @Bean
    public MqttPahoClientFactory mqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        MqttConnectOptions options = new MqttConnectOptions();
        options.setServerURIs(new String[]{"tcp://127.0.0.1:1883"});
        options.setUserName("username");
        options.setPassword("password".toCharArray());
        factory.setConnectionOptions(options);
        return factory;
    }

    @Bean
    @ServiceActivator(inputChannel = "mqttOutboundChannel")
    public MessageHandler mqttOutbound() {
        MqttPahoMessageHandler messageHandler =
                new MqttPahoMessageHandler("testClient2", mqttClientFactory());
        messageHandler.setAsync(true);
        messageHandler.setDefaultTopic("test");
        return messageHandler;
    }

    @Bean
    public MessageChannel mqttOutboundChannel() {
        return new DirectChannel();
    }


    /**
     * =====================================发送的接口=====================================
     */
    @Component
    @MessagingGateway(defaultRequestChannel = "mqttOutboundChannel")
    public interface MyGateway {

        void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, String data);
    }
}
```

​	上面就是代码，解释都在官方文档里面了，大家搞个翻译，翻译一下官方文档就大致明白了。这里说一些注意的点。

* 代码中配置了两个入站适配器，一个出站适配器，这三个适配器的ClientID最好不要一样，要不然会一直出现断连重连现象。

* 发送的接口最好分离出去一个单独的类，我写在一起，但是实际使用的时候一直出问题，因为时间问题也没有去找原因。

* 代码的这种写法不是MQTT的单独的写法，我了解到这个好像是Spring的一种模式吧，没有来得及了解，后续肯定会去了解一下。

* SpringBoot的这种整合有一些BUG，好像是MQTT官方解决了，但是SpringBoot官方还存在这个问题。具体没有遇到这个问题，所以不清楚现象，留个坑。

* 官方还可以写一个监听函数，用来监听连接成功和断连的时候，这里没有上代码，没有啥可以讲的。在官方文档的最后面有提到。

​	下面就是上调用的代码，这里直接使用controller调用了，没有啥复杂的。

```java
@RequestMapping(value = "/mqtt", method = RequestMethod.GET)
public String sendMQTTMsg() {
    myGateway.sendToMqtt("testTopic1", "hello1");
    myGateway.sendToMqtt("testTopic2", "hello2");
    return "hello";
}
```

​	剩下就没有啥可以讲的了，时间赶的急，后面遇到再说吧，再次留个坑。

​	就这样吧，结束。