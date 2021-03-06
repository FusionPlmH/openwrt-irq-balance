# OpenWrt路由器多核终端均衡脚本
## 目的
通过均衡分配用于处理中断的CPU核心，提升系统的吞吐性能
## 原理
OpenWrt默认将所有中断都分配到CPU0执行，很容易产生拥挤，因此需要调整中断与CPU的对应关系。
不过，由于程序流程的局部性，如果将中断完全分散到不同的核心中，会产生频繁的缓存交换，拖累速度。
因此，根据场景对中断的分布进行调整，才能最大化的提升效率。
## 结构
直接执行脚本即可设置irq与核心的绑定关系。
CPU核心与Mask遮罩值的对应关系如下所示：
| CPU0 | CPU1 | CPU2 | CPU3 |
| :----: | :----: | :----: | :----: |
| 1 | 2 | 4 | 8 |

如果需要同时使用多个核心，则直接将遮罩值相加取16进制，如使用CPU2与CPU3则值为C。
项目中脚本与平台的对应关系如下表所示：
| 脚本名称 | 对应平台 | 说明 |
| :----: | :----: | :----: |
| ipq40xx.sh | 高通IPQ4018/4019/... | 针对有线与无线转发优化的配置 |
| lean-ipq40xx.sh | 高通IPQ4018/4019/... | Lean固件中的默认配置 |
## 状态
目前只基于Asus RT-ACRH17测试了IPQ4019平台。
### 测试场景
使用两台间隔7米的ACRH17组建Mesh网络，主路由中运行iperf3服务器，主从路由通过QCA9984 5G进行连接，测试电脑连接于从路由的LAN1接口。
两台ACRH17均使用OpenWrt官方19.07.2固件与ath10k无线驱动。
测试电脑中运行代码`iperf3 -c 主路由IP -t 15 -P 4`。
使用从路由测试均衡脚本。
吞吐量的结果如下表所示：
| 中断绑定配置 | 速度/Mbps |
| :----: | :----: |
| OpenWrt默认 | 284 |
| 完全由CPU0执行 | 284 |
| irqbalance | 315 |
| CPU0执行无线，CPU2执行以太网 | 315 |
| Lean默认 | 362 |
| CPU2执行无线与以太网 | 385 |
