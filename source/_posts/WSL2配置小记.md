---
title: WSL2配置小记
categories: 瞎折腾记录
toc: true
abbrlink: 31b7e97e
date: 2023-12-27 01:20:23
tags:
---

对于使用Windows进行开发的程序员来说，WSL2（Windows Subsystem for Linux 2）的引入无疑是一个革命性的工具。它提供了一个轻量级且高效的Linux内核环境，完美融合了Windows和Linux生态系统的优势。然而，随着项目复杂度的增加，默认的WSL2配置可能无法满足所有需求。这时，位于用户目录下的`.wslconfig`文件就能发挥关键作用。本文将深入探讨如何通过这一配置文件优化你的开发环境。

<!-- more -->
## 为什么需要自定义`.wslconfig`？

默认情况下，WSL2会根据宿主机的硬件资源自动分配内存、CPU等资源。但对于以下场景，这种通用配置可能不够灵活：

* 内存敏感型应用（如Docker、Java服务）可能导致WSL2过度占用资源
* 需要限制WSL2使用的CPU核心数以腾出系统资源给其他应用
* 需要调整Linux虚拟机（VM）的存储或网络配置以提升性能
* 希望在SSD上限制分页文件（Swap）写入以延长硬盘寿命

通过手动配置.wslconfig，你可以精准控制WSL2的资源分配，实现开发效率与系统稳定性的平衡。

## `.wslconfig`的相关配置

默认情况下，`.wslconfig`文件并不存在。你可能需要通过以下命令创建它：

```PowerShell
cd ~
notepad.exe .wslconfig
```

在记事本窗口中对该文件进行修改。以下是一个示例的`.wslconfig`文件：

```conf
# Settings apply across all Linux distros running on WSL 2
[wsl2]

# 限制WSL2 VM内存
memory=64GB

# 限制WSL2 VM处理器数量
processors=32

# 自定义Linux内核位置
kernel=C:\\temp\\myCustomKernel

# 额外的内核命令行参数
kernelCommandLine = vsyscall=emulate

# SWAP文件大小
swap=8GB

# SWAP文件位置
swapfile=C:\\temp\\wsl-swap.vhdx

# 使 Windows 能够回收分配给WSL2 VM的未使用内存。
pageReporting=false

# 用于指定绑定到 WSL 2 VM 中的通配符或 localhost 的端口是否应可通过 localhost:port 从主机连接
localhostforwarding=true

# WSL2 的原生嵌套虚拟化支持，可以启用KVM
nestedVirtualization=false

# 是否显示 dmesg 内容的输出控制台窗口
debugConsole=true

# 网络模式，如果值为 mirrored，则会启用镜像网络模式。默认为 NAT 模式，个人推荐使用镜像模式来方便开发，仅在Win11中可用
networkingMode=mirrored

# 强制 WSL 使用 Windows 的 HTTP 代理信息，方便挂梯子，仅在Win11中可用
autoProxy=true
```

配置文件中的路径必须添加转义反斜杠。
