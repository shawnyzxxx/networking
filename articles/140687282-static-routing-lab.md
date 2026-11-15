# 静态路由实验报告

## 文章导读
本篇是静态路由综合实验报告，在一张 `192.168.1.0/24` 网段的网络上，通过**环回接口模拟用户网段、DHCP 分配地址、Telnet 远程管理**，并完成**最简化静态路由**配置实现全网可达。适合学习华为设备静态路由与综合组网的读者。

## 一、实验需求
1. 整张网络使用 `192.168.1.0/24` 网段进行划分
2. R1、R2、R3、R4 均有**两个环回**来模拟用户网段
3. R5 使用 DHCP 为 PC 分配 IP 地址
4. R2、R3、R4、R5 开启 Telnet 功能，账号密码：`admin/admin`
5. 完成所有路由器的静态路由配置，要求使用**最简化路由表**的方式
6. 使用 R1 进行 Telnet 测试，登录其他设备
7. 实现全网可达

## 二、网段划分

**七个互联网段**

| 网段 | 地址范围 |
|---|---|
| 第一个网段 | 192.168.1.0 ~ 192.168.1.15 |
| 第二个网段 | 192.168.1.16 ~ 192.168.1.31 |
| 第三个网段 | 192.168.1.32 ~ 192.168.1.47 |
| 第四个网段 | 192.168.1.48 ~ 192.168.1.63 |
| 第五个网段 | 192.168.1.64 ~ 192.168.1.79 |
| 第六个网段 | 192.168.1.80 ~ 192.168.1.95 |
| 第七个网段 | 192.168.1.96 ~ 192.168.1.111 |

**各路由器的环回接口（模拟用户网段）**

| 设备 | 环回接口 | 地址范围 |
|---|---|---|
| R1 | LoopBack 1 / LoopBack 2 | 192.168.1.208~223 / 192.168.1.224~250 |
| R3 | LoopBack 1 / LoopBack 2 | 192.168.1.112~127 / 192.168.1.128~143 |
| R5 | LoopBack 1 / LoopBack 2 | 192.168.1.144~159 / 192.168.1.160~175 |
| R7 | LoopBack 1 / LoopBack 2 | 192.168.1.176~191 / 192.168.1.192~207 |

## 三、实验步骤

### 1. 配置环回接口（以 R1 为例）
```
[r1]interface LoopBack 1
[r1-LoopBack1]ip address 192.168.1.222 28

[r1]interface LoopBack 2
[r1-LoopBack2]ip address 192.168.1.248 28
```

查看接口与环回状态：
```
[r1]display ip interface brief

Interface                   IP Address/Mask      Physical   Protocol
GigabitEthernet0/0/0       192.168.1.1/28       up         up
GigabitEthernet0/0/1       192.168.1.62/28      up         up
GigabitEthernet0/0/2       unassigned           down       down
LoopBack1                   192.168.1.222/28     up         up(s)
LoopBack2                   192.168.1.248/28     up         up(s)
NULL0                       unassigned           up         up(s)
```
> `(l)` 表示 loopback，`(s)` 表示 spoofing，可以看到两个环回接口已正常 UP。

### 2. R5 使用 DHCP 为 PC 端分配地址
```
[r5]interface GigabitEthernet 0/0/0
[r5-GigabitEthernet0/0/0]ip address 192.168.1.78 28

[r5]interface GigabitEthernet 0/0/1
[r5-GigabitEthernet0/0/1]ip address 192.168.1.81 28

[r5]dhcp enable
[r5]ip pool r5
[r5-ip-pool-r5]network 192.168.1.80 mask 28
[r5-ip-pool-r5]gateway-list 192.168.1.81

[r5]interface GigabitEthernet 0/0/1
[r5-GigabitEthernet0/0/1]dhcp select global
```

### 3. R2、R3、R4、R5 开启 Telnet 功能
以 R2 为例，账号密码为 `admin/admin`：
```
[r2]telnet server enable
[r2]user-interface vty 0 4
```
> 其余 R3、R4、R5 采用相同的 Telnet 配置方式。

## 四、实验测试
- 查看 PC1/PC2 的基础配置，并测试 PC1 与 PC2 之间的连通性
- 完成全部静态路由配置后，使用 **R1 进行 Telnet 测试**，登录其他设备验证

## 五、实验结果
全网可达，静态路由按最简化路由表方式配置完成，Telnet 测试登录成功，实验成功。