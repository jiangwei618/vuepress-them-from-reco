#主要功能
&emsp;监听Datastore中由neutron-northbound plugin的transcriber转换后的数据结构，再次转换为vpndervice中的数据结构，并存入datastroe。  

#主要结构
api:  
    yang: neutronvpn
impl:  


类 | 监听库 | 操作库|
---------|----------|---------
NeutronBgpvpnChangeListener|neutron:neutron/bgpvpns|
NeutronFloatingToFixedIpMappingChangeListener|
NeutronHostConfigChangeListener|
NeutronNetworkChangeListener|neutron:neutron/networks|
NeutronPortChangeListener     |
NeutronRouterChangeListener|
NeutronSecurityRuleConstants|  
NeutronSubnetChangeListener|
NeutronSubnetGwMacResolver|
NeutronTrunkChangeListener  |
UpgradeStateListener |
IPV6InternetDefaultRouteProgrammer  |
NeutronvpnManager  |
NeutronvpnManagerImpl  |
NeutronvpnNatManager  |
NeutronvpnUtils  |
NeutronExternalSubnetHandler|

# NeutronNetworkChangeListener
## add 

{% uml %}
```plantuml
@startuml

PARTICIPANT "neutron\n:neutron/networks" as DsNetworks #99FF99

DsNetworks -->  NeutronNetworkChangeListener:数据库监听通知
activate NeutronNetworkChangeListener
NeutronNetworkChangeListener -> NeutronNetworkChangeListener:isNetworkTypeSupported \n && isVlanOrVxlanNetwork\n <color red>no</color>
NeutronNetworkChangeListener -> DsNetworks:game over
deactivate NeutronNetworkChangeListener

== 创建 Elan instance== 

NeutronNetworkChangeListener --> neutronvpnUtils:缓存
NeutronNetworkChangeListener -> NeutronNetworkChangeListener: createElanInstance
activate NeutronNetworkChangeListener
PARTICIPANT "/config\n/elan\n:elan-instances" as DsElan #99FF99

NeutronNetworkChangeListener -> DsElan:<color blue>read datastore</color>
DsElan --> NeutronNetworkChangeListener
alt isExist
NeutronNetworkChangeListener -> NeutronNetworkChangeListener: exist instance
else
NeutronNetworkChangeListener -> DsElan:创建； <color blue>write datastore</color>
end
deactivate NeutronNetworkChangeListener

== 外部网络处理  == 
NeutronNetworkChangeListener -> NeutronNetworkChangeListener: isExternal
NeutronNetworkChangeListener -> DsNetworks:<color red>no</color>

NeutronNetworkChangeListener ->IElanService:createExternalElanNetwork 
note left
Create ELAN interface and IETF interfaces for the physical network
end note

PARTICIPANT "config\n/ietf-interfaces\n:interfaces" as DsIetf #99FF99
IElanService -> IElanService :createIetfInterfaces
IElanService -> DsIetf: <color blue>write datastore; node1</color>

IElanService -> IElanService:addElanInterface
PARTICIPANT "config\n/elan\n:elan-interfaces" as DsElanInterfaces #99FF99
IElanService -> DsElanInterfaces: <color blue>write datastore</color>


IElanService -> NeutronvpnNatManager: addExternalNetwork
PARTICIPANT "/config\n/odl-nat\n:external-networks  " as DsOdlNat #99FF99
NeutronvpnNatManager -> DsOdlNat: <color blue>write network to datastore</color>

alt isFlatOrVlanNetwork
PARTICIPANT "/config\n/l3vpn\n:vpn-instances  " as DsL3vpn #99FF99
IElanService -> NeutronvpnNatManager: createL3InternalVpn
NeutronvpnNatManager -> DsL3vpn: <color blue>write datastore</color>

IElanService -> NeutronvpnNatManager: createExternalVpnInterfaces
activate NeutronvpnNatManager
NeutronvpnNatManager -> DsElanInterfaces:获取 elan interface
DsElanInterfaces -> NeutronvpnNatManager
NeutronvpnNatManager -> DsL3vpn:  <color blue>write datastore</color>
deactivate NeutronvpnNatManager
end

@enduml

```
{% enduml %}

**node1**
```
Create ietf-interfaces based on the ELAN segment type.
    For segment type flat - create transparent interface pointing to thepatch-port attached to the physnet port.<br>
    For segment type vlan - create trunk interface pointing to the patch-portattached to the physnet port + trunk-member interface pointing to thetrunk interface.
```

**node2**
```
使用非集群监听，并通过 ConcurrentMap<Uuid, Network> 缓存数据，若A为master时，创建网络并缓存，一旦集群震荡，B切换为master，此时删除网络，则A中ConcurrentMap会产生数据残留，
不清楚是否会影响业务，但是肯定会造成内存泄漏。
是否可以使用集群监听，所有节点均对缓存进行操作，但是仅仅只有owner处理业务逻辑EntityOwnershipUtils
```

#NeutronSubnetChangeListener
##add
{% uml %}
```plantuml

@startuml

PARTICIPANT "/config/neutron:neutron/subnets " as DsSubnets #99FF99

DsSubnets -->  NeutronSubnetChangeListener:数据库监听通知
activate NeutronSubnetChangeListener
NeutronSubnetChangeListener -> NeutronVpnUtils:getNeutronNetwork
PARTICIPANT "neutron\n:neutron/networks" as DsNetworks #99FF99
NeutronVpnUtils->DsNetworks:read && !=null && \n isNetworkTypeSupported(network)\n <color red>no</color>
NeutronSubnetChangeListener -> DsSubnets:game over
deactivate NeutronSubnetChangeListener

== 创建 subnet==

NeutronSubnetChangeListener --> NeutronVpnUtils:缓存
NeutronSubnetChangeListener -> NeutronSubnetChangeListener: handleNeutronSubnetCreated
NeutronSubnetChangeListener -> NeutronvpnManager:createSubnetmapNode
PARTICIPANT "config/neutronvpn:subnetmaps" as DsSubnetMaps #99FF99

NeutronSubnetChangeListener -> DsSubnetMaps:<color blue>write datastore</color>
NeutronSubnetChangeListener -> NeutronSubnetChangeListener:createSubnetToNetworkMapping
PARTICIPANT "/config/neutronvpn:networkMaps " as DsNetworkMaps #99FF99
NeutronSubnetChangeListener -> DsNetworkMaps:<color blue>write datastore</color>


== 外部网络处理  NeutronvpnUtils.getIsExternal(network) && NeutronvpnUtils.isFlatOrVlanNetwork(network)==
NeutronSubnetChangeListener -> NeutronExternalSubnetHandler:handleExternalSubnetAdded
NeutronExternalSubnetHandler -> NeutronvpnNatManager:updateOrAddExternalSubnet
NeutronvpnNatManager -> NeutronvpnUtils:getOptionalExternalSubnets
PARTICIPANT "config/odl-nat:external-subnets" as DsExternalSubnets #99FF99
NeutronvpnUtils -> DsExternalSubnets:read datastore
DsExternalSubnets --> NeutronVpnUtils
alt optionalExternalSubnets.isPresent()
    NeutronvpnNatManager -> NeutronvpnNatManager: updateExternalSubnet
    NeutronvpnNatManager -> DsExternalSubnets:<color blue>update datastore</color>
else
    NeutronvpnNatManager -> NeutronvpnNatManager:addExternalSubnet
    NeutronvpnNatManager -> DsExternalSubnets:<color blue>write datastore</color>
end
NeutronvpnNatManager -> NeutronExternalSubnetHandler
NeutronExternalSubnetHandler -> NeutronvpnManager:updateSubnetNode
NeutronvpnManager -> DsSubnetMaps:<color blue>update datastore routerId || vpnId </color>

NeutronExternalSubnetHandler -> NeutronvpnManager:createVpnInstanceForSubnet

PARTICIPANT "config/l3vpn:vpn-instances" as DsVpnInstance #99FF99
NeutronvpnManager -> DsVpnInstance:<color blue>write datastore </color>

@enduml

```
{% enduml %}


#NeutronRouterChangeListener
## add 
