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

## flume ng 可能带来的提升

* 解耦合，不再依赖 zookeeper 和 flume master node 。

