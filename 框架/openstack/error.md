#端口绑定失败
&emsp;PortBindingFailed: Binding failed for
&emsp;检查ml2_conf.ini
```
    [ml2_type_vlan]
    network_vlan_ranges = external:1:1000
```

#控制台无法登陆
&emsp;检查防火墙,即使防火墙关闭，仍有可能在生效。使用 ipstables -F 清除所有防火墙规则

