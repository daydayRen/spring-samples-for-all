# spring 整合 rabbitmq（xml配置方式）

## 目录<br/>
<a href="#一说明">一、说明</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#11-项目结构说明">1.1 项目结构说明</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#12-依赖说明">1.2 依赖说明</a><br/>
<a href="#二spring-rabbit-基本配置">二、spring rabbit 基本配置</a><br/>
<a href="#三简单消费的发送">三、简单消费的发送</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#31-声明交换机队列绑定关系和消费者监听器">3.1 声明交换机、队列、绑定关系和消费者监听器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#32-测试简单消息的发送">3.2 测试简单消息的发送</a><br/>
<a href="#四传输对象">四、传输对象</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#41-创建消息的委托处理器">4.1 创建消息的委托处理器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#42-声明交换机队列绑定关系和消费者监听器">4.2 声明交换机、队列、绑定关系和消费者监听器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#43-测试对象消息的发送">4.3 测试对象消息的发送</a><br/>
## 正文<br/>


## 一、说明

### 1.1 项目结构说明

1. 本用例关于 rabbitmq 的整合提供**简单消息发送**和**对象消费发送**两种情况下的 sample。

2. rabbitBaseAnnotation.java 中声明了 topic 类型的交换机、持久化队列、及其绑定关系，用于测试说明 topic 交换机路由键的绑定规则。

3. rabbitObjectAnnotation.java 中声明了 direct 类型的交换机，持久化队列，及其绑定关系，用于示例对象消息的传输。

   注：关于 rabbitmq 安装、交换机、队列、死信队列等基本概念可以参考我的手记[《RabbitMQ 实战指南》读书笔记](https://github.com/heibaiying/LearningNotes/blob/master/notes/%E4%B8%AD%E9%97%B4%E4%BB%B6/RabbitMQ/%E3%80%8ARabbitMQ%E5%AE%9E%E6%88%98%E6%8C%87%E5%8D%97%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0.md),里面有详细的配图说明。



<div align="center"> <img src="https://github.com/heibaiying/spring-samples-for-all/blob/master/pictures/spring-rabbitmq.png"/> </div>



### 1.2 依赖说明

除了 spring 的基本依赖外，需要导入 spring rabbitmq 整合依赖

```xml
 <!--spring rabbitmq 整合依赖-->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>
<!--rabbitmq 传输对象序列化依赖了这个包-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.8</version>
</dependency>
```



## 二、spring rabbit 基本配置

```properties
rabbitmq.addresses=localhost:5672
rabbitmq.username=guest
rabbitmq.password=guest
# 虚拟主机，可以类比为命名空间 默认为/  必须先用图形界面或者管控台添加 程序不会自动创建且会抛出异常
rabbitmq.virtualhost=/
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation=
               "http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/rabbit
          http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">

    <context:property-placeholder location="rabbitmq.properties"/>

    <!--声明连接工厂-->
    <rabbit:connection-factory id="connectionFactory"
                               addresses="${rabbitmq.addresses}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"
                               virtual-host="${rabbitmq.virtualhost}"/>

    <!--创建一个管理器（org.springframework.amqp.rabbit.core.RabbitAdmin），用于管理交换，队列和绑定。
    auto-startup 指定是否自动声明上下文中的队列,交换和绑定, 默认值为 true。-->
    <rabbit:admin connection-factory="connectionFactory" auto-startup="true"/>

    <!--声明 template 的时候需要声明 id 不然会抛出异常-->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory"/>


    <!--可以在 xml 采用如下方式声明交换机、队列、绑定管理 但是建议使用代码方式声明 方法更加灵活且可以采用链调用-->
    <rabbit:queue name="remoting.queue"/>

    <rabbit:direct-exchange name="remoting.exchange">
        <rabbit:bindings>
            <rabbit:binding queue="remoting.queue" key="remoting.binding"/>
        </rabbit:bindings>
    </rabbit:direct-exchange>

    <!--扫描 rabbit 包 自动声明交换器、队列、绑定关系-->
    <context:component-scan base-package="com.heibaiying.rabbit"/>

</beans>
```



## 三、简单消费的发送

#### 3.1 声明交换机、队列、绑定关系和消费者监听器

```java
import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.rabbit.listener.api.ChannelAwareMessageListener;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author : heibaiying
 * @description : 声明队列、交换机、绑定关系、和队列消息监听
 */

@Configuration
public class RabbitBaseAnnotation {

    @Bean
    public TopicExchange exchange() {
        // 创建一个持久化的交换机
        return new TopicExchange("topic01", true, false);
    }

    @Bean
    public Queue firstQueue() {
        // 创建一个持久化的队列 1
        return new Queue("FirstQueue", true);
    }

    @Bean
    public Queue secondQueue() {
        // 创建一个持久化的队列 2
        return new Queue("SecondQueue", true);
    }

    /**
     * BindingKey 中可以存在两种特殊的字符串“#”和“*”，其中“*”用于匹配一个单词，“#”用于匹配零个或者多个单词
     * 这里我们声明三个绑定关系用于测试 topic 这种类型交换器
     */
    @Bean
    public Binding orange() {
        return BindingBuilder.bind(firstQueue()).to(exchange()).with("*.orange.*");
    }

    @Bean
    public Binding rabbit() {
        return BindingBuilder.bind(secondQueue()).to(exchange()).with("*.*.rabbit");
    }

    @Bean
    public Binding lazy() {
        return BindingBuilder.bind(secondQueue()).to(exchange()).with("lazy.#");
    }


    /*创建队列 1 消费者监听*/
    @Bean
    public SimpleMessageListenerContainer firstQueueLister(ConnectionFactory connectionFactory) {

        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        // 设置监听的队列
        container.setQueues(firstQueue());
        // 指定要创建的并发使用者数。
        container.setConcurrentConsumers(1);
        // 设置消费者数量的上限
        container.setMaxConcurrentConsumers(5);
        // 设置是否自动签收消费 为保证消费被成功消费，建议手工签收
        container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        container.setMessageListener(new ChannelAwareMessageListener() {
            @Override
            public void onMessage(Message message, Channel channel) throws Exception {
                // 可以在这个地方得到消息额外属性
                MessageProperties properties = message.getMessageProperties();
                //得到消息体内容
                byte[] body = message.getBody();
                System.out.println(firstQueue().getName() + "收到消息:" + new String(body));
                /*
                 * DeliveryTag 是一个单调递增的整数
                 * 第二个参数 代表是否一次签收多条，如果设置为 true,则所有 DeliveryTag 小于该 DeliveryTag 的消息都会被签收
                 */
                channel.basicAck(properties.getDeliveryTag(), false);
            }
        });
        return container;
    }


    /*创建队列 2 消费者监听*/
    @Bean
    public SimpleMessageListenerContainer secondQueueLister(ConnectionFactory connectionFactory) {

        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        container.setQueues(secondQueue());
        container.setMessageListener(new ChannelAwareMessageListener() {
            @Override
            public void onMessage(Message message, Channel channel) throws Exception {
                byte[] body = message.getBody();
                System.out.println(secondQueue().getName() + "收到消息:" + new String(body));
            }
        });
        return container;
    }

}
```

#### 3.2 测试简单消息的发送

```java
/**
 * @author : heibaiying
 * @description : 传输简单字符串
 */

@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "classpath:rabbitmq.xml")
public class RabbitTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void sendMessage() {
        MessageProperties properties = new MessageProperties();

        String allReceived = "我的路由键 quick.orange.rabbit 符合 queue1 和 queue2 的要求，我应该被两个监听器接收到";
        Message message1 = new Message(allReceived.getBytes(), properties);
        rabbitTemplate.send("topic01", "quick.orange.rabbit", message1);

        String firstReceived = "我的路由键 quick.orange.fox 只符合 queue1 的要求，只能被 queue 1 接收到";
        Message message2 = new Message(firstReceived.getBytes(), properties);
        rabbitTemplate.send("topic01", "quick.orange.fox", message2);

        String secondReceived = "我的路由键 lazy.brown.fox 只符合 queue2 的要求，只能被 queue 2 接收到";
        Message message3 = new Message(secondReceived.getBytes(), properties);
        rabbitTemplate.send("topic01", "lazy.brown.fox", message3);

        String notReceived = "我的路由键 quick.brown.fox 不符合 topic1 任何绑定队列的要求，你将看不到我";
        Message message4 = new Message(notReceived.getBytes(), properties);
        rabbitTemplate.send("topic01", "quick.brown.fox", message4);
    }
}
```

```java
结果:
  SecondQueue 收到消息:我的路由键 quick.orange.rabbit 符合 queue1 和 queue2 的要求，我应该被两个监听器接收到
  FirstQueue 收到消息:我的路由键 quick.orange.rabbit 符合 queue1 和 queue2 的要求，我应该被两个监听器接收到
  FirstQueue 收到消息:我的路由键 quick.orange.fox 只符合 queue1 的要求，只能被 queue 1 接收到
  SecondQueue 收到消息:我的路由键 lazy.brown.fox 只符合 queue2 的要求，只能被 queue 2 接收到
```



## 四、传输对象

#### 4.1 创建消息的委托处理器

这里为了增强用例的实用性，我们创建的处理器的 handleMessage 方法是一个重载方法，对于同一个队列的监听，不仅可以传输对象消息，同时针对不同的对象类型调用不同的处理方法。

```java
/**
 * @author : heibaiying
 * @description :消息委派处理类
 */
public class MessageDelegate {

    public void handleMessage(ProductManager manager) {
        System.out.println("收到一个产品经理" + manager);
    }

    public void handleMessage(Programmer programmer) {
        System.out.println("收到一个程序员" + programmer);
    }

}
```

#### 4.2 声明交换机、队列、绑定关系和消费者监听器

```java
/**
 * @author : heibaiying
 * @description : 声明队列、交换机、绑定关系、用于测试对象的消息传递
 */

@Configuration
public class RabbitObjectAnnotation {

    @Bean
    public DirectExchange objectTopic() {
        // 创建一个持久化的交换机
        return new DirectExchange("objectTopic", true, false);
    }

    @Bean
    public Queue objectQueue() {
        // 创建一个持久化的队列
        return new Queue("objectQueue", true);
    }

    @Bean
    public Binding binding() {
        return BindingBuilder.bind(objectQueue()).to(objectTopic()).with("object");
    }


    /*创建队列消费者监听*/
    @Bean
    public SimpleMessageListenerContainer objectQueueLister(ConnectionFactory connectionFactory) {

        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        // 设置监听的队列
        container.setQueues(objectQueue());
        // 将监听到的消息委派给实际的处理类
        MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
        // 指定由哪个方法来处理消息 默认就是 handleMessage
        adapter.setDefaultListenerMethod("handleMessage");

        // 消息转换
        Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter();
        DefaultJackson2JavaTypeMapper javaTypeMapper = new DefaultJackson2JavaTypeMapper();

        Map<String, Class<?>> idClassMapping = new HashMap<>();
        // 针对不同的消息体调用不同的重载方法
        idClassMapping.put(Type.MANAGER, com.heibaiying.bean.ProductManager.class);
        idClassMapping.put(Type.PROGRAMMER, com.heibaiying.bean.Programmer.class);

        javaTypeMapper.setIdClassMapping(idClassMapping);

        jackson2JsonMessageConverter.setJavaTypeMapper(javaTypeMapper);
        adapter.setMessageConverter(jackson2JsonMessageConverter);
        container.setMessageListener(adapter);
        return container;
    }

}
```

#### 4.3 测试对象消息的发送

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "classpath:rabbitmq.xml")
public class RabbitSendObjectTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void sendProgrammer() throws JsonProcessingException {
        MessageProperties messageProperties = new MessageProperties();
        //必须设置 contentType 为 application/json
        messageProperties.setContentType("application/json");
        // 必须指定类型
        messageProperties.getHeaders().put("__TypeId__", Type.PROGRAMMER);
        Programmer programmer = new Programmer("xiaoming", 34, 52200.21f, new Date());
        // 序列化与反序列化都使用的 Jackson
        ObjectMapper mapper = new ObjectMapper();
        String programmerJson = mapper.writeValueAsString(programmer);
        Message message = new Message(programmerJson.getBytes(), messageProperties);
        rabbitTemplate.send("objectTopic", "object", message);
    }


    @Test
    public void sendProductManager() throws JsonProcessingException {
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setContentType("application/json");
        messageProperties.getHeaders().put("__TypeId__", Type.MANAGER);
        ProductManager manager = new ProductManager("xiaohong", 21, new Date());
        ObjectMapper mapper = new ObjectMapper();
        String managerJson = mapper.writeValueAsString(manager);
        Message message = new Message(managerJson.getBytes(), messageProperties);
        rabbitTemplate.send("objectTopic", "object", message);
    }
}
```
