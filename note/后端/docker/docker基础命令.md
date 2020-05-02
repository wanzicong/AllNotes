---
title: docker基础命令
date: 2020-03-08 10:49:50
tags:	
- docker
---

# Docker命令终结 vim命令

## 1. docker查看报错信息

```docker
$ docker logs --since 30m 容器id(名字)
```

## 2. 宿主机 启动Docker

```dockerfile
sysyemctl start docker 
sysyemctl restart docker 
```

## 3. docker 启动容器

```dockerfile
docker start 容器id(名字)
docker restart 容器id(名字)
```

## 4. docker 创建容器

```dockerfile
docker run -di --name=名字 -p 9000:9000 镜像名字：版本号 (不写代表是last最新版本)
```

## 5. vi + 文件名字打开 文件

```vim
vi 文件名
```

##  6 .  进入vim中 i键表示开始插入

```
i
```

## 7. shift +z +z 表示退出 + 保存

shift +z +z  {vim}

## 8. docker进入docker 容器虚拟机中

```dockerfile
docker exec -it 容器名(id) /bin/bash
```

## 9. 在docker中挂载外部文件

```dockerfile
docker cp 文件 容器名:/path/path/path
```





