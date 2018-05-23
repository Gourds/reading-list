---
title: DevOps实践读书笔记
date: 2018-05-22 18:17:24
tags: [读书]
---

目前身边有三本关于DevOps的书，这本最薄，决定花个1~2天先通读一下。
update: 读完了，本身就是抱着课外读物的心态读的，总体来说实际应用价值不大，但是也是接触了一些概念，没有太多具体可操作实践的东西。个人不推荐购买阅读，这个书是公司购买的图书，所以就顺便看了。

>《DevOps实践：驭DevOps之力强化技术栈并优化IT运行
>作者:*[瑞典]Joakim Verona*。


### 随手记的

- 模块内的高内聚是可取的
- 高内聚低耦合的系统自带关注点分离
- git flow分支策略
- 版本号规范4部分（1代表主要代码变更2表示次要变更3表示修复缺陷4可以是构建号）http://semver.org
- Jenkins是Hudson构建服务器的一个fork
- 好玩的：yum install furtune-mod

### 几个概念

- KISS原则（Keep it simple stupid）

- TDD（测试驱动开发）
>- 实现测试：先编写测试后开发，将重心从编码切换到理解需求
>- 验证新写的测试会失败
>- 编写实现测试的功能
>- 验证新的测试和旧的测试会一起通过
>- 重构代码：清理代码的同时让代码更容易理解和维护

- REPL（交互式命令驱动开发）


### 部署工具

Puppet：拉模式，Puppet代理在Puppet服务器注册的同时打开一个通信通道来获取命令。定期执行科修改频率
Ansible：推模式，Server通过SSH推送期望的修改
Salt：推模式，利用ZeroMQ消息服务器，Client连接到消息服务器，监听修改通知。工作原理与Puppet类似，但速度更快
PalletOps



### 监控工具

Nagios
Munin
Ganglia
Graphite


### 日志处理

ELK


### 日志类库

log4j
logback


### 问题跟踪

Jira
Trello
Worktile









