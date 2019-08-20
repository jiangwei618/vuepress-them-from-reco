---
title: test
date: 2017-01-07T22:55:54.000Z
updated: teat
categories:
    - Diary
permalink: null
tags:
    - Games
toc: null
depth_from: 1
depth_to: 6
ordered: false
---  
  
  
#  网络处理
  
  
##  1. 常规步骤
  
###  1.1. add
  
  
  
  
***
<p style="text-align: center;"><Strong>第一步：添加网络</Strong></p>
  
当北向创建网络api被调用，neutron写入config neutron/network/network库，并触发datastore监听。
  
* neutronVpn/NeutronNetworkChangeListener：在进行一些基础检查后，创建elanInstance（一个网络对应一个elan instance）。创建elanInstance时会查询config elan-instances/elan-instance库，如果存在，则返回，如果不存在则创建enlanInstace数据，写入config库<br/><br/>
  
* ipv6Service/NeutronNetworkChangeListener：ipv6相关，暂不考虑<br/><br/>
  
  
* qosservice/QosNeutronNetworkChangeListener：qos相关，暂不考虑<br/><br/>
  
  
  
  
<details>
 <summary>第一步：网络创建</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1310.png?0.7974689986960424)  
</details>
  
  
***
  
<p style="text-align: center;"><Strong>第二步：创建elan</Strong></p>
  
&emsp;&emsp;当elan-instances/elan-instance config库被写入数据后，开始触发elan创建的流程。在创建elan的过程中，会写入其他的多个operational数据库，不过这些数据库都没有监听器，仅仅作为数据存储所用。
&emsp;&emsp;创建elan-instances/elan-instance时还会更新elan-instances/elan-instance config库
  
  
  
<details>
 <summary>第二步：elan创建</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1311.png?0.08913765804454887)  
</details>
  
  
  
<details>
 <summary>第二步：elan创建续</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1312.png?0.8193403893071831)  
</details>
  
  
  
  
***
<p style="text-align: center;"><Strong>第三步：elan 更新</Strong></p>
  
&emsp;&emsp;当elan-instances/elan-instance config库被更新数据后，开始触发elan更新的流程。
  
  
  
<details>
 <summary>第三步：elan更新</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1313.png?0.15435292325857053)  
</details>
  
<details>
 <summary>第三步：elan 更新续</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1314.png?0.5138934759324512)  
</details>
  
  
###  1.2. update
  
  
&emsp;&emsp;当北向api 更新neutron/network/network后，NeutronNetworkChangeListener不作其他操作.
  
<details>
 <summary>网络更新</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1315.png?0.23528989969317338)  
</details>
  
  
###  1.3. remove
  
***
<p style="text-align: center;"><Strong>第一步：删除网络</Strong></p>
&emsp;&emsp;删除数据库中的elan instance数据后，触发相关的其他流程。
  
<details>
 <summary>网络删除</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1316.png?0.7049653054342104)  
</details>
  
  
***
<p style="text-align: center;"><Strong>第二步：elan删除</Strong></p>
  
  
##  2. vlan网络创建
  
&emsp;&emsp;vlan 网络创建时，会提前创建一个elan interface 到业务口的映射。（odl原生代码中没有）在软件vtep 环境下创建vlan 网络或者硬件vtep环境下创建网络（overlay会将openstack创建的vxlan网络修改为vlan网络）会被触发。
  
<p style="text-align: center;"><Strong>第一步：vlan 网络创建</Strong></p>
  
<details>
 <summary>vlan网络创建</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1317.png?0.5738721806904379)  
</details>
  
&emsp;&emsp;在上图中，有几处会写数据库的操作，因此，会触发其他监听程序。
  
&emsp;&emsp;上图中trunk 口可elan interface的解释如下。其中`IfmConstants.OF_URI_SEPARATOR=":"`；`parentRef = dpnid:业务口`
  
|    端口性质     |                          名称规则                          |             示例             |
| --------------- | ---------------------------------------------------------- | ---------------------------- |
| trunk           | parentRef + IfmConstants.OF_URI_SEPARATOR + "trunk"        | 203100663178074:ens192:trunk |
| elan interfaced | parentRef + IfmConstants.OF_URI_SEPARATOR + segmentationId | 62457717962162:ens192:100    |
  
  
***
<p style="text-align: center;"><Strong>第二步：ietf interface 创建</Strong></p>
  
<details>
 <summary>ietf interface创建</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1318.png?0.5625248955504656)  
</details>
  
***
<p style="text-align: center;"><Strong>第三步：elan interface 创建</Strong></p>
  
  
<details>
 <summary>elan interface创建</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c1319.png?0.5665974674764349)  
</details>
  
***
<p style="text-align: center;"><Strong>第三步续：elan interface 创建</Strong></p>
  
  
<details>
 <summary>elan interface创建续</summary>
  

![](../../../source/assert/image/7a59aa52d998fa84c8c6a345c6e8c13110.png?0.8359787429719177)  
</details>
  
<details>
 <summary>创建vlan 网络 下发流表示例</summary>
  
 ```text
 table=0, priority=4,in_port=3,vlan_tci=0x0000/0x1fff actions=write_metadata:0x90000000001/0xffffff0000000001,goto_table:17
 table=0, priority=10,in_port=3,dl_vlan=10 actions=pop_vlan,write_metadata:0xa0000000001/0xffffff0000000001,goto_table:17
 table=17, priority=10,metadata=0xa0000000000/0xffffff0000000000 actions=load:0xa->NXM_NX_REG1[0..19], load:0x138a->NXM_NX_REG7[0..15],write_metadata:0xa0000a138a000000/0xfffffffffffffffe,goto_table:43
 table=52, priority=5,metadata=0x138a000000/0xffff000001 actions=write_actions(group:210004)
 table=52, priority=5,metadata=0x138a000001/0xffff000001 actions=write_actions(group:210003)
 table=55, priority=10,tun_id=0xa,metadata=0xa0000000000/0xfffff0000000000 actions=drop
 table=55, priority=9,tun_id=0xa actions=load:0xa00->NXM_NX_REG6[],resubmit(,220)
 table=220, priority=10,reg6=0x900,metadata=0x1/0x1 actions=drop
 table=220, priority=10,reg6=0xa00,metadata=0x1/0x1 actions=drop
 table=220, priority=9,reg6=0x900 actions=output:3
 table=220, priority=9,reg6=0xa00 actions=push_vlan:0x8100,set_field:4106->vlan_vid,output:3
 ```
 ```text
 group_id=210003,type=all
 group_id=210004,type=all,bucket=actions=group:210003,bucket=actions=load:0xa00->NXM_NX_REG6[],resubmit(,220)
 ```
  
</details>
  