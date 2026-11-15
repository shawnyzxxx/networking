# OSPF 综合实验：多区域、特殊区域与路由重发布

## 文章导读
本篇是 OSPF 综合实验，覆盖**多区域划分、MD5 认证、ABR 区域汇总、Stub/NSSA 特殊区域、多进程双向重发布、默认路由下发**等核心知识点。适合希望深入掌握 OSPF 大型组网的读者。

## 一、实验拓扑与网段划分
整张网络围绕 `172.16.0.0/20` 等网段划分多个 OSPF 区域（Area 0 骨干区、Area 1、Area 2、Area 3、Area 4），各路由器配置相应的接口地址与环回，形成多区域 OSPF 网络。

## 二、实验要求
1. 合理划分网段，实现子网汇总，减少路由条目
2. 规划特殊区域（Stub、NSSA），确保邻居关系建立
3. 正确宣告 IP，避免配置错误
4. 配置不规则区域，进行多进程双向重发布
5. 配置特殊区域后下发默认路由，保证路由信息完整

## 三、实验基础配置

### R1
```
[r1]interface GigabitEthernet0/0/0
[r1-GigabitEthernet0/0/0]ip address 172.16.32.1 255.255.240.0
[r1]ospf authentication-mode md5 1 cipher <密码>    # 认证
[r1-GigabitEthernet0/0/0]ospf 1 router-id 1.1.1.1
[r1-ospf-1]area 0.0.0.1
[r1-ospf-1-area-0.0.0.1]network 1.1.1.1 0.0.0.0
[r1-ospf-1-area-0.0.0.1]network 172.16.32.0 0.0.15.255
[r1-ospf-1-area-0.0.0.1]stub        # 配置 stub 区域
```

### R2
```
[r2]int g 0/0/0
[r2-GigabitEthernet0/0/0]ip ad 172.16.32.2 20
[r2]ospf 1 r 2.2.2.2
[r2-ospf-1]a 1
[r2-ospf-1-area-0.0.0.1]network 172.16.32.0 0.0.15.255
[r2-LoopBack0]ip ad 2.2.2.2 24
[r2]ospf 1 r 2.2.2.2
[r2-ospf-1]a 1
[r2-ospf-1-area-0.0.0.1]network 2.2.2.2 0.0.0.0
[r2]int g 0/0/0
[r2-GigabitEthernet0/0/0]ospf authentication-mode md5 1 c 123456    # 认证
[r2-ospf-1]a 1
[r2-ospf-1-area-0.0.0.1]stub
```

### R3（ABR，区域 0 与区域 1 的边界）
```
interface GigabitEthernet0/0/0
ip address 172.16.32.3 255.255.240.0
interface GigabitEthernet0/0/1
ip address 172.16.0.1 255.255.248.0

ospf 1 router-id 3.3.3.3
area 0.0.0.0
abr-summary 172.16.0.0 255.255.224.0        # 对区域 0 的汇总
authentication-mode md5 1 cipher <密码>     # 认证
network 172.16.0.0 0.0.7.255
area 0.0.0.1
abr-summary 172.16.32.0 255.255.240.0       # 对区域 1 的汇总
authentication-mode md5 1 cipher <密码>     # 认证
network 3.3.3.3 0.0.0.0
network 172.16.32.0 0.0.15.255
stub no-summary
```

### R4
```
ospf 1 router-id 4.4.4.4
default-route-advertise                        # 下发默认路由
area 0.0.0.0
network 172.16.0.0 0.0.7.255
network 172.16.8.0 0.0.7.255
network 172.16.16.0 0.0.7.255
network 172.16.24.0 0.0.7.255
[r4]acl 2000
[r4-acl-basic-2000]rule permit source 172.16.0.0 0.0.255.255
[r4]int s 4/0/0
[r4-Serial4/0/0]nat outbound 2000              # 做 NAT
```

### R5
```
[r5]int s 4/0/0
[r5-Serial4/0/0]ip ad 172.16.8.2 21
[r5-LoopBack0]
interface LoopBack0
ip address 5.5.5.5 255.255.255.0
```

### R6
```
ospf 1 router-id 6.6.6.6
area 0.0.0.0
abr-summary 172.16.0.0 255.255.224.0         # 对区域 0 的汇总
authentication-mode md5 1 cipher <密码>
network 172.16.16.0 0.0.7.255
area 0.0.0.2
abr-summary 172.16.64.0 255.255.192.0        # 对区域 2 的汇总
authentication-mode md5 1 cipher <密码>
network 172.16.64.0 0.0.15.255
nssa no-import-route
```

### R7
```
ospf 1 router-id 7.7.7.7
area 0.0.0.0
abr-summary 172.16.64.0 255.255.192.0
abr-summary 172.16.0.0 255.255.224.0
authentication-mode md5 1 cipher <密码>
network 172.16.24.0 0.0.7.255
area 0.0.0.3
abr-summary 172.16.96.0 255.255.224.0
authentication-mode md5 1 cipher <密码>
network 172.16.96.0 0.0.15.255
nssa no-summary            # 配置完全的非完全末梢区域
```

### R8
```
ospf 1 router-id 8.8.8.8
area 0.0.0.3
network 8.8.8.8 0.0.0.0
network 172.16.96.0 0.0.15.255
network 172.16.112.0 0.0.15.255
nssa no-summary            # 配置完全的非完全末梢区域
```

### R9
```
ospf 1 router-id 9.9.9.9
asbr-summary 172.16.128.0 255.255.224.0
asbr-summary 172.16.64.0 255.255.192.0
default-route-advertise
import-route ospf 2        # 重发布
authentication-mode md5 1 cipher <密码>
network 172.16.112.0 0.0.15.255
nssa no-summary
```

### R10
```
ospf 1 router-id 10.10.10.10
area 0.0.0.4
network 10.10.10.10 0.0.0.0
network 172.16.128.0 0.0.127.255
```

### R11
```
ospf 1 router-id 11.11.11.11
area 0.0.0.2
network 11.11.11.11 0.0.0.0
network 172.16.64.0 0.0.15.255
network 172.16.80.0 0.0.15.255
nssa no-import-route
```

### R12
```
ospf 1 router-id 12.12.12.12
asbr-summary 172.16.64.0 255.255.192.0
import-route rip 1          # 对 RIP 网络的重发布
area 0.0.0.2
network 172.16.80.0 0.0.15.255
nssa no-summary             # 配置 NSSA 区域
```

## 四、实验测试

**1. 联通测试**
- R1 ping R9
- R1 ping R12
- R1 ping R5 的环回接口

**2. 特殊区域划分**
- 查看 R3 的 LSDB 表，可见三类 LSA 消失，只剩一条三类缺省路由
- 查看 R9、R12，可见已产生 NSSA

**3. 路由汇总**
查看 R4 的路由信息，确认路由汇总生效。

## 五、实验结果与要点
1. **合理划分网段，实现子网汇总**：根据网络规模确定子网数量，选择合适子网掩码，规划每个子网的 IP 范围，在路由器上正确配置接口与掩码，从而减少路由条目、提高路由效率。
2. **规划特殊区域，确保邻居关系建立**：根据需求选择 Stub 或 NSSA，区域内所有路由器**统一配置**特殊区域参数，避免因配置不当导致邻居关系建立失败。
3. **注意宣告 IP，避免配置错误**：确保宣告的 IP 与接口地址一致，避免重复宣告造成冲突，定期检查 OSPF 路由信息。
4. **配置不规则区域，进行多进程双向重发布**：在边界路由器上创建多个 OSPF 进程，配置重发布规则，将路由信息正确传递到其他进程。
5. **配置特殊区域后下发默认路由**：在边界路由器上配置默认路由并设置合适优先级，保证不规则区域的路由信息完整。

> 说明：原文中的 MD5 认证密码为占位密文，重写时统一以 `<密码>` 表示，实际配置时应替换为真实一致的密码。