## 一、服务器架构概念解析  
#### 1，什么是“服务器架构”  
对服务器软件&硬件&运行的一体化规划  

框架结构：分层分块。  
构建技术选择：编程语言；通信方式；存储技术。  
运行质量：运行环境；部署工具方法；更新方案。  
 

## 二、案例讲解：分布式服务架构设计演讲——MMORPG（大型多人在线角色扮演）《轩辕传奇》  
服务器架构_分区多世界  
#### 1，运营视角  
世界与世界是隔离的  
世界之间的互通方式：跨服、转服、合服  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/%E8%BD%A9%E8%BE%95%E4%BC%A0%E5%A5%87%E8%BF%90%E8%90%A5.png)  


#### 2，运维视角  
SET部署：每开一组服就增加一组机器，部署一套进程。  
#### 3，客户端视角  
TCLS组件：显示所有服务器列表。  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/%E8%BD%A9%E8%BE%95%E4%BC%A0%E5%A5%87%E5%AE%A2%E6%88%B7%E7%AB%AF.png)  

#### 4，服务器视角  
一组服：一套进程。  
 

轩辕服务器为什么要这么多进程和机器_多维度切分  
#### 1，分区多世界原型v1  
一个大区包含多台物理机  
一台物理机仅运营一个游戏世界  
一个游戏世界对应一个游戏进程  
问题：跨世界共享的功能？  

游戏账号登陆
客户端版本升级
游戏大区列表
游戏账号信息
#### 2，公共服分离：将各游戏世界公共的功能分离部署到公共服上。  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/%E5%85%AC%E5%85%B1%E6%9C%8D%E5%8A%A1%E5%99%A8.png)  


问题：公共服的单点故障

“主-备-从”模式：主节点出问题切到从节点（热切换），从节点出问题切到备份节点。  
#### 3，按“接入-逻辑-存储”分离  
分离业务逻辑（不稳定的）与基础功能（稳定的）  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/%E5%88%86%E7%A6%BB%E4%B8%9A%E5%8A%A1%E9%80%BB%E8%BE%91.png)  


问题：逻辑处理和持久化数据在一个物理机上  

DB的文件IO会拖慢整体系统（百万级玩家，几百个G数据）  
进程运行中每日输出大量的日志（几个G数据）  
物理机故障时DB可能会丢失  
#### 4，按重要性分离逻辑计算与持久化存储的部署  
方案：数据库独立部署&热备，log服分离  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/%E5%88%86%E7%A6%BB%E4%B8%8E%E9%83%A8%E7%BD%B2.png)  

#### 5，分区多世界原型v2  
cluster级服务：整个游戏一组  
world级服务：每个游戏世界一组  
各组服均包含接入、逻辑、存储（DB/DR/LOG分离）  
#### 6，继续分离公共服  
公共服 分离“服务器列表”、“版本升级”、“账号信息”等功能  
#### 7，切分轩辕逻辑进程  
现状：所有的鸡蛋都在一个篮子里  

所有玩家都在一个进程上；  
好处：一个特性可以方便地操作到所有玩家；  
风险：一个特性的BUG可能会影响所有玩家。  
所有特性都在一个进程上；  
风险：特性的不断引入会使该进程稳定性和服务质量降低。  
方案：切分xysvr，让多个scene分别服务于一些用户，world负责拉取数据。并协调控制多scene。  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/%E8%BD%A9%E8%BE%95%E4%BC%A0%E5%A5%87%E9%80%BB%E8%BE%91%E8%BF%9B%E7%A8%8B.png)  


#### 8，分区多世界原型v3  
cluster级、world级服务按功能不能切分。  
#### 9，如何做切分_参考原则  
为可扩展性：一组服一套进程；（SET部署）  
为可运维性：一组服一套机器；  
为可靠性：弱相关的功能可分离；  
提高更新便利性：将频繁更新的部分分离；  
按服务重要性切分：如支付系统独立；  
按服务特点切分：接入、存储、逻辑分离；  
切分稳定的（基础功能）与不稳定（业务逻辑）的。  
 

## 三、案例讲解：面向运营的架构设计
#### 1，接入与负载  
多tconnd：分离下行广播包的压力。  

多scene：可扩展性（单服承载人数）。  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/%E6%8E%A5%E5%85%A5%E4%B8%8E%E8%B4%9F%E8%BD%BD.png)  


#### 2，可用性  
resume机制  

minidump  

避免因为coredump导致resume时间过长；  
支持输出出错时基本上下文：调用栈、寄存器。  
#### 3，在线控制  
reload机制：资源、配置文件热加载；  

GM系统：管理游戏运行内容。  

#### 4，过载保护  
请求频率控制：按功能模块控制：移动、技能...  
DB频率控制：按业务模块配额；区分优先级-存盘优先；  
边界情况检测：某类内存分配占用量超出阀值；某请求/time执行时间过长。  
#### 5，DB设计  
表格设计：按QQ号分表；简单列+统一blob格式（meta）  
数据升级：旁路进程，自动分批升级。  
 

## 四、案例讲解：旁路服务
版本升级  

tcus  

支持升级策略：强制、推荐、后台、预下载、自定义；  
支持灰度更新。  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/tcus.png)


tdir  

显示服务器列表，繁忙程度（参照在线人数）  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/2_%E6%B8%B8%E6%88%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A6%81/tdir.png)  



目录服务  

账号服务与账号登录

角色登录  

游戏过程  

运营支持  

脏字过滤  
验证码  
 

## 五、参考资料
[让我们谈谈游戏服务器开发（上）](http://gad.qq.com/article/detail/21401)

[游戏服务器架构演进(完整版](https://my.oschina.net/u/1859679/blog/1438724?p=2)