# 代码结构

**代码结构**  
目前，合并之后的Netvirt主要以VPNService为基础来进行演进的，下面我们将以carbon为例来介绍VPNService的实现过程。VPNService中的主要模块，如下表所示：  

主要模块 |功能
---------|----------
Neutronvpn |	监听Datastore中由neutron-northbound plugin的transcriber转换后的数据结构，再次转换为vpndervice中的数据结构，并存入datastroe。
dhcpservice0	|控制器代答dhcp
ElanManager|	维护租户网络的二层连通性
VPNManager	|维护租户网络的三层连通性，需要通过bgpMansger操纵控制满，通过fibManager来操作数据平面。
bgpManager	|接受VPNManager的控制，通过Thrift与外部的BGP引擎进行通信
fibManager	|FIB（Forward Information dataBase转发表）接受VPNmansger的控制，向OVS下发流表，转发三层流量。
natService	|支持SNAT和DANT
aclservice	|提供安全组功能
