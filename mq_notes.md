# 安装
在 Ubuntu 16.04.1 LTS 系统下安装及配置 RabbitMQ 的方法

## 安装 Erlang

```bash
# https://tecadmin.net/install-erlang-on-ubuntu/
cd /tmp
wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
sudo dpkg -i erlang-solutions_1.0_all.deb
sudo apt-get update
sudo apt-get install erlang
```

## 安装RabbitMQ

```bash
sudo echo "deb http://www.rabbitmq.com/debian/ testing main" >> /etc/apt/sources.list
sudo wget https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
sudo apt-key add rabbitmq-release-signing-key.asc
sudo apt-get update
sudo apt-get install rabbitmq-server
```

# 队列
## 订阅队列
- basic.consume:

    持续订阅队列，队列中一有消息就会取到

- basic.get:

    订阅队列，获取一条消息，取消订阅。频繁使用get会严重影响吞吐量

## 负载均衡
当队列由有多个消费者订阅时，rabbitmq 采用 `round robin` 算法进行负责均衡

## 消费队列
- 确认消息

    消费者从队列取到消息需要显示的发送 `basic.ack` 确认消息接受，或者在订阅时设置auto_ack参数，取道消息后自动ack。消费者正确确认消息后，队列才能安全删除消息。

    如果消费者接到一条消息还未确认就断开链接或取消订阅，队列会重新分发此消息

    如果消费者一直未 ack，队列不会再向其分发消息，直到消费者 ack 确认。利用这一点可以延迟确认防止在处理耗时的任务时负载过高

- 拒绝消息

    把消费者从 rabbit 断开连接。所有版本都支持。但是频繁的断开、链接会增加负担

    2.0以上版本支持 `basic.reject` 命令，reject命令的 `requee` 参数为 true 时，队列会重新分发，如果设置为 false，队列会移除次消息

- basic.reject，requeue=fase 与什么都不做，直接 ack 的区别

    在将来的版本中，rabbitmq 将支持一个名为 `dead leetter` 的队列，用来存放那些被拒绝而不重新入队的消息。通过观察这些消息可以发现存在的问题

## 创建队列