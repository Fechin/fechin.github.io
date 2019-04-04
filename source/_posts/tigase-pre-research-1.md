---
title: Tigase 集成预研
date: 2016-10-10 01:56:10
categories:
    - Java
tags:
    - Java
    - IM
mp3: http://link.hhtjim.com/163/283846.mp3
cover: /static/images/tigase-pre-research_xiangji_2_mini.jpg
---

### 1. 安装 Tigase-server

Tigase 支持多种安装方式， 包括 WEB UI 引导安装、Linux Console 安装、解压配置安装、Windows 程序安装等。这里以最便捷的安装方法`解压配置安装`为例。

#### 1.1 安装环境
`Linux 非 Root 权限` `JDK1.8` `MySQL 及 Root 权限`

#### 1.2 开始安装
从官方项目库：https://projects.tigase.org/projects/tigase-server/files 下载 tigase-server-7.0.4-b3844-dist-max.tar.gz 并解压
```
wget https://projects.tigase.org/attachments/download/4625/tigase-server-7.0.4-b3844-dist-max.tar.gz

tar -zxvf tigase-server-7.0.4-b3844-dist-max.tar.gz

cd tigase-server-7.0.4-b3844-dist-max
```
配置 JDK， 修改 JAVA_HOME="/usr/java/jdk1.8.0_101" // 你的 JDK 路径
```
vim etc/tigase.conf
```
执行数据库初始化脚本，注意配置参数
```
./scripts/db-create-mysql.sh
```
启动 Tigase
```
./scripts/tigase.sh start etc/tigase.conf
```
#### 1.3 验证安装
常用验证工具`Spark` `Psi` `Pidgin` 更多请参考：https://xmpp.org/software/clients.html


### 2. 阅读源码
#### 2.1 部署源码
tigase-server 核心源码
```
git clone https://repository.tigase.org/git/tigase-server.git
```
tigase 依赖工具包
```
git clone https://repository.tigase.org/git/tigase-utils.git
```
tigase 依赖 XML 解析工具
```
git clone https://repository.tigase.org/git/tigase-xmltools.git
```

#### 2.2 执行入口
```
/tigase-server/src/main/java/tigase/server/XMPPServer.java
```

#### 2.3 Tigase 的三大模块
- **component（组件）**是 tigase 服务的主要模块。它使用大量的代码实现了“接收和发送 stanzas（可以理解为各种各样的消息），可配置，并依据配置对大量事件做出应答”，它可以有独立 ip 地址。像 c2s connection manager，s2s connection manager，session manager，XEP-01140 外部组件连接管理，MUC-multi user char rooms；它们都是 tigase 的组件。

- **plug-in（插件）**在大多数情况下是处理特定的 xmpp stanzas 的一小段代码（相对于 components 那种大片大片的代码而言）。它没有自己的 ip 地址，处理完 xmpp stanzas 之后的结果是产生一个新的 xmpp stanzas。plug-ins 被 session manager 或 c2s connection manager 所加载。像 vCard stanza 处理，jabber:iq:register（用来注册新的用户帐号），presence stanza 处理（在线 / 忙碌 / 离开状态处理），jabber:iq:auth（对非 sasl 用户进行认证）等。

- **Connector（连接器）**是用来访问各类数据库的模块，例如访问 ldap/database。有两类 connector：认证数据库（校验用户名密码是否正确）connector 和用户数据库（用户的联系人信息 / 离线消息等）connector。它们是彼此独立的，可以分别连接到不同的数据库。像 JDBC database connector，XMLDB- 嵌入式 database connector，drupal database connector，Libresource database connector 都属于 Connector。

#### 2.4 扩展
```
// 如何编写一个组件
http://my.oschina.net/wjwei113/blog/375326

// 如何编写一个插件
http://my.oschina.net/wjwei113/blog/375304
```
### 3. 需求及规范

####  3.1 需求
1. 如何集成 HTTP API， 如何跟 IOS、Android、WEB 三端集成？
2. 如何集成第三方账户系统？
3. 区分管理员和普通用户的权限。
4. 实现过滤、拦截、白名单及黑名单功能。
5. 实现集群配置、发布订阅功能。
6. ......

####  3.2 规范
1. 以开源的心态预研、开发，每一个人都是架构师。
2. 规范报文格式，扩展报文。
3. 支持功能可配，动态调控。
4. ......

