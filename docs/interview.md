### 基础类
1. 什么是rabbitmq？
   
    RabbitMQ是一款开源的，Erlang编写的，基于AMQP协议的一种消息队列技术，消息中间件；
    
    可以用它来：解耦、异步、削峰。
    
     
    
2. 为什么要使用rabbitmq？
    1. 在分布式系统下具备异步,削峰,负载均衡等一系列高级功能;
    2. 拥有持久化的机制，进程消息，队列中的信息也可以保存下来。
    3. 实现消费者和生产者之间的解耦。
    4. 对于高并发场景下，利用消息队列可以使得同步访问变为串行访问达到一定量的限流，利于数据库的操作。
    5. 可以使用消息队列达到异步下单的效果，排队中，后台进行逻辑下单。

3. 使用rabbitmq的场景？
    - 服务间异步通信

    - 顺序消费

    - 定时任务

    - 请求削峰

4. 使用RabbitMQ有什么好处？
        服务间高度解耦，
        异步通信性能高，
        流量削峰

5. 使用mq的优缺点？

    - 优点：解耦、异步、削峰；
    - 降低了系统的**稳定性**：加入个消息队列，如果消息队列挂了，系统崩溃，会导致系统不稳定，可用性会降低；
    - 增加了系统的**复杂性**：
        增加了系统的复杂性：加入了消息队列，要多考虑很多方面的问题，比如：一致性问题、如何保证消息不被重复消费、如何保证消息可靠性传输等。因此，需要考虑的东西更多，复杂性增大。
    - 一致性问题：
        A系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是，要是BCD三个系统那里，BD两个系统写库成功了，结果C系统写库失败了，咋整？你这数据就不一致了。
        所以消息队列实际是一种非常复杂的架构，你引入它有很多好处，但是也得针对它带来的坏处做各种额外的技术方案和架构来规避掉，最好之后，你会发现，妈呀，系统复杂度提升了一个数量级，也许是复杂了10倍。但是关键时刻，用，还是得用的。。。

6. 如何确保消息正确地发送至RabbitMQ？ 如何确保消息接收方消费了消息？

    - 发送方确认模式：
        将信道设置成confirm模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的ID。
        一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一ID）。
        如果RabbitMQ发生内部错误从而导致消息丢失，会发送一条nack（not acknowledged，未确认）消息。
        发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。
    - 接收方确认机制
        接收方消息确认机制：消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ才能安全地把消息从队列中删除。
        这里并没有用到超时机制，RabbitMQ仅通过Consumer的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ给了Consumer足够长的时间来处理消息。保证数据的最终一致性；
    - 下面罗列几种特殊情况：
        如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）
        如果消费者接收到消息却没有确认消息，连接也未断开，则RabbitMQ认为该消费者繁忙，将不会给该消费者分发更多的消息。

7. 如何避免消息重复投递或重复消费？
        在消息生产时，MQ内部针对每条生产者发送的消息生成一个inner-msg-id，作为去重的依据（消息投递失败并重传），避免重复的消息进入队列；
        在消息消费时，要求消息体中必须要有一个bizId（对于同一业务全局唯一，如支付ID、订单ID、帖子ID等）作为去重的依据，避免同一条消息被重复消费。

8. 消息基于什么传输？
    由于TCP连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ使用信道的方式来传输数据。信道是建立在真实的TCP连接内的虚拟连接，且每条TCP连接上的信道数量没有限制。

9. 消息如何分发？
     若该队列至少有一个消费者订阅，消息将以循环（round-robin）的方式发送给消费者。每条消息只会分发给一个订阅的消费者（前提是消费者能够正常处理消息并进行确认）。
       通过路由可实现多消费的功能

10. 消息怎么路由？
     消息提供方->路由->一至多个队列
       消息发布到交换器时，消息将拥有一个路由键（routing key），在消息创建时设定。
       通过队列路由键，可以把队列绑定到交换器上。
       消息到达交换器后，RabbitMQ会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）；
       常用的交换器主要分为一下三种：
       fanout：如果交换器收到消息，将会广播到所有绑定的队列上
       direct：如果路由键完全匹配，消息就被投递到相应的队列
       topic：可以使来自不同源头的消息能够到达同一个队列。 使用topic交换器时，可以使用通配符

11. 如何确保消息不丢失？
      消息持久化，当然前提是队列必须持久化
      RabbitMQ确保持久性消息能从服务器重启中恢复的方式是，将它们写入磁盘上的一个持久化日志文件，当发布一条持久性消息到持久交换器上时，Rabbit会在消息提交到日志文件后才发送响应。
      一旦消费者从持久队列中消费了一条持久化消息，RabbitMQ会在持久化日志中把这条消息标记为等待垃圾收集。如果持久化消息在被消费之前RabbitMQ重启，那么Rabbit会自动重建交换器和队列（以及绑定），并重新发布持久化日志文件中的消息到合适的队列。

12. 如何保证rabbitmq高可用

      1. 普通集群模式
         
          ![img](Interview.assets/36cafe76057683333bd6cf6a427836270c8.jpg)



在多台机器上启动多个rabbitmq实例，每个机器启动一个。但是创建的queue，只会放在一个rabbtimq实例上，但是每个实例都同步queue的元数据。消费的时候，实际上如果连接到了另外一个实例，那么那个实例会从queue所在实例上拉取数据过来。
这种方式确实很麻烦，也不怎么好，没做到所谓的分布式，就是个普通集群。        

因为这导致要么消费者每次随机连接一个实例然后拉取数据，要么固定连接那个queue所在实例消费数据，前者有数据拉取的开销，后者导致单实例性能瓶颈。
而且如果那个放queue的实例宕机了，会导致接下来其他实例就无法从那个实例拉取，如果开启了消息持久化，让rabbitmq落地存储消息的话，消息不一定会丢，得等这个实例恢复了，然后才可以继续从这个queue拉取数据。这种方案没有什么所谓的高可用性，主要是提高吞吐量的，就是说让集群中多个节点来服务某个queue的读写操作。

2. 镜像集群模式

   ![img](/Interview.assets/10297697-95bcf8e25c194150.webp)

这种模式，才是所谓的rabbitmq的高可用模式，跟普通集群模式不一样的是，创建的queue，无论元数据还是queue里的消息都会存在于多个实例上，然后每次写消息到queue的时候，都会自动把消息到多个实例的queue里进行消息同步。
这样的话，好处在于，你任何一个机器宕机了，没事儿，别的机器都可以用。坏处在于，第一，这个性能开销也太大了吧，消息同步所有机器，导致网络带宽压力和消耗很重！第二，这么玩儿，就没有扩展性可言了，如果某个queue负载很重，你加机器，新增的机器也包含了这个queue的所有数据，并没有办法线性扩展你的queue
怎么开启这个镜像集群模式呢？rabbitmq有很好的管理控制台，就是在后台新增一个策略，这个策略是镜像集群模式的策略，指定的时候可以要求数据同步到所有节点的，也可以要求就同步到指定数量的节点，然后再次创建queue的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。

