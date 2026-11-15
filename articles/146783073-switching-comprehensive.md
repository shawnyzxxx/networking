# 交换综合实验：VLAN、VLANIF 与 MSTP 组网

## 文章导读
本篇是交换技术的综合实验，通过**VLAN 划分、VLANIF 三层接口、Eth-Trunk 链路聚合、路由器互联、MSTP 多生成树**，构建一个多区域互通的交换网络。适合学习华为交换机高级交换配置的读者。

## 一、实验拓扑与需求
通过 VLAN 划分实现不同业务网段隔离，使用 Eth-Trunk 增加链路带宽与冗余，借助 VLANIF 实现 VLAN 间三层通信，并用 MSTP 防止环路、实现负载分担。

## 二、实验步骤

### 1. VLAN 配置

**SW3**
```
[SW3]vlan batch 2 to 3
[SW3-GigabitEthernet0/0/1]port link-type access
[SW3-GigabitEthernet0/0/1]port default vlan 2
[SW3-GigabitEthernet0/0/2]port link-type access
[SW3-GigabitEthernet0/0/2]port default vlan 3
[SW3]port-group group-member g0/0/3
[SW3]port-group group-member g0/0/4
[SW3-port-group]port link-type trunk
[SW3-port-group]port trunk allow-pass vlan 2 3
```

**SW4**
```
[SW4]vlan batch 2 to 3
[SW4-GigabitEthernet0/0/1]port link-type access
[SW4-GigabitEthernet0/0/1]port default vlan 2
[SW4-GigabitEthernet0/0/2]port link-type access
[SW4-GigabitEthernet0/0/2]port default vlan 3
[SW4]port-group group-member g0/0/3
[SW4]port-group group-member g0/0/4
[SW4-port-group]port link-type trunk
[SW4-port-group]port trunk allow-pass vlan 2 3
```

**SW1**
```
[SW1]vlan batch 2 3 10 20
[SW1]int Eth-Trunk 0
[SW1-Eth-Trunk0]port link-type trunk
[SW1-Eth-Trunk0]port trunk allow-pass vlan 2 3
[SW1]port-group group-member g0/0/3 to g0/0/4
[SW1-port-group]port link-type trunk
[SW1-port-group]port trunk allow-pass vlan 2 3
[SW1]int g0/0/5
[SW1-GigabitEthernet0/0/5]port link-type access
[SW1-GigabitEthernet0/0/5]port default vlan 10
```

**SW2**
```
[SW2]vlan batch 2 3 10 20
[SW2]int Eth-Trunk 0
[SW2-Eth-Trunk0]port link-type trunk
[SW2-Eth-Trunk0]port trunk allow-pass vlan 2 3
[SW2]port-group group-member g0/0/3 to g0/0/4
[SW2-port-group]port link-type trunk
[SW2-port-group]port trunk allow-pass vlan 2 3
[SW2]int g0/0/5
[SW2-GigabitEthernet0/0/5]port link-type access
[SW2-GigabitEthernet0/0/5]port default vlan 20
```

### 2. VLAN 接口（VLANIF）配置

**SW1**
```
[SW1]int Vlanif 2
[SW1-Vlanif2]ip address 172.16.0.1 255.255.255.192
[SW1]int Vlanif 3
[SW1-Vlanif3]ip address 172.16.0.65 255.255.255.192
[SW1]int Vlanif 10
[SW1-Vlanif10]ip address 172.16.0.129 255.255.255.192
```

**SW2**
```
[SW2]int Vlanif 2
[SW2-Vlanif2]ip address 172.16.0.2 255.255.255.192
[SW2]int Vlanif 3
[SW2-Vlanif3]ip address 172.16.0.66 255.255.255.192
[SW2]int Vlanif 20
[SW2-Vlanif20]ip address 172.16.0.193 255.255.255.192
```

### 3. Eth-Trunk 配置

```
[SW1]int Eth-Trunk 0
[SW1-Eth-Trunk0]trunkport g0/0/1 g0/0/2

[SW2]int Eth-Trunk 0
[SW2-Eth-Trunk0]trunkport g0/0/1 g0/0/2
```

### 4. 路由器配置

**R1**
```
[r1-GigabitEthernet0/0/1]ip address 172.16.0.130 255.255.255.192
[r1-GigabitEthernet0/0/2]ip address 172.16.0.194 255.255.255.192
[r1-GigabitEthernet0/0/0]ip address 12.0.0.1 255.255.255.0
```

**ISP**
```
[ISP-GigabitEthernet0/0/0]ip address 12.0.0.2 255.255.255.0
[ISP-LoopBack0]ip address 2.2.2.2 255.255.255.0
```

### 5. STP（MSTP）配置

**SW1**
```
[SW1]stp enable
[SW1]stp mode mstp
[SW1]stp region-configuration
[SW1-mst-region]region-name aa
[SW1-mst-region]instance 1 vlan 2
[SW1-mst-region]instance 2 vlan 3
[SW1-mst-region]active region-configuration
[SW1]stp instance 1 root primary
[SW1]stp instance 2 root secondary
```

**SW2**
```
[SW2]stp enable
[SW2]stp mode mstp
[SW2]stp region-configuration
[SW2-mst-region]region-name aa
[SW2-mst-region]instance 1 vlan 2
[SW2-mst-region]instance 2 vlan 3
[SW2-mst-region]active region-configuration
[SW2]stp instance 1 root secondary
[SW2]stp instance 2 root primary
```

**SW3 和 SW4**
```
[SW3]stp enable
[SW3]stp mode mstp
[SW3]stp region-configuration
[SW3-mst-region]region-name aa
[SW3-mst-region]instance 1 vlan 2
[SW3-mst-region]instance 2 vlan 3
[SW3-mst-region]active region-configuration
[SW3]port-group group-member GigabitEthernet 0/0/1 to 0/0/2
[SW3-port-group]stp edged-port enable
[SW3]stp bpdu-protection

[SW4]stp enable
[SW4]stp mode mstp
[SW4]stp region-configuration
[SW4-mst-region]region-name aa
# ...（SW4 后续实例配置与 SW3 一致）
```

## 三、实验小结
本实验综合运用了 **VLAN 划分、VLANIF 三层网关、Eth-Trunk 链路聚合、MSTP 多生成树**。其中 `region-name` 必须一致才能组成同一个 MST 域，通过把不同 VLAN 映射到不同实例并指定不同的根桥，可以实现 VLAN 间的负载分担与环路防护。