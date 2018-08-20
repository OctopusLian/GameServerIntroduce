## 一、数据表示的基础  
#### 什么是数据表示？  
数据是信息的载体。  

数据表示是一组操作，可以描述、显示、操作信息。  

 

#### 数据表示的要素  
IDL - 接口描述语言  

- IDL是用来描述软件组件接口的一种计算机语言。IDL通过一种中立的方式来描述接口，使得在不同平台上运行的对象和用不同语言编写的程序可以相互通信交流；  

Data - Operation - 数据操作支持  

- serialize（序列化）  
- deserialize（反序列化）  
- visualize（可视化）  
- transform（转化）  
- compression（压缩）：数据运行时。  
 

Version Control - 版本控制支持  

- 数据可以有不同版本，且版本间可按照一定规则兼容
 

#### 业界现状  
- Google Proto Buffer (Protobuf)
- Apache Thrift Binary Protocol
- Tencent Data Representation (TDR)


 

#### Protobuf In Action  


Protobuf具体的使用  

通过IDL语言去定义一个.PROTO文件，然后PROTOBUF会对各个平台提供PROTO C这么一个编译器，然后PROTO C编译器我们可以指定我要生成对应的C#的DR的表示还是对应的C++的DR表示。  

一个学习的机会：编译一个自己熟悉的语言，比如用PROTOBUF编一个点.c和.cpp文件出来，它里面怎么操作数据的，怎么压缩数据的，整个步骤都是可以看到的。  

 

## 二、数据表示在游戏开发中的应用
#### 游戏开发 - 协议（以天刀为例）  
- 交互内容复杂：多重嵌套结构体/二进制数据。  
- 协议数量巨大：4000+条协议定义；13000+结构体定义。  
- 变更频繁  
- 网络流量巨大  
 


#### 游戏开发 - 异构系统协议交互  
多类型终端协议交互  

- server：Linux / x86 / C++  
- client：PC / iPhone / Android；C++ / C# / lua  
 
游戏开发 - 协议版本兼容  


 

#### 游戏开发 - 协议流量优化  
流量优化：通过DR提供的数据压缩功能进行流量优化。  

 

#### 游戏开发 - 数据存储的特点  
- 数据结构复杂：每个玩家的存储涉及到成千上万个字段；  
- 数据结构不稳定：每次版本更新有可能会新增字段或扩大原有字段；  
- update > read > insert > delete。  
 

#### 游戏开发 - 数据存储设计  
Key - Value数据存储模型  

- Key - 角色ID  
- Value - 二进制角色数据  
- MySQL Blob  

使用DR管理Blob数据  

- 数据序列化/反序列化  
- 数据兼容  
- 数据压缩  
![](https://github.com/OctopusLian/GameServerIntroduce/blob/master/3_%E6%B8%B8%E6%88%8F%E5%BC%80%E5%8F%91%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E8%A1%A8%E7%A4%BA/DR%E7%AE%A1%E7%90%86Blob%E6%95%B0%E6%8D%AE.png)  


我们存盘的时候就是把玩家的数据先serialize成我们的DR描述的中间格式，然后存到DB里面，DB里面再读取出来然后再恢复成我们的runtime格式