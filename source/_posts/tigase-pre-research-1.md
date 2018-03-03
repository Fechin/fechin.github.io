---
title: Tigase 集成预研
date: 2016-10-10 01:56:10
tags:
mp3: http://link.hhtjim.com/163/283846.mp3
cover: http://odwjyz4z6.bkt.clouddn.com/tigase-pre-research_xiangji_2_mini.jpg
---

### 1. 安装Tigase-server

Tigase支持多种安装方式， 包括WEB UI引导安装、Linux Console安装、解压配置安装、Windows程序安装等。这里以最便捷的安装方法`解压配置安装`为例。

#### 1.1 安装环境
`Linux非Root权限` `JDK1.8` `MySQL及Root权限`

#### 1.2 开始安装
从官方项目库：https://projects.tigase.org/projects/tigase-server/files 下载 tigase-server-7.0.4-b3844-dist-max.tar.gz 并解压
```
wget https://projects.tigase.org/attachments/download/4625/tigase-server-7.0.4-b3844-dist-max.tar.gz

tar -zxvf tigase-server-7.0.4-b3844-dist-max.tar.gz

cd tigase-server-7.0.4-b3844-dist-max
```
配置JDK， 修改JAVA_HOME="/usr/java/jdk1.8.0_101" // 你的JDK路径
```
vim etc/tigase.conf
```
执行数据库初始化脚本，注意配置参数
```
./scripts/db-create-mysql.sh
```
启动Tigase
```
./scripts/tigase.sh start etc/tigase.conf
```
#### 1.3 验证安装
常用验证工具`Spark` `Psi` `Pidgin` 更多请参考：https://xmpp.org/software/clients.html


### 2. 阅读源码
#### 2.1 部署源码
tigase-server核心源码
```
git clone https://repository.tigase.org/git/tigase-server.git
```
tigase依赖工具包
```
git clone https://repository.tigase.org/git/tigase-utils.git
```
tigase依赖XML解析工具
```
git clone https://repository.tigase.org/git/tigase-xmltools.git
```

#### 2.2 执行入口
```
/tigase-server/src/main/java/tigase/server/XMPPServer.java
```

#### 2.3 Tigase的三大模块
- **component（组件）**是tigase服务的主要模块。它使用大量的代码实现了“接收和发送stanzas（可以理解为各种各样的消息），可配置，并依据配置对大量事件做出应答”，它可以有独立ip地址。像c2s connection manager，s2s connection manager，session manager，XEP-01140外部组件连接管理，MUC-multi user char rooms；它们都是tigase的组件。

- **plug-in（插件）**在大多数情况下是处理特定的xmpp stanzas的一小段代码（相对于components那种大片大片的代码而言）。它没有自己的ip地址，处理完xmpp stanzas之后的结果是产生一个新的xmpp stanzas。plug-ins被session manager或c2s connection manager所加载。像vCard stanza处理，jabber:iq:register（用来注册新的用户帐号），presence stanza 处理（在线/忙碌/离开状态处理），jabber:iq:auth（对非sasl用户进行认证）等。

- **Connector（连接器）**是用来访问各类数据库的模块，例如访问ldap/database。有两类connector：认证数据库（校验用户名密码是否正确）connector和用户数据库（用户的联系人信息/离线消息等）connector。它们是彼此独立的，可以分别连接到不同的数据库。像JDBC database connector，XMLDB-嵌入式database connector，drupal database connector，Libresource database connector都属于Connector。

#### 2.4 扩展
```
// 如何编写一个组件
http://my.oschina.net/wjwei113/blog/375326

// 如何编写一个插件
http://my.oschina.net/wjwei113/blog/375304
```
### 3. 需求及规范

####  3.1 需求
1. 如何集成HTTP API， 如何跟IOS、Android、WEB三端集成？
2. 如何集成第三方账户系统？
3. 区分管理员和普通用户的权限。
4. 实现过滤、拦截、白名单及黑名单功能。
5. 实现集群配置、发布订阅功能。
6. 。。。。。

####  3.2 规范
1. 以开源的心态预研、开发，每一个人都是架构师。
2. 规范报文格式，扩展报文。
3. 支持功能可配，动态调控。
4. 。。。。。

