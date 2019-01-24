
### Producer
> 产生消息病发送到Broker；

一个Producer可以产生多个Topic下的消息；

Producer Group为多个同类Producer实例的集合，一般情况下一个组只有一个实例；


### Consumer
> 消费Broker中的消息；

一个Broker可以消费多个Topic下的消息；

Consumer Group为多个同类Consumer实例的集合，这些实例必须订阅相同的主题消息；

消息消费分为：推、拉 两种；

### Topic
> 为消息的分类

一个主题上面可以对应有多个Producer和Consumer，也可以没有，两者之间是松耦合的关系；


### Message
> 需要传递的信息；

一个消息必须附带一个Topic，有且只有一个，用于确定消息发往哪个Broker，也用于关联订阅信息；

### Message Queue
> 一个Topic再细分到多个sub-topic

### Tag
> sub-topic

用于对一个Topic下面的消息进行再分类，方便业务处理逻辑；

### Broker
> 消息服务核心，接收Producer生产的消息，并进行存储，然后接收Consumer的消费请求；

### Name Server
> 存储Broker中的消息注册信息，包含消息主题、Broker地址等信息；

