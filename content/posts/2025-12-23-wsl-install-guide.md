+++
# ====================================================
# 文章元数据
# ====================================================

# 基础信息
title = "Windows 11 将 WSL + Ubuntu 22.04 安装并迁移到 E 盘"
date = "2022-05-01T15:02:05+08:00"
lastmod = "2025-12-23T21:30:00+08:00"
draft = false

# 作者与版权
author = "zpasser"
notByAI = true

# SEO 优化
description = "本文详细介绍如何在 Windows 11 上安装 WSL2 并将 Ubuntu 22.04 迁移到其他盘符，解决 C 盘空间不足问题。包含完整的启用、安装、导出、导入和验证步骤，适合 Windows 用户学习使用 WSL。"
keywords = ["WSL", "WSL2", "Ubuntu 22.04", "Windows 11", "Linux", "系统迁移"]

# 分类与标签
categories = ["技术调试"]
tags = ["WSL", "Ubuntu", "Linux", "Windows 11", "系统配置", "磁盘迁移"]

# 内容展示
hideOnIndex = false
toc = true

# 功能开关
math = false
mermaid = false
+++

## 一、为什么要迁移

* Microsoft Store 安装的Ubuntu默认在C盘，太占空间
---

## 二、启用并安装 WSL（Windows 11）

### 1️⃣ 打开 PowerShell（管理员）
* 系统设置
控制面板->程序->启用或关闭 windows 功能，开启 Windows 虚拟化和 Linux 子系统（WSL2)以及Hyper-V

![image](https://i.ibb.co/qYFS2Wjj/image.png)

勾选完成后，Windows11 会自己下载些东西，并提示你重启。等电脑彻底重启完以后，进行后续操作


```powershell
wsl --install
```


---

### 2️⃣ 验证 WSL 状态

```powershell
wsl --status
```

确保看到：

* Default Version: **2**

如未设置：

```powershell
wsl --set-default-version 2
```

---

## 三、从 Microsoft Store 安装 Ubuntu 22.04

### 1️⃣ 打开 Microsoft Store

搜索并安装：

> **Ubuntu 22.04 LTS**

安装自己需要的版本即可

---

### 2️⃣ 启动一次 Ubuntu

* 创建 Linux 用户（如 `user`）
* 让系统完成初始化

验证版本(linux 内执行)：

```bash
lsb_release -a
```

应显示：

```
Ubuntu 22.04 LTS
jammy
```

---

## 四、查看当前 WSL 安装位置（此时仍在 C 盘）

### 1️⃣ 查看发行版名称

```powershell
wsl -l -v
```

通常为：

```
Ubuntu-22.04
```

---

### 2️⃣ 查看真实存储路径（关键）

```powershell
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Lxss /s
```

找到对应发行版，查看：

```
BasePath    REG_SZ    C:\Users\<用户名>\AppData\Local\Packages\...
```

👉 此路径下的 `LocalState\ext4.vhdx`
就是 Ubuntu 的真实存储位置。

---

## 五、将 Ubuntu 22.04 迁移到 E 盘（核心步骤）

### 1️⃣ 关闭 WSL

```powershell
wsl --shutdown
```

---

### 2️⃣ 导出 Ubuntu 为 tar 文件

⚠️ **必须是 tar 文件，不能是目录**

```powershell
mkdir E:\WSL
wsl --export Ubuntu-22.04 E:\WSL\ubuntu2204.tar
```

---

### 3️⃣ 注销（删除）原发行版注册

⚠️ 不会影响 tar 备份

```powershell
wsl --unregister Ubuntu-22.04
```

---

### 4️⃣ 导入 Ubuntu 到 E 盘（决定最终位置）

```powershell
wsl --import Ubuntu-22.04 E:\WSL\Ubuntu-22.04 E:\WSL\ubuntu2204.tar --version 2
```

说明：

* `Ubuntu-22.04`：发行版名称
* `E:\WSL\Ubuntu-22.04`：最终存储目录
* `ubuntu2204.tar`：导出的系统镜像
* `--version 2`：确保使用 WSL 2

---

## 六、验证迁移是否成功（非常重要）

### 1️⃣ 查看 WSL 列表

```powershell
wsl -l -v
```

期望结果：

```
Ubuntu-22.04   Stopped   2
docker-desktop Stopped   2
```

---

### 2️⃣ 启动 Ubuntu

```powershell
wsl -d Ubuntu-22.04
```

验证系统版本：

```bash
lsb_release -a
```

---

### 3️⃣ 确认真实存储位置

```powershell
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Lxss /s
```

确认看到：

```
BasePath    REG_SZ    E:\WSL\Ubuntu-22.04
```

👉 说明：

* `ext4.vhdx` 已位于 **E 盘**
* 迁移 **完全成功**

注意，安装后的wsl 默认无法使用系统代理，可通过获取Windows IP的方式设置代理
```powershell
WIN_IP=$(grep nameserver /etc/resolv.conf | awk '{print $2}')
export http_proxy=http://$WIN_IP:7890
export https_proxy=http://$WIN_IP:7890

```

---




## 九、总结

> **WSL 的"安装位置"本质是 ext4.vhdx 的位置**
> **唯一安全的迁移方式是 `wsl --export / --import`**
