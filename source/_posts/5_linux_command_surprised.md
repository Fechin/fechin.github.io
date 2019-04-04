---
title: 5 个相见恨晚的 Linux 命令 - 终端之美
date: 2017-11-28 15:16:26
categories:
    - Linux
tags:
    - Linux
    - 命令
mp3: /static/mp3/Jam%20-%20%E4%B8%83%E6%9C%88%E4%B8%8A.mp3
cover: /static/images/CotOTsNUAAENHGa.jpg
---

### tldr（命令手册）
作为一个开发人员，会时常用到终端命令，最让人头疼的是记不住繁琐的参数。用谷哥度娘检索效率低下；通过`man`查看帮助，超长文章不易阅读。

`tldr`命令正是解决这一痛点，`tldr`是什么？从它的 GitHub 页翻译说“一个简洁的社区驱动的帮助手册”，这是对它最好的解释，根据二八原则给出命令的常用场景示例，简单易读；存放在 Github 上的命令库接受来自五湖四海的朋友提交的内容，社区驱动。

`man`命令有更详细的说明，单从实用角度讲，`tldr`才是王者。

```bash
fechinwork in ~/Desktop at 11:57:57 λ tldr tar

tar

Archiving utility.
Often combined with a compression method, such as gzip or bzip.

- Create an archive from files:
    tar cf target.tar file1 file2 file3

- Create a gzipped archive:
    tar czf target.tar.gz file1 file2 file3

- Extract an archive in a target folder:
    tar xf source.tar -C folder

- Extract a gzipped archive in the current directory:
    tar xzf source.tar.gz

- Extract a bzipped archive in the current directory:
    tar xjf source.tar.bz2

- Create a compressed archive, using archive suffix to determine the compression program:
    tar caf target.tar.xz file1 file2 file3

- List the contents of a tar file:
    tar tvf source.tar
```

小提示：支持在进 20 中语言环境下运行，通过`tldr --update`更新本地命令库。



### tree（树形目录）

当我们编写项目文档，想更直观的表达项目结构及内容的时候，这个小小的命令就可以派上用场了，它以类似于图像的树状图排列目录和文件。
```bash
fechinwork in ~/work/script/nginx2mysql at 12:43:10 λ tree -L 2
.
├── README.md
├── config.yml
├── libs
│   ├── __init__.py
│   ├── __init__.pyc
│   ├── nginx_log_parser.py
│   ├── nginx_log_parser.pyc
│   ├── simplemysql.py
│   └── simplemysql.pyc
├── logs
│   └── cdn-2016-06-04.log
├── parser.py
└── requirements.txt

2 directories, 11 files
```

小提示：支持定制层级，过滤内容等各种个性化设置。通过`tldr tree`查看具体使用示例。添加`-N`参数解决中文乱码问题。


### rlwrap（历史命令）

经常使用命令的同学一定有习惯，通过上下按键切换历史命令，但是让人头疼的是`telnet`命令不支持切换，甚至是退格删除，所以时常遇到如下尴尬场面。莫急，`rlwrap`便是用来解决这一痛点的。

通过`telnet`执行 Dubbo 接口：
```bash
> telnet 192.168.1.147 23457
> invoke com.yinyuetai.yuan.user.api.UserService.get(1) > ^[[A^[[A^[[A^[[B^[[B
# 好尴尬~
```
通过`telnet`连接 memcached 服务器：
```bash
> telnet 192.168.1.36 11211 > ^[[A^[[A^[[A^[[B^[[B
# 好尴尬~
```
什么是`rlwrap`？它是基于 readline 库，实现命令行补全和记录的包装命令。如今交互式输入是最基本的需求，Linux 正是通过 readline 这个库来记录用户的操作，实现交互式输入、自动补全、搜索等功能。对于没有支持 readline 操作的命令，`rlwrap`就是最好的伙伴了。

用法：在执行`telnet`命令前加上 rlwrap 命令即可。
```bash
fechinwork in ~/Documents at 14:43:15 λ
fechinwork in ~/Documents at 14:43:41 λ rlwrap telnet 127.0.0.1 6379
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
set product alpha
+OK
get prod
$-1
get product
$5
alpha
```
小提示：除了`telnet`还有 Oracle 系列命令需要支持 rlwrap 包装装`sqlplus`、`rman`、`asmcmd`
快捷别名：`alias telnet='rlwrap telnet'`



### script（记录会话输出）

很多时候，为了安全和备份，需要对工作内容进行保存。那么，`script`命令就是隐藏在终端的记录器，它可以记录终端会话的所有内容，形成文件。对于需要工作留痕的同学来说，`script`便是良药。
如何使用，用`script`启动它，此时它已经开始记录。完事后用`exit`退出记录，默认生成了一个叫“typescript”的文件。
```bash
fechinwork in ~ at 13:34:04 λ script
Script started, output file is typescript
fechinwork in ~ at 13:34:07 λ echo '导演，开始了吗？'
导演，开始了吗？
fechinwork in ~ at 13:34:27 λ echo '你退出自己看咯，略略~'
你退出自己看咯，略略~
fechinwork in ~ at 13:34:48 λ exit

Script done, output file is typescript
fechinwork in ~ at 13:34:51 λ cat typescript
Script started on Sat Oct 21 13:34:06 2017
fechinwork in ~ at 13:34:07 λ echo '导演，开始了吗？'
导演，开始了吗？
fechinwork in ~ at 13:34:27 λ echo '你退出自己看咯，略略~'
你退出自己看咯，略略~
fechinwork in ~ at 13:34:48 λ exit

Script done on Sat Oct 21 13:34:51 2017
```
小提示：`script`可以在什么场景下使用呢？
1、我需要把大批量视频推送到 CDN，耗时一晚上，这时通过`script`记录执行的日志，第二天对没有推送成功的做单独处理。
2、别人远程你的服务器或电脑，安全起见`script`一下。
3、与同事协同工作时，自己工作做了一半，交给另一个人来做，此时发给它你的`script`，让它接着干。
……



### autojump（一键直达）

最后一个压轴神器，也是我用的最多的命令之一。

相信多数终端用户使用频率最高的命令是`cd`、`ls`, 在我不知道切换到哪里的时候不得不`ls`确认目录名，如此反复，到达想去的目录可能要经历几次甚至十次以上的 cd，经历了多少风雨才找到我的文件。俗话说“不会偷懒的程序员不是好程序员”，如此饱受挫折那是我们的风格，于是有了 autojump 的诞生，它注定不凡。
顾名思义，autojump，自动跳转，而不是切换，因为它可以做到一键直达。

```bash
fechinwork in ~/work/script/xls2sql at 14:27:24 λ j Des
/Users/fechinwork/Desktop
fechinwork in ~/Desktop at 14:27:27 λ pwd
/Users/fechinwork/Desktop
fechinwork in ~/Desktop at 14:27:31 λ j Docu
/Users/fechinwork/Documents
fechinwork in ~/Documents at 14:27:41 λ pwd
/Users/fechinwork/Documents
fechinwork in ~/Documents at 14:27:44 λ
```
小提示：可以通过`j -s`命令查看它的数据库，以及数据库中的目录权重。

