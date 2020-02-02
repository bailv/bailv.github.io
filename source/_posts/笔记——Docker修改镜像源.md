---
title: 笔记——Docker修改镜像源
date: 2020-02-02 17:48:58
tags:
categories:
---

## 1. 当前命令携带`--registry-mirror`参数

例：

```bash
docker run hello-world --registry-mirror=https://docker.mirrors.ustc.edu.cn
```

### 2. 使用 json 配置文件

配置文件默认路径`/etc/docker/daemon.json`，非默认路径需要修改 `dockerd`的`–config-file`，在该配置文件添加一下内容：

```json
{
  "registry-mirrors": [
    "https://md4nbj2f.mirror.aliyuncs.com",
    "http://registry.docker-cn.com",
    "http://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com"
  ]
}
```

修改后需执行以下命令

```bash
# 重载配置文件
sudo systemctl daemon-reload

# 重启docker
sudo systemctl restart docker
```
