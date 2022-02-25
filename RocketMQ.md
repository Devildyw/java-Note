#  [RocketMQ](https://github.com/apache/rocketmq/blob/master/docs/cn/concept.md)

**MQ(Message Queue)**:消息队列

## 基本概念

### 消息模型（Message Model）:

RocketMQ主要由 **Producer、Broker、Consumer 三部分组成**，其中Producer 负责生产消息，Consumer 负责消费消息，**Broker 负责存储消息**。Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。Message Queue 用于存储消息的物理地址，每个Topic中的消息地址存储于多个 Message Queue 中。ConsumerGroup 由多个Consumer 实例构成。

### 消息生产者（Producer）: 

负责生产消息，一般由业务系统负责生产消息。一个消息生产者会把业务应用系统里产生的消息发送到broker服务器。RocketMQ提供多种发送方式，**同步发送、异步发送、顺序发送、单向发送**。**同步和异步方式均需要Broker返回确认信息，单向发送不需要。**

### 消息消费者（Consumer）:

负责消费消息，一般是后台系统负责异步消费。一个消息消费者会从Broker服务器拉取消息、并将其提供给应用程序。从用户应用的角度而言提供了两种消费形式：拉取式消费、推动式消费。

### 主题（Topic）:

表示一类消息的集合，**每个主题包含若干条消息，每条消息只能属于一个主题**，(topic)**是RocketMQ进行消息订阅的基本单位。**

### 代理服务器（Broker Server）:

**消息中转角色，负责存储消息、转发消息。**代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。

### 名字服务（Name Server）:

名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。多个Namesrv实例组成集群，但相互独立，没有信息交换。

![2ms3i8hm3a](https://gitee.com/Devildyw/blogimage/raw/master/img/2ms3i8hm3a.jpg)

![image-20220222203606812](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220222203606812.png)

由上图可知 Broker集群,producer集群,consumer集群都要与NameServer集群进行通信.

### 拉取式消费（Pull Consumer）:

Consumer消费的一种类型，应用通常主动调用Consumer的拉消息方法从Broker服务器拉消息、**主动权由应用控制**。一旦获取了批量消息，应用就会启动消费过程。

### 推动式消费（Push Consumer）:

Consumer消费的一种类型，**该模式下Broker收到数据后会主动推送给消费端**，该消费模式一般实时性较高。

### 生产者组（Producer Group）:

**同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。**如果发送的是事务消息且原始生产者在发送之后崩溃，则Broker服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。

### 消费者组（Consumer Group）:

**同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。**消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。要注意的是，**消费者组的消费者实例必须订阅完全相同的Topic。**RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。

### 集群消费（Clustering）:

集群消费模式下,相同Consumer Group的每个Consumer实例平均分摊消息。

### 广播消费（Broadcasting）:

广播消费模式下，相同Consumer Group的每个Consumer实例都接收全量的消息。

### 普通顺序消息（Normal Ordered Message）:

普通顺序消费模式下，消费者通过同一个消息队列（ Topic 分区，称作 Message Queue） 收到的消息是有顺序的，不同消息队列收到的消息则可能是无顺序的。

### 严格顺序消息（Strictly Ordered Message）:

严格顺序消息模式下，消费者收到的所有消息均是有顺序的。

### 消息（Message）:

**消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须属于一个主题。**RocketMQ中**每个消息拥有唯一的Message ID**，且**可以携带具有业务标识的Key**。系统提供了**通过Message ID和Key查询消息**的功能。

### 标签（Tag）:

**为消息设置的标志，用于同一主题下区分不同类型的消息。**来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。**标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统**。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。

### MQ(Messages Queue)三大优点:

* 应用解耦: **提高系统的容错性和可维护性**
* 削峰填谷: **提升用户体验和系统的吞吐量**
* 异步提速:**提高系统的稳定性**

### 通常的MQ 三大缺点: 

* 应用可用性降低

  > 系统引入的外部依赖越多, 系统的稳定性越差,一旦MQ宕机,就会对业造成影响.

* 系统的复杂度提高

  > MQ的加入大大增加了系统的复杂度,以前系统间是同步的远程调用,现在是通过MQ进行异步调用.

* 一致性的问题(A B系统正常 但是C系统处理失败 会发生事务问题)

  > A系统处理完业务,通过MQ给BCD三个系统发送消息数据,如果B系统,C系统处理成功,D系统处理失败.



---

## 单对单模式(初始RocketMQ)

### 生产者

>* 谁来发
>* 发给谁
>* 启动连接
>* 发什么
>* 怎么发
>* 发的结果是?
>* 关闭连接

```java
public class Producer {
    public static void main(String[] args) throws MQBrokerException, RemotingException, InterruptedException, MQClientException {
        //1. 谁来发?
        //创建一个生产者
        DefaultMQProducer producer = new DefaultMQProducer(/*可以在这里设置名称*/"group1");

        //2.发给谁
        //发送给命名服务器 通过Name Server分配Brokerip 再由生产者发送给broker
        producer.setNamesrvAddr("localhost:9876");

        //启动连接
        producer.start();

        //3.怎么发
        //发送Message apache包下的 网络传输都是字节流传输
        Message message = new Message("Topic1","Tag1",("Hello World").getBytes());

        //4.发什么
        SendResult sendResult = producer.send(message);

        //5.发的结果是什么
        //SendResult 就是发送后的结果
        System.out.println(sendResult);

        //6.打扫战场
        //生产者是与name Server建立了一个长连接进行发送消息 所以发送完毕后 关闭连接
        producer.shutdown();
    }
}
```

### 消费者

>* 谁来收
>* 从哪里收
>* 监听那个消息队列
>* 处理业务流程
>* 启动连接

```java
public class Consumer {
    public static void main(String[] args) throws Exception{
        //1.谁来收
        //消费者有两种模式 一种是拉去(需要消费者自己去拉去) 一种是推送(消息主动推送给消费者)
        DefaultMQPushConsumer pushConsumer = new DefaultMQPushConsumer("group1");

        //2.从哪里收
        //与生产者一样 消费者 也许要去name Server中获得对应broker的地址去获得消息
        pushConsumer.setNamesrvAddr("localhost:9876");

        //3.监听那个消息队列
        //设置监听队列 subscribe:订阅 指定主题 和订阅表达式 "*"表示订阅主题中的所有
        pushConsumer.subscribe("Topic1","*");

        //4.处理业务流程
        //注册一个监听器 去监听是否有消息被生产 一有就立刻接收
        pushConsumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                //接收到的消息就是 List<MessageExt> msgs 这时我们就能写我们的业务逻辑
                for (MessageExt msg : msgs) {
                    System.out.println(new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //启动连接
        pushConsumer.start();

        System.out.println("消费者启动起来了");

        //注意不要关闭消费者(如果还有对应主题的生产者的情况下 关闭就无法监听消息 就无法收到消息了)
    }
}
```

## 一对多(单生产者 多消费者模式)

### 多消费者都在同一组中时

**消息会被分配到该组的不同消费者手中(当一个组中的消费者为偶数时平分)**

![image-20220223173114990](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220223173114990.png)

### 多消费者在不同组时

**每个组都会有完整的消息数目和消息信息(广播式 消息先被复制到不同的消费者组)**

![image-20220223173349238](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220223173349238.png)

**特别的:**如果想在同一组中实现广播模式 可以在接收消息前设置消息的模式

> `Consumer.setMessageModel(消息模式);`
>
> 默认是CLUSTERING 负载均衡模式
>
> 可以设置为BROADCASTING 就是广播模式

## 多对多(多生产者 多消费者模式)

对于生产者生产的消息而言



---

## 消息类别

### 同步消息

**特征:** 即时性较强,重要的消息,且必须有回执的消息,例如短息,通知(转账成功)

```java
public class SyncProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("Devilsproducer");
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();
        for (int i = 0; i < 100; i++) {
            Message message = new Message("TopicTest","TagA",("Hello RocketMq "+ i).getBytes(RemotingHelper.DEFAULT_CHARSET));
            SendResult sendResult = producer.send(message);
            System.out.printf("%s%n",sendResult);
        }
        producer.shutdown();
    }
}
```

### 异步消息

**特征:** 即时性较弱,但需要有回执的消息,例如订单中的某些信息

```java
public class AsyncProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("group3");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();

        for (int i = 0; i < 10; i++) {
            String msg = "Hello World";
            Message message = new Message("Topic3", "tag1", msg.getBytes());
            //异步消息 Callback也是一个多线的接口
            producer.send(message, new SendCallback() {
                //发送成功的回调方法a
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println(sendResult);
                }
                //发送失败的回调方法
                @Override
                public void onException(Throwable e) {
                    System.out.println(e);
                }
            });
        }
        System.out.println("异步发送完成");
    }
}
```

### 单向消息

**特征:** 不需要有回执的信息,例如日志类消息

```java
public class OneWayProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("group3");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();

        //单项消息
        for (int i = 0; i < 10; i++) {
            String msg = "Hello World"+i;
            Message message = new Message("Topic1", "tag1", msg.getBytes());
            //发送单项消息 没有回执消息
            producer.sendOneway(message);
        }
        System.out.println("发送完成了");
        producer.shutdown();
    }
}
```



### 延时消息

**特征:** 消息发送时并不直接发送到消息服务器,而是根据设定等待的时间到达,起到延时到达的缓冲作用

![image-20220223203009432](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220223203009432.png)

```java
public class DelayProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();

        //延时消息
        for (int i = 0; i < 10; i++) {
            String msg = "Hello World"+i;
            Message message = new Message("Topic1", "tag1", msg.getBytes());

            //设置延时 能分别设置每一条消息的延时等级 数字对应等级 而不是真正的时间
            message.setDelayTimeLevel(4);
            //发送延时消息
            SendResult sendResult = producer.send(message);
            System.out.println(sendResult);
        }
        System.out.println("发送成功了");
        //断开连接
        producer.shutdown();
    }
}
```

### 批量消息

**特征:** 一次发送多条消息,节约网络开销

原理就是通过producer可以通过send方法发送Collection(集合)的缘故 这样我们就可以将Message对象封装到一个集合中 通过send方法完成批量消息的发送

```java
public class BatchProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();

        //通过producer的send方法可以传输Collection的机制 我们只需要将消息封装到一个集合中 我们就能发送批量消息了
        ArrayList<Message> messages = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            String msg = "Hello World"+i;
            Message message = new Message("Topic1", "tag1", msg.getBytes());
            messages.add(message);
        }
        //批量发送
        SendResult send = producer.send(messages);
        System.out.println(send);
        System.out.println("批量消息发送成功");

        producer.shutdown();
    }
}

```

**注意:**

> * 这些批量消息应该有相同的topic
>
> * 相同的waitStoreMsgOK
>
> * 不能是延时消息
>
> * 消息内容的总长度不能超过4M
>
> * 消息内容总长度包含如下:
>
>   >1. topic(字符串字节数)
>   >2. body(字节数组长度)
>   >3. 消息追加的属性(key与value对应的字符串字节数)
>   >4. 日志(固定20字节)



## 消息过滤

语法过滤(属性过滤/语法过滤/SQL过滤):按照消息的某些属性过滤;

![image-20220223210153476](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220223210153476.png)

针对消费者而言在设置订阅消息的模式时, 可以设置主题(Topic) 还可以设置订阅表达式 该订阅表示就是用来过滤你要接收的消息的

---

###  Tag过滤

**`pushConsumer.subscribe("Topic1",MessageSelector.byTag("Tag1 || vip"));`**

表示只接收标签为Tag1 或者 vip的消息(默认不指定也是以Tag执行)

```java
public class Consumer {
    public static void main(String[] args) throws Exception{
        //1.谁来收
        //消费者有两种模式 一种是拉去(需要消费者自己去拉去) 一种是推送(消息主动推送给消费者)
        DefaultMQPushConsumer pushConsumer = new DefaultMQPushConsumer("group1");

        //2.从哪里收
        //与生产者一样 消费者 也许要去name Server中获得对应broker的地址去获得消息
        pushConsumer.setNamesrvAddr("localhost:9876");

        //3.监听那个消息队列
        //设置监听队列 subscribe:订阅 指定主题 和订阅表达式 "*"表示订阅主题中的所有
        pushConsumer.subscribe("Topic1","Tag1 || vip");

        //4.处理业务流程
        //注册一个监听器 去监听是否有消息被生产 一有就立刻接收
        pushConsumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                //接收到的消息就是 List<MessageExt> msgs 这时我们就能写我们的业务逻辑
                for (MessageExt msg : msgs) {
                    System.out.println(new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //启动连接
        pushConsumer.start();

        System.out.println("消费者启动起来了");

        //注意不要关闭消费者(如果还有对应主题的生产者的情况下 关闭就无法监听消息 就无法收到消息了)
    }
}
```

### SQL过滤

要是使用sql过滤首先生产者方在发送消息时需要给消息添加参数 **`message.putUserProperty("key","value");`**(因为这不是Tag过滤 并且tag也无法搭载过多的信息)

使用SQL过滤之前需要在broker.conf添加

```conf
# 开启对 propertyfilter的支持
enablePropertyFilter = true
filterSupportRetry = true
```

然后再调用**`pushConsumer.subscribe("Topic1",MessageSelector.bySql("age>18"));`**

`producer`

```java
Message message = new Message("Topic1","vip",("Hello World").getBytes());

message.putUserProperty("name","张三");
message.putUserProperty("age","18");

//4.发什么
SendResult sendResult = producer.send(message);
```

`consumer`

```java
public class Consumer {
    public static void main(String[] args) throws Exception{
        //1.谁来收
        //消费者有两种模式 一种是拉去(需要消费者自己去拉去) 一种是推送(消息主动推送给消费者)
        DefaultMQPushConsumer pushConsumer = new DefaultMQPushConsumer("group1");

        //2.从哪里收
        //与生产者一样 消费者 也许要去name Server中获得对应broker的地址去获得消息
        pushConsumer.setNamesrvAddr("localhost:9876");

        //3.监听那个消息队列
        //设置监听队列 subscribe:订阅 指定主题 和订阅表达式 "*"表示订阅主题中的所有
        pushConsumer.subscribe("Topic1", MessageSelector.bySql("age > 16"));

        //4.处理业务流程
        //注册一个监听器 去监听是否有消息被生产 一有就立刻接收
        pushConsumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                //接收到的消息就是 List<MessageExt> msgs 这时我们就能写我们的业务逻辑
                for (MessageExt msg : msgs) {
                    System.out.println(new String(msg.getBody()));
                    Map<String, String> properties = msg.getProperties();
                    Iterator<Map.Entry<String,String>> iter = properties.entrySet().iterator();
                    while(iter.hasNext()){
                        Map.Entry<String, String> next = iter.next();
                        System.out.println(next.getKey()+" = "+next.getValue());
                    }
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //启动连接
        pushConsumer.start();

        System.out.println("消费者启动起来了");

        //注意不要关闭消费者(如果还有对应主题的生产者的情况下 关闭就无法监听消息 就无法收到消息了)
    }
}
```

## Springboot整合RocketMQ

* 导入Springboot与RocketMQ整合starter

```xml
	    <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-spring-boot-starter</artifactId>
            <version>2.2.1</version>
        </dependency>
```

* 可以在application中配置rocketmq name-server的ip地址 和生产者的信息 或是消费者的信息

```yml
rocketmq:
  name-server: localhost:9876
  consumer:
    group: group1
  producer:
    group: group1
```

### Producer

* 在使用时 我们需要将springboot容器中的RocketMQTemplate(使用@Autowired)注册到我们的类中

* **RocketMQTemplate:RocketMQ模板类 : 建立连接 断开连接**

```java
    @Autowired
    private RocketMQTemplate rocketMQTemplate;//RocketMQ模板类: 建连接 短链接
```

* Springboot中传输的消息是Springboot框架提供的 **`org.springframework.messaging.Message<T>`**

* 可以使用 **`org.springframework.messaging.support.MessageBuilder`**的**静态方法withPayload(T payload)新建一个消息构建器 再调用build()方法** 就可以将**payload转换为一个Message对象**
* 上述都是使用send方法发送信息的需要做的 我们可以使用rocketMQTemplate的其他方法 例如 **`converAndSend()`**该方法由名字就知道它可以转化并且发送 它可以将java对象转化为**`org.springframework.messaging.Message<T>`**发送
* 除了上述两种方法 还有**`syncSend(), asyncSend(), sendOneWay() `**分别对应着同步,异步,单向消息 还可以在方法的参数上添加**`timeout delayLevel`**等参数以达到延时效果

**注意:** 这里不再是单纯的填入Topic了而是**destination** 并且格式是 **`topicName:tags`** 

RocketMQ获取destination的源码

![image-20220224202331947](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220224202331947.png)

### Consumer

* Consumer方面我们使用了监听器的方式来接收消息 实现RocketMQ自带的**`RocketMQListener<T>`** T指的**`withpayload`**中的消息类型

  ```java
  @Service //注册到容器中
  public class DemoConsumer implements RocketMQListener<User> {
      /**
       * 接收成功的回调方法
       * @param message
       */
      @Override
      public void onMessage(User message) {
          System.out.println(message);
      }
  
  }
  ```

  * 设置了接收的监听器 我们还要设置监听的消息的主题 消息过滤 还有消费者组的名称 才能满足RocketMQ的规范

    * 这里我们使用**`rocketmq-spring-boot-starter`**的注解**`@RocketMQMessageListener`**设置参数 因为name-server在application.yml中我们已经设置了 springboot会自动识别并且设置.

  * @RocketMQMessageListener

    > * topic:主题
    > * selectorExpression:过滤表达式
    > * selectorType:设置过滤类型(Tag or Sql)
    > * consumerGroup: 消费者组的名称
    > * messageModel:消息的模式(广播或是集群)

**`Producer`**

```java
@RestController
@RequestMapping("/demo")
public class SendController {
    @Autowired
    private RocketMQTemplate rocketMQTemplate;//RocketMQ模板类: 建连接 短链接
    @GetMapping("/send")
    public String send(){
        User user = new User("Devil", 10);
        rocketMQTemplate.convertAndSend("Topic2",user);//convert: 消息转换为字节数组 甚至可以自动将对象转化为字节数组 但必须实现序列化

        rocketMQTemplate.syncSend("Topic2",user);//发送同步消息

        //发送异步消息
        rocketMQTemplate.asyncSend("Topic2", user, new SendCallback() {
            //发送成功的回调方法
            @Override
            public void onSuccess(SendResult sendResult) {

            }
            //发送失败的回调方法
            @Override
            public void onException(Throwable e) {

            }
        });

        //发送单项消息
        rocketMQTemplate.sendOneWay("Topic2",user);

        //发送延时消息
        rocketMQTemplate.syncSend("Topic2:tag1", MessageBuilder.withPayload(user).build(),10,3);

        return "success";
    }
}

```

**`Consumer`**

```java
@Service
@RocketMQMessageListener(topic = "Topic2",selectorExpression = "tag1 || tag2",consumerGroup = "${rocketmq.producer.group}",
        selectorType = SelectorType.TAG,messageModel = MessageModel.BROADCASTING)
public class DemoConsumer implements RocketMQListener<User> {
    /**
     * 接收成功的回调方法
     * @param message
     */
    @Override
    public void onMessage(User message) {
        System.out.println(message);
    }

}
```

---

## 消息顺序

 **消息错乱的原因:**

> 默认消息的发送是每条消息按照 依次按照queue的顺序进入queue 即:**队列内无序,队列外有序**
>
> ![image-20220224214903923](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220224214903923.png)
>
> **`Producer`**
>
> ```java
>         //这样发送会导致消息错乱
>         for (OrderStep orderStep : orderSteps) {
>             Message message = new Message("topic3", "tag1", orderStep.toString().getBytes());
>             SendResult send = producer.send(message);
> 
>             System.out.println(send);
>         }
> ```
>
> **`Consumer`**
>
> ```java
>         //这样接收会导致消息错乱
>         pushConsumer.registerMessageListener(new MessageListenerConcurrently() {
>             @Override
>             public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
>                 //接收到的消息就是 List<MessageExt> msgs 这时我们就能写我们的业务逻辑
>                 for (MessageExt msg : msgs) {
>                     System.out.println(new String(msg.getBody()));
>                 }
>                 return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
>             }
>         });
> ```
>
> 

**纠正消息错乱:**

> 修改消息的顺序,即指定消息进入的队列, 完整的顺序(订单的完整流程 创建 支付 完成)应当进入同一个消息队列. 即:**队列内有序,队列外无序**
>
> ![image-20220224214826242](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220224214826242.png)
>
> 为了使得生产的消息有序可以在producer中发送消息时指定消息进入的消息队列
>
> **`producer.send(message, new MessageQueueSelector() {...},null);`** 其中的**`MessageQueueSelector()`**接口的select方法就可以指定消息填充的队列的队列id 更具这个id就可以获得这个队列 再通过send方法 发送到这个队列中
>
> **`Producer`**
>
> ```java
>         //正确的发送
>         for (OrderStep orderStep : orderSteps) {
>             Message message = new Message("topic3", "tag1", orderStep.toString().getBytes());
>             SendResult send = producer.send(message, new MessageQueueSelector() {
>                 //这个方法就是队列悬着的方法
>                 @Override
>                 public MessageQueue select(List<MessageQueue> mqs/*消息队里额*/, Message msg, Object arg) {
>                     //队列数
>                     int size = mqs.size();
>                     //确定的orderId对应确定的队列 取模运算
>                     int orderId = (int) (orderStep.getOrderId());
>                     int queueId = orderId % size;
> 
>                     //根据 计算出的queueId 从List<MessageQueue> mqs中获取消息队列
>                     return mqs.get(queueId);
>                 }
>             }, null);
>             System.out.println(send);
>         }
> ```
>
> 对于Consumer需要注册**顺序的监听器** 作用就是一个线程只监听一个MessageQueue 这样就可以接收一个queue中的消息了
>
> 而一个queue中都是producer生产的顺序的消息.
>
> >  **`new MessageListenerOrderly(){...}`**
>
> **`Consumer`**
>
> ```java
>         //消费者注册一个顺序的监听器 作用就是一个线程只监听一个MessageQueue
>         pushConsumer.registerMessageListener(new MessageListenerOrderly() {
>             @Override
>             public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
>                 for (MessageExt msg : msgs) {
>                     System.out.println(new String(msg.getBody()));
>                 }
>                 System.out.println(context.getMessageQueue().getQueueId());
>                 return ConsumeOrderlyStatus.SUCCESS;
>             }
>         });
> ```
>
> 

---

## 事务消息

### RocketMQ事务流程概要

RocketMQ实现事务主要分为两个阶段: 正常事务的发送及提交、事务信息的补偿流程(都是针对生产者 因为事务只出现在DataBase中 有些情况需要将消息存储在数据库中 如果发生事务问题....)

**整体流程为:**

> * 正常事务发送与提交阶段
>   1. 生产者发送一个半消息给broker(半消息是指的暂时不能消费的消息)
>   2. 服务端响应消息写入结果,半消息发送成功
>   3. 开始执行本地事务
>   4. 根据本地事务的执行情况执行Commit或者Rollback
> * 事务信息的补偿流程
>   1. 如果broker长时间没有收到本地事务的执行状态,会向生产者发起一个确认会查的操作请求
>   2. 生产者收到确认会查请求后,检查本地事务的执行状态
>   3. 根据检查后的结果执行Commit或者Rollback操作 补偿阶段主要是用于解决生产者在发送Commit或者Rollbacke操作时发生超时或失败的情况
>
> ![img](https://pic3.zhimg.com/80/v2-325e5949a667b144f2684caac49dd41a_720w.png)

### RocketMQ事务流程关键

* **事务消息在一阶段对用户不可见** 

  事务消息相对普通消息最大的特点就是一阶段发送的消息对用户是不可见的,也就是说消费者不能直接消费.这里RocketMQ实现方法是原消息的主题与消息消费队列,然后把主题改成**`RMQ_SYS_TRANS_HALF_TOPIC`**.这样由于消费者没有订阅这个主题,所以不会消费.

* **如何处理第二阶段的发送消息?**

  在本地事务执行完成后回向Broker发送Commit或者Rollback操作,此时如果在发送消息的时候生产者出故障了,要保证这条消息最终被消费,broker就会向服务端发送回查请求,确认本地事务的执行状态.当然RocketMQ并不会无休止的发送事务状态回查请求,**默认是15次**,如果15次回查还是无法得知事务的状态,RocketMQ默认回滚消息(broker就会将这条半消息删除)

* 事务的三种状态:

  > - **TransactionStatus.CommitTransaction**：提交事务消息，消费者可以消费此消息
  > - **TransactionStatus.RollbackTransaction**：回滚事务，它代表该消息将被删除，不允许被消费。
  > - **TransactionStatus.Unknown** ：中间状态，它代表需要检查消息队列来确定状态。

### 使用

创建生产者时我们不在简单地创建**`DefaultMQProducer`** 而是RocketMQ事务专属的 **`TransactionMQProducer`** 并且不再简单地发送消息了 而是设置一个事务监听器 **`setTransactionListener(new TransactionListener(){...});`** 实现接口方法 并且由于监听器需要等待本地事务的执行情况我们不能再生产者发送完消息后关闭

**`Producer`**

```java
public class TransProducer {
    public static void main(String[] args) throws Exception {
        TransactionMQProducer producer = new TransactionMQProducer("group1");
        producer.setNamesrvAddr("localhost:9876");

        //设置事务监听
        producer.setTransactionListener(new TransactionListener() {
            //执行本地事务 这就是正常事务过程
            @Override
            public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
                //消息保存到数据库中
                //sql代码
                //根据数据库事务状态 返回事务状态
                System.out.println("正常执行的过程");

                //LocalTransactionState.ROLLBACK_MESSAGE 表示事务回滚 这时broker就会删除掉half消息 消费者接收不到
                //如果是LocalTransactionState.COMMIT_MESSAGE 表示提交消息 这时broker就会提交half消息 消费能接收
                //LocalTransactionState.UNKNOW 事务结果未知 执行事务补偿过程 即broker主动询问生产者事务结果
                return LocalTransactionState.UNKNOW;
            }
            //检查本地事务 这就是事务补偿过程
            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt msg) {
                System.out.println("执行事务补偿过程");
                return LocalTransactionState.UNKNOW;
            }
        });

        producer.start();

        String msg = "Hello Transaction";
        Message message = new Message("topic4", "tag1", msg.getBytes());
        SendResult send = producer.sendMessageInTransaction(message,null);
        System.out.println(send);

        System.out.println("消息生产完毕");

        //不能关闭 涉及事务的提交和回滚 以及事务与broker的交互过程 不能一发出消息就关闭
        //producer.shutdown();



    }
}
```

**`Consumer`** 整个事务消息环节与Consumer相关不大,所以不用对原来的Consumer进行修改 正常接收消息即可.

## 集群搭建

 ### 集群分类

* **单机**
  * 一个broker提供服务(宕机后服务瘫痪)
* **集群**
  * 多个broker提供服务(单机宕机后消息无法及时被消费)
  * 多个master和多个slave
    * master到slave消息同步方式为同步(较异步方式性能略低,消息无延迟)
    * master到slave消息同步方式为异步(较同步方式性能略高,数据略有延迟)
* **根据配置文件中的信息来设置主从集群**

### RocketMQ集群工作流程

* NameServer启动,开启监听,等待broker,producer与consumer连接
* broker启动,根据配置信息,连接所有的NameServer,并保持长连接 
  * 如果broker中现存数据,NameServer将保存topic与broker关系
* producer发送信息,连接某个NameServer,并建立长连接
* producer发送消息
  * 如果topic存在,由NameServer直接分配
  * 如果topic不存在,由NameServer创建topic与broker关系,并分配
* producer与broker的topic选择一个消息队列(从列表中选择)
* producer与broker建立长连接,用于发送消息
* producer发送消息

**`Consumer`**工作流程同**`Producer`**

---

## RocketMQ高级特性

**RocketMQ消息发送底层**

1. 消息的生产者发送消息到MQ

2. MQ返回ACK给生产者

3. MQ push消息给对于的消费者

4. 消息消费者返回ACK给MQ

说明: ACK(Acknowledge character)

注意: **如果broker出现问题不能发送和接收ACK  生产者就会接收不到broker发送的ACK 就会导致生产者一直发送同一条消息 也会导致消费者一直消费同一条消息**

![image-20220225203843889](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225203843889.png)

### 消息的存储

1. 消息生产者发送消息到MQ

2. MQ接收到消息,将消息持久化,存储该消息

3. MQ返回ACK给生产者

4. MQpush消息给对应的消费者

5. 消息消费者返回ACK给MQ
6. MQ删除消息

![image-20220225204155340](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225204155340.png)

**注意:**

> * 第5步 MQ在指定时间接收到消息消费者返回ACK, MQ认定消息消费成功,执行6
> * 第5步 MQ在指定时间未接收到消息消费者返回ACK,MQ认定消费失败,重新执行456

### 消息的存储介质

为了防止数据库出现故障和数据库I/O降低性能(数据库最后也是将数据存储再磁盘上(文件系统))

**所以我们直接绕过数据库 直接将消息存在本地的文件系统上**

![image-20220225204909019](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225204909019.png)

**数据库:**

* ActiveMQ使用
* 缺点: 数据库瓶颈将成为MQ瓶颈

**文件系统:**

* RocketMQ/Kafka/RabbitMQ
* 解决方案: 采用消息刷盘的机制进行数据的存储
* 缺点:硬盘损坏的问题无法避免

### 高效的消息存储与读写方式

* **SSD(Solid State Disk): 固态硬盘**
  * 随机写 100kb/s
  * 顺序写 600-3000m/s 
* 由上可知 顺序写的速度是远远快于随机写的

---

* **RocketMQ中向文件系统预先申请了一定大小的磁盘空间 用于顺序读写**(这就是RocketMQ高速读写的第一个原因)

* Linux系统发送数据的方式

  >"零拷贝"技术
  >
  >* 数据传输由传统的4次复制简化成3次复制,减少1次复制过程
  >* java语言中使用MappedByteBuffer类实现了该技术
  >* 要求:预留存储空间,用于保存数据(1G存储空间起步)

  传统模式

  ![image-20220225211130484](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225211130484.png)

"零拷贝模式"

![image-20220225211155810](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225211155810.png)

**总结(RocketMQ高速读写的原因):**

> * 磁盘读写方式
> * "零拷贝"技术

### 消息存储的结构

MQ数据存储区域包括如下内容

* 消息数据存储区域
  * topic
  * queueId
  * message
* 消费逻辑队列(会记录每一个队列被每一个消费者消费到了什么(多少偏移量))
  * minOffset
  * maxOffset
  * consumerOffset

* 索引
  * key索引
  * 创建时间索引
  * ......

![image-20220225212158831](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225212158831.png)

### 刷盘机制

  #### 同步刷盘:

1. 生产者发送消息到MQ,MQ接到消息数据
2. MQ挂起生产者发送消息的线程
3. MQ将消息数据写入内存
4. 内存数据写入硬盘
5. 磁盘存储后返回SUCCESS
6. MQ回复挂起的生产者线程
7. 发送ACK到生产者

![image-20220225212521664](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225212521664.png)

#### 异步刷盘

1. 生产者发送消息到MQ,MQ接收到消息数据

3. MQ将消息写入内存

7. 发送ACK到生产者

4. 待到内存中的消息数据积累到一定量 就将消息数据写入硬盘

#### 总结:

> * 同步刷盘: 安全性高,效率低,速度慢(适用于对数据安全性要求较高的业务)
> * 异步刷盘:安全性低,效率高,速度块(使用与对数据处理速度要求较高的业务)

![image-20220225213018812](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225213018812.png)

---

### 高可用性

* NameServer
  * 无状态(相互之间无联系)+全服务器注册
* 消息服务器
  * 主从框架(2M-2S)
* 消息生产
  * 生产者将相同的topic绑定到多个group组,保障master挂掉后,其他master仍可以正常进行消息接收
* 消息消费
  * RocketMQ自身会根据master的压力确认是否由master承担消息读取的功能,当master繁忙的时候,自动切换slave成单数据读取的工作(主从分离 当压力过大时 master只写入 因为slave中的数据与master实时更新 所以这时slave可以承担读的功能)

**主从数据复制**

![image-20220225213710132](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225213710132.png)

---

### 负载均衡

* **`Producer`**负载均衡

> * 内部实现了不同broker集群中对同一个topic对应消息队列的负载均衡
>
> ![image-20220225214232482](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225214232482.png)

* **`Consumer`**负载均衡(针对相同的消费者组间)

> * 平均分配
>
> ![image-20220225214249237](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225214249237.png)
>
> * 循环平均分配(解决宕机问题)
>
> ![image-20220225214305341](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225214305341.png)

---

### 消息重试

当消息消费后未正常返回消费成功的消息将启动消息重试机制

**消息重试机制**

> * **顺序消息重试**
>
>   > 当消费者消费失败后,RocketMQ会自动进行消息重试(每次间隔为1s)
>   >
>   > **注意:** 应用会出现消息消费被堵塞的情况,因此要对顺序消息的消费情况进行监控,避免阻塞的现象发生
>   >
>   > ![image-20220225214842367](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225214842367.png)
>
> * **无序消息重试**
>
>   > * 无序消息包括普通消息、定时消息、延时消息、事务消息
>   > * 无序消息重试仅适用于负载均衡（集群）模型下的消息消费，不适用于广播模式下的消息
>   > * 消费为保障无序消息的消费，MQ设定了合理的消息重试间隔时长
>   >
>   > ![image-20220225215028265](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225215028265.png)

---

### 死信队列

**死信队列就是那些重试无果的消息存在的队列**

* **死信队列特征**

  > * 归属某一个组（Gourp Id)，而不归属Topic，也不归属消费者。
  > * 一个死信队列中可以包含同一个组下的多个Topic中的死信消息
  > * 死信队列不会进行默认初始化，当第一个死信出现后，此队列首次初始化

* **死信队列中消息特征**

  >* 不会被再次重复消费
  >
  >* 死信队列中的消息有效期为3天，达到时限后将被清除

* **死信处理**

  > 在监控平台中,通过查找死信,获取死信的messageId,然后通过id对死信进行精准消费

#### 总结:

* 死信

  > * 死信队列与死信
  >
  > * 死信处理方式

---

### 消息重复消费

#### 消息重复发送的原因

* 生产者发送了重复的消息
  * 网络闪断(例如: 消息服务器没有发送ACK给生产者)
  * 生产者宕机
* 消息服务器投递了重复的消息
  * 网络闪断(例如: 消费者没有发送ACK给消息服务器)
* 动态的负载均衡过程
  * 网络闪断/抖动
  * broker重启
  * 订阅方应用重启(消费者)
  * 客户端扩容
  * 客户端缩容

![image-20220225220215976](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225220215976.png)

#### 消息幂等

* 对于同一条消息,无论消费了多少次,结果保持一致,称为**消息幂等性**
* 解决方案
  * 使用业务id作为消息的key
  * 在消费消息时,客户端对key做判定,未使用过放行,使用过抛弃
* 注意: **messageId由RocketMQ产生,MessageId并不具有唯一性,不能作用幂等判定条件**

* **常见的幂等方法示例**

  ![image-20220225220811978](https://gitee.com/Devildyw/blogimage/raw/master/img/image-20220225220811978.png)













# ------End------
