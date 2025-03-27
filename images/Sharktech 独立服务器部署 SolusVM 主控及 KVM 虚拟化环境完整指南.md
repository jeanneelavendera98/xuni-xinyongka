# Sharktech 独立服务器部署 SolusVM 主控及 KVM 虚拟化环境完整指南

## 前言：为什么选择 SolusVM？

作为 OnApp 旗下的成熟虚拟化管理平台，SolusVM 1.x 系列凭借以下优势成为众多主机商的首选方案：
- 与 WHMCS 等财务系统无缝对接
- 完善的 KVM/Xen/OpenVZ 虚拟化支持
- 丰富的行业文档和技术支持资源

👉 [【点击查看】2025年最新 Sharktech 优惠码及特价云服务器方案汇总](https://bit.ly/Sharktech)

## 环境准备

### 硬件要求
| 组件类型       | 配置要求                          |
|----------------|-----------------------------------|
| 主控服务器     | 2核CPU/4GB内存/50GB存储（VPS即可）|
| KVM被控服务器  | 独立服务器+IPMI管理/多个公网IP段  |

### 系统要求
- **必须使用纯净系统**（不可安装cPanel/宝塔等面板）
- 推荐 CentOS 6 Minimal 版本（CentOS 7存在兼容性问题）
- 基础软件包安装：
  bash
  yum -y update
  yum -y install wget unzip screen lrzsz
  

## 授权与证书配置

### 授权购买
- 主控端：$10/月（含30天试用）
- 被控端：$2.5/月/终端

### SSL证书方案
1. **付费证书**：替换默认证书文件
   bash
   /usr/local/svmstack/nginx/ssl/ssl.crt
   /usr/local/svmstack/nginx/ssl/ssl.key
   
2. **Let's Encrypt**（推荐）：
   bash
   curl https://get.acme.sh | sh
   acme.sh --issue -d cp.example.com -w /usr/local/solusvm/www/.verification
   

## 主控端安装流程

1. 执行安装脚本：
   bash
   wget https://files.soluslabs.com/install.sh
   sh install.sh
   
2. 选择选项 `1`（仅安装主控程序）
3. 完成后的关键信息：
   - 控制面板访问地址
   - 默认管理员凭证
   - SSL证书配置路径

## KVM被控端配置

### 网络架构设计
mermaid
graph LR
    A[公网IP段1] -->|Bridge| B(br0)
    A2[公网IP段2] -->|Bridge| B
    C[内网IP段] -->|Bridge| D(intbr0)
    B --> E[KVM虚拟机]
    D --> E

### 关键配置步骤
1. 创建网桥配置文件示例：
   bash
   # /etc/sysconfig/network-scripts/ifcfg-br0
   DEVICE=br0
   TYPE=Bridge
   ONBOOT=yes
   BOOTPROTO=static
   IPADDR=56.56.56.34
   NETMASK=255.255.255.252
   GATEWAY=56.56.56.33
   

2. 必须修改的内网参数：
   ini
   ; /usr/local/solusvm/data/config.ini
   domain_simple_internal_network = true
   

## 虚拟机管理实战

### 模板同步流程
1. 从官方源下载系统模板
2. 创建 Media Group
3. 设置自动密码生成和网络配置
4. 同步到节点服务器

### 典型问题排查
- **网络不通**：检查`brctl show`输出
- **虚拟机无法启动**：确认SELinux已禁用
- **IP分配异常**：验证IP Blocks配置范围

## 性能优化建议
- 启用`host-passthrough`CPU模式
- 合理设置内存气球驱动
- 使用VirtIO磁盘和网卡驱动

## 商业部署注意事项
- 建议购买 Sharktech 独立服务器时选择：
  - 硬件RAID配置
  - 多个IP段的订购选项
  - 带外管理(IPMI)功能

👉 [【限时特惠】Sharktech 高配独立服务器专属折扣通道](https://bit.ly/Sharktech)

## 常见问题解答

**Q：能否主控和被控安装在同一服务器？**
A：KVM/Xen需分开安装，OpenVZ可通过"Master with OpenVZ"实现

**Q：如何添加额外的内网IP？**
A：需在虚拟机内手动创建网络配置文件，例如：
bash
DEVICE=eth1
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.0.0.2
NETMASK=255.255.255.0

## 总结
本教程详细演示了在Sharktech独立服务器环境部署SolusVM的全流程，重点攻克了网络配置等难点。实际部署时建议：
1. 严格按照步骤操作
2. 做好各环节的配置备份
3. 充分测试网络连通性
4. 参考官方文档进行深度定制