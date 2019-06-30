@[TOC](目录)
# 前言
关于RabbitMQ 与ActiveMQ的文章在网上有很多。我个人比较好奇它们的通信机制，为什么或RabbitMQ的路由很灵活。它们的通信机制是怎么样的呢？内容来源下面几篇文章，非常感谢。
https://yq.aliyun.com/articles/44584
https://blog.csdn.net/zwgdft/article/details/53561277

# 消息域
**点对点**：其实就是生成者者和消费者共用一个**队列**。当然，消费者和生产者可以有多个。只不过当消息被某个消费者消费后，就会在队列上销毁掉。也就是一条消息，只会被一个消费者消费。

**订阅模式**：一个发布者，或者多个发布者，共同发布到一个主题。然后订阅者可以订阅该**主题**。当主题有消息时，多订阅者都能共同接收这些消息。但是如果消息不是持久订阅的话，消费者只有在线的情况下才能接收。

# 通信方式
## RabbitMQ
- **通信方式**
 RabbitMQ是基于AMQP协议来实现的消息中间件。AMQP，类似于HTTP协议，也是一个应用层的协议，网络层使用TCP来通信。因此，RabbitMQ也是典型的C-S模型，准确地说是C-S-C模型，因为伴随着RabbitMQ的使用，总是会有Producer与Consumer两个Client和一个Broker Server。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190323181557986.png)
  Client要与Server进行通信，就必须先建立连接，RabbitMQ中有Connection与Channel两个概念，前者就是一个TCP连接，后者是在这个连接上的虚拟概念，负责逻辑上的数据传递，因此，为了节省资源，一般在一个客户端中建立一个Connection，每次使用时再分配一个Channel即可。**一个Connection可以由多个Channel**

- **消息体** 
  Message是RabbitMQ中的消息体概念。类似HTTP传输中，有header和body两部分数据，Message中也有Attributes和Payload两部分数据，前者是一些元信息，后者是传递的消息数据实体。

- **消息投递** 
  Exchange、Queue与Routing Key三个概念是理解RabbitMQ消息投递的关键。RabbitMQ中一个核心的原则是，消息不能直接投递到Queue中。Producer只能将自己的消息投递到Exchange中，由Exchange按照routing_key投递到对应的Queue中，具体的架构参见下图。细细品味就会体会到这样设计的精妙之处。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190323181537680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xzd19kYWJhb2ppYW4=,size_16,color_FFFFFF,t_70)
  那么，具体实现时，如何完成这三者关系的绑定？总结起来是两点：第一，在Consumer Worker中，声明自己对哪个Exchange感兴趣，并将自己的Queue绑定到自己感兴趣的一组routing_key上，建立相应的映射关系；第二，在Producer中，将消息投递一个Exchange中，并指明它的routing_key。由此可见，Queue这个概念只是对Consumer可见，Producer并不关心消息被投递到哪个Queue中。

   
## ActiveMQ
点对点的的消息模式：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190323182331414.png)
  **消息结构**
- **消息头**
消息头包含消息的识别信息和路由信息，消息头包含一些标准的属性如：JMSDestination（在点对点中就是指队列名），JMSMessageID等。 

- **消息属性**
如果需要除消息头字段以外的值，那么可以使用消息属性。这种新属性包含以下几种：
应用需要用到的属性;
消息头中原有的一些可选属性;
JMS Provider 需要用到的属性。 
- **消息体**
 有以下类型：
TextMessage：  java.lang.String对象，如xml文件内容；
MapMessage：   名/值对的集合，名是String对象，值类型可以是Java任何基本类型；
BytesMessage： 字节流
StreamMessage：Java中的输入输出流
ObjectMessage：Java中的可序列化对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190323180629892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xzd19kYWJhb2ppYW4=,size_16,color_FFFFFF,t_70)
通过上图，我们以TCP协议来分析ActiveMQ的通讯机制

**客户(Client)**：消息生产者和消费者对于MQ都是客户。

**消息中转器(Message Broker)**：它是MQ的核心，负责处理消息。

**通讯步骤**

- **建立链接**：

activeMQ初始化时，通过TcpTransportServer类根据配置打开TCP侦听端口，客户通过该端口发起建立链接的动作。
把accept的Socket放入阻塞队列中。 
另外一个线程Socket handler阻塞着等待队列中是否有新的Socket，如果有则取出来
生成一个TransportConnection的实例。TransportConnection类的主要作用是处理链路的状态信息，并实现CommandVisitor接口来完成各类消息的处理

TransportConnection会使用一个由多个TransportFilter实例组成的消息处理链条，负责对接收到的各类消息进行处理并发送相应的应答。这个链条的典型组成顺序：MutexTransport->WireFormatNegotiator->InactivityMonitor->TcpTransport。在这条链条中最后的一环就是TcpTransport类，它是实际和Client获取和发送数据的地方，该类的重要方法有run()和oneway()，一个负责读取，一个负责发送。 
建链完成，可以进行通讯操作。 

- **关闭链接**

activeMQ发现TCP链接的关闭，最关键的代码在TcpBufferedInputStream类中的 int n = in.read(buffer, position, buffer.length - position);

- **心跳**

为了更好的维护TCP链路的使用，activeMQ采用了心跳机制作为判断双方链路的健康情况。activeMQ使用的是双向心跳，也就是activeMQ的Broker和Client双方都进行相互心跳，但不管是Broker或Client心跳的具体处理情况是完全一样的，都在InactivityMonitor类中实现，下面具体介绍
心跳会产生两个线程“InactivityMonitor ReadCheck”和“InactivityMonitor WriteCheck”，它们都是Timer类型，都会隔一段固定时间被调用一次。ReadCheck线程主要调用的方法是readCheck()，当在等待时间内，有消息接收到，则该方法会返回true。
   

RabbitMQ的一大特色就是用到Exchange交换机这个设计，有了这个“交换机”，我们就可以在上面做很多灵活的复杂的路由配置来进行通信。