## 安装RabbitMQ
#### 下载rabbitmq镜像
`docker pull rabbitmq:3.7.7-management`

#### 运行
```
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 \
-v $PWD/rabbitmq/data:/var/lib/rabbitmq \
-e RABBITMQ_DEFAULT_USER=youpik \
-e RABBITMQ_DEFAULT_PASS=qaz123wsx rabbitmq:3.7.7-management
```
> -d 后台运行容器；<br/>
--name 指定容器名；<br/>
-p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）； <br/>
-v 映射目录或文件；<br/>
--hostname  主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；<br/>
-e 指定环境变量；（RABBITMQ_DEFAULT_VHOST：默认虚拟机名；RABBITMQ_DEFAULT_USER：默认的用户名；RABBITMQ_DEFAULT_PASS：默认用户名的密码）

> 访问地址 http://ip:15672 <br/>
用户名/密码：**admin/admin**



### SpringBoot整合RabbitMQ

> + 「中华石杉」https://doocs.gitee.io/advanced-java/#/./docs/high-concurrency/mq-interview
>
> + 「一灰灰Blog」https://spring.hhui.top/spring-blog/categories/SpringBoot/MQ%E7%B3%BB%E5%88%97/RabbitMq/

#### pom 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### 配置 application.yml

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: youpik
    password: qaz123wsx
    virtual-host: /
```

#### 定义MQ相关常量

```java
public interface MQConstant {

    /** ------------------- EXCHANGE ---------------------- **/

    String EXCHANGE_MIGRATION_MALL = "migration_mall.direct";


    /** ------------------- QUEUE ---------------------- **/

    String QUEUE_MIGRATION_ORDER = "migration.order.queue";
    String QUEUE_MIGRATION_ORDER_ITEM = "migration.order_item.queue";


    /** ------------------- ROUTING_KEY ---------------------- **/

    String ROUTER_MIGRATION_ORDER = "migration.order.key";
    String ROUTER_MIGRATION_ORDER_ITEM = "migration.order_item.key";
}
```

#### 配置MqConfig

```java
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MqConfig {

  	/**
     * 定义一个TOPIC
     */
    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange(MQConstant.EXCHANGE_MIGRATION_MALL);
    }

  	/**
     * 定义多个消息队列
     */
    @Bean("orderQueue")
    public Queue orderQueue() {
        return new Queue(MQConstant.QUEUE_MIGRATION_ORDER, true);
    }
    @Bean("orderItemQueue")
    public Queue orderItemQueue() {
        return new Queue(MQConstant.QUEUE_MIGRATION_ORDER_ITEM, true);
    }

  	/**
     * 绑定 exchange > queue > routingKey
     */
    @Bean
    public Binding bindingOrder(TopicExchange topicExchange, @Qualifier("orderQueue") Queue queue) {
        return BindingBuilder.bind(queue).to(topicExchange).with(MQConstant.ROUTER_MIGRATION_ORDER);
    }
    @Bean
    public Binding bindingOrderItem(TopicExchange topicExchange, @Qualifier("orderItemQueue") Queue queue) {
        return BindingBuilder.bind(queue).to(topicExchange).with(MQConstant.ROUTER_MIGRATION_ORDER_ITEM);
    }

  	/**
     * 配置RabbitTemplate实例 && 消息体的Json数据转换
     */
    @Bean
    public RabbitTemplate jsonRabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());
        return rabbitTemplate;
    }
}
```

#### 消息生产者

```java
import org.springframework.amqp.AmqpException;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageDeliveryMode;
import org.springframework.amqp.core.MessagePostProcessor;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class RabbitMqSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /*
     * 发送持久化消息的方法
     */
    public void sendMsg(String exchange, String routingKey, Object msg){
        /**
         * convertAndSend - 转换并发送消息的template方法。
         * 是将传入的普通java对象，转换为rabbitmq中需要的message类型对象，并发送消息到rabbitmq中。
         * 参数一：交换器名称。 类型是String
         * 参数二：路由键。 类型是String
         * 参数三：消息，是要发送的消息内容对象。类型是Object
         */
        this.rabbitTemplate.convertAndSend(exchange, routingKey, msg);
    }


    /**
     * 推送一个非持久化的消息，这个消息推送到持久化的队列时，mq重启，这个消息会丢失；上面的持久化消息不会丢失
     *
     * @param msg
     * @return
     */
    public void sendTempMsg(String exchange, String routingKey, Object msg) {
        rabbitTemplate.convertAndSend(exchange, routingKey, msg, new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                message.getMessageProperties().setHeader("ta", "测试");
                message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.NON_PERSISTENT);
                return message;
            }
        });
    }
}
```

> 发送消息
>
> ```java
> rabbitMqSender.sendMsg(MQConstant.EXCHANGE_MIGRATION_MALL, MQConstant.ROUTER_MIGRATION_ORDER, msg);
> ```

#### 消息消费者

```java
import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Service;

import java.io.IOException;

@Slf4j
@Service
public class OrderMqConsumer {

    /**
     * durable 是否持久化
     * autoDelete 没有消费者时是否自动删除queue （一般 durable 和 autoDelete 设置互斥）
     * ackMode (noack, auto, manual) 设置消息的确认机制
     * concurrency 并发消费，设置消费者的个数 （还有一种 m-n 的格式，表示m个并行消费者，最多可以有n个）
     */
    @RabbitListener(
        bindings = @QueueBinding(
            value = @Queue(value = MQConstant.QUEUE_MIGRATION_ORDER, durable = "true", autoDelete = "false"),
            exchange = @Exchange(value = MQConstant.EXCHANGE_MIGRATION_MALL, ignoreDeclarationExceptions = "true", type = ExchangeTypes.TOPIC),
            key = MQConstant.ROUTER_MIGRATION_ORDER
        ), ackMode = "MANUAL", concurrency = "2"
    )
    public void consumerOrder(String data, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag, Channel channel) throws IOException {
        try {
          	//TODO 具体消费消息的业务逻辑
            log.info(">>> consumerOrder >>> data={}", data)

            // RabbitMQ的ack机制中，第二个参数返回true，表示需要将这条消息投递给其他的消费者重新消费
            channel.basicAck(deliveryTag, false);
        } catch (Exception e) {
            log.error(">>>>>>>> consumerOrder error >>", e);
            // 第三个参数true，表示这个消息会重新进入队列
            channel.basicNack(deliveryTag, false, true);
        }
    }
}
```



