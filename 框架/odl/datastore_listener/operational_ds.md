```
{
    "request": {
        "mbean": "org.opendaylight.controller:Category=ShardDataTreeListenerInfo,name=member-1-shard-default-operational,type=DistributedOperationalDatastore",
        "attribute": "DataTreeChangeListenerInfo",
        "type": "read"
    },
    "value": [
        {
            "notificationCount": 4,
            "listener": "org.opendaylight.genius.interfacemanager.listeners.InterfaceStateListener@2c191ca7",
            "enabled": true,
            "registeredPath": "/(urn:ietf:params:xml:ns:yang:ietf-interfaces?revision=2014-05-08)interfaces-state/interface/interface"
        },
        {
            "notificationCount": 13,
            "listener": "org.opendaylight.genius.interfacemanager.listeners.InterfaceTopologyStateListener@d91913c",
            "enabled": true,
            "registeredPath": "/(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)network-topology/topology/topology/node/node/AugmentationIdentifier{childNames=[(urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)auto-attach, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)stp_enable, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)bridge-openflow-node-ref, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)datapath-id, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)datapath-type, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)protocol-entry, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)bridge-name, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)bridge-other-configs, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)controller-entry, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)fail-mode, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)bridge-uuid, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)flow-node, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)managed-by, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)bridge-external-ids]}"
        },
        {
            "notificationCount": 2,
            "listener": "org.opendaylight.netvirt.ipv6service.Ipv6NodeListener@583faa0",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node/AugmentationIdentifier{childNames=[(urn:opendaylight:flow:inventory?revision=2013-08-19)description, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-actions, (urn:opendaylight:flow:inventory?revision=2013-08-19)hardware, (urn:opendaylight:flow:inventory?revision=2013-08-19)switch-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-instructions, (urn:opendaylight:flow:inventory?revision=2013-08-19)meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)serial-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-group, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-match-types, (urn:opendaylight:flow:inventory?revision=2013-08-19)port-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)table, (urn:opendaylight:flow:inventory?revision=2013-08-19)group, (urn:opendaylight:flow:inventory?revision=2013-08-19)manufacturer, (urn:opendaylight:flow:inventory?revision=2013-08-19)table-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)software, (urn:opendaylight:flow:inventory?revision=2013-08-19)ip-address]}"
        },
        {
            "notificationCount": 1,
            "listener": "org.opendaylight.netvirt.dhcpservice.DhcpNodeListener@1c78549b",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.netvirt.elan.l2gw.ha.listeners.HAOpClusteredListener@1cd39767",
            "enabled": true,
            "registeredPath": "/(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)network-topology/topology/topology[{(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)topology-id=hwvtep:1}]/node/node"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.netvirt.elan.l2gw.listeners.HwvtepPhysicalSwitchListener@30b0bf7e",
            "enabled": true,
            "registeredPath": "/(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)network-topology/topology/topology[{(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)topology-id=hwvtep:1}]/node/node/AugmentationIdentifier{childNames=[(urn:opendaylight:params:xml:ns:yang:ovsdb:hwvtep?revision=2015-09-01)switch-fault-status, (urn:opendaylight:params:xml:ns:yang:ovsdb:hwvtep?revision=2015-09-01)management-ips, (urn:opendaylight:params:xml:ns:yang:ovsdb:hwvtep?revision=2015-09-01)tunnel-ips, (urn:opendaylight:params:xml:ns:yang:ovsdb:hwvtep?revision=2015-09-01)hwvtep-node-name, (urn:opendaylight:params:xml:ns:yang:ovsdb:hwvtep?revision=2015-09-01)managed-by, (urn:opendaylight:params:xml:ns:yang:ovsdb:hwvtep?revision=2015-09-01)tunnels, (urn:opendaylight:params:xml:ns:yang:ovsdb:hwvtep?revision=2015-09-01)hwvtep-node-description, (urn:opendaylight:params:xml:ns:yang:ovsdb:hwvtep?revision=2015-09-01)physical-switch-uuid]}"
        },
        {
            "notificationCount": 1,
            "listener": "org.opendaylight.netvirt.natservice.ha.SnatNodeEventListener@108f5521",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node"
        },
        {
            "notificationCount": 4,
            "listener": "org.opendaylight.genius.interfacemanager.servicebindings.flowbased.listeners.FlowBasedServicesInterfaceStateListener@668d5cc9",
            "enabled": true,
            "registeredPath": "/(urn:ietf:params:xml:ns:yang:ietf-interfaces?revision=2014-05-08)interfaces-state/interface/interface"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.genius.interfacemanager.listeners.InterfaceInventoryStateListener@62d50775",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node/node-connector/node-connector/AugmentationIdentifier{childNames=[(urn:opendaylight:flow:inventory?revision=2013-08-19)advertised-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)current-speed, (urn:opendaylight:flow:inventory?revision=2013-08-19)queue, (urn:opendaylight:flow:inventory?revision=2013-08-19)state, (urn:opendaylight:flow:inventory?revision=2013-08-19)hardware-address, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported, (urn:opendaylight:flow:inventory?revision=2013-08-19)peer-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)name, (urn:opendaylight:flow:inventory?revision=2013-08-19)port-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)configuration, (urn:opendaylight:flow:inventory?revision=2013-08-19)current-feature, (urn:opendaylight:flow:inventory?revision=2013-08-19)maximum-speed, (urn:opendaylight:flow:inventory?revision=2013-08-19)reason]}"
        },
        {
            "notificationCount": 110,
            "listener": "org.opendaylight.genius.lockmanager.impl.LockListener@34cbe89d",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:genius:lockmanager?revision=2016-04-13)locks/lock/lock"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.netvirt.vpnmanager.ArpMonitoringHandler@5b7d7196",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:netvirt:l3vpn?revision=2013-09-11)learnt-vpn-vip-to-port-data/learnt-vpn-vip-to-port/learnt-vpn-vip-to-port"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.genius.mdsalutil.internal.MDSALManager$FlowListener@424683c1",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node/AugmentationIdentifier{childNames=[(urn:opendaylight:flow:inventory?revision=2013-08-19)description, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-actions, (urn:opendaylight:flow:inventory?revision=2013-08-19)hardware, (urn:opendaylight:flow:inventory?revision=2013-08-19)switch-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-instructions, (urn:opendaylight:flow:inventory?revision=2013-08-19)meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)serial-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-group, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-match-types, (urn:opendaylight:flow:inventory?revision=2013-08-19)port-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)table, (urn:opendaylight:flow:inventory?revision=2013-08-19)group, (urn:opendaylight:flow:inventory?revision=2013-08-19)manufacturer, (urn:opendaylight:flow:inventory?revision=2013-08-19)table-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)software, (urn:opendaylight:flow:inventory?revision=2013-08-19)ip-address]}/(urn:opendaylight:flow:inventory?revision=2013-08-19)table/table/flow/flow"
        },
        {
            "notificationCount": 2,
            "listener": "org.opendaylight.netvirt.vpnmanager.VpnNodeListener@23671181",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.genius.interfacemanager.listeners.CacheBridgeRefEntryListener@68f8eb53",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:genius:interfacemanager:meta?revision=2016-04-06)bridge-ref-info/bridge-ref-entry/bridge-ref-entry"
        },
        {
            "notificationCount": 4,
            "listener": "org.opendaylight.netvirt.elan.internal.ElanInterfaceStateClusteredListener@10cda555",
            "enabled": true,
            "registeredPath": "/(urn:ietf:params:xml:ns:yang:ietf-interfaces?revision=2014-05-08)interfaces-state/interface/interface"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.netvirt.elan.l2gw.listeners.LocalUcastMacListener@35c1bc04",
            "enabled": true,
            "registeredPath": "/(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)network-topology/topology/topology[{(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)topology-id=hwvtep:1}]/node/node"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.genius.alivenessmonitor.internal.AlivenessMonitor@5d47d9d9",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:genius:alivenessmonitor?revision=2016-04-11)monitoring-states/monitoring-state/monitoring-state"
        },
        {
            "notificationCount": 2,
            "listener": "org.opendaylight.genius.mdsalutil.cache.DataObjectCache$$Lambda$1186/1573209624@366ceede",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:genius:interfacemanager:meta?revision=2016-04-06)if-indexes-interface-map/if-index-interface/if-index-interface"
        },
        {
            "notificationCount": 4,
            "listener": "org.opendaylight.netvirt.ipv6service.Ipv6ServiceInterfaceEventListener@74862a8",
            "enabled": true,
            "registeredPath": "/(urn:ietf:params:xml:ns:yang:ietf-interfaces?revision=2014-05-08)interfaces-state/interface/interface"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.genius.mdsalutil.cache.DataObjectCache$$Lambda$1186/1573209624@736706bd",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:netvirt:natservice?revision=2016-01-11)neutron-vip-states/vip-state/vip-state"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.genius.mdsalutil.internal.MDSALManager$GroupListener@1d15c0a7",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node/AugmentationIdentifier{childNames=[(urn:opendaylight:flow:inventory?revision=2013-08-19)description, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-actions, (urn:opendaylight:flow:inventory?revision=2013-08-19)hardware, (urn:opendaylight:flow:inventory?revision=2013-08-19)switch-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-instructions, (urn:opendaylight:flow:inventory?revision=2013-08-19)meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)serial-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-group, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-match-types, (urn:opendaylight:flow:inventory?revision=2013-08-19)port-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)table, (urn:opendaylight:flow:inventory?revision=2013-08-19)group, (urn:opendaylight:flow:inventory?revision=2013-08-19)manufacturer, (urn:opendaylight:flow:inventory?revision=2013-08-19)table-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)software, (urn:opendaylight:flow:inventory?revision=2013-08-19)ip-address]}/(urn:opendaylight:flow:inventory?revision=2013-08-19)group/group"
        },
        {
            "notificationCount": 1094,
            "listener": "org.opendaylight.genius.mdsalutil.cache.DataObjectCache$$Lambda$1186/1573209624@18de7178",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:neutron?revision=2015-07-12)neutron/hostconfigs/hostconfig/hostconfig"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.genius.interfacemanager.pmcounters.NodeConnectorStatsImpl@913ec99",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.ovsdb.hwvtepsouthbound.reconciliation.configuration.HwvtepReconciliationManager@9f33412",
            "enabled": true,
            "registeredPath": "/(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)network-topology/topology/topology[{(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)topology-id=hwvtep:1}]/node/node"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.serviceutils.srm.impl.ServiceRecoveryListener@69601cf3",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:serviceutils:srm:ops?revision=2018-06-26)service-ops/services/services/operations/operations"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.openflowplugin.openflow.ofswitch.config.DefaultConfigPusher@1ab71b65",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node/AugmentationIdentifier{childNames=[(urn:opendaylight:flow:inventory?revision=2013-08-19)description, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-actions, (urn:opendaylight:flow:inventory?revision=2013-08-19)hardware, (urn:opendaylight:flow:inventory?revision=2013-08-19)switch-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-instructions, (urn:opendaylight:flow:inventory?revision=2013-08-19)meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)serial-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-group, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-match-types, (urn:opendaylight:flow:inventory?revision=2013-08-19)port-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)table, (urn:opendaylight:flow:inventory?revision=2013-08-19)group, (urn:opendaylight:flow:inventory?revision=2013-08-19)manufacturer, (urn:opendaylight:flow:inventory?revision=2013-08-19)table-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)software, (urn:opendaylight:flow:inventory?revision=2013-08-19)ip-address]}"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.genius.interfacemanager.listeners.TerminationPointStateListener@5ff5efa4",
            "enabled": true,
            "registeredPath": "/(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)network-topology/topology/topology/node/node/termination-point/termination-point/AugmentationIdentifier{childNames=[(urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-type, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ofport_request, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-external-ids, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-uuid, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)qos-entry, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)trunks, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)port-uuid, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)vlan-tag, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)port-external-ids, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)options, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-bfd-status, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ingress-policing-rate, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ingress-policing-burst, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)port-other-configs, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)name, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-bfd, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ifindex, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-other-configs, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-lldp, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ofport, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)vlan-mode]}"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.ovsdb.hwvtepsouthbound.HwvtepOperGlobalListener@2238bc50",
            "enabled": true,
            "registeredPath": "/(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)network-topology/topology/topology[{(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)topology-id=hwvtep:1}]/node/node"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.openflowplugin.applications.lldpspeaker.NodeConnectorInventoryEventTranslator@18d4189a",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node/node-connector/node-connector/AugmentationIdentifier{childNames=[(urn:opendaylight:flow:inventory?revision=2013-08-19)advertised-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)current-speed, (urn:opendaylight:flow:inventory?revision=2013-08-19)queue, (urn:opendaylight:flow:inventory?revision=2013-08-19)state, (urn:opendaylight:flow:inventory?revision=2013-08-19)hardware-address, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported, (urn:opendaylight:flow:inventory?revision=2013-08-19)peer-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)name, (urn:opendaylight:flow:inventory?revision=2013-08-19)port-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)configuration, (urn:opendaylight:flow:inventory?revision=2013-08-19)current-feature, (urn:opendaylight:flow:inventory?revision=2013-08-19)maximum-speed, (urn:opendaylight:flow:inventory?revision=2013-08-19)reason]}"
        },
        {
            "notificationCount": 4,
            "listener": "org.opendaylight.netvirt.aclservice.listeners.AclInterfaceStateListener@3eaab9d1",
            "enabled": true,
            "registeredPath": "/(urn:ietf:params:xml:ns:yang:ietf-interfaces?revision=2014-05-08)interfaces-state/interface/interface"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.netvirt.qosservice.QosTerminationPointListener@38b77502",
            "enabled": true,
            "registeredPath": "/(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)network-topology/topology/topology[{(urn:TBD:params:xml:ns:yang:network-topology?revision=2013-10-21)topology-id=ovsdb:1}]/node/node/termination-point/termination-point/AugmentationIdentifier{childNames=[(urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-type, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ofport_request, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-external-ids, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-uuid, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)qos-entry, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)trunks, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)port-uuid, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)vlan-tag, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)port-external-ids, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)options, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-bfd-status, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ingress-policing-rate, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ingress-policing-burst, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)port-other-configs, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)name, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-bfd, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ifindex, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-other-configs, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)interface-lldp, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)ofport, (urn:opendaylight:params:xml:ns:yang:ovsdb?revision=2015-01-05)vlan-mode]}"
        },
        {
            "notificationCount": 4,
            "listener": "org.opendaylight.netconf.sal.streams.listeners.ListenerAdapter@39485a8f",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:neutron?revision=2015-07-12)neutron/ports"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.genius.mdsalutil.cache.DataObjectCache$$Lambda$1186/1573209624@7aa56fd3",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:netvirt:l3vpn?revision=2013-09-11)vpn-instance-op-data/vpn-instance-op-data-entry/vpn-instance-op-data-entry"
        },
        {
            "notificationCount": 1,
            "listener": "org.opendaylight.netvirt.qosservice.QosInterfaceStateChangeListener@720db655",
            "enabled": true,
            "registeredPath": "/(urn:ietf:params:xml:ns:yang:ietf-interfaces?revision=2014-05-08)interfaces-state/interface/interface"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.openflowplugin.applications.lldpspeaker.NodeConnectorInventoryEventTranslator@18d4189a",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node/node-connector/node-connector/AugmentationIdentifier{childNames=[(urn:opendaylight:flow:inventory?revision=2013-08-19)advertised-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)current-speed, (urn:opendaylight:flow:inventory?revision=2013-08-19)queue, (urn:opendaylight:flow:inventory?revision=2013-08-19)state, (urn:opendaylight:flow:inventory?revision=2013-08-19)hardware-address, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported, (urn:opendaylight:flow:inventory?revision=2013-08-19)peer-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)name, (urn:opendaylight:flow:inventory?revision=2013-08-19)port-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)configuration, (urn:opendaylight:flow:inventory?revision=2013-08-19)current-feature, (urn:opendaylight:flow:inventory?revision=2013-08-19)maximum-speed, (urn:opendaylight:flow:inventory?revision=2013-08-19)reason]}/(urn:opendaylight:flow:inventory?revision=2013-08-19)state"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.genius.mdsalutil.cache.DataObjectCache$$Lambda$1186/1573209624@29db3710",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:genius:itm:meta?revision=2017-12-10)ovs-bridge-ref-info/ovs-bridge-ref-entry/ovs-bridge-ref-entry"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.genius.mdsalutil.cache.DataObjectCache$$Lambda$1186/1573209624@6d424da1",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:genius:itm:op?revision=2016-04-06)tunnels_state/state-tunnel-list/state-tunnel-list"
        },
        {
            "notificationCount": 1,
            "listener": "org.opendaylight.netvirt.elan.internal.ElanDpnInterfaceClusteredListener@22c0bb48",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:netvirt:elan?revision=2015-06-02)elan-dpn-interfaces/elan-dpn-interfaces-list/elan-dpn-interfaces-list/dpn-interfaces/dpn-interfaces"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.daexim.impl.DataExportImportAppProvider$$Lambda$1129/356649316@450a7097",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:daexim-internal?revision=2016-09-21)daexim/daexim-control"
        },
        {
            "notificationCount": 3,
            "listener": "org.opendaylight.openflowplugin.applications.frm.impl.DeviceMastershipManager@42c537e",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:inventory?revision=2013-08-19)nodes/node/node/AugmentationIdentifier{childNames=[(urn:opendaylight:flow:inventory?revision=2013-08-19)description, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-actions, (urn:opendaylight:flow:inventory?revision=2013-08-19)hardware, (urn:opendaylight:flow:inventory?revision=2013-08-19)switch-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-instructions, (urn:opendaylight:flow:inventory?revision=2013-08-19)meter, (urn:opendaylight:flow:inventory?revision=2013-08-19)serial-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)stale-group, (urn:opendaylight:flow:inventory?revision=2013-08-19)supported-match-types, (urn:opendaylight:flow:inventory?revision=2013-08-19)port-number, (urn:opendaylight:flow:inventory?revision=2013-08-19)table, (urn:opendaylight:flow:inventory?revision=2013-08-19)group, (urn:opendaylight:flow:inventory?revision=2013-08-19)manufacturer, (urn:opendaylight:flow:inventory?revision=2013-08-19)table-features, (urn:opendaylight:flow:inventory?revision=2013-08-19)software, (urn:opendaylight:flow:inventory?revision=2013-08-19)ip-address]}"
        },
        {
            "notificationCount": 0,
            "listener": "org.opendaylight.netvirt.vpnmanager.LearntVpnVipToPortEventProcessor@4773bc25",
            "enabled": true,
            "registeredPath": "/(urn:opendaylight:netvirt:l3vpn?revision=2013-09-11)learnt-vpn-vip-to-port-event-data/learnt-vpn-vip-to-port-event/learnt-vpn-vip-to-port-event"
        }
    ],
    "timestamp": 1564712469,
    "status": 200
}
```