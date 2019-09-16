---
title: 'DomDatastore 基本介绍'
date: 2019-09-04T09:58:55.000Z
description: 'DomDatastore 基本介绍'
categories:
    - 框架
    - odl
    - controller
tags:
    - odl
    - controller
---  
  
  
##  官方介绍
  
  
[官方介绍网址](https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Architecture:DOM_DataStore#In-Memory_Datastore_.2F_Cache )
  
##  术语
  
  
| 属于                                | 解释                                                                                                       |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Binding                             | 由 YANG Schema 生成的 Java 接口，类和约定。                                                                |
| Binding Aware                       | 绑定感知，使用了 YANG Schema 生成的数据和 API 的组件或功能。                                               |
| Binding Indepent                    | 绑定无关，使用 DOM 方式进行数据和 API 调用的组件或功能，它独立于由 YANG 生成的 Java 语言绑定。             |
| Binding-independent type identifier | 类似 QName 的格式的数据结构或 RPC 方法的标识符                                                             |
| Consumer                            | 使用由另一个提供者提供的模型和/或 API 的组件（例如应用程序）。                                             |
| Data operation                      | 描述整个系统状态（配置，运行数据）的数据子集之上的操作。                                                   |
| DTO                                 | 数据传输对象(Data Transfer Object)：用于在 Binding-Aware 组件之间传输数据的简单对象。 DTO 是绑定的一部分。 |
| Infrastructure Component            | 既不是提供者也不是消费者，但暴露或扩展 SAL 功能的组件                                                      |
| Provider                            | 通过特定于模型的 API 或以独立于绑定的格式为应用程序提供功能的组件                                          |
| SAL                                 | 服务抽象层                                                                                                 |
| NSF                                 | 网络服务功能（例如 TopologyManager，ForwardingRulesManager）                                               |
## 
  
 ![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/DomDatastore介绍.md-2019-09-16-13-29-15.png )
  
##  新标准化树形数据模型
  
  
对于数据的新标准化模型将代表后面的 YANG 规格的实际概念。 这将不再是基于序列化
格式（定义见 YANG 规格和使用的 SAL- 经纪参数 impl1.0 ）。
  
- <strong> NormalizedNode </strong>
  代表在树结构中的一个 节点的基本类型 ;所有其他节点类型是来自这个基本类型。它包
  含一个叶子识别符和一个值。
- - DataContainerNode
    其中包含 多个叶节点 ;它在 YANG 语法中没有直接表示。
- - - ContainerNode
      容器节点 ，它包含了多个子叶片和映射到 YANG 容器陈述。 （ Node, which
      represents a leaf which can occur only once per parent node; it contains multiple child
      leaves and maps to the container statement in YANG. ）
- - - MapEntryNode
      节点代表一个叶子， 叶子是唯一识别的键的值。 MapEntryNode 中可以包含多
      个。（ Node which represents a leaf, which can occur multiple times; a leave is
      uniquely identified by the value of its key. A MapEntryNode may contain
      multiple ）
- - - ChoiceNode
      节点代表一个叶子， 这大多发生一次， 每个父节点， 但可能的值可以有不同的
      类型。地图来选择语句。类型映射到该选择的 case 语句。
- - - AugmentationNode
      Node which represents a leaf, which occurs mostly once per parent node.
- - LeafNode
    叶子节点，存放相关值（ PS：官网解释是 Contains simple value. ）。
- - LeafSetEntryNode
- - LeafSetNode
- - MapNode
  
##  DOM Data Broker
  
  
提供访问概念性数据树存储方法，并提供接口对存储树的某一分支下的数据更改。
具体定义的类
  
执行过程
当前的 DOM 数据代理（ DOM Data Broker ）并没有设计成一个智能化的内存缓存树形
结构， 这种内存缓存树形结构能够跟踪依赖关系， 计算变更和维护提交处理程序， 通知监听
器和实现数据之间的关系假设。 这可能会导致效率低下， 执行两个阶段提交， 其中在由数据
代理本身所做的一切状态跟踪。如下所示：
  
1. 计算受影响的子树。
2. 过滤提交处理受影响的子树。
3. 过滤数据受影响子树的数据改变监听器。
4. 捕获初始状态数据的更改监听器。
5. 启动所有受影响的提交处理程序的提交申请。
6. 完成提交的所有受影响的提交处理程序。
7. 捕获最后的数据状态改变监听器。
8. 向受影响的数据更改监听器发布数据更改事件。
  
dom broker 需要保持和维护以下部分的状态
1. 将子树路径映射到已注册的提交处理程序
2. 将子树路径映射到已注册的数据会更改侦听器
3. 将子树路径映射到已注册的数据读取器
  
此外，dom broker职责是：
  
1. 路由读取数据的请求
2. 数据两阶段提交
3. 发布数据更改事件：捕获数据更改之前的状态和数据更改之后的状态
  
  
https://wiki.opendaylight.org/view/OpenDaylight_Controller:Binding-Independent_Components
  
  
https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Architecture:DOM_DataStore:Plugging_in_a_Datastore_into_MD-SAL
  
  
https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Architecture:DOM_DataStore:Transactions
  
https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Architecture:Clustering#ShardTransaction
  
  
https://www.sdnlab.com/15782.html
  
  
https://wiki.opendaylight.org/view/OpenDaylight_Controller:Binding-Independent_Data_Format
  
  
https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Architecture:Clustering
  
  
https://wiki.opendaylight.org/view/YANG_Tools:YANG_to_Java_Mapping
  