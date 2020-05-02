---
title: npm创建vue项目
date: 2020-01-30 20:52:19
tags:
	- npm
	- vue

---

## 创建vue3 项目的步骤

### 第一步

安装全局的vue 脚手架（电脑安装一次就行了）

```makefile
 npm install -g @vue/cli 			
```

### 第二步

检查vue脚手架的 版本

```java
vue -V (!必须是大写的V)
```

### 第三步

创建新文件夹 ，进入文件夹执行命令

```java
vue create myVue  (!项目名称)
```

### 第四步

进行项目的初始化，选择安装插件

```
出现界面的选择， 选择手动设置
```

### 第五步

1) .  选择配置的内容（关键的一步）

```java
选择第一个 Babel
选择第第四个 Router（路由）
    
    
    
Linter/Formattor 可以选， 初学不建议选， 最后可以在项目中进行组件追加的方式
其他的组件有进行单元测试用的 可以进行添加配置
npm install 命令 就是进行组件的下载用
```

2) . 选择别的一些设置

	1. 是否使用历史的设置 -----选择 y

   	2. 其他的....  选择 y 
   	3. 选择 json
   	4. 全部默认直到结束 

### 第六步

启动安装好的vue项目

```java
npm run server
```

### 补充

安装淘宝镜像

```
 npm  install  -g  cnpm  --registry=https://registry.npm.taobao.org
```



## 启动一个项目

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).