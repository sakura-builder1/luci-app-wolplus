# luci-app-wolplus（网络唤醒++）

<br>中文 | [English](README_en.md)

一款更方便的**网络唤醒（Wake on LAN）**插件 —— 在 LuCI 里管理待唤醒的主机，一键开机。

> 🤖 **本次 24.10 适配由 AI 完成**（ucode 后端 + JS 前端重写）。

## 🙏 致谢与来源

本项目改编自 **[@sundaqiang](https://github.com/sundaqiang/openwrt-packages/tree/master/luci-app-wolplus)** 的
`luci-app-wolplus`，在原版基础上适配 **OpenWrt 24.10**（新版 LuCI 的 JS/ucode 架构）。

原版采用 Apache-2.0 授权，本项目**沿用同一协议**，并保留原作者署名。

## ✨ 特性

- 🖥️ **主机列表管理** —— 增删改、拖动排序
- 🔍 **自动补全 MAC** —— 从局域网主机表（`luci-rpc` 的 `getHostHints`）里直接选，不用手敲
- 🎛️ **指定网卡** —— 多网卡机器可指定从哪块网卡发唤醒包
- ⚡ **一键唤醒** —— 点一下按钮就发包（底层用 `etherwake`）
- 🌏 **中文界面** —— 自带翻译
- 🔧 **适配 OpenWrt 24.10** —— ucode 后端 + 新版 LuCI JS 前端

## 📋 适用场景

- 想远程开机的**台式机 / NAS / 服务器**
- 软路由上管理一堆待唤醒设备
- 家里多台电脑，懒得走过去按电源键 😄

## 📦 安装

### 方式一：编译成 ipk（推荐）

在 OpenWrt 源码根目录执行：

```bash
# 1. 进入 package 目录，克隆插件
cd package
git clone https://github.com/sakura-builder1/luci-app-wolplus.git

# 2. 回到源码根目录
cd ..

# 3. 更新一下 feeds（首次编译需要）
./scripts/feeds update -a && ./scripts/feeds install -a

# 4. 只编译这个插件
make package/luci-app-wolplus/compile V=s
```

编译产物在：

```
bin/packages/<架构>/base/luci-app-wolplus_*.ipk
```

拷到路由器安装：

```bash
scp bin/packages/x86_64/base/luci-app-wolplus_*.ipk root@192.168.1.1:/tmp/
ssh root@192.168.1.1
opkg install /tmp/luci-app-wolplus_*.ipk
/etc/init.d/rpcd restart          # 重要！否则菜单不显示
```

### 方式二：集成进固件

```bash
make menuconfig
```

进入：

```
LuCI
  └── Applications
        └── <M> luci-app-wolplus
```

> 💡 依赖会自动带上：`luci-base` `etherwake` `rpcd` `ucode` `ucode-mod-fs` `ucode-mod-uci`

然后正常编译固件：

```bash
make -j$(nproc)
```

## 🎯 使用方法

### 1. 打开 LuCI

```
服务 → Wake on LAN +
```

### 2. 添加待唤醒主机

| 字段 | 说明 |
|---|---|
| **名称** | 随便起，比如「书房台式机」 |
| **MAC 地址** | 目标机器的网卡 MAC（可从下拉里选，会自动列出局域网已知设备）|
| **网卡** | 从哪块网卡发包（多网卡时选，单网卡随便）|

点「**添加**」新增一条，可以拖动手柄排序。

### 3. 唤醒

点该主机右侧的 **唤醒按钮** 即可 ✅

底层执行的是：

```bash
etherwake -D -i <网卡> <MAC>
```

### 4. 命令行验证（可选）

```bash
# 看看包发出去了没
tcpdump -i br-lan -c 3 udp port 9
```

## ⚙️ 配置文件

`/etc/config/wolplus`

```uci
config macclient
	option name '书房台式机'
	option macaddr 'AA:BB:CC:DD:EE:FF'
	option maceth 'br-lan'
```

## 🏗️ 工作原理

```
LuCI 前端 (wolplus.js)
        │  rpc: luci.wolplus / awake
        ▼
ucode 后端 (luci.wolplus)
        │  读取 uci 配置
        ▼
etherwake -D -i <网卡> <MAC>     ← 发送魔术包
        ▼
目标机器网卡收到魔术包 → 开机 ✅
```

## 🔧 兼容性

| OpenWrt 版本 | 状态 |
|---|---|
| 24.10 | ✅ 已适配并测试 |
| 23.05 | ⚠️ 需要旧版 LuCI（Lua/CBI）|
| 25.12 | ⚠️ 需要适配 `apk` |

## 📄 License

Apache-2.0

沿用原版协议。原作者 [@sundaqiang](https://github.com/sundaqiang/openwrt-packages)，
本适配版由 AI 编写。
