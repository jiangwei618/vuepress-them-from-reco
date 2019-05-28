
# 日志分析
## 创建网络

**日志**
```
#### 创建网络 overlay netruon 介入 #####
 IceNeutronNetworkChangeListener   IceNeutronNetworkChangeListener data tree changed.

  IceNeutronNetworkChangeListener   WRITE IceNetwork IceNeutronNetworkChangeListener
  
  IceNeutronNetworkChangeListener   Adding Network:Name=network, isAdminStateUp=true, isShared=false,getNetworkType=IceNetworkTypeVxlan, getSegmentationId=10071}, IceNetworkL3Extension{isExternal=false}}}

#### mapper 介入 写入vxlan id ####
 IceNeutronUtils                   getSegmentationId 10071

  IceNeutronNetworkChangeListener   Network segmentationID: value=10071

  IceNeutronNetworkChangeListener   handle vxlan network created with segmentationId 10071

#### 获取server信息  ####
 IceNeutronUtils                   serverLeafSet:[Uri [_value=iceDeviceId:172.20.1.174]]

 IceVxlanManager                   ChannelId is : Uri [_value=iceDeviceId:172.20.1.174] with device : Uri [_value=iceDeviceId:172.20.1.174]

#### 获取vlan id #### 
 IceOverlayPoolsManager           Allocate id: 500 ,poolType :1

#### ICe vxlan介入 #### 
 IceVxlanCacheManager              writeL2Vxlan IceDeviceVxlanL2 iceDeviceId:172.20.1.174, _iceNetworkId=21e67093-9cf3-4172-9943-64fca7be2da5, _iceNetworkType=IceNetworkTypeVxlan, _iceVlanId=500, _iceVxlanId=10071

  IceVxlanL2ChangeListener         IceVxlanL2ChangeListener Write

#### overlay调用underlay下发配置 ####
  DistributeConfig                  distribute vxlanConfigl2Dynamic input VconfigType=Config, _deviceIp=Ipv4Address [_value=172.20.1.174], vlanId=500, _vni=10071, _isMulticastMode=true

  ConfigCache                       init vni : 10071 for device : iceDeviceId:172.20.1.174

  DistributeConfig                  modifyDeviceVniCache true

  EvpnVni                           config segmentId start

  DevInfo                           deviceIp=172.20.1.174, deviceType=CN61108PC-V

#### underlay 第一条命令（主要配置vni） l2Config.configEvpnL2 ####
  HttpPostCmd                       devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;evpn ;vni 10071 L2 ;rd auto ;route-target import auto ;route-target export auto ;exit
  
  EvpnVni                           save segment id of 172.20.1.174 succeeded

  EvpnVni                           configSegmentId command exec succeed, deviceIp is :172.20.1.174

  DevInfo                           deviceIp=172.20.1.174, deviceType=CN61108PC-V

#### underlay 第二条命令（配置vlan-vxlan映射） l2Config.configVlanVnSegmentL2 ####
  HttpPostCmd                       devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;vlan 500 ;no vn-segment ;vn-segment 10071 ;end

  Vlan                              configVlanVnSegment sendCommands return :true

  Vlan                              vlan is not created, so create vlan!

#### overlay mapper收到对device switch 的vlan号修改 l2Config.configNve1L2Multicast ####
  IceLeafSwitchChangeListener       LeafSwitch getDeviceVlanId=1,500, 修改device vlan
  
  IceUnderlayPoolManager           | 44 - com.inspur.ice.resourcepool-impl - 0.1.0 | Allocate ip: 225.0.0.1 ,poolType :9

  DevInfo                           deviceIp=172.20.1.174, deviceType=CN61108PC-V

#### underlay 第三条命令（配置bgp loopback）####
  HttpPostCmd                       devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;interface nve1 ;host-reachability protocol bgp ;shutdown ;source-interface loopback 95 ;no shutdown ;member vni 10071 ;suppress-arp  ;mcast-group 225.0.0.1 ;end

```
**交换机配置**

&emsp;部分配置对不上，是因为不是同一次截取
```
vlan 2
  vn-segment 10070

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback95
  member vni 10070
    suppress-arp
    mcast-group 225.0.0.1

router bgp 65101
evpn
  vni 10070 l2
    rd auto
    route-target import auto
    route-target export auto
```
**命令**
```
devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;evpn ;vni 10071 L2 ;rd auto ;route-target import auto ;route-target export auto ;exit

devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;vlan 500 ;no vn-segment ;vn-segment 10071 ;end

devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;interface nve1 ;host-reachability protocol bgp ;shutdown ;source-interface loopback 95 ;no shutdown ;member vni 10071 ;suppress-arp  ;mcast-group 225.0.0.1 ;end
```

## 创建子网（同时会创建dhcp端口）
**日志分析**
```
 
#### Ice Neutron介入监听到子网创建  ####
IceNeutronSubnetChangeListener      IceNeutronSubnetChangeListener data tree changed.
IceNeutronSubnetChangeListener      WRITE IceSubnet IceNeutronSubnetChangeListener
IceNeutronSubnetChangeListener      Adding Subnet : 
IceNeutronUtils                     getSegmentationId 10071

#### overlay mapper 介入，写 vxlan l2  ####
IceNeutronUtils                     serverLeafSet:[Uri [_value=iceDeviceId:172.20.1.177], Uri [_value=iceDeviceId:172.20.1.174]]
IceVxlanL2ChangeListener            IceVxlanL2ChangeListener 写入subnet信息
ConfigCache                         ADD IceDeviceVxlanL2{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.177], getIceInterfaceConfig=[], getIceNetworkId=Uuid [_value=21e67093-9cf3-4172-9943-64fca7be2da5], getIceSubnet=[IceSubnet{getIceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceVlanId=IceVlanId [_value=500], getIceVxlanId=IceVxlanId [_value=10071], augmentations={}} failed

#### Ice Neutron介入监听dhcp port创建 (多次信息写入)####
IceNeutronPortChangeListener        IceNeutronPortChangeListener data tree changed.
IceNeutronPortChangeListener        IcePort data tree changed.
IceNeutronPortChangeListener        port vtepType null
IceNeutronPortChangeListener        IceNeutronPortChangeListener data tree changed.
IceNeutronPortChangeListener        port vtepType HW
IceNeutronPortChangeListener        Add ice port with hw host
IceVxlanManager                     getHostNodeFromHostName jw-controller
IceVxlanManager                     uncompleted topology 
IceNeutronPortChangeListener        Get portBindingInfo [IceVxlanManager$PortBindingInfo@16b5b00c] with icePort Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9]

#### overlay mapper 介入，写vxlan l2####
IceVlanCacheManager                 writeNeutronSubnet 
IceVlanCacheManager                 writeNeutronPort Port
netvirt.natservice.internal.NatUtil getSubnetGwMac : No GW ip found for subnet 49c27f08-05d1-3f62-a810-91a6f700856a
NeutronPortChangeListener           NeutronPortChangeListener data tree changed.
NeutronPortChangeListener           NeutronPortChangeListener data tree changed write.
IceVxlanCacheManager                writeIceInterfacePort IceInterfaceConfig 

#### overlay config 介入，准备下发配置 ####
IceVxlanL2ChangeListener            IceVxlanL2ChangeListener Write 端口写入
DevInfo                             deviceIp=172.20.1.177, deviceType=CN61108TC-V
SshcProvider                        sshInput is : SshInput [_enablePassword=inspur@123, _sshCommand=[SshCommand [_command=show running-config interface Ethernet1/1, augmentation=[]]], _sshIp=172.20.1.177, _sshPassword=inspur@123, _sshUser=admin, _nxapiUsed=false, augmentation=[]]

#### netvirt 介入####
NeutronvpnManager                   updateSubnetmapNodeWithPorts 
NatUtil                             getSubnetGwMac : No GW ip found for subnet 49c27f08-05d1-3f62-a810-91a6f700856a
VpnSubnetRouteHandler               SUBNETROUTE: onPortAddedToSubnet: Port 85617daf-4a17-49c0-a3a2-9841625aa5c9 being added to subnet 49c27f08-05d1-3f62-a810-91a6f700856a

#### genius 介入####
InterfacemgrProvider                Interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 is not present
ElanInterfaceManager                Interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 is removed from Interface Oper DS due to port down
InterfaceConfigListener             parent refs not specified for 85617daf-4a17-49c0-a3a2-9841625aa5c9

#### 返回netvirt ####
VpnSubnetRouteHandler               SUBNETROUTE


#### underlay  下发配置 ####
SshcProvider
Vlan                            checkPortIsInPortChannel sendCommands return :true
VlanProvider                    setPort2VlanMode start
IceLeafSwitchChangeListener     LeafSwitch  write.
DevInfo                         deviceIp=172.20.1.177, deviceType=CN61108TC-V
HttpPostCmd                     devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;vlan  500 ;interface Ethernet1/1 ;switchport  ;switchport mode trunk ;switchport trunk allowed vlan  500 ;no shutdown
Vlan                            setPort2VlanMode1 sendCommands return :true

#### genius netvirt 介入监听端口状态 ietf-interfaces:interfaces-state ####
statehelpers.OvsInterfaceStateAddHelper Adding Interface State to Oper DS for interface: tap85617daf-4a
netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: add: Processed interface tap85617daf-4a up event


genius.utils.batching.ResourceBatchingManager Total taken ##time = 2ms for resourceList of size 1 for resourceType INTERFACEMGR-DEFAULT-OPERATIONAL
genius.interfacemanager.renderer.ovs.confighelpers.OvsInterfaceConfigUpdateHelper port attributes modified, requires a delete and recreate of 85617daf-4a17-49c0-a3a2-9841625aa5c9 configuration
genius.interfacemanager.commons.InterfaceManagerCommonUtils Creating child interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 of type Trunk bound on parent-interface tap85617daf-4a
genius.interfacemanager.renderer.ovs.confighelpers.OvsInterfaceConfigAddHelper adding vlan configuration for interface 85617daf-4a17-49c0-a3a2-9841625aa5c9

#### genius id manager分配id####
genius.idmanager.IdManager Got pool IdLocalPool [poolName=interfaces.2130706689, availableIds=AvailableIdHolder [low=1, high=6553, cur=3], releasedIds=ReleasedIdHolder [availableIdCount=2, timeDelaySec=30, delayedEntries=[{Id: 3 ReadyTimeSec: 1553930804}, {Id: 1 ReadyTimeSec: 1553930806}]]]
genius.idmanager.IdManager The newIdValues [3] for the idKey 85617daf-4a17-49c0-a3a2-9841625aa5c9

#### genius netvirt 绑定服务 （datastore interface-service-bindings）####
####
    {
        "name": "85617daf-4a17-49c0-a3a2-9841625aa5c9",
        "type": "iana-if-type:l2vlan",
        "enabled": true,
        "odl-interface:l2vlan-mode": "trunk",
        "odl-interface:parent-interface": "tap85617daf-4a"
      }
####

genius.interfacemanager.IfmUtil Binding Service default.85617daf-4a17-49c0-a3a2-9841625aa5c9 for : 85617daf-4a17-49c0-a3a2-9841625aa5c9
netvirt.vpnmanager.InterfaceStateChangeListener VPN Interface add event - intfName 85617daf-4a17-49c0-a3a2-9841625aa5c9 from InterfaceStateChangeListener
netvirt.natservice.internal.NatInterfaceStateChangeListener call : Unable to process add for interface 85617daf-4a17-49c0-a3a2-9841625aa5c9
netvirt.vpnmanager.InterfaceStateChangeListener Detected interface add event for interface 85617daf-4a17-49c0-a3a2-9841625aa5c9
genius.interfacemanager.servicebindings.flowbased.listeners.FlowBasedServicesConfigListener Service Binding Entry created for Interface: 85617daf-4a17-49c0-a3a2-9841625aa5c9, ServiceName: default.85617daf-4a17-49c0-a3a2-9841625aa5c9, ServicePriority 9
genius.interfacemanager.servicebindings.flowbased.utilities.FlowBasedServicesUtils adding bound-service state information for interface : 85617daf-4a17-49c0-a3a2-9841625aa5c9, service-mode : yang.gen.v1.urn.opendaylight.genius.interfacemanager.servicebinding.rev160406.ServiceModeEgress
netvirt.ipv6service.Ipv6ServiceInterfaceEventListener Port Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9] does not include IPv6Address, skipping.

#### genius netvirt 收到 85617daf-4a17-49c0-a3a2-9841625aa5c9 端口状态改变事件####
netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: add: Processed interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 up event
netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: add: Received port UP event for interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 subnetId 49c27f08-05d1-3f62-a810-91a6f700856a

genius.interfacemanager.servicebindings.flowbased.config.helpers.FlowBasedEgressServicesConfigBindHelper binding egress service default.85617daf-4a17-49c0-a3a2-9841625aa5c9 for interface: 85617daf-4a17-49c0-a3a2-9841625aa5c9
genius.idmanager.jobs.UpdateIdEntryJob Updated id entry with idValues [3], idKey 85617daf-4a17-49c0-a3a2-9841625aa5c9, pool interfaces.2130706689

#### DpnInterfaceListener 监听elan ，保证路由通告消息进入####
DpnInterfaceListener DpnInterfaceListener data tree changed.
DpnInterfaceListener begin processFlowTableEgressAclDel
DpnInterfaceListener data is null or dpninterface is null!!
genius.interfacemanager.InterfacemgrProvider Create VLAN interface : 115377256237675:ens192:500
DpnInterfaceListener begin processFlowTableEgressAclAdd!!!
DpnInterfaceListener begin for ifname is 85617daf-4a17-49c0-a3a2-9841625aa5c9!!!
DpnInterfaceListener begin getIcePortByIfName 85617daf-4a17-49c0-a3a2-9841625aa5c9
com.inspur.ice.overlaymapper.util.Utils begin getIcePortByIfName 85617daf-4a17-49c0-a3a2-9841625aa5c9
com.inspur.ice.overlaymapper.util.Utils begin read ds 85617daf-4a17-49c0-a3a2-9841625aa5c9
DpnInterfaceListener begin check is nova port
genius.interfacemanager.InterfacemgrProvider Interface 115377256237675:ens192:500 is not present
netvirt.elan.internal.ElanInterfaceManager Interface 115377256237675:ens192:500 is removed from Interface Oper DS due to port down
genius.interfacemanager.renderer.ovs.confighelpers.OvsVlanMemberConfigAddHelper adding vlan member configuration for interface 115377256237675:ens192:500

#### genius id manager分配id(上面获取不到后面才创建？)####
genius.idmanager.IdManager Got pool IdLocalPool [poolName=interfaces.2130706689, availableIds=AvailableIdHolder [low=1, high=6553, cur=3], releasedIds=ReleasedIdHolder [availableIdCount=1, timeDelaySec=30, delayedEntries=[{Id: 1 ReadyTimeSec: 1553930806}]]]
genius.idmanager.IdManager The newIdValues [1] for the idKey 115377256237675:ens192:500
genius.idmanager.jobs.UpdateIdEntryJob Updated id entry with idValues [1], idKey 115377256237675:ens192:500, pool interfaces.2130706689
genius.interfacemanager.IfmUtil Binding Service default.115377256237675:ens192:500 for : 115377256237675:ens192:500
netvirt.vpnmanager.InterfaceStateChangeListener VPN Interface add event - intfName 115377256237675:ens192:500 from InterfaceStateChangeListener
netvirt.vpnmanager.InterfaceStateChangeListener Detected interface add event for interface 115377256237675:ens192:500
netvirt.natservice.internal.NatInterfaceStateChangeListener call : Unable to process add for interface 115377256237675:ens192:500

#### 发现服务被绑定，下发流表 datastore interface-service-bindings####
genius.interfacemanager.servicebindings.flowbased.listeners.FlowBasedServicesConfigListener Service Binding Entry created for Interface: 115377256237675:ens192:500, ServiceName: default.115377256237675:ens192:500, ServicePriority 9
genius.interfacemanager.servicebindings.flowbased.utilities.FlowBasedServicesUtils adding bound-service state information for interface : 115377256237675:ens192:500, service-mode : yang.gen.v1.urn.opendaylight.genius.interfacemanager.servicebinding.rev160406.ServiceModeEgress
genius.interfacemanager.servicebindings.flowbased.config.helpers.FlowBasedEgressServicesConfigBindHelper binding egress service default.115377256237675:ens192:500 for interface: 115377256237675:ens192:500
genius.utils.batching.ResourceBatchingManager Total taken ##time = 2ms for resourceList of size 4 for resourceType INTERFACEMGR-DEFAULT-OPERATIONAL
netvirt.elan.internal.InterfaceAddWorkerOnElanInterface Handling elan interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 add for elan 476baec5-fd39-349d-a84e-57c1d2221ec5
DpnInterfaceListener DpnInterfaceListener data tree changed.
DpnInterfaceListener begin processFlowTableEgressAclDel
DpnInterfaceListener begin get EgressAclFlow 85617daf-4a17-49c0-a3a2-9841625aa5c9
DpnInterfaceListener begin get EgressAclFlowoption 85617daf-4a17-49c0-a3a2-9841625aa5c9
DpnInterfaceListener processFlowTableEgressAclDel EgressAclFlow is null!! ifname is 85617daf-4a17-49c0-a3a2-9841625aa5c9
DpnInterfaceListener begin processFlowTableEgressAclAdd!!!
DpnInterfaceListener begin for ifname is 85617daf-4a17-49c0-a3a2-9841625aa5c9!!!
DpnInterfaceListener begin getIcePortByIfName 85617daf-4a17-49c0-a3a2-9841625aa5c9
com.inspur.ice.overlaymapper.util.Utils begin getIcePortByIfName 85617daf-4a17-49c0-a3a2-9841625aa5c9
com.inspur.ice.overlaymapper.util.Utils begin read ds 85617daf-4a17-49c0-a3a2-9841625aa5c9
DpnInterfaceListener begin check is nova port
DpnInterfaceListener begin for ifname is 115377256237675:ens192:500!!!
DpnInterfaceListener ifname contain dpid , don't need process 115377256237675:ens192:500

####elan，/operational/elan:elan-forwarding-tables  ####

netvirt.elan.internal.ElanInterfaceManager Programming remote dmac flows on the newly connected dpn 115377256237675 for elan 476baec5-fd39-349d-a84e-57c1d2221ec5
netvirt.elan.internal.ElanInterfaceManager programming smac and dmacs for fa:16:3e:d8:2c:da on source and other DPNs for elan 476baec5-fd39-349d-a84e-57c1d2221ec5 and interface 85617daf-4a17-49c0-a3a2-9841625aa5c9
netvirt.elan.evpn.listeners.ElanMacEntryListener ElanMacEntryListener : ADD macEntry KeyedInstanceIdentifier{targetType=interface yang.gen.v1.urn.opendaylight.netvirt.elan.rev150602.forwarding.entries.MacEntry, path=[yang.gen.v1.urn.opendaylight.netvirt.elan.rev150602.ElanForwardingTables, yang.gen.v1.urn.opendaylight.netvirt.elan.rev150602.elan.forwarding.tables.MacTable[key=MacTableKey [_elanInstanceName=476baec5-fd39-349d-a84e-57c1d2221ec5]], yang.gen.v1.urn.opendaylight.netvirt.elan.rev150602.forwarding.entries.MacEntry[key=MacEntryKey [_macAddress=PhysAddress [_value=fa:16:3e:d8:2c:da]]]]}
genius.interfacemanager.servicebindings.flowbased.listeners.FlowBasedServicesConfigListener Service Binding Entry created for Interface: 85617daf-4a17-49c0-a3a2-9841625aa5c9, ServiceName: elan.476baec5-fd39-349d-a84e-57c1d2221ec5.85617daf-4a17-49c0-a3a2-9841625aa5c9, ServicePriority 9
genius.interfacemanager.servicebindings.flowbased.utilities.FlowBasedServicesUtils adding bound-service state information for interface : 85617daf-4a17-49c0-a3a2-9841625aa5c9, service-mode : yang.gen.v1.urn.opendaylight.genius.interfacemanager.servicebinding.rev160406.ServiceModeIngress
genius.interfacemanager.servicebindings.flowbased.config.helpers.FlowBasedIngressServicesConfigBindHelper binding ingress service elan.476baec5-fd39-349d-a84e-57c1d2221ec5.85617daf-4a17-49c0-a3a2-9841625aa5c9 for vlan port: 85617daf-4a17-49c0-a3a2-9841625aa5c9

#### genius netvirt 收到odl-interface:parent-interface": "tap85617daf-4a 端口状态改变事件####

netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: update: Processed Interface tap85617daf-4a update event
netvirt.vpnmanager.InterfaceStateChangeListener VPN Interface update event - intfName 85617daf-4a17-49c0-a3a2-9841625aa5c9 from InterfaceStateChangeListener
netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: update: Processed Interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 update event
netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: update: Received port UP event for interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 in subnet 49c27f08-05d1-3f62-a810-91a6f700856a
netvirt.elan.internal.InterfaceAddWorkerOnElanInterface Handling elan interface 115377256237675:ens192:500 add for elan 476baec5-fd39-349d-a84e-57c1d2221ec5
genius.interfacemanager.servicebindings.flowbased.listeners.FlowBasedServicesConfigListener Service Binding Entry created for Interface: 115377256237675:ens192:500, ServiceName: elan.476baec5-fd39-349d-a84e-57c1d2221ec5.115377256237675:ens192:500, ServicePriority 9
genius.interfacemanager.servicebindings.flowbased.utilities.FlowBasedServicesUtils adding bound-service state information for interface : 115377256237675:ens192:500, service-mode : yang.gen.v1.urn.opendaylight.genius.interfacemanager.servicebinding.rev160406.ServiceModeIngress
genius.interfacemanager.servicebindings.flowbased.config.helpers.FlowBasedIngressServicesConfigBindHelper binding ingress service elan.476baec5-fd39-349d-a84e-57c1d2221ec5.115377256237675:ens192:500 for vlan port: 115377256237675:ens192:500
netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: update: Processed Interface tap85617daf-4a update event
netvirt.vpnmanager.InterfaceStateChangeListener VPN Interface update event - intfName 85617daf-4a17-49c0-a3a2-9841625aa5c9 from InterfaceStateChangeListener
netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: update: Processed Interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 update event
netvirt.vpnmanager.SubnetRouteInterfaceStateChangeListener SUBNETROUTE: update: Received port UP event for interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 in subnet 49c27f08-05d1-3f62-a810-91a6f700856a
IceNeutronPortChangeListener IceNeutronPortChangeListener data tree changed.
IceNeutronPortChangeListener IcePort data tree changed.
IceNeutronPortChangeListener WRITE
IceNeutronPortChangeListener Updating Port : original = IcePortKey [_uuid=Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9]] update = IcePortKey [_uuid=Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9]]
IceVlanCacheManager writeNeutronPort Port [_allowedAddressPairs=[], _deviceId=dhcp639d7107-1693-5abe-bb94-18ba198147ec-21e67093-9cf3-4172-9943-64fca7be2da5, _deviceOwner=network:dhcp, _extraDhcpOpts=[], _fixedIps=[FixedIps [_ipAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.2]], _key=FixedIpsKey [_subnetId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], _ipAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.2]]], _subnetId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], augmentation=[]]], _key=PortKey [_uuid=Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9]], _macAddress=MacAddress [_value=fa:16:3e:d8:2c:da], _name=, _networkId=Uuid [_value=476baec5-fd39-349d-a84e-57c1d2221ec5], _revisionNumber=7, _securityGroups=[], _tenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], _uuid=Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9], _adminStateUp=true, augmentation=[PortBindingExtension [_hostId=jw-controller, _vifDetails=[VifDetails [_detailsKey=has_datapath_type_netdev, _key=VifDetailsKey [_detailsKey=has_datapath_type_netdev], _value=false, augmentation=[]], VifDetails [_detailsKey=uuid, _key=VifDetailsKey [_detailsKey=uuid], _value=dc328b61-430d-4d06-a432-a9e3afc9660a, augmentation=[]], VifDetails [_detailsKey=support_vhost_user, _key=VifDetailsKey [_detailsKey=support_vhost_user], _value=false, augmentation=[]]], _vifType=ovs, _vnicType=normal], PortSecurityExtension [_portSecurityEnabled=false]]]
IceVlanCacheManager vlanPortMap ：ADD Port [_allowedAddressPairs=[], _deviceId=dhcp639d7107-1693-5abe-bb94-18ba198147ec-21e67093-9cf3-4172-9943-64fca7be2da5, _deviceOwner=network:dhcp, _extraDhcpOpts=[], _fixedIps=[FixedIps [_ipAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.2]], _key=FixedIpsKey [_subnetId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], _ipAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.2]]], _subnetId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], augmentation=[]]], _key=PortKey [_uuid=Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9]], _macAddress=MacAddress [_value=fa:16:3e:d8:2c:da], _name=, _networkId=Uuid [_value=476baec5-fd39-349d-a84e-57c1d2221ec5], _revisionNumber=7, _securityGroups=[], _tenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], _uuid=Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9], _adminStateUp=true, augmentation=[PortBindingExtension [_hostId=jw-controller, _vifDetails=[VifDetails [_detailsKey=has_datapath_type_netdev, _key=VifDetailsKey [_detailsKey=has_datapath_type_netdev], _value=false, augmentation=[]], VifDetails [_detailsKey=uuid, _key=VifDetailsKey [_detailsKey=uuid], _value=dc328b61-430d-4d06-a432-a9e3afc9660a, augmentation=[]], VifDetails [_detailsKey=support_vhost_user, _key=VifDetailsKey [_detailsKey=support_vhost_user], _value=false, augmentation=[]]], _vifType=ovs, _vnicType=normal], PortSecurityExtension [_portSecurityEnabled=false]]] failed
NeutronPortChangeListener NeutronPortChangeListener data tree changed.
NeutronPortChangeListener NeutronPortChangeListener data tree changed write.
netvirt.neutronvpn.NeutronPortChangeListener Update port 85617daf-4a17-49c0-a3a2-9841625aa5c9 from network Uuid [_value=476baec5-fd39-349d-a84e-57c1d2221ec5]
netvirt.neutronvpn.NeutronPortChangeListener Update port 85617daf-4a17-49c0-a3a2-9841625aa5c9 from network Uuid [_value=476baec5-fd39-349d-a84e-57c1d2221ec5]
IceNeutronPortChangeListener IceNeutronPortChangeListener data tree changed.
IceNeutronPortChangeListener IcePort data tree changed.
before IcePort{getDeviceId=dhcp639d7107-1693-5abe-bb94-18ba198147ec-21e67093-9cf3-4172-9943-64fca7be2da5, getDeviceOwner=network:dhcp, getIceAllowedAddressPairs=[], getIceExtraDhcpOpts=[], getIceFixedIps=[IceFixedIps{getIpAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.2]], getSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceSecurityGroups=[], getMacAddress=MacAddress [_value=fa:16:3e:d8:2c:da], getName=, getNetworkId=Uuid [_value=21e67093-9cf3-4172-9943-64fca7be2da5], getRevisionNumber=7, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], getUuid=Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9], isAdminStateUp=true, augmentations={interface yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.binding.rev180302.IcePortBindingExtension=IcePortBindingExtension{getHostId=jw-controller, getIceVifDetails=[IceVifDetails{getDetailsKey=has_datapath_type_netdev, getValue=false, augmentations={}}, IceVifDetails{getDetailsKey=uuid, getValue=dc328b61-430d-4d06-a432-a9e3afc9660a, augmentations={}}, IceVifDetails{getDetailsKey=support_vhost_user, getValue=false, augmentations={}}], getVifType=ovs, getVnicType=normal}, interface yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.portsecurity.rev150712.IcePortSecurityExtension=IcePortSecurityExtension{isPortSecurityEnabled=false}}}
 after IcePort{getDeviceId=dhcp639d7107-1693-5abe-bb94-18ba198147ec-21e67093-9cf3-4172-9943-64fca7be2da5, getDeviceOwner=network:dhcp, getIceAllowedAddressPairs=[], getIceExtraDhcpOpts=[], getIceFixedIps=[IceFixedIps{getIpAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.2]], getSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceSecurityGroups=[], getMacAddress=MacAddress [_value=fa:16:3e:d8:2c:da], getName=, getNetworkId=Uuid [_value=21e67093-9cf3-4172-9943-64fca7be2da5], getRevisionNumber=8, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], getUuid=Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9], isAdminStateUp=true, augmentations={interface yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.binding.rev180302.IcePortBindingExtension=IcePortBindingExtension{getHostId=jw-controller, getIceVifDetails=[IceVifDetails{getDetailsKey=has_datapath_type_netdev, getValue=false, augmentations={}}, IceVifDetails{getDetailsKey=uuid, getValue=dc328b61-430d-4d06-a432-a9e3afc9660a, augmentations={}}, IceVifDetails{getDetailsKey=support_vhost_user, getValue=false, augmentations={}}], getVifType=ovs, getVnicType=normal}, interface yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.portsecurity.rev150712.IcePortSecurityExtension=IcePortSecurityExtension{isPortSecurityEnabled=false}}}

```
**交换机配置**

```
interface Ethernet1/1
  switchport mode trunk
  switchport trunk allowed vlan 2
```
**命令**

```
devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;vlan  2 ;interface Ethernet1/1 ;switchport  ;switchport mode trunk ;switchport trunk allowed vlan  2 ;no shutdown

```

## 创建路由
**日志**
```
#### neutron 监听到路由器创建 ####
IceNeutronRouterChangeListener IceNeutronRouterChangeListener data tree changed.
IceNeutronRouterChangeListener WRITE IceRouter IceNeutronRouterChangeListener
IceNeutronRouterChangeListener Adding IceRouter : value=IceRouter{getIceRoutes=[], getName=router, getProjectId=baec961077ea4e67933a8c140ca444d1, getRevisionNumber=1, getStatus=ACTIVE, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], getUuid=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], isAdminStateUp=true, isDistributed=false, augmentations={}}

#### 分配vlan vxlan id ####
com.inspur.ice.impl.overlay.IceOverlayPoolsManager Allocate id: 11500 ,poolType :4
com.inspur.ice.impl.overlay.IceOverlayPoolsManager Allocate id: 2500 ,poolType :2

#### overlay mapper 介入，写vxlan L3 ####
IceNeutronUtils serverLeafSet:[Uri [_value=iceDeviceId:172.20.1.177], Uri [_value=iceDeviceId:172.20.1.174]]
IceVxlanCacheManager writeL3Vxlan IceDeviceVxlanL3 [_iceDeviceId=Uri [_value=iceDeviceId:172.20.1.177], _iceExtGateways=[], _iceRouterSubnet=[], _iceVlanId=IceVlanId [_value=2500], _iceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], _iceVpnType=Router, _iceVrfRoute=[], _iceVxlanId=IceVxlanId [_value=11500], _key=IceDeviceVxlanL3Key [_iceDeviceId=Uri [_value=iceDeviceId:172.20.1.177], _iceVxlanId=IceVxlanId [_value=11500]], _distributed=true, augmentation=[]] create true
IceVxlanCacheManager writeL3Vxlan IceDeviceVxlanL3 [_iceDeviceId=Uri [_value=iceDeviceId:172.20.1.174], _iceExtGateways=[], _iceRouterSubnet=[], _iceVlanId=IceVlanId [_value=2500], _iceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], _iceVpnType=Router, _iceVrfRoute=[], _iceVxlanId=IceVxlanId [_value=11500], _key=IceDeviceVxlanL3Key [_iceDeviceId=Uri [_value=iceDeviceId:172.20.1.174], _iceVxlanId=IceVxlanId [_value=11500]], _distributed=true, augmentation=[]] create true

#### overlay config 介入，监听到vxlan L3变化，准备下发配置 ####
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener IceVxlanL3ChangeListener Write - before : null after : IceDeviceVxlanL3{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.177], getIceExtGateways=[], getIceRouterSubnet=[], getIceVlanId=IceVlanId [_value=2500], getIceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], getIceVpnType=Router, getIceVrfRoute=[], getIceVxlanId=IceVxlanId [_value=11500], isDistributed=true, augmentations={}}
com.inspur.ice.overlayconfig.impl.DistributeConfig distribute l3 vxlan config input VxlanConfigl3Input [_configType=Config, _deviceIp=Ipv4Address [_value=172.20.1.177], _routerId=de55a13c-2352-4dd6-b418-3cec4b149890, _vlanId=2500, _vni=VniRange [_value=11500], _vrfContext=de55a13c-23:11500, augmentation=[]]
com.inspur.ice.overlayconfig.impl.ConfigCache init vrf : 11500 for device : iceDeviceId:172.20.1.177
com.inspur.ice.impl.common.DevInfo deviceIp=172.20.1.177, deviceType=CN61108TC-V

#### underlay 介入 ，下发第一条配置 ####
com.inspur.ice.impl.HttpPostCmd devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;vlan 2500 ;no vn-segment ;vn-segment 11500 ;end
com.inspur.ice.impl.HttpPostCmd nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "body": "Warning: Enable double-wide arp-ether tcam carving if igmp snooping/Hsrp over vxlan is enabled. Ignore if tcam carving is already configured.\n", "code": "200", "msg": "Success" }] } }}
com.inspur.ice.impl.devices.config.vlan.Vlan configVlanVnSegment sendCommands return :true
com.inspur.ice.impl.devices.config.vlan.Vlan vlan is not created, so create vlan!
IceLeafSwitchChangeListener LeafSwitch before:Switch{getDevDesc=ICNT NOS-CN(tm) inos, Software (inos-cn), Version 9.2(2), RELEASE SOFTWARE Copyright (c) 2016-2018, Used subject to license from copyright owner. Compiled 11/8/2018 7:00:00, getDevId=Uri [_value=iceDeviceId:172.20.1.177], getDevIpAdd=Ipv4Address [_value=172.20.1.177], getDevLocation=, getDevManu=Inspur, getDevName=server177, getDevType=CN61108TC-V, getDeviceRole=ServerLeaf, getDeviceVlanId=1,500, getEnablePassword=inspur@123, getEvpnTemp=1, getMacAddress=00:00:00:00, getOnlineStatus=onLine, getPointX=96, getPointY=45, getSnmpRead=public, getSnmpVersion=v2c, getSnmpWrite=public, getSshPassword=inspur@123, getSshUserName=admin, getSyncStatus=sync, isHasFex=false, augmentations={}} after:Switch{getDevDesc=ICNT NOS-CN(tm) inos, Software (inos-cn), Version 9.2(2), RELEASE SOFTWARE Copyright (c) 2016-2018, Used subject to license from copyright owner. Compiled 11/8/2018 7:00:00, getDevId=Uri [_value=iceDeviceId:172.20.1.177], getDevIpAdd=Ipv4Address [_value=172.20.1.177], getDevLocation=, getDevManu=Inspur, getDevName=server177, getDevType=CN61108TC-V, getDeviceRole=ServerLeaf, getDeviceVlanId=1,500,2500, getEnablePassword=inspur@123, getEvpnTemp=1, getMacAddress=00:00:00:00, getOnlineStatus=onLine, getPointX=96, getPointY=45, getSnmpRead=public, getSnmpVersion=v2c, getSnmpWrite=public, getSshPassword=inspur@123, getSshUserName=admin, getSyncStatus=sync, isHasFex=false, augmentations={}} data tree changed write.
com.inspur.ice.impl.devices.config.vxlan.VxlanManager config nve1L3Multicast result=true
com.inspur.ice.impl.devices.config.vxlan.VrfContext config vrf start
com.inspur.ice.impl.common.DevInfo deviceIp=172.20.1.177, deviceType=CN61108TC-V

#### underlay 介入 ，下发第二条配置 ####
com.inspur.ice.impl.HttpPostCmd devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;vrf context de55a13c-23:11500 ;vni 11500 ;rd auto ;address-family ipv4 unicast ;route-target both auto ;route-target both auto evpn ;address-family ipv6 unicast ;route-target both auto ;route-target both auto evpn ;end
com.inspur.ice.impl.HttpPostCmd nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }] } }}

#### netvirt 蒋婷到router创建 ####
org.opendaylight.netvirt.ipv6service.NeutronRouterChangeListener Add Router notification handler is invoked Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890].
org.opendaylight.netvirt.ipv6service.IfMgr No unprocessed interfaces for the router Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890]
com.inspur.ice.impl.devices.config.vxlan.VrfContext Succeed to mergeVrfRouteTargetList, routeTargetList is :RouteTargetList [_key=RouteTargetListKey [_routeTarget=Both, _routeTargetSourceNN=, _routeTargetDestNN=], _routeTarget=Both, _routeTargetDestNN=, _routeTargetSourceNN=, augmentation=[]]
com.inspur.ice.impl.devices.config.vxlan.VxlanManager config VrfContextL3Multicast result=true
com.inspur.ice.impl.common.DevInfo deviceIp=172.20.1.177, deviceType=CN61108TC-V

#### genius 介入 ，分配id ####
org.opendaylight.genius.idmanager.IdManager Got pool IdLocalPool [poolName=vpnservices.2130706689, availableIds=AvailableIdHolder [low=100000, high=102999, cur=99999], releasedIds=ReleasedIdHolder [availableIdCount=0, timeDelaySec=30, delayedEntries=[]]]
org.opendaylight.genius.idmanager.IdManager The newIdValues [100000] for the idKey de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.netvirt.vpnmanager.VpnInstanceListener VPN-ADD: addVpnInstance: VPN Id 100000 generated for VpnInstanceName de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.genius.idmanager.jobs.UpdateIdEntryJob Updated id entry with idValues [100000], idKey de55a13c-2352-4dd6-b418-3cec4b149890, pool vpnservices.2130706689
org.opendaylight.netvirt.vpnmanager.VpnInstanceListener VPN-ADD: addVpnInstance: VpnInstanceOpData populated successfully for vpn de55a13c-2352-4dd6-b418-3cec4b149890 rd de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager processSavedInterfaces: No interfaces in queue for VPN de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.netvirt.vpnmanager.VpnInstanceListener$PostAddVpnInstanceWorker VPN-ADD: onSuccess: Vpn Instance Op Data addition for de55a13c-2352-4dd6-b418-3cec4b149890 successful.

#### underlay 介入 ，下发第三条配置 ####
com.inspur.ice.impl.HttpPostCmd devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;feature interface-vlan ;vlan 2500 ;interface vlan 2500 ;no shutdown ;vrf member de55a13c-23:11500 ;ip forward
com.inspur.ice.impl.HttpPostCmd nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "body": "Warning: Deleted all L3 config on interface Vlan2500\n", "code": "200", "msg": "Success" }, { "code": "200", "msg": "Success", "body": { } }] } }}
com.inspur.ice.impl.devices.config.vlan.Vlan interfaceVlanVrf sendCommands return :true
com.inspur.ice.impl.devices.config.vlan.Vlan vlan 2500 is created in device 172.20.1.177, do nothing!
com.inspur.ice.impl.devices.config.vxlan.VxlanManager config InterfaceVlanMulticast result=true
com.inspur.ice.impl.common.DevInfo deviceIp=172.20.1.177, deviceType=CN61108TC-V
com.inspur.ice.impl.HttpPostCmd devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;interface nve1 ;member vni 11500 associate-vrf ;end
com.inspur.ice.impl.HttpPostCmd nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }] } }}
com.inspur.ice.impl.devices.config.vxlan.Nve1Config save nve1 id of 172.20.1.177 succeeded
com.inspur.ice.impl.devices.config.vxlan.VxlanManager config Nve1L3Multicast result=true
com.inspur.ice.impl.devices.config.vxlan.L3Config successed to mergeRouterIdWithVrfContext, deviceIp is :Ipv4Address [_value=172.20.1.177]
com.inspur.ice.overlayconfig.impl.DistributeConfig distribute vrfRoutingInstanceConfig input VrfRoutingInstanceConfigInput [_configType=Config, _deviceIp=Ipv4Address [_value=172.20.1.177], _vrfContext=de55a13c-23:11500, _notCheckDeviceRoute=false, augmentation=[]]
com.inspur.ice.impl.devices.config.bgp.BorderLeafConfig device do not config router bgp, deviceIp is :172.20.1.177
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener ADD VrfRoute
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener ADD RouteGateway
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener ADD RouteTable
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener IceVxlanL3ChangeListener Write - before : null after : IceDeviceVxlanL3{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.174], getIceExtGateways=[], getIceRouterSubnet=[], getIceVlanId=IceVlanId [_value=2500], getIceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], getIceVpnType=Router, getIceVrfRoute=[], getIceVxlanId=IceVxlanId [_value=11500], isDistributed=true, augmentations={}}
com.inspur.ice.overlayconfig.impl.DistributeConfig distribute l3 vxlan config input VxlanConfigl3Input [_configType=Config, _deviceIp=Ipv4Address [_value=172.20.1.174], _routerId=de55a13c-2352-4dd6-b418-3cec4b149890, _vlanId=2500, _vni=VniRange [_value=11500], _vrfContext=de55a13c-23:11500, augmentation=[]]
com.inspur.ice.overlayconfig.impl.ConfigCache init vrf : 11500 for device : iceDeviceId:172.20.1.174
com.inspur.ice.impl.common.DevInfo deviceIp=172.20.1.174, deviceType=CN61108PC-V
com.inspur.ice.impl.HttpPostCmd devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;vlan 2500 ;no vn-segment ;vn-segment 11500 ;end
com.inspur.ice.impl.HttpPostCmd nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "body": "Warning: Enable double-wide arp-ether tcam carving if igmp snooping/Hsrp over vxlan is enabled. Ignore if tcam carving is already configured.\n", "code": "200", "msg": "Success" }] } }}
com.inspur.ice.impl.devices.config.vlan.Vlan configVlanVnSegment sendCommands return :true
com.inspur.ice.impl.devices.config.vlan.Vlan vlan is not created, so create vlan!
IceLeafSwitchChangeListener LeafSwitch before:Switch{getDevDesc=ICNT NOS-CN(tm) inos, Software (inos-cn), Version 9.2(2), RELEASE SOFTWARE Copyright (c) 2016-2018, Used subject to license from copyright owner. Compiled 11/8/2018 7:00:00, getDevId=Uri [_value=iceDeviceId:172.20.1.174], getDevIpAdd=Ipv4Address [_value=172.20.1.174], getDevLocation=, getDevManu=Inspur, getDevName=server174, getDevType=CN61108PC-V, getDeviceRole=ServerLeaf, getDeviceVlanId=1,500-501, getEnablePassword=inspur@123, getEvpnTemp=1, getMacAddress=00:00:00:00, getOnlineStatus=onLine, getPointX=255, getPointY=246, getSnmpRead=public, getSnmpVersion=v2c, getSnmpWrite=public, getSshPassword=inspur@123, getSshUserName=admin, getSyncStatus=sync, isHasFex=false, augmentations={}} after:Switch{getDevDesc=ICNT NOS-CN(tm) inos, Software (inos-cn), Version 9.2(2), RELEASE SOFTWARE Copyright (c) 2016-2018, Used subject to license from copyright owner. Compiled 11/8/2018 7:00:00, getDevId=Uri [_value=iceDeviceId:172.20.1.174], getDevIpAdd=Ipv4Address [_value=172.20.1.174], getDevLocation=, getDevManu=Inspur, getDevName=server174, getDevType=CN61108PC-V, getDeviceRole=ServerLeaf, getDeviceVlanId=1,500-501,2500, getEnablePassword=inspur@123, getEvpnTemp=1, getMacAddress=00:00:00:00, getOnlineStatus=onLine, getPointX=255, getPointY=246, getSnmpRead=public, getSnmpVersion=v2c, getSnmpWrite=public, getSshPassword=inspur@123, getSshUserName=admin, getSyncStatus=sync, isHasFex=false, augmentations={}} data tree changed write.
com.inspur.ice.impl.devices.config.vxlan.VxlanManager config nve1L3Multicast result=true
com.inspur.ice.impl.devices.config.vxlan.VrfContext config vrf start
com.inspur.ice.impl.common.DevInfo deviceIp=172.20.1.174, deviceType=CN61108PC-V
com.inspur.ice.impl.HttpPostCmd devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;vrf context de55a13c-23:11500 ;vni 11500 ;rd auto ;address-family ipv4 unicast ;route-target both auto ;route-target both auto evpn ;address-family ipv6 unicast ;route-target both auto ;route-target both auto evpn ;end
com.inspur.ice.impl.HttpPostCmd nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }] } }}
com.inspur.ice.impl.devices.config.vxlan.VrfContext Succeed to mergeVrfRouteTargetList, routeTargetList is :RouteTargetList [_key=RouteTargetListKey [_routeTarget=Both, _routeTargetSourceNN=, _routeTargetDestNN=], _routeTarget=Both, _routeTargetDestNN=, _routeTargetSourceNN=, augmentation=[]]
com.inspur.ice.impl.devices.config.vxlan.VxlanManager config VrfContextL3Multicast result=true
com.inspur.ice.impl.common.DevInfo deviceIp=172.20.1.174, deviceType=CN61108PC-V
com.inspur.ice.impl.HttpPostCmd devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;feature interface-vlan ;vlan 2500 ;interface vlan 2500 ;no shutdown ;vrf member de55a13c-23:11500 ;ip forward
com.inspur.ice.impl.HttpPostCmd nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "body": "Warning: Deleted all L3 config on interface Vlan2500\n", "code": "200", "msg": "Success" }, { "code": "200", "msg": "Success", "body": { } }] } }}
com.inspur.ice.impl.devices.config.vlan.Vlan interfaceVlanVrf sendCommands return :true
com.inspur.ice.impl.devices.config.vlan.Vlan vlan 2500 is created in device 172.20.1.174, do nothing!
com.inspur.ice.impl.devices.config.vxlan.VxlanManager config InterfaceVlanMulticast result=true
com.inspur.ice.impl.common.DevInfo deviceIp=172.20.1.174, deviceType=CN61108PC-V
com.inspur.ice.impl.HttpPostCmd devIpAdd is :172.20.1.174 and nxapi commands: config terminal ;interface nve1 ;member vni 11500 associate-vrf ;end
com.inspur.ice.impl.HttpPostCmd nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }] } }}
com.inspur.ice.impl.devices.config.vxlan.Nve1Config save nve1 id of 172.20.1.174 succeeded
com.inspur.ice.impl.devices.config.vxlan.VxlanManager config Nve1L3Multicast result=true
com.inspur.ice.impl.devices.config.vxlan.L3Config successed to mergeRouterIdWithVrfContext, deviceIp is :Ipv4Address [_value=172.20.1.174]
com.inspur.ice.overlayconfig.impl.DistributeConfig distribute vrfRoutingInstanceConfig input VrfRoutingInstanceConfigInput [_configType=Config, _deviceIp=Ipv4Address [_value=172.20.1.174], _vrfContext=de55a13c-23:11500, _notCheckDeviceRoute=false, augmentation=[]]
com.inspur.ice.impl.devices.config.bgp.BorderLeafConfig device do not config router bgp, deviceIp is :172.20.1.174
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener ADD VrfRoute
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener ADD RouteGateway
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener ADD RouteTable

```

**交换机配置**
```
vlan 2002
  vn-segment 11002

vrf context 1b58fa19-02:11002
  vni 11002
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
  address-family ipv6 unicast
    route-target both auto
    route-target both auto evpn


interface Vlan2002
  no shutdown
  vrf member 1b58fa19-02:11002
  ip forward

interface nve1
  member vni 11002 associate-vrf


router bgp 65101
vrf context 1b58fa19-02:11002
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
  address-family ipv6 unicast
    route-target both auto
    route-target both auto evpn

IP Route Table for VRF "1b58fa19-02:11002"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

```

**命令**
```
devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;vlan 2002 ;no vn-segment ;vn-segment 11002 ;end

devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;vrf context 1b58fa19-02:11002 ;vni 11002 ;rd auto ;address-family ipv4 unicast ;route-target both auto ;route-target both auto evpn ;address-family ipv6 unicast ;route-target both auto ;route-target both auto evpn ;end

devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;feature interface-vlan ;vlan 2002 ;interface vlan 2002 ;no shutdown ;vrf member 1b58fa19-02:11002 ;ip forward

devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;interface nve1 ;member vni 11002 associate-vrf ;end

 devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;router bgp 65101 ;vrf 1b58fa19-02:11002 ;address-family ipv6 unicast ;advertise l2vpn evpn ;redistribute direct route-map all ;address-family ipv4 unicast ;advertise l2vpn evpn ;redistribute direct route-map all

```
## 路由器添加接口
**日志分析**
```
#### port 创建 ####
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      IceNeutronPortChangeListener data tree changed.
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      IcePort data tree changed.
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      WRITE
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      Adding Port : value=IcePort{getDeviceId=de55a13c-2352-4dd6-b418-3cec4b149890, getDeviceOwner=network:router_interface, getIceAllowedAddressPairs=[], getIceFixedIps=[IceFixedIps{getIpAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceSecurityGroups=[], getMacAddress=MacAddress [_value=fa:16:3e:70:03:a0], getName=, getNetworkId=Uuid [_value=21e67093-9cf3-4172-9943-64fca7be2da5], getProjectId=baec961077ea4e67933a8c140ca444d1, getStatus=ACTIVE, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], getUuid=Uuid [_value=562cd511-c923-44b1-993a-ed3647e02439], isAdminStateUp=true, augmentations={interface org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.binding.rev180302.IcePortBindingExtension=IcePortBindingExtension{getHostId=, getIceVifDetails=[], getVifType=unbound, getVnicType=normal}, interface org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.portsecurity.rev150712.IcePortSecurityExtension=IcePortSecurityExtension{isPortSecurityEnabled=false}}}
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      IcePort{getDeviceId=de55a13c-2352-4dd6-b418-3cec4b149890, getDeviceOwner=network:router_interface, getIceAllowedAddressPairs=[], getIceFixedIps=[IceFixedIps{getIpAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceSecurityGroups=[], getMacAddress=MacAddress [_value=fa:16:3e:70:03:a0], getName=, getNetworkId=Uuid [_value=21e67093-9cf3-4172-9943-64fca7be2da5], getProjectId=baec961077ea4e67933a8c140ca444d1, getStatus=ACTIVE, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], getUuid=Uuid [_value=562cd511-c923-44b1-993a-ed3647e02439], isAdminStateUp=true, augmentations={interface org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.binding.rev180302.IcePortBindingExtension=IcePortBindingExtension{getHostId=, getIceVifDetails=[], getVifType=unbound, getVnicType=normal}, interface org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.portsecurity.rev150712.IcePortSecurityExtension=IcePortSecurityExtension{isPortSecurityEnabled=false}}}  router interface created.

#### 写vxlan L3 subnet 下的interface ####
com.inspur.ice.overlaymapper.impl.IceVxlanCacheManager      writeIceRouterSubnet IceRouterSubnet [_cidr=IpPrefix [_ipv4Prefix=Ipv4Prefix [_value=100.100.1.0/24]], _gatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], _iceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], _key=IceRouterSubnetKey [_iceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], _gatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]]], augmentation=[]]
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      IceVxlanL3ChangeListener Write - before : IceDeviceVxlanL3{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.177], getIceExtGateways=[], getIceRouterSubnet=[], getIceVlanId=IceVlanId [_value=2500], getIceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], getIceVpnType=Router, getIceVrfRoute=[], getIceVxlanId=IceVxlanId [_value=11500], isDistributed=true, augmentations={}} after : IceDeviceVxlanL3{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.177], getIceExtGateways=[], getIceRouterSubnet=[IceRouterSubnet{getCidr=IpPrefix [_ipv4Prefix=Ipv4Prefix [_value=100.100.1.0/24]], getGatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getIceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceVlanId=IceVlanId [_value=2500], getIceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], getIceVpnType=Router, getIceVrfRoute=[], getIceVxlanId=IceVxlanId [_value=11500], isDistributed=true, augmentations={}}
com.inspur.ice.overlayconfig.impl.ConfigCache      ADD IceDeviceVxlanL3{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.177], getIceExtGateways=[], getIceRouterSubnet=[IceRouterSubnet{getCidr=IpPrefix [_ipv4Prefix=Ipv4Prefix [_value=100.100.1.0/24]], getGatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getIceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceVlanId=IceVlanId [_value=2500], getIceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], getIceVpnType=Router, getIceVrfRoute=[], getIceVxlanId=IceVxlanId [_value=11500], isDistributed=true, augmentations={}} failed with {}
com.inspur.ice.overlaymapper.impl.IceNeutronvpnUtils      Get subnetmap : Subnetmap{getId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], getNetworkId=Uuid [_value=476baec5-fd39-349d-a84e-57c1d2221ec5], getNetworkType=VLAN, getPortList=[Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9]], getSegmentationId=500, getSubnetIp=100.100.1.0/24, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], augmentations={}}
com.inspur.ice.overlayconfig.impl.ConfigCache      init l3 vxlan : IceDeviceVxlanL3Key [_iceDeviceId=Uri [_value=iceDeviceId:172.20.1.177], _iceVxlanId=IceVxlanId [_value=11500]] : IceRouterSubnetKey [_iceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], _gatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]]]
com.inspur.ice.overlayconfig.impl.DistributeConfig      distribute interface vlan config input InterfaceVlanVrfMemberInput [_configVlanType=Config, _deviceIp=Ipv4Address [_value=172.20.1.177], _fabricForwarding=AnycastGateway, _vlanId=500, _vlanIp=100.100.1.1/24, _vrfMember=de55a13c-23:11500, _secondaryIp=true, augmentation=[]]
com.inspur.ice.overlaymapper.impl.IceVlanCacheManager      writeL3VpnInterface VpnInterface [_key=VpnInterfaceKey [_name=85617daf-4a17-49c0-a3a2-9841625aa5c9], _name=85617daf-4a17-49c0-a3a2-9841625aa5c9, _vpnInstanceName=de55a13c-2352-4dd6-b418-3cec4b149890, _routerInterface=false, augmentation=[Adjacencies [_adjacency=[Adjacency [_adjacencyType=PrimaryAdjacency, _ipAddress=100.100.1.2, _key=AdjacencyKey [_ipAddress=100.100.1.2], _macAddress=fa:16:3e:d8:2c:da, _subnetId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], augmentation=[]]]]]]
com.inspur.ice.overlaymapper.impl.IceVxlanCacheManager      writeIceRouterSubnet IceRouterSubnet [_cidr=IpPrefix [_ipv4Prefix=Ipv4Prefix [_value=100.100.1.0/24]], _gatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], _iceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], _key=IceRouterSubnetKey [_iceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], _gatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]]], augmentation=[]]
com.inspur.ice.overlaymapper.impl.IceNeutronvpnUtils      Get subnetmap : Subnetmap{getId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], getNetworkId=Uuid [_value=476baec5-fd39-349d-a84e-57c1d2221ec5], getNetworkType=VLAN, getPortList=[Uuid [_value=85617daf-4a17-49c0-a3a2-9841625aa5c9]], getSegmentationId=500, getSubnetIp=100.100.1.0/24, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], augmentations={}}
com.inspur.ice.overlaymapper.impl.IceVlanCacheManager      writeL3VpnInterface VpnInterface [_key=VpnInterfaceKey [_name=85617daf-4a17-49c0-a3a2-9841625aa5c9], _name=85617daf-4a17-49c0-a3a2-9841625aa5c9, _vpnInstanceName=de55a13c-2352-4dd6-b418-3cec4b149890, _routerInterface=false, augmentation=[Adjacencies [_adjacency=[Adjacency [_adjacencyType=PrimaryAdjacency, _ipAddress=100.100.1.2, _key=AdjacencyKey [_ipAddress=100.100.1.2], _macAddress=fa:16:3e:d8:2c:da, _subnetId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], augmentation=[]]]]]]
com.inspur.ice.overlaymapper.impl.NeutronL3VpnInterfaceListener      NeutronL3VpnInterfaceListener add  recieve !!!
com.inspur.ice.overlaymapper.impl.IceVlanCacheManager      vpnPortMap ：ADD VpnInterface [_key=VpnInterfaceKey [_name=85617daf-4a17-49c0-a3a2-9841625aa5c9], _name=85617daf-4a17-49c0-a3a2-9841625aa5c9, _vpnInstanceName=de55a13c-2352-4dd6-b418-3cec4b149890, _routerInterface=false, augmentation=[Adjacencies [_adjacency=[Adjacency [_adjacencyType=PrimaryAdjacency, _ipAddress=100.100.1.2, _key=AdjacencyKey [_ipAddress=100.100.1.2], _macAddress=fa:16:3e:d8:2c:da, _subnetId=Uuid [_value=49c27f08-05d1-3f62-a810-91a6f700856a], augmentation=[]]]]]] failed
com.inspur.ice.overlaymapper.impl.NeutronCommonUtils      IceFwconnectionrouters is null!!!
com.inspur.ice.overlaymapper.impl.NeutronL3VpnInterfaceListener      NeutronL3VpnInterfaceListener iceFwconnectionrouter  is null !!!
org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager      add: intfName 85617daf-4a17-49c0-a3a2-9841625aa5c9 onto vpnName de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager      addVpnInterface: VPN Interface add event - intfName 85617daf-4a17-49c0-a3a2-9841625aa5c9 onto vpnName de55a13c-2352-4dd6-b418-3cec4b149890 from addVpnInterface
com.inspur.ice.overlaymapper.impl.NeutronL3VpnInterfaceListener      NeutronL3VpnInterfaceListener update  recieve !!!
org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager      update: VPN Interface update event - intfName 85617daf-4a17-49c0-a3a2-9841625aa5c9 on dpn null oldVpn de55a13c-2352-4dd6-b418-3cec4b149890 newVpn de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager      addVpnInterface: VPN Interface add event - intfName 85617daf-4a17-49c0-a3a2-9841625aa5c9 vpnName de55a13c-2352-4dd6-b418-3cec4b149890 on dpn null
org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager      processVpnInterfaceUp: Binding vpn service to interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 onto dpn 115377256237675 for vpn de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.netvirt.vpnmanager.VpnFootprintService      createOrUpdateVpnToDpnList: Created/Updated vpn footprint for vpn de55a13c-2352-4dd6-b418-3cec4b149890 vpnId 100000 interfacName85617daf-4a17-49c0-a3a2-9841625aa5c9 on dpn 115377256237675
org.opendaylight.netvirt.vpnmanager.VpnOpStatusListener      update: Processing update for vpn de55a13c-2352-4dd6-b418-3cec4b149890 with rd de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.netvirt.fibmanager.VrfEntryListener      populateFibOnNewDpn: dpn: 115377256237675: VRF Table not yet available for RD de55a13c-2352-4dd6-b418-3cec4b149890
org.opendaylight.netvirt.vpnmanager.VpnFootprintService      publishAddNotification: Successful in notifying listeners for add dpn 115377256237675 in vpn de55a13c-2352-4dd6-b418-3cec4b149890 rd de55a13c-2352-4dd6-b418-3cec4b149890 event
org.opendaylight.netvirt.vpnmanager.VpnFootprintService$DpnEnterExitVpnWorker      onSuccess: FootPrint established for vpn de55a13c-2352-4dd6-b418-3cec4b149890 rd de55a13c-2352-4dd6-b418-3cec4b149890 on dpn 115377256237675
org.opendaylight.netvirt.vpnmanager.VpnFootprintService      createOrUpdateVpnToDpnList: Sent populateFib event for new dpn 115377256237675 in VPN de55a13c-2352-4dd6-b418-3cec4b149890 for interface 85617daf-4a17-49c0-a3a2-9841625aa5c9
org.opendaylight.genius.datastoreutils.DataStoreJobCoordinator      Exception when executing jobEntry: JobEntry{key='VPNINTERFACE-85617daf-4a17-49c0-a3a2-9841625aa5c9', mainWorker=org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager$$Lambda$741/1527639192@6e597ade, rollbackWorker=null, retryCount=0, futures=null}
java.lang.NullPointerException: null
        at org.opendaylight.netvirt.vpnmanager.VpnUtil.getVpnSubnetGatewayIp(VpnUtil.java:1464) ~[395:org.opendaylight.netvirt.vpnmanager-impl:0.5.2]
        at org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager.processVpnInterfaceAdjacencies(VpnInterfaceManager.java:679) ~[395:org.opendaylight.netvirt.vpnmanager-impl:0.5.2]
        at org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager.processVpnInterfaceUp(VpnInterfaceManager.java:379) ~[395:org.opendaylight.netvirt.vpnmanager-impl:0.5.2]
        at org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager.lambda$addVpnInterface$0(VpnInterfaceManager.java:247) ~[395:org.opendaylight.netvirt.vpnmanager-impl:0.5.2]
        at org.opendaylight.genius.datastoreutils.DataStoreJobCoordinator$MainTask.run(DataStoreJobCoordinator.java:285) [304:org.opendaylight.genius.mdsalutil-api:0.3.2]
        at java.util.concurrent.ForkJoinTask$RunnableExecuteAction.exec(ForkJoinTask.java:1402) [?:?]
        at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289) [?:?]
        at java.util.concurrent.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1056) [?:?]
        at java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1692) [?:?]
        at java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:157) [?:?]
org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager      VPN Interface update event - intfName 85617daf-4a17-49c0-a3a2-9841625aa5c9 onto vpnName de55a13c-2352-4dd6-b418-3cec4b149890 running config-driven
org.opendaylight.netvirt.vpnmanager.VpnInterfaceManager      update: vpn interface updated for interface 85617daf-4a17-49c0-a3a2-9841625aa5c9 oldVpn de55a13c-2352-4dd6-b418-3cec4b149890 newVpn de55a13c-2352-4dd6-b418-3cec4b149890 processed successfully
com.inspur.ice.impl.common.DevInfo      deviceIp=172.20.1.177, deviceType=CN61108TC-V
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      IceNeutronPortChangeListener data tree changed.
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      IcePort data tree changed.
before IcePort{getDeviceId=de55a13c-2352-4dd6-b418-3cec4b149890, getDeviceOwner=network:router_interface, getIceAllowedAddressPairs=[], getIceFixedIps=[IceFixedIps{getIpAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceSecurityGroups=[], getMacAddress=MacAddress [_value=fa:16:3e:70:03:a0], getName=, getNetworkId=Uuid [_value=21e67093-9cf3-4172-9943-64fca7be2da5], getProjectId=baec961077ea4e67933a8c140ca444d1, getStatus=ACTIVE, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], getUuid=Uuid [_value=562cd511-c923-44b1-993a-ed3647e02439], isAdminStateUp=true, augmentations={interface org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.binding.rev180302.IcePortBindingExtension=IcePortBindingExtension{getHostId=, getIceVifDetails=[], getVifType=unbound, getVnicType=normal}, interface org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.portsecurity.rev150712.IcePortSecurityExtension=IcePortSecurityExtension{isPortSecurityEnabled=false}}}
 after IcePort{getDeviceId=de55a13c-2352-4dd6-b418-3cec4b149890, getDeviceOwner=network:router_interface, getIceAllowedAddressPairs=[], getIceExtraDhcpOpts=[], getIceFixedIps=[IceFixedIps{getIpAddress=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceSecurityGroups=[], getMacAddress=MacAddress [_value=fa:16:3e:70:03:a0], getName=, getNetworkId=Uuid [_value=21e67093-9cf3-4172-9943-64fca7be2da5], getRevisionNumber=5, getTenantId=Uuid [_value=baec9610-77ea-4e67-933a-8c140ca444d1], getUuid=Uuid [_value=562cd511-c923-44b1-993a-ed3647e02439], isAdminStateUp=true, augmentations={interface org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.binding.rev180302.IcePortBindingExtension=IcePortBindingExtension{getHostId=, getIceVifDetails=[], getVifType=unbound, getVnicType=normal}, interface org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ice.neutron.portsecurity.rev150712.IcePortSecurityExtension=IcePortSecurityExtension{isPortSecurityEnabled=false}}}
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      WRITE
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      Updating Port : original = IcePortKey [_uuid=Uuid [_value=562cd511-c923-44b1-993a-ed3647e02439]] update = IcePortKey [_uuid=Uuid [_value=562cd511-c923-44b1-993a-ed3647e02439]]
com.inspur.ice.overlaymapper.impl.IceNeutronPortChangeListener      Search port failed from neutron data store Uuid [_value=562cd511-c923-44b1-993a-ed3647e02439]
com.inspur.ice.impl.HttpPostCmd      devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;feature interface-vlan ;vlan 500 ;interface vlan 500 ;no shutdown ;vrf member de55a13c-23:11500 ;no ip forward ;ip address 100.100.1.1/24 ;fabric forwarding mode anycast-gateway
com.inspur.ice.impl.HttpPostCmd      nxapi return is: { "ins_api": { "sid": "eoc", "type": "cli_conf", "version": "1.0", "outputs": { "output": [{ "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "body": "Warning: Deleted all L3 config on interface Vlan500\n", "code": "200", "msg": "Success" }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }, { "code": "200", "msg": "Success", "body": { } }] } }}
com.inspur.ice.impl.devices.config.vlan.Vlan      interfaceVlanVrf sendCommands return :true
com.inspur.ice.impl.devices.config.vlan.Vlan      vlan is not created, so create vlan!
org.opendaylight.netvirt.vpnmanager.ArpNotificationHandler      ArpNotification Gratuitous Request Received from interface 115377256237675:ens192:500 and IP 100.100.1.1 having MAC 00:01:00:01:00:01 target destination 100.100.1.1, learning MAC
org.opendaylight.netvirt.vpnmanager.ArpNotificationHandler      ARP NO_RESOLVE: VPN  not configured. Ignoring responding to ARP requests from this Interface 115377256237675:ens192:500.
org.opendaylight.netvirt.vpnmanager.ArpNotificationHandler      ArpNotification Gratuitous Request Received from interface 115377256237675:ens192:500 and IP 100.100.1.1 having MAC 00:01:00:01:00:01 target destination 100.100.1.1, learning MAC
org.opendaylight.netvirt.vpnmanager.ArpNotificationHandler      ARP NO_RESOLVE: VPN  not configured. Ignoring responding to ARP requests from this Interface 115377256237675:ens192:500.
com.inspur.ice.overlaymapper.impl.IceLeafSwitchChangeListener      LeafSwitch before:Switch{getDevDesc=ICNT NOS-CN(tm) inos, Software (inos-cn), Version 9.2(2), RELEASE SOFTWARE Copyright (c) 2016-2018, Used subject to license from copyright owner. Compiled 11/8/2018 7:00:00, getDevId=Uri [_value=iceDeviceId:172.20.1.177], getDevIpAdd=Ipv4Address [_value=172.20.1.177], getDevLocation=, getDevManu=Inspur, getDevName=server177, getDevType=CN61108TC-V, getDeviceRole=ServerLeaf, getDeviceVlanId=1,500,2500, getEnablePassword=inspur@123, getEvpnTemp=1, getMacAddress=00:00:00:00, getOnlineStatus=onLine, getPointX=96, getPointY=45, getSnmpRead=public, getSnmpVersion=v2c, getSnmpWrite=public, getSshPassword=inspur@123, getSshUserName=admin, getSyncStatus=sync, isHasFex=false, augmentations={}} after:Switch{getDevDesc=ICNT NOS-CN(tm) inos, Software (inos-cn), Version 9.2(2), RELEASE SOFTWARE Copyright (c) 2016-2018, Used subject to license from copyright owner. Compiled 11/8/2018 7:00:00, getDevId=Uri [_value=iceDeviceId:172.20.1.177], getDevIpAdd=Ipv4Address [_value=172.20.1.177], getDevLocation=, getDevManu=Inspur, getDevName=server177, getDevType=CN61108TC-V, getDeviceRole=ServerLeaf, getDeviceVlanId=1,500,2500, getEnablePassword=inspur@123, getEvpnTemp=1, getMacAddress=00:00:00:00, getOnlineStatus=onLine, getPointX=96, getPointY=45, getSnmpRead=public, getSnmpVersion=v2c, getSnmpWrite=public, getSshPassword=inspur@123, getSshUserName=admin, getSyncStatus=sync, isHasFex=false, augmentations={}} data tree changed write.
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      ADD VrfRoute
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      REMOVE VrfRoute
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      ADD RouteGateway
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      UPDATE RouteGateway
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      ADD RouteTable
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      REMOVE RouteTable
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      IceVxlanL3ChangeListener Write - before : IceDeviceVxlanL3{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.174], getIceExtGateways=[], getIceRouterSubnet=[IceRouterSubnet{getCidr=IpPrefix [_ipv4Prefix=Ipv4Prefix [_value=100.100.1.0/24]], getGatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getIceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceVlanId=IceVlanId [_value=2500], getIceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], getIceVpnType=Router, getIceVrfRoute=[], getIceVxlanId=IceVxlanId [_value=11500], isDistributed=true, augmentations={}} after : IceDeviceVxlanL3{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.174], getIceExtGateways=[], getIceRouterSubnet=[IceRouterSubnet{getCidr=IpPrefix [_ipv4Prefix=Ipv4Prefix [_value=100.100.1.0/24]], getGatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getIceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceVlanId=IceVlanId [_value=2500], getIceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], getIceVpnType=Router, getIceVrfRoute=[], getIceVxlanId=IceVxlanId [_value=11500], isDistributed=true, augmentations={}}
com.inspur.ice.overlayconfig.impl.ConfigCache      ADD IceDeviceVxlanL3{getIceDeviceId=Uri [_value=iceDeviceId:172.20.1.174], getIceExtGateways=[], getIceRouterSubnet=[IceRouterSubnet{getCidr=IpPrefix [_ipv4Prefix=Ipv4Prefix [_value=100.100.1.0/24]], getGatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]], getIceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], augmentations={}}], getIceVlanId=IceVlanId [_value=2500], getIceVpnId=Uuid [_value=de55a13c-2352-4dd6-b418-3cec4b149890], getIceVpnType=Router, getIceVrfRoute=[], getIceVxlanId=IceVxlanId [_value=11500], isDistributed=true, augmentations={}} failed with {}
com.inspur.ice.overlayconfig.impl.ConfigCache      ADD IceRouterSubnetKey [_iceSubnetId=Uuid [_value=08617069-adb2-476c-90e9-b6bb4f425091], _gatewayIp=IpAddress [_ipv4Address=Ipv4Address [_value=100.100.1.1]]] failed with IceDeviceVxlanL3Key [_iceDeviceId=Uri [_value=iceDeviceId:172.20.1.174], _iceVxlanId=IceVxlanId [_value=11500]]
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      ADD VrfRoute
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      REMOVE VrfRoute
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      ADD RouteGateway
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      UPDATE RouteGateway
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      ADD RouteTable
com.inspur.ice.overlayconfig.impl.IceVxlanL3ChangeListener      REMOVE RouteTable
org.opendaylight.netvirt.vpnmanager.ArpNotificationHandler      ArpNotification Gratuitous Request Received from interface 115377256237675:ens192:500 and IP 100.100.1.1 having MAC 00:01:00:01:00:01 target destination 100.100.1.1, learning MAC
org.opendaylight.netvirt.vpnmanager.ArpNotificationHandler      ARP NO_RESOLVE: VPN  not configured. Ignoring responding to ARP requests from this Interface 115377256237675:ens192:500.

```

**交换机配置**
```
 添加到路由

 interface Vlan2
  no shutdown
  vrf member 1b58fa19-02:11002
  ip address 10.10.10.1/24
  fabric forwarding mode anycast-gateway
```

**命令**
```

 devIpAdd is :172.20.1.177 and nxapi commands: config terminal ;feature interface-vlan ;vlan 2 ;interface vlan 2 ;no shutdown ;vrf member 1b58fa19-02:11002 ;no ip forward ;ip address 10.10.10.1/24 ;fabric forwarding mode anycast-gateway

```

#添加静态路由

**交换机配置**
```
vrf context vni-11001
  vni 11001
  rd auto
  ip route 10.0.0.0/24 99.99.99.2 目的网段/下一跳
  address-family ipv4 unicast
    route-target both auto
route-target both auto evpn
address-family ipv6 unicast
    route-target both auto
    route-target both auto evpn

在BGP中发布（underlay bgpvrf）
router bgp 65101
  vrf vni-11001
  address-family ipv4 unicast
    network 10.0.0.0/24  /下一跳

```

#动态路由
**交换机配置**
```
------underlay bgpVrfNeighborConfig
router bgp 65101
  vrf vni-11001
    address-family ipv4 unicast
      advertise l2vpn evpn
    neighbor 99.99.99.2 remote-as 200
      address-family ipv4 unicast 

```

# 常用接口说明
##overlay

**distributeL2VxlanConfig --> vxlanConfigl2Dynamic**

" config terminal ;feature nv overlay  ;feature vn-segment-vlan-based  ;nv overlay evpn  ;evpn ;vni 10071 l2 ;rd auto ;route-target import auto ;route-target export auto ;exit"