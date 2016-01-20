# Flume Ng 与Og 的区别

## 核心组件变化

图 1 和图 3 是两个版本的架构图。

FLUM OG 的特点是：
* FLUM OG 有三种角色的节点，如图 1：代理节点（agent）、收集节点（collector）、主节点（master）。
* agent 从各个数据源收集日志数据，将收集到的数据集中到 collector，然后由收集节点汇总存入 hdfs。master 负责管理 agent，collector 的活动。
* agent、collector 都称为 node，node 的角色根据配置的不同分为 logical node（逻辑节点）、physical node（物理节点）。对 logical nodes 和     physical nodes 的区分、配置、使用一直以来都是使用者最头疼的地方。
* agent、collector 由 source、sink 组成，代表在当前节点数据是从 source 传送到 sink。如图 2。

图 1. FLUM OG 架构图

![图一](http://www.ibm.com/developerworks/cn/data/library/bd-1404flumerevolution/1.jpg)

图 2. OG 节点组成图

![图二](http://www.ibm.com/developerworks/cn/data/library/bd-1404flumerevolution/2.jpg)

对应于 OG 的特点，FLUM NG 的特点是：
* NG 只有一种角色的节点：代理节点（agent）。
* 没有 collector、master 节点。这是核心组件最核心的变化。
* 去除了 physical nodes、logical nodes 的概念和相关内容。
* agent 节点的组成也发生了变化。如图 4，NG agent 由 source、sink、channel 组成。

图 3. FLUM NG 架构图

![图三](http://www.ibm.com/developerworks/cn/data/library/bd-1404flumerevolution/3.jpg)

图 4. NG 节点组成图

![图三](http://www.ibm.com/developerworks/cn/data/library/bd-1404flumerevolution/4.jpg)

从整体上讲，NG 在核心组件上进行了大规模的调整，核心组件的数目由 7 删减到 4。由于 Flume 的使用涉及到众多因素，如 avro、thrift、hdfs、jdbc、zookeeper 等，而这些组件和 Flume 的整合都需要关联到所有组件。所以核心组件的改革对整个 Flume 的使用影响深远：

* 大大降低了对用户的要求，如核心组件的变化使得 Flume 的稳定使用不再依赖 zookeeper，用户无需去搭建 zookeeper 集群；另外用户也不再纠结于   OG 中的模糊概念（尤其是 physical nodes、logical nodes，agent、collector）。
* 有利于 Flume 和其他技术、hadoop 周边组件的整合，比如在 NG 版本中，Flume 轻松实现了和 jdbc、hbase 的集成。
* 将 OG 版本中复杂、大规模、不稳定的标签移除，Flume实现了向灵活、轻便的转变，而且在功能上更加强大、可扩展性更高，这一点主要表现在用户   使用 Flume 搭建日志收集集群的过程中。

## 删减节点角色，脱离 zookeeper

Zookeeper 是针对大型分布式系统的可靠协调系统，适用于有多类角色集群管理。比如在 hbase 中，对 HMaster、HRegionServer 的管理。

在 OG 版本中，Flume 的使用稳定性依赖 zookeeper。它需要 zookeeper 对其多类节点（agent、collector、master）的工作进行管理，尤其是在集群中配置多个 master 的情况下。当然，OG 也可以用内存的方式管理各类节点的配置信息，但是需要用户能够忍受在机器出现故障时配置信息出现丢失。所以说 OG 的稳定行使用是依赖 zookeeper 的。

而在 NG 版本中，节点角色的数量由 3 缩减到 1，不存在多类角色的问题，所以就不再需要 zookeeper 对各类节点协调的作用了，由此脱离了对 zookeeper 的依赖。由于 OG 的稳定使用对 zookeeper 的依赖表现在整个配置和使用过程中，这就要求用户掌握对 zookeeper 集群的搭建及其使用（尤其是要熟悉 zookeeper 数据文件夹 data，Flume 节点配置信息存放在此文件夹中）；掌握 Flume 中和 zookeeper 相关的配置。对初次接触 Flume 的用户来讲，这是非常痛苦的事。

## 用户配置变化

从用户角度来讲，配置过程无疑是整个集群搭建的核心步骤。Flume 的配置分为两个部分：安装和数据传输配置。

### 安装

OG 在安装时：

* 在 flume-env.sh 中设置$JAVA_HOME。
* 需要配置文件 flume-conf.xml。其中最主要的、必须的配置与 master 有关。集群中的每个 Flume 都需要配置 master 相关属性（如     flume.master.servers、flume.master.store、flume.master.serverid）。
* 如果想稳定使用 Flume 集群，还需要安装 zookeeper 集群，这需要用户对 zookeeper 有较深入的了解。
* 安装 zookeeper 之后，需要配置 flume-conf.xml 中的相关属性，如 flume.master.zk.use.external、flume.master.zk.servers。
* 在使用 OG 版本传输数据之前，需要启动 master、agent。

NG 在安装时，只需要在 flume-env.sh 中设置$JAVA_HOME。

### 数据传输配置

* shell 命令：需要用户掌握 Flume shell 命令；
* master console 页面：这是 OG 用户最常用的配置方式；弊端在于，除非用户熟悉复杂的各类 source，sink 配置函数以及格式（source：大约 25 个，sink：大约 46 个），否则在复杂的集群环境下，用户每次只能配置一个节点（指定 source、sink）来保证配置的准确性；**我们目前经常使用的配置方式为master console**。

NG 的配置只需要一个配置文件，这个配置文件中存放 source、sink、channel 的配置。如图 5 中是配置文件 example.conf 里面的内容，其中 agent_foo 是 agent 名字。然后在启动 agent 时，使用一下命令指定这个配置文件：
```shell
$ bin/flume-ng agent --conf-file example.conf --name agent_foo -Dflume.root.logger=INFO,console
```
图 5. example.conf

![图五](http://www.ibm.com/developerworks/cn/data/library/bd-1404flumerevolution/5.jpg)

# 公司 flume 升级的可行性分析

## flume ng 整体架构介绍

Flume架构整体上看就是 source-->channel-->sink 的三层架构（参见最上面的 图三），类似生产者和消费者的架构，他们之间通过queue（channel）传输，解耦。

* Source:完成对日志数据的收集，分成 transtion 和 event 打入到channel之中。 
* Channel:主要提供一个队列的功能，对source提供中的数据进行简单的缓存。 
* Sink:取出Channel中的数据，进行相应的存储文件系统，数据库，或者提交到远程服务器。 

## 升级 flume 可能会遇到的问题。

### 如何做到平滑升级

我们目前日志收集的流程：

* flume 收集前段日志，以 avro 的格式打到 LVS 上；
* LVS 将请求分发给三台 elevator 服务器；
* 由 elevator 将数据写入kafka。

### 升级的思路

#### 只升级 flume 模块

* 优点：升级平滑，不影响其他模块使用。
* 缺点：原有 flume og 通过 avro 协议发送的数据是经过自己封装的, 升级需要重写 flume sink 插件，ng并不支持原有 og 的插件。

#### 去掉 elevator 直接将日志写入kafka

* 优点：简化逻辑，增加系统的健壮性，可以直接使用官方的包实现功能；
* 缺点：少了 elevator 的白名单、统计、日志落地等功能。

这里说下我对 elevator 模块几个重点功能的理解：

* 白名单：设置 topic 过滤名单，因为 flume master console 页面是可能会泄露的，所以为防止不需要的 topic 进入 kafka 而设置白名单功能。但 flume ng 是没有 master console 页面的，所有 topic 都是通过配置文件来配置，所以不存在白名单功能的需求。
* metrics：流量统计，用于监控 elevator 各个 topic 流量的功能。
* 日志落地：在 kafka 宕机或网络问题无法写入 kafka 时，将日志落地，写入 levelDb 等待服务恢复后重新写入。

flume ng 提供了两种常用的 MemoryChannel 和 FileChannel 供大家选择。其优劣如下：
* MemoryChannel: 所有的events被保存在内存中。优点是高吞吐。缺点是容量有限并且Agent死掉时会丢失内存中的数据。
* FileChannel: 所有的events被保存在文件中。优点是容量较大且死掉时数据可恢复。缺点是速度较慢。
可用于代替elevator中的日志落地功能。

## 升级的关键技术

### 重写 AvroSink
 
原有的 AvroDFOTopicSink，它的本质还是创建一个 AvroTopicSink
```java
public class AvroDFOTopicSink extends EventSink.Base {
    public static final String BATCH_COUNT = "batchCount";
    public static final String BATCH_MILLIS = "batchMillis";

    public static SinkFactory.SinkBuilder dfoBuilder() {
        return new SinkFactory.SinkBuilder() {
                public EventSink build(Context context, String[] args) {
                    Preconditions.checkArgument(args.length >= 3,
                        "usage: avroInsistentTopic(hostname, portno, [ topic, isJson ]{, batchCount=1}{, batchMillis=0}");

                    String host = FlumeConfiguration.get().getCollectorHost();
                    int port = FlumeConfiguration.get().getCollectorPort();
                    String topic = "undefine";
                    boolean isJson = false;

                    if (args.length >= 1) {
                        host = args[0];
                    }

                    if (args.length >= 2) {
                        port = Integer.parseInt(args[1]);
                    }

                    if (args.length >= 3) {
                        topic = args[2];
                    }

                    if (args.length >= 4) {
                        isJson = Boolean.parseBoolean(args[3]);
                    }

                    String batchDeco = "";
                    String batchN = context.getValue("batchCount");
                    int n = 1;

                    if (batchN != null) {
                        n = Integer.parseInt(batchN);
                    }

                    String batchLatency = context.getValue("batchMillis");
                    int ms = 1000;

                    if (batchLatency != null) {
                        ms = Integer.parseInt(batchLatency);
                    }

                    if (n > 1) {
                        batchDeco = batchDeco + " batch(" + n + "," + ms +
                            ") ";
                    }

                    FlumeConfiguration conf = FlumeConfiguration.get();
                    long maxSingleBo = conf.getFailoverMaxSingleBackoff();
                    long initialBo = conf.getFailoverInitialBackoff();
                    long maxCumulativeBo = conf.getFailoverMaxCumulativeBackoff();

                    String avroTopic = String.format("avroTopic(\"%s\", %d, \"%s\", \"%s\")",
                            new Object[] {
                                host, Integer.valueOf(port), topic,
                                Boolean.valueOf(isJson)
                            });

                    String snk = String.format(batchDeco +
                            " < %s ? diskFailover insistentAppend " +
                            " stubbornAppend insistentOpen(%d,%d,%d) %s >",
                            new Object[] {
                                avroTopic, Long.valueOf(maxSingleBo),
                                Long.valueOf(initialBo),
                                Long.valueOf(maxCumulativeBo), avroTopic
                            });

                    try {
                        return FlumeBuilder.buildSink(context, snk);
                    } catch (FlumeSpecException e) {
                        throw new IllegalArgumentException(e);
                    }
                }
            };
    }

    public static List<Pair<String, SinkFactory.SinkBuilder>> getSinkBuilders() {
        return Arrays.asList(new Pair[] { new Pair("avroDFOTopic", dfoBuilder()) });
    }
}
```

AvroTopicSink
```java
public class AvroTopicSink extends EventSink.Base {
    static final Logger LOG = LoggerFactory.getLogger(AvroTopicSink.class);
    public static final String A_SERVERHOST = "serverHost";
    public static final String A_SERVERPORT = "serverPort";
    public static final String A_SENTBYTES = "sentBytes";
    protected AvroTopicEventServer avroClient;
    String host;
    int port;
    String topic;
    boolean isJson;
    AccountingTransceiver transport;

    public AvroTopicSink(String host, int port, String topic, boolean isJson) {
        this.host = host; 
        this.port = port;
        this.topic = topic;
        this.isJson = isJson;
    }

    // 通过avroClient发送数据
    public void append(Event e) throws IOException, InterruptedException {
        List afe = toAvroListEvent(e, this.topic, this.isJson);

        ensureInitialized();

        try {
            this.avroClient.append(afe);
            super.append(e);
        } catch (Exception e1) {
            throw new IOException("Append failed " + e1.getMessage(), e1);
        }
    }

    // 检查链接状态
    private void ensureInitialized() throws IOException {
        if ((this.avroClient == null) || (this.transport == null)) {
            throw new IOException(
                "avroTopic called while not connected to server");
        }
    }

    // 将batch event转化为List<AvroTopicEvent>
    public static List<AvroTopicEvent> toAvroListEvent(Event e, String topic,
        boolean isJson) throws IOException {
        if (!BatchingDecorator.isBatch(e)) {
            return ImmutableList.of(toAvroEvent(e, topic, isJson));
        }

        int sz = ByteBuffer.wrap(e.get("batchSize")).getInt();
        List list = new ArrayList(sz);
        byte[] data = e.get("batchData");
        DataInput in = new DataInputStream(new ByteArrayInputStream(data));

        for (int i = 0; i < sz; i++) {
            WriteableEvent we = new WriteableEvent();
            we.readFields(in);
            list.add(toAvroEvent(we, topic, isJson));
        }

        return list;
    }
    
    // 将event转化为AvroTopicEvent
    public static AvroTopicEvent toAvroEvent(Event e, String topic,
        boolean isJson) {
        AvroTopicEvent tempAvroEvt = new AvroTopicEvent();
        tempAvroEvt.timestamp = e.getTimestamp();
        tempAvroEvt.body = new String(e.getBody());
        tempAvroEvt.host = e.getHost();
        tempAvroEvt.topic = topic;
        tempAvroEvt.json = isJson;
        tempAvroEvt.path = (e.getAttrs().containsKey("tailSrcFile")
            ? new String((byte[]) e.getAttrs().get("tailSrcFile")) : "");

        return tempAvroEvt;
    }

    //开启avroClient
    public void open() throws IOException {
        URL url = new URL("http", this.host, this.port, "/");
        Transceiver http = new HttpTransceiver(url);
        this.transport = new AccountingTransceiver(http);

        try {
            this.avroClient = ((AvroTopicEventServer) SpecificRequestor.getClient(AvroTopicEventServer.class,
                    this.transport));
        } catch (Exception e) {
            throw new IOException("Failed to open Avro event sink at " +
                this.host + ":" + this.port + " : " + e.getMessage());
        }

        LOG.info("AvroEventSink open on port  " + this.port);
    }

    //关闭链接
    public void close() throws IOException {
        if (this.transport != null) {
            this.transport.close();

            LOG.info("AvroEventSink on port " + this.port + " closed");
        } else {
            LOG.warn("Trying to close AvroEventSink, which was closed already");
        }
    }

    public long getSentBytes() {
        return this.transport.getSentBytes();
    }

    public ReportEvent getMetrics() {
        ReportEvent rpt = super.getMetrics();
        rpt.setStringMetric("serverHost", this.host);
        rpt.setLongMetric("serverPort", this.port);
        rpt.setLongMetric("sentBytes", this.transport.getSentBytes());

        return rpt;
    }
    
    //sink builder
    public static SinkFactory.SinkBuilder builder() {
        return new SinkFactory.SinkBuilder() {
                public EventSink build(Context context, String[] args) {
                    if (args.length < 3) {
                        throw new IllegalArgumentException(
                            "usage: avrotopic(hostname, portno, [ topic, isJson ]) ");
                    }

                    String host = FlumeConfiguration.get().getCollectorHost();
                    int port = FlumeConfiguration.get().getCollectorPort();
                    String topic = "undefine";
                    boolean isJson = false;

                    if (args.length >= 1) {
                        host = args[0];
                    }

                    if (args.length >= 2) {
                        port = Integer.parseInt(args[1]);
                    }

                    if (args.length >= 3) {
                        topic = args[2];
                    }

                    if (args.length >= 4) {
                        isJson = Boolean.parseBoolean(args[3]);
                    }

                    return new AvroTopicSink(host, port, topic, isJson);
                }
            };
    }

    public static List<Pair<String, SinkFactory.SinkBuilder>> getSinkBuilders() {
        return Arrays.asList(new Pair[] { new Pair("avroTopic", builder()) });
    }
}
```

flume ng 官方的AvroSink类

```java
//继承自AbstractRpcSink，可以将events发送到rpc服务器
public class AvroSink extends AbstractRpcSink {

  private static final Logger logger = LoggerFactory.getLogger(AvroSink.class);

  @Override
  protected RpcClient initializeRpcClient(Properties props) {
    logger.info("Attempting to create Avro Rpc client.");
    // 它会根据配置文件从RpcClientFactory中实例化不同的RpcClient
    return RpcClientFactory.getInstance(props); 
  }
}
```

AbstractRPCSink

```java
  @Override
  public Status process() throws EventDeliveryException {
    Status status = Status.READY;
    Channel channel = getChannel();
    Transaction transaction = channel.getTransaction();

    if(resetConnectionFlag.get()) {
      resetConnection();
      // if the time to reset is long and the timeout is short
      // this may cancel the next reset request
      // this should however not be an issue
      resetConnectionFlag.set(false);
    }

    try {
      transaction.begin();

      verifyConnection();

      List<Event> batch = Lists.newLinkedList();

      for (int i = 0; i < client.getBatchSize(); i++) {
        Event event = channel.take();

        if (event == null) {
          break;
        }

        batch.add(event);
      }

      int size = batch.size();
      int batchSize = client.getBatchSize();

      if (size == 0) {
        sinkCounter.incrementBatchEmptyCount();
        status = Status.BACKOFF;
      } else {
        if (size < batchSize) {
          sinkCounter.incrementBatchUnderflowCount();
        } else {
          sinkCounter.incrementBatchCompleteCount();
        }
        sinkCounter.addToEventDrainAttemptCount(size);
        client.appendBatch(batch);
      }

      transaction.commit();
      sinkCounter.addToEventDrainSuccessCount(size);

    } catch (Throwable t) {
      transaction.rollback();
      if (t instanceof Error) {
        throw (Error) t;
      } else if (t instanceof ChannelException) {
        logger.error("Rpc Sink " + getName() + ": Unable to get event from" +
            " channel " + channel.getName() + ". Exception follows.", t);
        status = Status.BACKOFF;
      } else {
        destroyConnection();
        throw new EventDeliveryException("Failed to send events", t);
      }
    } finally {
      transaction.close();
    }

    return status;
  }

```

flume ng 官方有四种RpcClient:

* NettyAvroRpcClient.appendBatch(batch)方法会调用appendBatch(events, requestTimeout, TimeUnit.MILLISECONDS)方法，该方法会首先确认链接处于READY状态，否则报错；然后将每个event重新封装成AvroFlumeEvent，放入avroEvents列表中；然后构造一个CallFuture和avroEvents一同封装成一个Callable放入线程池 handshake = callTimeoutPool.submit(callable)中去执行，其call方法内容是avroClient.appendBatch(avroEvents, callFuture)就是在此批量提交到RPC服务器；然后handshake.get(connectTimeout, TimeUnit.MILLISECONDS)在规定时间等待执行的返回结果以及等待append的完成waitForStatusOK(callFuture, timeout, tu)，详细的可看这里Flume的Avro Sink和Avro Source研究之二 ： Avro Sink ，有对于这两个future更深入的分析。一个批次传输的event的数量是min(batchSize,events.size())

* FailoverRpcClient.appendBatch(batch)方法会做最多maxTries次尝试直到获取到可以正确发送events的Client，通过localClient=getClient()--》getNextClient()来获取client，这个方法每次会获取hosts中的下一个HostInfo，并使用NettyAvroRpcClient来作为RPC Client，这就又回到了(1)中，这个方法还有一个要注意的就是会先从当前的lastCheckedhost+1位置向后找可以使用的Client，如果不行会再从开始到到lastCheckedhost再找，再找不到就报错。使用localClient.appendBatch(events)来处理events，可参考(1)。

* LoadBalancingRpcClient.appendBatch(batch)方法，首先会获取可以发送到的RPC服务器的迭代器Iterator<HostInfo> it = selector.createHostIterator()；然后取一个HostInfo,RpcClient client = getClient(host)这个Client和(2)一样都是NettyAvroRpcClient，但是getClient方法会设置一个保存名字和client映射的clientMap；client.appendBatch(events)执行之后就会跳出循环，下一次appendBatch会选择下一个client执行。

* ThriftRpcClient.appendBatch(batch)方法，以thrift协议发送events，与我们系统不相关，不再赘述。

### ng的高可用方案

#### Agent死掉
Agent死掉分为两种情况：机器死机或者Agent进程死掉。

对于机器死机的情况来说，由于产生日志的进程也同样会死掉，所以不会再产生新的日志，不存在不提供服务的情况。

对于Agent进程死掉的情况来说，确实会降低系统的可用性。对此，有下面两种种方式来提高系统的可用性。首先，我们服务的进程都是在watchdog下启动的，如果进程死掉会被系统立即重启，以提供服务。其次，对所有的Agent进行存活监控，发现Agent死掉立即报警。

#### Collector死掉

由于中心服务器提供的是对等的且无差别的服务，且Agent访问Collector做了LoadBalance和重试机制。所以当某个Collector无法提供服务时，Agent的重试策略会将数据发送到其它可用的Collector上面。所以整个服务不受影响。

#### kafka 服务器宕机或不可访问

假如kafka宕机或不可访问，colletor可以将所收到的events缓存到FileChannel，保存到磁盘上继续提供服务。等kafka服务恢复后再将FileChannel中缓存的events发送给kafka。

### ng的可靠性分析
对日志收集系统来说，可靠性(reliability)是指Flume在数据流的传输过程中，保证events的可靠传递。

对Flume来说，所有的events都被保存在Agent的Channel中，然后被发送到数据流中的下一个Agent或者最终的存储服务中。那么一个Agent的Channel中的events什么时候被删除呢？当且仅当它们被保存到下一个Agent的Channel中或者被保存到最终的存储服务中。这就是Flume提供数据流中点到点的可靠性保证的最基本的单跳消息传递语义。

那么Flume是如何做到上述最基本的消息传递语义呢？

首先，Agent间的事务交换。Flume使用事务的办法来保证event的可靠传递。Source和Sink分别被封装在事务中，这些事务由保存event的存储提供或者由Channel提供。这就保证了event在数据流的点对点传输中是可靠的。在多级数据流中，如下图，上一级的Sink和下一级的Source都被包含在事务中，保证数据可靠地从一个Channel到另一个Channel转移。

其次，数据流中 Channel的持久性。Flume中MemoryChannel是可能丢失数据的（当Agent死掉时），而FileChannel是持久性的，提供类似mysql的日志机制，保证数据不丢失。

# 总结

flume ng 相对于 og 改动还是挺大的，系统稳定性也有所提升。从对现有系统破坏性最小的角度来看，升级时重写AvroSink来支持elevator无疑是最好的选择。但放弃elevator使用flume ng官方api仍然是一个不错的选择。

# Refer

* [基于Flume的美团日志收集系统(一)架构和设计](http://tech.meituan.com/mt-log-system-arch.html)
* [基于Flume的美团日志收集系统(二)架构和设计](http://tech.meituan.com/mt-log-system-optimization.html)
* [Flume NG 简介及配置实战](http://my.oschina.net/leejun2005/blog/288136?fromerr=LbU0tNi4)
* [Flume NG：Flume 发展史上的第一次革命](http://www.ibm.com/developerworks/cn/data/library/bd-1404flumerevolution/index.html)
* [高可用Hadoop平台－Flume NG实战图解篇](http://www.tuicool.com/articles/ruyANn)
* [【Flume】Flume-NG源码阅读之AvroSink](http://blog.csdn.net/szwangdf/article/details/34098807)
* [Flume配置-Failover](http://www.aboutyun.com/blog-70-465.html)
* [flume学习（八）：自定义source](http://blog.csdn.net/xiao_jun_0820/article/details/38312091)
