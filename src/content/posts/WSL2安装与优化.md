---
title: WSL2安装与优化
published: 2026-04-24
pinned: false 
description: 安装WSL功能
image: ./images/firefly2.avif
category: 安装教程
tags: ["wsl","linux"]
author: yhx 
draft: false 
comment: true 
--- 
### 1.使用管理员模式打开PowerShell安装WSL功能

```bash
wsl.exe --list --online
wsl --install -d Ubuntu-24.04
```

### 2.将wsl文件迁移至D盘

首先查看WSL状态

```bash
wsl -l -v
```

关闭运行中的wsl

```bash
wsl -t Ubuntu-24.04
wsl --shutdown
```

以压缩包的形式导出系统文件

```bash
wsl --export Ubuntu-24.04 E:\WSL\Ubuntu-24.04.tar
```

注销原有的linux系统

```bash
wsl --unregister Ubuntu-24.04
```

 导入系统到E盘

```bash
wsl --import Ubuntu-24.04 E:\WSL\Ubuntu-24.04 E:\WSL\Ubuntu-24.04.tar
```

### 3.自启动

按 Win + R 键打开运行对话框，输入 ```shell:startup```这会打开 Windows 启动文件夹,新建  start_wsl.vbs

```bash
Set ws = WScript.CreateObject("WScript.Shell")
ws.Run "wsl", 0
```

版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

原文链接：<https://blog.csdn.net/weixin_56029873/article/details/142439651>
