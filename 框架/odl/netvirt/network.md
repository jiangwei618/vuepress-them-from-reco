# 网络处理

## 1.add

***
<strong><center> **第一步** </center></strong>

&emsp;&emsp;当北向创建网络api被调用，neutron写入config neutron/network/network库，并触发datastore监听。

* **neutronVpn/NeutronNetworkChangeListener:** 在进行一些基础检查后，创建elanInstance（一个网络对应一个elan instance）。创建elanInstance时会查询config elan-instances/elan-instance库，如果存在，则返回，如果不存在则创建enlanInstace数据，写入config库
  
<br/>  

* **ipv6Service/NeutronNetworkChangeListener:**
 
<br/>
  
* **qosservice/QosNeutronNetworkChangeListener:**
  
<br/>

```plantuml
@startuml
start
    #99FF99:neutron/network/network;
    :add; 
    fork
        :bundle:neutronVpn; 
        :NeutronNetworkChangeListener;
        :createElanInstance;
        #99FF99:elan-instances/elan-instance;
        if (readDs ElanInstance isPresent?) then (yes)
            :get from ds; 
        else (no)
            :write ds;
        endif
        :外部网络处理; 
    fork again
        :bundle:ipv6Service;
        :NeutronNetworkChangeListener;
    fork again
        :bundle:qosservice;
        :QosNeutronNetworkChangeListener;
    end fork
    
end

@enduml

```

***

<strong><center> **第二步** </center></strong>

&emsp;&emsp;当elan-instances/elan-instance config库被写入数据后，开始触发elan创建的流程。在创建elan的过程中，会写入其他的多个operational数据库，不过这些数据库都没有监听器，仅仅作为数据存储所用。
&emsp;&emsp;创建elan-instances/elan-instance时还会更新elan-instances/elan-instance config库

```plantuml
@startuml
start
    #99FF99:elan-instances/elan-instance;
    :add; 
    fork
        :ElanInstanceDpnListener; 
        if (非外部网络 && vlan) then (yes)
            :创建elan interface \n（连接到业务网络接口）; 
        else (no)
            :return;
        endif
    fork again
        :EvpnElanInstanceListener;
        stop;
    fork again
        :DataObjectCache;
        :添加缓存;
    fork again
        :ElanInstanceListener;
        stop;  
    end fork
    
end

@enduml

```

&emsp;&emsp;**接下图**

```plantuml
@startuml
start
    #99FF99:elan-instances/elan-instance;
    :add; 
    fork
        :...;
        stop  
    fork again
        :ElanInstanceManager;
        :ElanUtils.updateOperationalDataStore;
        fork
        #00B2EE:elan:elan-state;
        fork again
        #00B2EE:elan:elan-forwarding-tables/mac-table;
        fork again
        #00B2EE:EtreeInstance 处理;
        stop;
        fork again
        #00B2EE:elan:elan-tag-name-map;
        fork again
        :update;
        #99FF99:elan-instances/elan-instance（with elanTag）; 
        end fork     
    end fork
   
end

@enduml

```


***
<strong><center> **第三步** </center></strong>
&emsp;&emsp;当elan-instances/elan-instance config库被更新数据后，开始触发elan更新的流程。
```plantuml
@startuml
start
    #99FF99:elan-instances/elan-instance;
    :update; 
    fork
        :ElanInstanceDpnListener; 
        if (before.isVlanElanInstance \n && after.isVlanElanInstance) then (yes)
            if (before.SegmentationId != after.SegmentationId\n || before.PhysicalNetworkName != after.PhysicalNetworkName\n) then (yes)    
            :删除elan interface \n（连接到业务网络接口）; 
            :创建elan interface \n（连接到业务网络接口）;
            else(no) 
            endif 
        else(no) 
        endif

        if (!before.isVlanElanInstance \n && after.isVlanElanInstance) then (yes)
            :创建elan interface \n（连接到业务网络接口）;
        else(no) 
        endif

        if (before.isVlanElanInstance \n && !after.isVlanElanInstance) then (yes)
            :删除 interface \n（连接到业务网络接口）;
        else(no) 
        endif

    fork again
        :EvpnElanInstanceListener;
        if (isWithdrawEvpnRT2Routes) then (yes)
            :do something;
        else(no)
        endif
        if (isAdvertiseEvpnRT2Routes) then (yes)
            :do something;
        else(no)
        endif
        stop;
    fork again
        :DataObjectCache;
        :处理缓存;
    fork again
        :ElanInstanceListener;
        stop;  
    end fork
    
end

@enduml

```

&emsp;&emsp;**接下图**

```plantuml
@startuml
start
    #99FF99:elan-instances/elan-instance;
    :update; 
    fork
        :...;
        stop  
    fork again
        :ElanInstanceManager;
        if (existingElanTag == null || !existingElanTag.equals(update.getElanTag()))then(yes)
            if (update.getElanTag() == null  || update.getElanTag() == 0L)then(yes)
                :update the elan-Instance with new properties;
                :ElanUtils.updateOperationalDataStore;
                :见第二步流程;
            else(no)
                :handleunprocessedElanInterfaces(处理添加elan interface时，\nelan tag is not updated的端口，缓存可能有问题);
            endif
        else(no)
        endif
    end fork
   
end

@enduml

```


## 2.update

&emsp;&emsp;当北向api 更新neutron/network/network后，NeutronNetworkChangeListener不作其他操作.

```plantuml
@startuml
start
    #99FF99:neutron/network/network;
    :update; 
    fork
        :bundle:neutronVpn; 
        :NeutronNetworkChangeListener;
        :do nothing
    fork again
        :bundle:ipv6Service;
        :NeutronNetworkChangeListener;
    fork again
        :bundle:qosservice;
        :QosNeutronNetworkChangeListener;
    end fork
    
end

@enduml

```


## 3.remove
***
<strong><center> **第一步** </center></strong>

```plantuml
@startuml
start
    #99FF99:neutron/network/network;
    :remove; 
    fork
        :bundle:neutronVpn; 
        :NeutronNetworkChangeListener;
        :外部网络处理;
        #99FF99:elan-instances/elan-instance;
        if (readDs ElanInstance isPresent?) then (yes)
            :delete ds elan instance; 
        else (no)
        endif
    fork again
        :bundle:ipv6Service;
        :NeutronNetworkChangeListener;
    fork again
        :bundle:qosservice;
        :QosNeutronNetworkChangeListener;
    end fork
    
end

@enduml

```
    
&emsp;&emsp;删除数据库中的elan instance数据后，触发相关的其他流程。