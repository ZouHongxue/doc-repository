# Linux 部署运维

## 1.安装erlang

## 2.安装socat

```bash
$ yum install socat
```

## 3.安装rabbit MQ

```shell
$ rpm -ivh rabbitmq-server-3.7.5-1.el7.noarch.rpm
```

## 4.查看rabbitmq状态

```shell
$ systemctl status rabbitmq-server.service
$ rabbitmqctl status //当前状态
$ rabbitmqctl cluster_status //集群状态
```

## 5.rabbitmq启动

```shell
$ systemctl start rabbitmq-server.service
```

## 6.rabbitmq的管理界面

```shell
$ rabbitmq-plugins list //列出所有插件
$ rabbitmq-plugins enable rabbitmq_management //启动后在http://server-name:15672访问
$ rabbitmq-plugins disable rabbitmq_management //关闭rabbitmq管理插件
$ rabbitmq-plugins enable rabbitmq_tracing //启用trace插件
$ rabbitmqctl trace_on //打开trace的开关
$ rabbitmqctl trace_on -p vhost
$ rabbitmqctl trace_off //关闭trace的开关
$ rabbitmq-plugins disable rabbitmq_tracing //关闭trace插件
$ rabbitmq-plugins enable rabbitmq_delayed_message_exchange //安装延时队列
```

## 7.rabbitmq停止

```shell
$ systemctl stop rabbitmq-server.service
```

## 8.rabbitmq关闭

```shell
$ rabbitmqctl shutdown
```

## 9.rabbitmq重启

```shell
$ systemctl restart rabbitmq-server.service
$ rabbitmqctl reset
```

## 10.配置文件位于/etc/rabbitmq

```shell
$ rabbitmqctl environment //查看运行参数
```

## 11.用户管理

```shell
$ rabbitmqctl list_users //列出所有用户
$ rabbitmqctl list_user_permissions user //查看用户权限
$ 例子:rabbitmqctl list_user_permissions hyp
$ rabbitmqctl add_user user password //添加用户
$ 例子:rabbitmqctl add_user hyp hyp
$ rabbitmqctl delete_user user //删除用户
$ 例子:rabbitmqctl delete_user hyp
$ rabbitmqctl set_user_tags user <administrator | monitoring | management | policymaker> //授予角色
例子:$ rabbitmqctl set_user_tags hyp administrator
administrator角色,可查看所有信息,并且可以对用户,策略(policy)进行操作.
monitoring角色,可查看rabbitmq节点的相关信息(进程数,内存使用情况,磁盘使用情况等)
policymaker角色,可以对策略(policy)进行管理,但无法查看节点的相关信息.
management角色,只能查看队列和交换机等,无法看到节点信息,也无法对策略进行管理.
rabbitmqctl change_password user newpassword //修改密码
例子:$ rabbitmqctl change_password hyp hyppassword
```

## 12.集群

```shell
$ rabbitmqctl stop_app
$ rabbitmqctl reset
$ rabbitmqctl join_cluster --ram rabbit@v01-app-rabbitmq01 //加入集群,该节点为内存节点类型
$ rabbitmqctl join_cluster --disc rabbit@v01-app-rabbitmq01 //加入集群,该节点为磁盘节点类型
$ rabbitmqctl change_cluster_node_type <ram | disc> //修改节点类型
$ rabbitmqctl start_app
/*到目前为止,集群虽然搭建成功,但只是默认的普通集群,exchange,binding等数据可以复制到集群各节点
但对于队列来说,各节点只有相同的元数据,即队列结构,但队列实体只存在于创建改队列的节点,即队列内容不会复制
(从其余节点读取，可以建立临时的通信传输)
$ rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}' //将所有队列设置为镜像队列,即队列会被复制到各个节点,各个节点状态保持一直
$ rabbitmqctl list_policies //查看策略
```

# Client Demo-HelloWorld （JAVA）

## 1.Create Maven Project And Import Rabbit MQ amqp-client

```xml
<!-- https://mvnrepository.com/artifact/com.rabbitmq/amqp-client -->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.4.1</version>
</dependency>
```

## 2.Create Send Class

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class Send {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("IP or Host");
        connectionFactory.setUsername("your UserName");
        connectionFactory.setPassword("your Password");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        String message = "Hello world!!";
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
        System.out.println(" [x] Sent '" + message + "'");

        channel.close();
        connection.close();
    }
}
```

## 3.Create Receive Class

```java
import com.rabbitmq.client.*;
import java.nio.charset.StandardCharsets;

public class Received {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("IP or Host");
        connectionFactory.setUsername("your UserName");
        connectionFactory.setPassword("your Password");
        Connection connection = factory.newConnection();
        Channel channel = connectionFactory.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                String message = new String(body, StandardCharsets.UTF_8);
                System.out.println(" [x] Received '" + message + "'");
            }
        };
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

# Client Demo-HelloWorld（SpringBoot）

## 1.Use Spring Initializr And Add RabbitMQ Dependency 

## 2.配置 application.yml

```yaml
spring:
  rabbitmq:
    host: xxxx
    port: 5672
    username: xxxx
    password: xxxx
    virtual-host: /
```

## 3.编写启动类

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class RabbitmqTutorialsApplication {

    public static void main(String[] args) {
        SpringApplication.run(RabbitmqTutorialsApplication.class, args);
    }
}
```

## 4.编写Sender

```java
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class TutorialSender {

    @Autowired
    private RabbitTemplate template;

    @Autowired
    private Queue queue;

    @Scheduled(fixedDelay = 1000, initialDelay = 500)
    public void send() {
        String message = "Hello World!";
        this.template.convertAndSend(queue.getName(), message);
        System.out.println(" [x] Sent '" + message + "'");
    }
}
```

## 5.编写Receiver

```java
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;

@RabbitListener(queues = "hello")
public class TutorialReceiver {

    @RabbitHandler
    public void receive(String msg) {
        System.out.println(" [x] Received '" + msg + "'");
    }
}
```

## 6.编写Amqp配置类

```java
import com.rrc.rabbitmqtutorials.Service.TutorialReceiver;
import com.rrc.rabbitmqtutorials.Service.TutorialSender;
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AmqpConfig {

    private static final String QUEUE_NAME = "hello";
    @Bean
    public Queue hello() {
        return new Queue(QUEUE_NAME);
    }

    @Bean
    public TutorialReceiver receiver() {
        return new TutorialReceiver();
    }

    @Bean
    public TutorialSender sender() {
        return new TutorialSender();
    }
}
```

# Exchanges

> The core idea in the messaging model in RabbitMQ is that the producer never sends any messages directly to a queue. Actually, quite often the producer doesn't even know if a message will be delivered to any queue at all. 

在RabbitMQ的消息模型中最核心的想法是生产者绝不直接将消息发送到队列。实际上生产者往往不知道消息会交给哪个队列。

> Instead, the producer can only send messages to an *exchange*. An exchange is a very simple thing. On one side it receives messages from producers and the other side it pushes them to queues. The exchange must know exactly what to do with a message it receives. Should it be appended to a particular queue? Should it be appended to many queues? Or should it get discarded. The rules for that are defined by the *exchange type*. 

因此，生产者只能将消息交给一个交换机，这个交换机是一个非常简单的东西，它一边接受生产者的消息另一边将消息推给队列。这个交换机必须清楚的知道对于接收到的消息如何处理。是该交给一个特定的队列，还是发送给很多队列，还是应该丢弃掉这个消息。这些规则通过交换机的类型进行定义。

## 交换机类型

direct

topic

headers

fanout



# Notice

## 报错 connection Refuse

确认用户名密码，账号访问权限问题。可以在网页端设置host:15672/#users设置。

## 报错reply-code=406, reply-text=PRECONDITION_FAILED 

Exchange或Queue使用不同参数重复创建同名的会出错。换个名试下。





