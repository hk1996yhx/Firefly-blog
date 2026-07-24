---
title: 安装VMware并汉化
published: 1970-01-02
pinned: false
description: 显示在首页上的文章的简短描述
image: ./images/firefly3.avif
category: 文章分类
tags: ["文章标签"]
author: yhx
sourceLink: 文章内容的来源链接或参考 https://www.baidu.com
draft: true 
comment: true  
---   
VMware Workstation Pro 现在已经可以通过 Broadcom 官方免费获取。

为了方便佬友们能快速下载我就直接贴链接了 成功登录后直接打开[官方VMware下载页](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Workstation%20Pro&freeDownloads=true)就能到下载界面。

需要汉化文件的佬友可以跳过安装的步骤直接看对应的内容哦

## 下载官方最新版

### 1. 注册或登录

打开 [Broadcom 官网登录页](https://profile.broadcom.com/) 完成注册或登录。

![register-flow|690x381](https://cdn3.ldstatic.com/original/4X/a/d/c/adc8dd36a220489edf162177ed26a6ee768d39f1.jpeg)


### 2. 进入下载页

完成注册并且成功登录后，打开 [官方VMware下载页](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Workstation%20Pro&freeDownloads=true)。

图里用 `VMware Workstation Pro for Windows 25H2u1` 做示例，实际下载时选最新版本。

![download-flow|690x366, 100%](https://cdn3.ldstatic.com/original/4X/2/2/b/22b31fefe898721eda0f387f9c2db6f102ae2b39.webp)

## 安装 VMware Workstation Pro

下载完成后双击运行下载好的 VMware Workstation Pro 安装程序开始安装

- VMware Workstation Pro 25H2/25H2u1 目前不支持「简体中文」界面，所以安装界面是英文的。

- 安装完成后可选择[手动汉化](#汉化)。

![install-flow|78x500, 100%](https://cdn3.ldstatic.com/original/4X/2/8/3/283664d5c8ea15a068ff153ad12d84a7a1974afc.webp)

## 汉化

官方25H2版本界面默认是英文。要显示中文，可以使用我提供的 `zh_CN` 语言包。

汉化文件取自 VMware Workstation 17 Pro 语言文件。

[Github](https://github.com/Hermione027/VMware-Workstation-Pro-25H2-zh_CN)
适用版本

- `VMware-Workstation-Pro-25H2`

- `VMware-Workstation-Pro-25H2u1`

- 其他版本请自行测试

### 1. 下载并解压语言包

先到 [汉化文件下载页](https://github.com/Hermione027/VMware-Workstation-Pro-25H2-zh_CN/releases/tag/Luo) 下载 `zh_CN.zip`。

或者点击这里下载也是可以的 [zh_CN.zip|attachment](upload://tBHXIPre6t5tOe5RkKdbTLDoDA8.zip) (1.8 MB)

打开 VMware 安装目录下的 `Message` 文件夹。

把下载好的 `zh_CN.zip` 解压后，复制解压后的文件到 `Message` 目录下的 `zh_CN` 文件夹，如果 `zh_CN` 文件夹不存在，就先手动创建一个。

![hanhua-1|690x426](https://cdn3.ldstatic.com/original/4X/b/3/6/b36575e8e01c4a26a04f85a3a96f0a05e2a6df9c.webp)

### 2. 打开配置文件

按 `Win + R` 输入

```ini

%APPDATA%\VMware\preferences.ini

```

打开配置文件。

![hanhua-2|399x230](https://cdn3.ldstatic.com/original/4X/d/7/6/d76dea982dba864228f850bdabb6931fd54de206.webp)

### 3. 添加配置

在 `preferences.ini` 文件末尾添加：

```ini

pref.locale = "zh_CN"

```

保存后重启 VMware。

![hanhua-3|690x457](https://cdn3.ldstatic.com/original/4X/8/6/e/86e253da139999d143a81d5071b6a588d41508fd.webp)

### 4. 检查结果

重启后如果界面已经变成中文，就说明汉化成功。

如果重启后还是英文，通常是这几个原因：

- `preferences.ini` 没保存成功

- `zh_CN` 文件夹放错位置

- 没有重启 VMware

- 版本不适用

![hanhua-success|690x428](https://cdn3.ldstatic.com/original/4X/d/d/c/ddc56cafa298a400823a841cc983009a0b985fe1.webp)
