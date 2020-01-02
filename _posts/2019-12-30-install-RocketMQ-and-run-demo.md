---
layout: post
title: '安装RocketMQ，并运行示例'
subtitle: '基于官方文档的例子解决一些未说明的问题'
author: 'Max'
header-style: text
tags:
  - RocketMQ
  - Java
---

## \* 先决条件

1. 64 位 JDK1.8+
2. Maven 3.2.x
3. Git

## 1. 下载、安装

1. 从官网下载[RocketMQ](http://rocketmq.apache.org)

2. 解压 `zip` 包

   ```
   unzip rocketmq-all-4.6.0-bin-release.zip
   ```

## 2. 运行示例

- ### 快速开始

  1. 启动`namesrv`

     ```
     // 启动 namesrv
     nohup sh bin/mqnamesrv &
     // 查看日志
     tail -f ~/logs/rocketmqlogs/namesrv.log
     ```

  2. 启动`broker`

     ```
     // 启动 broker
     nohup sh bin/mqbroker -n localhost:9876 &
     // 查看日志
     tail -f ~/logs/rocketmqlogs/broker.log
     ```

     刚启动 `broker` 发现程序直接退出，查看日志

     ```
     Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000005c0000000, 8589934592, 0) failed; error='Cannot allocate memory' (errno=12)
     #
     # There is insufficient memory for the Java Runtime Environment to continue.
     # Native memory allocation (mmap) failed to map 8589934592 bytes for committing reserved memory.
     # An error report file with more information is saved as:
     # /opt/rocketmq-all-4.6.0/hs_err_pid10455.log
     ```

     发现说 无法分配内存，谷歌一把之后，修改脚本文件，在 `bin` 目录下找到 `runserver.sh` 和 `runbroker.sh` , 修改以下配置

     - 先退出 `namestr`

       ```
       ./bin/mqshutdown namesrv
       ```

     - 修改配置

       ```
       JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn125m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
       ```

     然后重启 `namesrv` 和 `broker`，能正常启动

3. 发送和接收消息

   - 设置环境变量

     ```
     export NAMESRV_ADDR=localhost:9876
     ```

   - 发送消息

     ```
     sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer

     SendResult [sendStatus=SEND_OK, msgId= ...
     ```

     可以看到日志说明发送成功

   - 接收消息

     ```
     sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer

     ConsumeMessageThread_%d Receive New Messages: [MessageExt...
     ```

     可以看到日志说明接收成功

- ### 简单消息示例

  1. **引入依赖 (以 `Java` 和 `maven` 为例)**

     ```
     <dependency>
     	<groupId>org.apache.rocketmq</groupId>
         <artifactId>rocketmq-client</artifactId>
         <version>4.3.0</version>
     </dependency>
     ```

  2. **发送同步消息**

     ```
     public class SyncProducer {
         public static void main(String[] args) throws Exception {
             //Instantiate with a producer group name.
             DefaultMQProducer producer = new
                 DefaultMQProducer("please_rename_unique_group_name");
             // 输入服务器地址
             producer.setNamesrvAddr("localhost:9876");
             //启动
             producer.start();
             for (int i = 0; i < 100; i++) {
                 //Create a message instance, specifying topic, tag and message body.
                 Message msg = new Message("TopicTest" /* Topic */,
                     "TagA" /* Tag */,
                     ("Hello RocketMQ " +
                         i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
                 );
                 //Call send message to deliver message to one of brokers.
                 SendResult sendResult = producer.send(msg);
                 System.out.printf("%s%n", sendResult);
             }
             //Shut down once the producer instance is not longer in use.
             producer.shutdown();
         }
     }
     ```

     > 如创建 Topic 报错，运行 broker 脚本时，添加如下参数
     >
     > autoCreateTopicEnable=true
     >
     > 检查客户端版本是否与服务端一致
     >
     > 若运行 ./mqadmin topicList -n localhost:9876 报错
     >
     > 如果是报 unable to calculate a request signature. error=Algorithm HmacSHA1 not available
     >
     > 在 bin 目录下找到 tools.sh
     >
     > 在\${JAVA_HOME}/jre/lib/ext 后加上 ext 文件夹的绝对路径 > 寻找路径参照 Centos7 寻找 JDK 路径

     我们可以在控制台启动 接收消息的代码，然后运行 Java 程序来验证是否成功

3. **发送异步消息**

   ```
   public class AsyncProducer {
       public static void main(String[] args) throws Exception {
           //Instantiate with a producer group name.
           DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
           // Specify name server addresses.
           producer.setNamesrvAddr("localhost:9876");
           //Launch the instance.
           producer.start();
           producer.setRetryTimesWhenSendAsyncFailed(0);
           for (int i = 0; i < 100; i++) {
                   final int index = i;
                   //Create a message instance, specifying topic, tag and message body.
                   Message msg = new Message("TopicTest",
                       "TagA",
                       "OrderID188",
                       "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
                   producer.send(msg, new SendCallback() {
                       @Override
                       public void onSuccess(SendResult sendResult) {
                           System.out.printf("%-10d OK %s %n", index,
                               sendResult.getMsgId());
                       }
                       @Override
                       public void onException(Throwable e) {
                           System.out.printf("%-10d Exception %s %n", index, e);
                           e.printStackTrace();
                       }
                   });
           }
           //Shut down once the producer instance is not longer in use.
           producer.shutdown();
       }
   }
   ```

   执行代码时，控制台报错

   ```
   org.apache.rocketmq.client.exception.MQClientException: No route info of this topic, TopicTest
   See http://rocketmq.apache.org/docs/faq/ for further details.
   	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:610)
   	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.access$300(DefaultMQProducerImpl.java:86)
   	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl$2.run(DefaultMQProducerImpl.java:443)
   	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
   	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
   	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
   	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
   	at java.lang.Thread.run(Thread.java:748)
   ```

   检查连接状况，发现都正常，在官网文档中，一个小哥指出了这个问题

   ```
   have you tested your demo code? AsyncProducer does not work. if you add some countdownlatch inbetween , and timeout the result, it's working. the code itself in github has problems as well..
   ```

   在使用`countdownlatch`，能解决这个问题，并且 github 上的[示例](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/simple/AsyncProducer.java)也是这样使用的，可能是因为主线程提前终结，导致的异步线程出现的问题，直接在 `producer.shutdown()` 使主线程睡眠也可解决此问题。

   ```
   TimeUnit.SECONDS.sleep(3);
   ```

4) **以单向模式发送消息**

   ```
   public class OnewayProducer {
       public static void main(String[] args) throws Exception{
           //Instantiate with a producer group name.
           DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
           // Specify name server addresses.
           producer.setNamesrvAddr("localhost:9876");
           //Launch the instance.
           producer.start();
           for (int i = 0; i < 100; i++) {
               //Create a message instance, specifying topic, tag and message body.
               Message msg = new Message("TopicTest" /* Topic */,
                   "TagA" /* Tag */,
                   ("Hello RocketMQ " +
                       i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
               );
               //Call send message to deliver message to one of brokers.
               producer.sendOneway(msg);

           }
           //Shut down once the producer instance is not longer in use.
           producer.shutdown();
       }
   }

   ```

5. **接收消息**

   ```
   public class Consumer {

       public static void main(String[] args) throws InterruptedException, MQClientException {

           // Instantiate with specified consumer group name.
           DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name");

           // Specify name server addresses.
           consumer.setNamesrvAddr("localhost:9876");

           // Subscribe one more more topics to consume.
           consumer.subscribe("TopicTest", "*");
           // Register callback to execute on arrival of messages fetched from brokers.
           consumer.registerMessageListener(new MessageListenerConcurrently() {

               @Override
               public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                   ConsumeConcurrentlyContext context) {
                   System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                   return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
               }
           });

           //Launch the consumer instance.
           consumer.start();

           System.out.printf("Consumer Started.%n");
       }
   }
   ```

- ### 有序消息

  1. 发送消息

     ```
     public class OrderedProducer {
         public static void main(String[] args) throws Exception {
             //Instantiate with a producer group name.
             MQProducer producer = new DefaultMQProducer("example_group_name");
             //Launch the instance.
             producer.start();
             String[] tags = new String[] {"TagA", "TagB", "TagC", "TagD", "TagE"};
             for (int i = 0; i < 100; i++) {
                 int orderId = i % 10;
                 //Create a message instance, specifying topic, tag and message body.
                 Message msg = new Message("TopicTestjjj", tags[i % tags.length], "KEY" + i,
                         ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                 SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                 @Override
                 public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                     Integer id = (Integer) arg;
                     int index = id % mqs.size();
                     return mqs.get(index);
                 }
                 }, orderId);

                 System.out.printf("%s%n", sendResult);
             }
             //server shutdown
             producer.shutdown();
         }
     }
     ```

2. 消费消息

   ```
   public class OrderedConsumer {
       public static void main(String[] args) throws Exception {
           DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("example_group_name");

           consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

           consumer.subscribe("TopicTest", "TagA || TagC || TagD");

           consumer.registerMessageListener(new MessageListenerOrderly() {

               AtomicLong consumeTimes = new AtomicLong(0);
               @Override
               public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs,
                                                          ConsumeOrderlyContext context) {
                   context.setAutoCommit(false);
                   System.out.printf(Thread.currentThread().getName() + " Receive New Messages: " + msgs + "%n");
                   this.consumeTimes.incrementAndGet();
                   if ((this.consumeTimes.get() % 2) == 0) {
                       return ConsumeOrderlyStatus.SUCCESS;
                   } else if ((this.consumeTimes.get() % 3) == 0) {
                       return ConsumeOrderlyStatus.ROLLBACK;
                   } else if ((this.consumeTimes.get() % 4) == 0) {
                       return ConsumeOrderlyStatus.COMMIT;
                   } else if ((this.consumeTimes.get() % 5) == 0) {
                       context.setSuspendCurrentQueueTimeMillis(3000);
                       return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
                   }
                   return ConsumeOrderlyStatus.SUCCESS;

               }
           });

           consumer.start();

           System.out.printf("Consumer Started.%n");
       }
   }

   ```


     > 记得设置 `namesrvAddr`，由于发送的消息的 Tags 和 接收消息的 Tags 不同，可以根据输出的日志来对比是不是顺序消费

- ### 广播

  1. 生产者

     ```
     public class BroadcastProducer {
         public static void main(String[] args) throws Exception {
             DefaultMQProducer producer = new DefaultMQProducer("ProducerGroupName");
             producer.start();

             for (int i = 0; i < 100; i++){
                 Message msg = new Message("TopicTest",
                     "TagA",
                     "OrderID188",
                     "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
                 SendResult sendResult = producer.send(msg);
                 System.out.printf("%s%n", sendResult);
             }
             producer.shutdown();
         }
     }
     ```

  2. 消费者

     ```
     public class BroadcastConsumer {
         public static void main(String[] args) throws Exception {
             DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("example_group_name");

             consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

             //set to broadcast mode
             consumer.setMessageModel(MessageModel.BROADCASTING);

             consumer.subscribe("TopicTest", "TagA || TagC || TagD");

             consumer.registerMessageListener(new MessageListenerConcurrently() {

                 @Override
                 public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                     ConsumeConcurrentlyContext context) {
                     System.out.printf(Thread.currentThread().getName() + " Receive New Messages: " + msgs + "%n");
                     return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                 }
             });

             consumer.start();
             System.out.printf("Broadcast Consumer Started.%n");
         }
     }
     ```

     > 记得设置 `namesrvAddr`，可以多开几个 Consumer 来验证是否成功

* ### 定时消息

  1. 消费者

     ```
      public class ScheduledMessageConsumer {

          public static void main(String[] args) throws Exception {
              // Instantiate message consumer
              DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ExampleConsumer");
              // Subscribe topics
              consumer.subscribe("TestTopic", "*");
              // Register message listener
              consumer.registerMessageListener(new MessageListenerConcurrently() {
                  @Override
                  public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> messages, ConsumeConcurrentlyContext context) {
                      for (MessageExt message : messages) {
                          // Print approximate delay time period
                          System.out.println("Receive message[msgId=" + message.getMsgId() + "] "
                                  + (System.currentTimeMillis() - message.getStoreTimestamp()) + "ms later");
                      }
                      return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                  }
              });
              // Launch consumer
              consumer.start();
          }
      }
     ```

  2. 生产者

     ```
     public class ScheduledMessageProducer {

          public static void main(String[] args) throws Exception {
              // Instantiate a producer to send scheduled messages
              DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
              // Launch producer
              producer.start();
              int totalMessagesToSend = 100;
              for (int i = 0; i < totalMessagesToSend; i++) {
                  Message message = new Message("TestTopic", ("Hello scheduled message " + i).getBytes());
                  // This message will be delivered to consumer 10 seconds later.
                  // 设置消息延时10秒发送
                  message.setDelayTimeLevel(3);
                  // Send the message
                  producer.send(message);
              }

              // Shutdown producer after use.
              producer.shutdown();
          }

      }
     ```

     ***

     **DelayTimeLevel 详解**

     > 官网愣是没找到相关说明，只能撸源码了

     在 `CommitLog` 有使用 `MessageConst.PROPERTY_DELAY_TIME_LEVEL`

     ```
     // Timing message processing
     {
         String t = propertiesMap.get(MessageConst.PROPERTY_DELAY_TIME_LEVEL);
         if (ScheduleMessageService.SCHEDULE_TOPIC.equals(topic) && t != null) {
             int delayLevel = Integer.parseInt(t);

             if (delayLevel > this.defaultMessageStore.getScheduleMessageService().getMaxDelayLevel()) {
                 delayLevel = this.defaultMessageStore.getScheduleMessageService().getMaxDelayLevel();
             }

             if (delayLevel > 0) {
                 tagsCode = this.defaultMessageStore.getScheduleMessageService().computeDeliverTimestamp(delayLevel,
                     storeTimestamp);
             }
         }
     }
     ```

     然后找 `ScheduleMessageService` 类，发现有解析 `DelayLevel` 的方法

     ```
     public boolean parseDelayLevel() {
         HashMap<String, Long> timeUnitTable = new HashMap<String, Long>();
         timeUnitTable.put("s", 1000L);
         timeUnitTable.put("m", 1000L * 60);
         timeUnitTable.put("h", 1000L * 60 * 60);
         timeUnitTable.put("d", 1000L * 60 * 60 * 24);

         String levelString = this.defaultMessageStore.getMessageStoreConfig().getMessageDelayLevel();
         try {
             String[] levelArray = levelString.split(" ");
             for (int i = 0; i < levelArray.length; i++) {
                 String value = levelArray[i];
                 String ch = value.substring(value.length() - 1);
                 Long tu = timeUnitTable.get(ch);

                 int level = i + 1;
                 if (level > this.maxDelayLevel) {
                     this.maxDelayLevel = level;
                 }
                 long num = Long.parseLong(value.substring(0, value.length() - 1));
                 long delayTimeMillis = tu * num;
                 this.delayLevelTable.put(level, delayTimeMillis);
             }
         } catch (Exception e) {
             log.error("parseDelayLevel exception", e);
             log.info("levelString String = {}", levelString);
             return false;
         }

         return true;
     }
     ```

     可以得知 `DelayLevel` 是从 `MessageStoreConfig` 来的，其中有个属性 `messageDelayLevel`

     ```
     private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
     ```

     并且， `MessageStoreConfig` 是在 `BrokerStartup` 启动时初始化的

     ```
     if (commandLine.hasOption('c')) {
         String file = commandLine.getOptionValue('c');
         if (file != null) {
             configFile = file;
             // 如果-c参数存在, 且配置文件存在, 则加载配置文件
             InputStream in = new BufferedInputStream(new FileInputStream(file));
             properties = new Properties();
             properties.load(in);

             properties2SystemEnv(properties);
             MixAll.properties2Object(properties, brokerConfig);
             MixAll.properties2Object(properties, nettyServerConfig);
             MixAll.properties2Object(properties, nettyClientConfig);
             // 覆盖默认配置
             MixAll.properties2Object(properties, messageStoreConfig);

             BrokerPathConfigHelper.setBrokerConfigPath(file);
             in.close();
         }
     }
     ```

     所以，在启动 `broker` 时，先修改 `conf` 目录下的 `broker.conf` 文件

     ```
     messageDelayLevel = 10s 1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
     ```

     然后指定配置文件启动就 ok 了

     ```
     nohup sh bin/mqbroker -n localhost:9876 autoCreateTopicEnable=true -c /opt/rocketmq-all-4.6.0/conf/broker.conf &
     ```
