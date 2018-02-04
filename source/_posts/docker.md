---
title: docker的安装和使用
date:
modified: 
categories:
- Linux
tags:
- docker
- Linux
---
### docker安装
在最小化安装的情况下，docker是没有安装的，所以不需要先卸载。

> 安装yum-config-manager工具

```
yum -y install yum-utils
```

> 设置Docker仓库

```
yum-config-manager --add-repo https://docs.docker.com/v1.13/engine/installation/linux/repo_files/centos/docker.repo
```

> 更新yum源并安装最新的docker

```
yum makecache fast
yum -y install docker-engine
```

> 启动docker

```
systemctl start docker
systemctl enable docker
```

> 升级docker

```
yum makecache fast
yum list docker-engine.x86_64  --showduplicates |sort -r
yum -y install docker-engine-<VERSION_STRING>
```
