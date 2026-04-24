---
title: WSL2安装常用软件与1panel+docker
published: 2026-04-24
pinned: false 
description: WSL2安装1panel与docker
image: ./images/firefly2.avif
category: 安装教程
tags: ["wsl","linux"]
author: yhx  
draft: false 
comment: true 
--- 

# 1.更新系统并安装基础软件

```
sudo apt update && sudo apt upgrade -y&&sudo apt install -y neofetch nano git unzip vim curl wget tmate && neofetch
```

# 2.ssh

打开终端，输入以下命令安装SSH：

```
sudo apt-get install openssh-server。
```

安装完成后，启动SSH服务：

```
sudo service ssh start。
```

重启SSH服务，使配置生效：

```
sudo service ssh restart。
```

# 3.1panel官方脚本

```
bash -c "$(curl -sSL https://resource.fit2cloud.com/1panel/package/v2/quick_start.sh)"
```

# 4.科技lion一键脚本

```
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

# 5.科技lion一键脚本(国内)

```
curl -sS -O https://kejilion.pro/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```
