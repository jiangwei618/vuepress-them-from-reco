---
title: ovs-appctl 用户手册中文翻译
date: 2019-08-22 17:09:22 
description: ovs-appctl 用户手册中文翻译
categories:
- 框架
- openvswitch
tags: 
- openvswitch
export_on_save:
    markdown: true

  
---

## 1. 通用命令
exit 优雅关闭ovs-vswitchd进程 

qos/show interface 查询内核中关于qos的配置以及和给出端口有关的状态 

cfm/show [interface] 显示在指定端口上CFM配置的详细信息。如果没有指定接口，则显示所有使能了CFM的接口 

cfm/set-fault [interface] status 强制将指定端口的CFM模块的错误状态（如果没指定接口则是全部接口）设置成指定的状态。可以是”true”,”false”,”normal” 

stp/tcn [bridge] 在运行了stp的bridge上强制进行拓扑变更。之将导致该dp发送拓扑变更通知并且刷新MAC表。。如果没有指定桥，则应用到所有dp   



## 2. 桥命令
这些命令用于管理桥。  

fdb/flush [bridge] 清除指定桥的MAC学习表，没有指定桥则应用于所有桥 

fdb/show bridge 列出指定桥上每个MAC直至与VLAN的对应信息，并且包含该学习到该MAC的端口号还有该条目的age信息，单位为秒 

bridge/reconnect [bridge] 命令桥断开和当前openFlow控制器的连接并且重连，如果没有指定桥，则应用于所有桥，这个命令可以在分析排查控制器错误的时候很有用 

bridge/dump-flows bridge 列出桥上所有的流，包括那些在其他命令中（例如 ovs-ofctl dump-flows）默认隐藏的流.一些机制比如带内管理等设置的流策略是不行允许修改和覆盖的，所以对控制器来说他们是隐藏的。  

BOND命令 这些命令管理ovs桥上绑定端口。要了解这些命令，你需要了解一种叫做源负载分担（SLB）的实施细节。作为直接将源MAC地址设成SLAVE的做法，通过特定的计算将48bit的MAC自动化映射到一个8bit的值（MAC hash）。所有匹配这个hash值得mac地址被指定为slave。 

bond/list 列出所有的绑定配置，以及slaves，范围包含所有桥 

bond/show[port] 给出指定端口的所有绑定有关的信息（updelay，downdelay，距离下次进行重新平衡的时间），如果没指定端口，则列出所有bond的端口。同时也列出所有slave的信息，包括这些slave是处于enable还是disable状态、完成一个正在实施中的updelay或者一个downdelay的时间、是否是激活态的slave。任何关于LACP的信息可以使用lacp/show来查看。 

bond/migrate port hash slave 仅适用于配置了SLB的绑定。分配一个指定的machash值给一个新的slave。Port指定了bond的端口，hash则是将要迁移的mac hash值（十进制0到255之间），slave即是要新的slave。 这个重新制定的关系不是永久的：rebalanceing或者发生fail-over时，这个mac hash蒋辉按照常规的方式切换到新的slave上面 MAC hash值不能指定到一个disable态的slave上  

bond/set-active-slave port slave 将给定slave设为激活态的slave。给定的slave必须是enable状态。 这个配置不是永久的：如果该slave变成disable，将会自动选择一个新的slave。 

bond/enable-slave port slave 

bond/disable-slave port slave Enable/disableslave在给定的bond port上，忽略任何updelay和downdelay。 这个设置不是永久的：他将保持到该slave的承载状态变化 

bond/hashmac [valn] [basis] 返回指定mac（伴随指定vlan和basis）的hash值 

lacp/show [port] 列出所有指定端口的lacp关联信息。包括active/passive、system id、systempriority。同时列出每个slave 的信息：enable/disable、连接上或者未连接上、端口id和优先级、主用信息和成员信息。如果没有指定端口，则显示所有应用了CFM的接口信息。   


## 3. 数据通道命令（datapath）
这些命令管理逻辑数据通道。类似ovs-dpctl的命令。 

dpif/dump-dps 在多行中显示所有配置的datapath名称 

dpif/show[dp….] 打印dp的汇总信息，包括dp的状态还有连接上的端口列表。端口的信息包括openFlow的端口号，datapath的端口号，以及类型（本地端口被标识为openflow port 65534） 如果指定了一个或多个datapath，将只显示指定的这些dp的信息。否则，则显示所有dp的信息。 

dpif/dump-flows dp 想控制端打印dp中流表的所有条目。 这个命令主要来与debugOpen Vswitch.它所打印的流表不是openFlow的流条目。它打印的是由dp模块维护的简单的流。如果你想查看OpenFlow条目，请使用ovs-ofctl dump-flows。dpif/del-fow dp 删除指定dp上所有流表。同上所述，这些不是OpenFlow流表。   



## 4. OpenFlow协议命令
这些命令管理核心OpenFlow交换的实施。 

ofproto/list 列出所有运行中ofproto实例。这些名字可能在ofproto/trace中用到。 

ofproto/trace[dpname] odp_flow [-generate] [packet] ofproto/tracebridgebr_flow [-generate] [packet] 追踪报告构造包在交换机中的路径。包头（例如源和目的）和元数据（比如：入端口），一起组成它的“flow”，根据这些“flow”决定包的目的地。你可以用些列途径地址流。 dpnameodp-flow odp-flow 是一个可以使用 ovs-dpctl dump-flows命令打印出来的流。如果你所有的桥都是同样样的类型，这也是通常的情况，那么你可以忽略 dp-name，但是如果你的桥拥有不同类型（即，ovs-netdev和ovs-system型），那么你必须要指定dp-name。 bridgebr_flow br_flow是一种可以使用ovs−ofctl  add−flow命令添加的流类型。（这不是一个OpenFlow流：除了其他的差异，这种流永远不会有通配符）bridge指定了被追踪的br-flow经过的桥名。   通常情况下，你可以只指定一个流，用以上提到的一种形式，但是有时候你可能需要值一个确切的数据包来代替流   副作用 有些动作是由副作用的，比如，normal 动作能刷新MAC学习表，learn动作会改变OpenFlow表。ofproto/trace只有在指定包的时候发生副作用。如果你需要虚作用，那么你必须提供一个包。 （output 动作也是明显的副作用，但是ofproto/trace 永远不会执行这个动作，即便是你制定了包的时候）   不完整的信息 大多数时候，Open Vswitch能够尽力用流就得出一个包所经路径的所有信息，但是在一些特定场景下，ovs可能需要查看一些不包含在流内的其他包的部分信息。这种情况下，如果你不提供一个包，那么ofproto/trace就会提示你需要一个包。   如果你希望在ofproto/trace 操作中包含一个包，你有两种方法实现：   -generate 这个选项，附加在之前叙述的两种流方式后面用来在内生成该流的一个包并且使用这个包。如果你的目地是利用副作用，那么这个选项是你达成目标的最容易的方法。但是-generate 不是一个填充不完整信息的好方式，因为生成的包是基于流信息的，即是说这个包并不能带有任何这个流以外的信息。   packet 这种形式提供了一个明确的以十六进制数字序列表示的包。一个以太网帧至少14 bytes长，即至少28个16进制的数字。很明显，使用手工输入是很不方便的。好在我们的ovs-pacp 和ovs-tcpundump 工具提供了简便的方法。 利用这种形式，包头直接从packet中提取，那么odp_flow或者br_flow应该只包含元数据。元数据可以是以下类型：   skb_priority 报文的qos优先级 skb_mark 报文的SKB标记 tun_id 报文到达的隧道id号 in_port 报文到达的端口 第一种流格式的in_port的值是内核 datapath的端口号，而OpenFlow的端口号值是OpenFlow的端口号。这两种端口号一般都是不一样的，而且没什么关系可言。   

ofproto/self-check [switch] 运行内部一致性检查，并且返回一个简要的汇总。指定桥的时候限定在该实例，否则是所有实例。如果汇总报告了任何错误，那么ovs的日志中会包含更多详细的信息。请将这些错误报告作为bug发送给ovs的开发者。  

## 5. vlog命令
这些命令管理ovs-vswitchd的日志配置   

vlog/set [spec] 设置日志等级。没有spec时，设置所有模块和设施的日志等级为dbg。其他情况下，spec是一个用逗号或者空格或者冒号分隔的单词列表，最多支持下面所述范畴的每样配置一个。 l  一个可用模块名，可以用ovs-appctlvlog/list 命令来查看所有可用模块名。 l syslog、console、file改变着三项任意项的日志等级。 l off、emer、err、warn、info、dbg，这些用来控制日志等级，不低于这些等级的消息蒋辉被记录在日志中，所有低于该等级的将被过滤。参考ovs-appctl查看日志等级的详细定义。 如果没有指定spec， 对于file选项，不论日志等级是否设置，只有当ovs-vswitchd调用 –log-file选项时，日志才会被记录至文件。 为了保持和老版本的ovs的兼容性，any可以作为合法参数但是不会发生作用。 vlog/set PATTERN:facility:pattern 设置应用于每个设施日志的格式，可以参考ovs-appctl查看格式的可用语法信息。 

vlog/list 列出所有支持记录日志的模块和他们当前的日志等级。

vlog/reopen 让ovs-vswitchd关闭并且重新打开日志文件（可以用于在转换日志后，重新建立一个新日志文件来使用） 需要ovs-vswitchd 使能 –log-file选项时才有效 vlog/disable-rate-limit [module]… vlog/enable-rate-limit [module]… 默认情况下，ovs-vswitchd 限制了记录日志的速率。当消息发生的频率高于默认值时，该消息将会被抑制。这将节省磁盘空间，让日志更加可读，并且让进程更加流畅，但是有些情况下的排错需要更多的细节。这样，vlog/disable−rate−limit允许特定独立模块的日志记录不限制在默认速率下。你可以指定一个或多个模块名，这些模块名可以通过vlog/list查看。不指定模块名或者使用any关键字将应用到所有记录日志的模块。 vlog/enable−rate−limit命令，和vlog/disable−rate−limit的语法一样，可以恢复速率限制。 内存命令（MEMORYCOMMANDS） 报告内存的使用率 memory/show 显示一些ovs-vswitchd内存使用的基础状态信息。ovs−vswitchd也会在启动后并且周期性的检测内存的增长   COVERAGE COMMANDS 这个命令管理ovs−vswitchd的“coverage counters”，即在守护进程运行期间发生的特殊事件的次数。除了使用这个命令意外，当ovs−vswitchd检测到主循环运行周期异常长的时候，会自动以INFO的日志等级记录coverage counters。 主要用于性能分析和debugging。 coverage/show 显示coverage counters值。  


## 6. 压力选项命令（STRESSOPTION COMMANDS）
这些命令允许开发者测试OpenvSwitch，从而触发一些只有在极端案例中出现的行为。开发和测试者因此可以更加容易的发现只有在偶然的后者极端情况下才会出现的bug。压力选项可能导致一些非正常的行为，但这些行为可能并不是bug，所以，这些命令应该仅仅用于测试。  

stress/enable

stress/disable 所有压力选项的开关 stress/list 以下用表格形式列出了所有可用的压力选项和他们的设置，表头如下： NAME 一个单词用来定义这个选项，并在stress/set中使用这个名字 DESCRIPTION 为不了解这个选项内部代码和该选项完成动作的人所添加的描述信息 PERIOD 现行配置的触发周期。如果压力选项是去使能状态，那么这也是disabled。否则这是一个表示压力选项触发的动作时间发生的间隔计数。 MODE 如果压力选项disabled，那么值为n/a。其他情况，如果选项制定了确切的触发周期那么是periodic，如果是指定周期内随机触发则是random COUNTER 如果压力选项disabled，那么n/a，其余情况，显示下个触发周期钱一共触发了多少次。 HITS 这个压力选项从程序启动开始一共触发了多少次 RECOMMENDED 对不熟悉间隔的人建议的周期。这是一个可信的压力，不会造成系统的崩溃。MINIMUM/MAXIMUM 周期的最大/最小值 DEFAULT 默认周期，这将会在今使能了压力选项（tress/enable），但是没有具体配置的时候（stress/set）。当压力选项为关闭时默认关闭。  
stress/set option period [random|periodic] 设置压力选项option的周期为给定period值。period为0是disable这个选项。指定random时，将以一个平均周期为给定period的值来随机触发这个选项，指定periodic时，将精确地以周期值来触发动作。之后是默认值。 有过压力选项没有用stress/enable使能，这个命令将不会生效。  


## 7. 限制(LIMITS)
我们相信限制和我们如下所写的一样精确。这些限制假设你使用linux内核的dp。 

l  大约256个桥需要5000个文件描述符来（ovs-switchd进程每个datapath需要17个文件描述符）

l  每个桥65280个端口。根据绑定的hash标的尺寸，每个桥接口在1024以上，性能将会降级 

l  每个桥可以学习2048个MAC条目 

l  内核的流仅受限于内核的可用内存。32位的内核每个桥维护的流数大于1048576或者64位的内核维护的流数目大于262144时，性能将会降级。（ovs-vswitchd永远都不应该加载那么多条流） 

l  OpenFlow流仅受限于可用内存。性能根据独特的通配符个数呈现线性分布。OpenFlow表中相同通配符的流信息都有相同的查找时间。但是当一个表拥有很多不同通配置的流时，查表的时间就线性上升了。 

l  每个桥255端口可以加入STP协议 

l  每个桥支持32个端口镜像（MIRROR） 

l  端口名最长15字节（这是linux系统内核的限制）

）