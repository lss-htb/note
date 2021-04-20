# RabbitMQ

消息队列主要解决耦合, 异步处理, 流量削峰等问题



## 问题

### 几种消息队列的对比

- RabbitMQ (队列模式/消息代理)
    - Erlang
    - 支持多种语言
    - 客户端 SDK: Java .NET, Ruby, Python, PHP, NodeJS, Rust, C/C++, GO, Erlang (由于是商业维护, 支持非常多客户端)
    - 多协议支持 AMQP, XMPP< SMTP, STOMP
    - 不支持批量消息操作
    - master/slave, slave 只做备份
    - 万级消息吞吐量
    - 微妙级消息延迟
    - 事务: 支持
    - 支持均衡负载
    - 支持集群
    - 持久化能力: 内存,文件, 支持数据堆积, 但数据堆积反过来影响生产效率
    - 是否有序: 若想有序, 只能用一个 Client

- ActiveMQ
    - Java
    - 多语言支持
    - 协议支持: OpenWire, STOMP, REST, XMPP, AMQP
    - 消息批量操作: 支持
    - 消息推拉模式: 多协议,Pull/Push 均支持
    - HA: 基于 ZooKeeper + LevelDB 的 Master-Slave 实现模式
    - 数据可靠性: master/slave
    - 单机吞吐量: 万级(最差)
    - 持久化能力: 内存,文件,数据库
    - 集群: 支持
    - 负载均衡: 支持
    - 管理界面: 一般

- RocketMQ
    - Java
    - 支持语言: Java, C++(不成熟)
    - 客户端 SDK: Java，C++，Python，Go，Node.Js
    - 协议支持: 自己定义的一套(社区提供的 JMS 不成熟)
    - 消息批量操作: 支持
    - 消息推拉模式: 多协议, Pull/Push 都支持
    - HA: 支持多 Master 模式, 多 Master 多 Slave 模式, 异步复制模式, 多 Master 多 Slave 模式, 同步双写
    - 数据可靠性: 支持异步实时刷盘, 同步刷盘, 同步复制, 异步复制
    - 单机吞吐量: 最高(十万级)
    - 持久化能力: 磁盘
    - 是否有序: 有序
    - 事务: 支持
    - 均衡负载: 支持
    - 管理界面: 命令行界面
    
- Kafka (发布订阅模式/分布式流式系统/消息当日志储存)
    - Scala & Java
    - 官方支持 Java, 开源社区有多语言版本
    - 客户端 SDK: C/C++, Python, Go, NodeJS, PHP 支持非常多  [官方文档](https://cwiki.apache.org/confluence/display/KAFKA/Clients)
    - 协议支持: 自有协议, 社区封装了 HTTP 协议
    - 消息批量操作: 支持
    - 消息推拉模式: Pull
    - HA: 支持 repllca 机制, leader 宕机后, 备份自动顶替, 并重新选举 leader (基于 Zookeeper)
    - 数据可靠性: 数据可靠, 并且有 repllca 机制, 有容错容灾能力
    - 单机吞吐量: 次之(十万级)
    - 消息延迟: 毫秒级
    - 事务: 不支持, 但可以通过 Low Level API 保证仅消费一次
    - 顺序处理: 有
    - 集群: 支持
    - 负载均衡: 支持
    - 管理界面: 官网命令行, 有开源界面

### RabbitMQ 的应用场景

1. 非实时性: 不需要立即获得结果
2. 应用耦合: 多应用间通过队列对同一任务进行处理, 避免一失败全失败
3. 异步处理: 多应用对同一消息进行处理, 应用间并发处理消息.
4. 限流削峰: 应用于秒杀或者抢购活动中, 避免流量过大导致应用挂掉的情况.
5. 消息驱动的系统: 系统分为消息队列, 生产者, 消费者等.

对比 Kafka:

RabbitMQ: 遵循 AMQP 协议, 由内在高并发的 erlang 语言开发, 用在实时的对可靠性要求比较高的消息传递上
Kafka 主要用于处理活跃的流式数据, 大数据量的数据处理上

### RabbitMQ 对比 Kafka

- 优先级队列
    - RabbitMQ 支持
    - Kafka 不支持
- 延迟队列
    - RabbitMQ 支持
- 死信队列
    - RabbitMQ 支持
- 重试队列
    - 不支持 (但是可以通过延迟队列来实现重试队列)
- 消费模式
    - RabbitMQ: Push/Pull 模式
    - Kafka: Pull
- 广播消费
    - RabbitMQ: 支持, 但力度较 Kafka 弱
    - Kafka: 支持
- 消息回溯
    - RabbitMQ: 不支持, 消息一旦被确认消费就会被标记删除
    - Kafka 支持按照 offset 和 timestamp 两种维度进行消息回溯
- 消息堆积
    - RabbitMQ: 支持, 一般情况下, 内存堆积达到特定阈值时会影响其性能, 但不是绝对的, 如果考虑到吞吐这因素, Kafka 的堆积效率比 RabbitMQ 总体上要高很多
    - Kafka: 支持
- 持久化
    - RabbitMQ: 支持
    - Kafka: 支持
- 消息追踪
    - RabbitMq: 支持. 可以采用 Firehose 或者 rabbitmq_tracing 插件实现. 不过开启 rabbitmq_tracing 插件会大幅度影响性能, 不建议生产环境开启, 反倒是可以使用 Firehost 与 外部链路系统结合提供高细腻度的消息追踪支持.
    - Kafka: 不支持. 但是可以通过外部系统来支持
- 消息过滤
    - RabbitMQ: 不支持, 但是可以封装
    - Kafka: 客户端级别的支持
- 多租户:
    - RabbitMQ: 支持
    - Kafka: 不支持
- 多协议支持:
    - RabbitMQ: 本身就是 AMQP 协议的实现, 同时支持 MQTT, STOMP 等协议
    - Kafka: 只支持定义协议, 目前在几个主流版本间存在兼容性问题
- 跨语言支持
    - RabbitMQ: 采用 Erlang 编写, 支持多种语言的客户端
    - Kafka: 采用 Scala 和 Java 编写, 支持多种语言的客户端
- 流量控制
    - RabbitMQ: 基于 Credit-Based 算法, 是内部被动触发的保护机制, 作用于生产者层面.
    - Kafka: 支持 client 和 user 级别, 通过主动设置可将流控作用于生产者或消费者
- 消息顺序性
    - RabbitMQ: 顺序性的条件比较苛刻, 需要单线程来发送, 单线程消费并不采用延迟队列, 优先级队列等一些高级功能, 从某种意义上来说不算支持顺序性.
    - Kafka: 支持单分区(partition)级别的顺序性
- 安全机制
    - RabbitMQ: 和 Kafka 类似
    - Kafka: TLS/SSL, SASL 身份认证和 读写控制权
- 幂等性
    - RabbitMQ: 不支持
    - Kafka: 支持单个生产者分区单会话的幂等性
- 事务性消息
    - RabbitMQ: 支持
    - Kafka: 支持

### 如果不用消息队列, 项目中会怎么实现对应功能?

开协程执行任务, 耦合度高

### RabbitMQ 内部使用原理? 工作流程?

- 发布流程
    1. Publisher 和 Broker 建立 TCP 连接
    2. Publisher 和 Broker 建立信道
    3. Publisher 通过 channel 将 message 发送给 Broker, 由 Exchange 将 message 进行转发
    4. Exchange 将 message 转发到指定的 Queue (队列)
- 接收
    1. Consumer 和 Broker 建立 TCP 连接
    2. Consumer 和 Broker 建立 channel
    3. Consumer 监听 Queue
    4. 当有消息到达 Queue 时, Broker 默认将 message 推送给 Consumer
    5. Consumer 接收到消息

### RabbitMQ 如何保证的数据可靠性?

- 丢失消息的可能位置
    - 生产者->交换机
        - 事务 TX
            - 将 channel 设置为事务模式
            - 事务提交/事务回滚
        - 确认 Confirm
            - 将 channel 设置为确认模式
            - 增加确认监听  Listener
            - 处理监听结果
    - 交换机->队列
        - Mandatory: 设置监听 Listener 实现
            - 发送消息 basicPublish() 时将设置 mandatory 参数设置为 true
            - 将 channel 增加 MandatoryListener 监听
        - 备用交换机
            - 当交换机消息未找到路由队列时将消息转发到备用交换机
    - 队列
        - 消息持久化
            - 队列与队列消息同时持久化
            - 队列持久化: 创建队列时, 将持久化参数设置为 true
            - 队列消息持久化: 发送消息方法参数列表要求传递 BasicProperties, deliveryMod 表示消息持久化.
    - 消费者->队列
        - 消费者确认
            - basicAck: 单个消息确认/多个消息确认
                - deliveryTag: 消息的唯一编号
                - multiple: 批量操作, 编码小于上面编号的消息都做本次一致的操作
            - basicReject: 单个消息拒绝
                - deliveryTag
                - requeue: 是否重新放回队列, 这里抛弃的消息如果设置了死信转发, 将会被路由到配置的死信交换器
            - basicNack: 批量拒绝
- 

### RabbitMQ 中各种名词之间的关系 

- Broker: 简单来说就是消息队列服务器实体
- Exchange: 消息交换机, 它指定消息按什么规则, 路由到哪个队列
- Queue: 消息队列载体, 每个消息都会被投入到一个或多个队列
- Binding: 绑定, 它的作用就是把 exchange 和 queue 按照路由规则绑定起来
- Routing Key: 路由关键字, exchange 根据这个关键字进行消息投递
- vhost: 虚拟主机, 一个 broker 里可以开设多个 vhost, 用作不同用户的权限分离
- producer: 消息生产者, 就是投递消息的程序
- consumer: 消息消费者, 就是接受消息的程序
- channel: 消息通道, 在客户端的每个连接里, 可建立多个 channel, 每个 channel 代表一个会话任务

### 使用流程

1. client 连接到消息队列服务器, 打开一个 channel
2. client 声明一个 exchange, 并设置相关属性
3. client 声明一个 queue, 并设置相关属性
4. client 使用 routing key, 在 exchange 和 queue 之间建立好绑定关系
5. client 投递消息到 exchange
6. exchange 接收到消息后, 就根据消息 key 和已经设置的 binding, 进行消息路由, 将消息投递到一个或多个队列里.

### 什么是 AMQP 协议 (Advanced Message QueuingProtocol)

开放式标准应用层协议。

可以简单的理解为一套消息传递的标准协议, 例如 HTTP 协议, HTTPS 协议都有自身的规则.

整体上就是一个 生产->消费流程: 连接创建与销毁, 生产消息, 消费消息

- 定义了这些特性
    - 消息方向
    - 消息队列
    - 消息路由 (包括: 点到点, 发布-订阅模式)
    - 可靠性
    - 安全性
- AMQP 协议栈
    - 包含三层
    - Model Layer: Exchanges, Queues, Transactions, Access Control Data Type
        - 协议最高层, 定义了一些供客户端调用的命令
    - Session Layer: Commands delivery, Exceptions handler, Sychronization
        - 主要负责将客户端命令发送给服务器, 在将服务器端的应答返回给客户端, 主要为客户端与服务端之间通信提供可靠性, 同步机制和错误处理.
    - Transport Layer: Data encoding, Framing, Failure detection, Multiplecing
        - 主要传输二进制数据流, 提供帧的处理,信道复用,错误检测和数据表示.
    
    
    
### 为什么选择 RabbitMQ

1. 除了 Qpid, RabbitMQ 是唯一一个实现了 AMQP 标砖的消息服务器
2. 可靠性, RabbitMQ 的持久化支持, 保证了消息的稳定性
3. 高并发, RabbitMQ 使用了 Erlang 开发语言, Erlang 是为电话交换机开发的语言, 天生自带高并发光环, 和高可用特性
4. 集群部署简单
5. 社区活跃度高

### 消息发送原理

首先应用程序和 Rabbit Server 之间会创建一个 TCP 连接, 一旦 TCP 打开, 并通过了认证, 认证就是你试图连接 Rabbit 之前发送的 Rabbit 服务器连接信息和用户名密码, 有点像程序连接数据. 一旦通过认证, 应用程序和 Rabbit Server 之间就创建了一条 AMQP 信道(channel).

信道是创建在真实 TCP 上的虚拟连接, AMQP 命令都是通过信道发送出去的, 每个信道都会有一个唯一的 ID, 不论是发布消息, 订阅队列或者介绍消息都是通过信道完成的.

> 为什么不直接通过 TCP 发送消息

对于操作系统来说创建和销毁 TCP 会话是非常昂贵的开销. 引入信道的概念, 我们可以在一条 TCP 连接上创建 N 多信道, 这样既能发送命令, 也能够保证每条信道的私密性, 我们可以想象成光纤电缆.

### 持久化原理

- 持久化选项 (三者同时满足才会将消息持久化)
    - (消息持久化) 投递消息的时候 durable 设置为 true, 消息持久化
    - (交换机持久化) 消息已经到达持久化交换机上
    - (队列持久化) 消息已经到达持久化的队列上
- 原理
    - Rabbit 会将持久化消息写入磁盘上的持久化日志文件, 等消息被消费后, Rabbit 会把这条消息标识为等待垃圾回收.
- 优缺点: 性能和稳定性的选择

### 确保消息不丢失

- 事务
    - 同步阻塞
- Confirm 机制
    - 生产消息后异步消息通知确认
- Ack 响应
    - 消费消息后发送 Ack 确认
    
### RabbitMQ 集群

- 多节点分担 Queue, 而不是每一个 Queue 在多个节点都存在
    - 减少数据冗余
    - 避免增加网络和磁盘的开销
- 交换机的绑定消息在所有节点上都是完整的
    - 队列元数据: 队列名称和它的属性
    - 交换机元数据: 交换机名称, 类型和属性
    - 绑定元数据: 一张简单的表格展示了如何将消息路由到队列
    - vhost 元数据: 为 vhost 内的队列, 交换机 和绑定提供命名空间和安全属性.
- 至少要一个磁盘节点
    - 保证节点发生故障后或重启节点后, 所有关于系统的配置与元信息都会从磁盘上恢复.
    - 保证节点故障后, 不会在其他的节点上声明同样的 Queue.
- 镜像队列
    - 镜像队列可以理解成节点的拷贝, 类似于 MySQL 的主从.
    - 当主节点下线时, 会从从节点中选举一个最老的从节点升级为主节点
    - 从节点加入时, 并不会主动的同步所有的消息过来. 显示调用命令同步时, 会阻塞队列, 无法进行操作, 直到同步完毕.
    - 主镜像消息的发布和消费时, 通过广播(GM)给所有的 slave 节点.
    - 对于 Basic.Publish, 消息会直接发送到所有的 master 和 slave 上, 这个时候 master 挂掉了 slave 上也有完整的消息.
    - 坏处是会造成数据的冗余, 造成性能的降低
    - GM (Guarenteed Multicast) 实现一种可靠的组播通讯歇息, 该协议能够保证组播消息的原子性, 即保证组中活着的节点要么都收到消息, 要么都收不到消息. 原理是将所有的节点形成一个循环链表, 消息从 master 发送后, 顺着链表传递, 最后又会回到 master 节点, 此时 master 节点就知道所有的节点都收到了消息了.
- 均衡负载
    - HAProxy 提供高可用, 均衡负载以及基于 TCP 和 HTTP 应用的代理, 支持虚拟主机, 是免费快速且可靠的一种解决方案.