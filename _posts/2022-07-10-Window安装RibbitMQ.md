Window安装RibbitMQ

RabbitMQ

RabbitMQ是由Erlang语言开发的AMQP的开源实现。AMQP：Advanced Message Queue,高级消息队列。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品，开发语言等条件的限制。RabbitMQ最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性，扩展性，高可用性等方面表现不俗。

具体特点：

1.可靠性：使用一些机制来保证可靠性，如持久性，传输确认，发布确认。

2.灵活的路由：在消息进入队列之前，通过Exchange来路由消息的。对于典型的路由功能，RabbitMQ已经提供了一些内置的Exchange来实现。针对更复杂的路由功能，可以将多个Exchange绑定在一起，也通过插件机制实现自己的Exchange。

3.消息集群：多个RabbitMQ服务器可以组成一个集群，形成一个逻辑Broker

4.高可用：队列可以在集群中的机器上进行镜像，使得在部分节点出问题情况下队列仍然可用。

5.多种协议：RabbitMQ支持多种消息队列协议，比如STOMP，MQTT等等。

6.多语言客户端：RabbitMQ几乎支持所有常用的语言，比如：java，.Net，Ruby等等。

7.管理界面：提供一个易用的用户界面，使得用户可以监视和管理消息Broker的许多方便。

8.跟踪机制：如果消息异常，RabbitMQ提供了消息跟踪机制，使用者可以找出发生了什么。

9.插件机制：提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

安装：

官方地址：https://www.rabbitmq.com/

找到Download+installation

选择window版本

找到Dependencies依赖关系

