---
layout:     post
title:      OMNeT++ Simulation tutorial
subtitle:   OMNeT++
date:       2019-11-10
author:     Richie
header-img: img/post-OMNeT++diagram1.jpg
catalog: true
tags:
    - OMNeT++
---




# 1.	OMNet++ 介绍
OMNet++提供一个C++库，它由仿真内核及一些用来创建仿真组件（简单模块和信息）的工具类（如随机数生成，统计收集，拓扑发现等）；组装和配置这些组件的基础设施（NED语言，ini文件）；运行时用户接口或仿真环境（TKenv,Cmdenv）；一个用来设计，运行和评估仿真的IDE环境；实时仿真的扩展接口；MRIP，并行的分布式仿真，数据库连接等等这些组成。除此之外，OMNet++ 本身并不提供任何特定用于计算机网络仿真，系统架构仿真和任意其它领域的组件；具体的仿真是由一些仿真模型和框架如Mobility Framework或INET Framework来支持，这些模型独立于OMNet++开发，并有自己的发布周期。
# 2.	消息、链路、门(gates)
当模块接收一个消息时,模块的”本地仿真时间”前进。消息能够从其他的模块或从相同的模块抵达(自身的消息用于执行定时器)。 
每个连接(也称之为链接)被创建成一个单一层次的模块层次:在一个复合模块中,可以连接相应的两个子模块的门,或一个子模块的门和一个复合模块的门。 
OMNet++提供了一个基于组件的架构，模型是由可重用的组件或模块组成的。模块之间可以通过gates（在其它系统中称为ports，即端口）进行连接，以构成复合模块。
门是模块的链接点。模块间起点和终点就是门。OMNet++支持单向链接，因此有输入和输出门。门是模块的输入/输出接口，消息通过输出门发送出去，通过输入门进行接收。 
支持门向量:一个门向量包含多个单一门。
模块描述的gates域:列出其名字即可声明门。空的方括号对[]表示门向量。向量的元素从0开始编号。  
```
simple NetworkInterface
  parameters: 
  //...  
  gates:  
  in: fromPort, fromHigherLayer;  
  out: toPort, toHigherLayer;  
```

```
simple RoutingUnit  
  parameters: 
  //...  
  gates:  
  in: output[];  
  out: input[];  
```  
门向量的大小在被用作复合模块的部件时给定。因此，每个模块实例的门向量大小不同。

# 3.    .ned
每个仿真模型是一个复合模块类型的实例。这一层次（组件和拓扑）由NED文件来处理。
NED文件只定义了模型的结构（拓扑），其行为和模块参数的某个子集则是开放的，行为是通过在简单模块相关联的C++代码来定义的，在NED文件中的参数赋值首先进行，而那些没有赋值的的参数则可以在ini文件中赋值（也就是说，在NED中的参数值不能被ini文件中的值覆盖）。如果最后仍然有一些参数没有被赋值，则会在运行时以交互的方式取得。

1)简单块由simple关键字来声明的。
简单模块是其他(复合)模块的基本构建块，简单模块类型由名称标识，简单模块通过声明参数和门来定义，一个名为sink的组件可以用NED来描述：
```
simple Sink
{
    parameters:
        @display("i=block/sink");
        @signal[packetReceived](type=cPacket);
        @statistic[packetReceived](title="packets received"; source=packetReceived; record=count,"sum(packetBytes)","vector(packetBytes)"; interpolationmode=none);
    gates:
        input in[];
}
```
这个组件也是下面的NED模型VlanEtherHostFullLoad中的组成部分。

2)复合模块则是用module关键字来声明的。
复合模块的定义类似于简单模块的定义:有gates域和parameters域,它还有两附加的域submodules和connections。
一个名为VlanEtherHostFullLoad的复合模块在NED文件中描述如下：
```
module VlanEtherHostFullLoad
{
    parameters:
        @display("i=device/pc2;bgb=363,398");
        @networkNode();
        @labels(node,ethernet-node);
        *.interfaceTableModule = default("");
        eth.mac.queueModule = "^.^.trafGenQueueApp";
    gates:
        inout ethg @labels(EtherFrame-conn);
    submodules:
        trafGenQueueApp: EtherTrafGenQueue {
            @display("p=192,69");
        }
        sink: Sink {
            @display("p=84,69");
        }
        eth: <default("VlanEthernetInterface")> like IEthernetInterface {
            parameters:
                @display("p=142,283,row,150;q=txQueue");
        }
    connections:
        eth.upperLayerIn <-- trafGenQueueApp.out;
        eth.upperLayerOut --> sink.in++;
        eth.phys <--> ethg;
}
```
所有的域(parameters, gates, submodules, connections) 都是可选的。通常复合模块参数是用于传递给子模块，对子模块的参数初始化的。

基于这些模块则可以构成一个NED网络。并用network关键字来表示它可以通过自身运行。
例如构成一个MulticastNetwork:

```
network MulticastNetwork
{
//... 
    submodules:
//... 
    connections:
//... 
}
```

# 4.    .ini
ini文件的其中一个功能是定义仿真某个特定的网络（因为可以在NED文件中定义一个或多个network.endnetwork定义。同时，也可以动态地指定载入的NED文件（即默认不载入），为模块参数赋值，指定仿真的运行时间和用于生成随机数的种子，以及需要收集的统计量，或者用不同的参数设置来建立几个实验。
一个omnetpp.ini的样例如下：
```
[General]
network = inet.examples.inet.broadcast.UDPBroadcastNetwork

**.client.numApps = 1
**.client.app[0].typename = "UdpBasicApp"
**.client.app[*].destAddresses =  "10.0.1.255"
**.client.app[0].destPort = 1000
**.client.app[0].messageLength = 100B
**.client.app[0].startTime = 10s
**.client.app[0].sendInterval = 1s

**.numTargets = 3
**.target[*].numApps = 1
**.target[*].app[0].typename = "UdpSink"
**.target[*].app[0].localPort = 1000

**.nonTarget.numApps = 1
**.nonTarget.app[0].typename = "UdpSink"
**.nonTarget.app[0].localPort = 1000

*.R2.ipv4.ip.directBroadcastInterfaces = "eth0"
```
由上可以看到，可以在为模块参数赋值时使用通配符。在NED文件中的参数赋值首先进行，而那些没有赋值的的参数则可以在ini文件中赋值（也就是说，在NED中的参数值不能被ini文件中的值覆盖）。如果最后仍然有一些参数没有被赋值，则会在运行时以交互的方式取得。

# 5.    .XML参数 
许多模块需要描述比简单模块参数更复杂的输入。然后输入这些参数至一个外部配置文件,然后让模块读文件,处理文件。在一串参数中通过文件到达模块。 
从3.0版的OMNet++就包含了支持XML配置文件。OMNet++包括了XML解释器(LibXML, Expat,等),读取和验证文件类型定义的文件(如果XML文档包括一个DOCTYPE),缓存文件(如果多个模块引用,只加载一次),通过一个XPath子集符号来选择文档的一部分,在类DOM的对象树中显示内容。
这种机制可以通过NED参数类型xml和xmldoc()操作访问。可以通过xmldoc()操作指定xml类型模块参数至一个具体的XML文件(或XML内的一个元素)。可以从NED和omnetpp.ini指定xml参数。 

# 6.    仿真输出 
仿真过程中会有向量输出和标题输出，默认文件名为omnetpp.vec和omnetpp.sca，尽管omnetpp.ini可以指定输出为不同的文件名，仍需要在简单模块中进行编码以让其拥有记录仿真结果的能力，所以在他人编写的仿真中可能并不创建这些文件。
一个输出向量文件包含若干输出向量，每一个向量都是一个 (timestamp,value) 对。输出向量可以存储例如关于时间的队列长度，接收数据包时端到端的延迟，丢弃的数据包或者信道吞吐量—任意在简单模块中的编程所定义的统计量都可以。并且，可以在omnetpp.ini中配置输出向量：即可以启用或者关闭记录某个输出向量，或者用一定的仿真时间间隔来进行限制。可以通过查看cOutVector对象的C++的源代码来查看一个简单模块可以获取的输出向量。
输出向量根据时间来捕获行为，而输出标量文件则包含总的统计量：发送包的数目，丢弃包的数目，平均的端到端延迟，吞吐量的峰值等。可以在recodScala() 方法的调用 ，特别是在一个简单模块类中的finish()方法中查看输出标量。
输出向量可以用Plove程序来绘图，而输出标量则可以使用Scalars程序来绘图。

# 7.    随机数 
OMNeT++默认的随机数生成器是 Mersenne Twister，种子可以自动获取，或者在omnetpp.ini中定义。OMNeT++ 支持很多种概率分布，并且在NED和C++中都是可用的(在 3.2 版本中有 14 个连续的分布和 6 个离散的分布, 具体可以查看 API doc)。非常量的模块参数可以用随机变量来赋值，如 exponential(0.2), 它表示C++代码会在每次读取参数时获得一个不同的数字；这是指定随机流量源的一种很方便的方法。( 常量参数也能用表达式如exponential(0.2)来赋值,但它只会被计算一次并且不再改变)。

# 8.    用C++编写模型
简单模块其实是C++类，从cSimpleModule继承子类，重定义一个虚成员函数，并通过Define_Module()宏来将新的类注册到OMNeT++中。

模块间主要使用消息传递进行通讯，而timers(timeouts)也处理模块发送给自身的消息。消息应当是cMessage类或者是其子类，消息将被传递到模块的 handleMessage(cMessage *msg) 方法，在这里应当添加你的代码。几乎你想定义的模块行为都在handleMessage() 之中定义，因此这段代码块可能会很长，所以将其重构为其它成员函数例如命名为processTimer(),processPacket()是一个好主意（可以将activity() 方法视为handleMessage()的一个替换，但是在实践中最好不要这么做）。

你可以使用send(cMessage *msg, const char *outGateName) 方法来向其它模块传递消息，对于无线仿真和一些情况下，则可以更方便地直接传递消息到其它模块而不用在NED文件中建立连接。这可以由sendDirect(cMessage *msg, double delay, cModule *targetModule, const char *inGateName)方法来完成。Self-messages (that is, timers can be scheduled) 可以通过scheduleAt(simtime_t time, cMessage *msg) 进行传送，并且在它们失效前可以通过cancelEvent(cMessage *msg) 来取消。这些函数都是cSimpleModule类的成员，在文档中可以找到更多的相关信息。

基本的cMessage类包含几个数据成员，最重要也最实用的是name,length,message”kind”(一个int成员)。其它数据成员则存储此消息最近发送/调度的信息：arrival time,arrival gate等。为了进行协议仿真，cMessage可以封装另外一个cMessage对象，可以查看它的encapsulate(cMessage *msg) 方法进行了解。这些方法也能逐步地修改域的长度。如果你需要在消息中运送更多的数据，其它数据成员可以通过继承来添加。然而，你并不需要手工编写新的C++类，可以更方便地在一个.msg文件中定义，从而让OMNet++( opp_msgc 工具) 来为你生成C++类。生成的C++文件会拥有_m.h,m.cc的后缀。.msg文件支持进一步的继承，组合，数组成员等等，并且它拥有可以让在C++类中自定义的语法。一个消息文件的例子如下：
message NetworkPacket { fields: int srcAddr; int destAddr; }

OMNet++经常用来进行网络协议的仿真，cMessage有一个叫control info的域，它包含额外的信息来促进协议层间的通讯。例如，当应用层发送数据（一个消息对象）到TCP来传输时，它能在包含socket标志符的信息中附加一小段控制信息(一个对象)。或者，当TCP发送一个TCP segment到IP层传输时，它附加包含目标IP地址的控制信息，也可能是其它的一些选项，如TTL。控制信息也能发送到其它方向（向上），以便向上一层标记源IP地址或者TCP连接。控制信息是通过cMessage的 setControlInfo(cPolymorpic *ctrl) 和 removeControlInfo() 方法来处理的。

其它cSimpleModule模块中需要重定义的虚成员函数是initialize()( 主要的初始化工作在这里进行，因为在构造函数的调用过程中模型也正在构建)，initialize(int stage) 和多阶段初始化的int numinitStages() const(这在一个模块的初始化代码依赖于其它已经初始化完毕的模块时很有用)，finish()来记录总的结果（析构函数不适宜用于这种用途）。如果你想要模块感知参数（参数可以交互式地在GUI中或由其它模块改变，因此你需要重新读取这个参数）在运行时的改变，则需要重定义handleParameterChange(const char *paramName)方法。

简单模块的NED参数可以使用使用par(const char paramName) 方法来读取，在典型的情况下，这个过程会在initialize()中进行，并将这些值存储在模块类的数据成员中。par()返回一个cPar对象的引用，这个引用可以用C++的语法或调用对象的doubleValue()方法等这些途径转为合适的类型（long,double,const char,etc）。除了基本的类型，也可以是XML文件：参数可以赋值到XML文件中，它能用以DOM的对象树的形式向C++代码展示。

消息传递并不总是模块间通讯的最好方法。例如，如果你设计了一个用于收集统计的模块（常使用全局变量），通过消息来传递统计量的更新，不如将统计模块作为一个C++对象，并调用它的用于此目的的公共成员函数（如updateStatistics(…)）来得简便。

对方法的直接调用有一些技术细节。首先你需要查找其它模块：cModule的parentModule() 和submodule(const char*name)方法可以查找到与当前相关的模块，而simulation.moduleByPath(const char path) 可以通过一个绝对的路径名在全局查找一个模块，一旦你拥有其它模块的指针，你需要将它转化为实际的类型(i.e. to StatisticsCollector from cModule*)，这是由拥有与C++的dynamic_cast相同语法的check_and_cast来完成，但如果转换不成功或指针为NULL时它会抛出一个错误。public的方法应当将Enther_Method()或者Enter_Method_Silent(…) 放置在顶端，它们启用了GUI的动画调用，同时进行了一些与临时上下文切换相近的行为（在这里并不深究，但如果想要在方法中进行消息处理这是必须的）。INET框架广泛地使用了方法调用来访问模块，如RoutingTable, InterfaceTable, NotificationBoard等。
INET的NotificationBoard在一些仿真中很有用，它通过对信息的产生者和消费者进行解耦，在它们之间转发变化的notifications或者事件的notifications，从而支持几个模块间的信息共享（NotificationBoard宽泛地基于blackboard 区域，但在实践中，仅转发notifications比存储blackboard中发布的信息更简单有效。进一步地，notifications可能包含实际的数据或者指向数据的指针的拷贝）。

对于调试，模块中打印一些必要的信息是很关键的。在OMNet++中使用ev<< （类似于printf和cout）将输出写到可以进行过滤的GUI窗口中，同时模块用图标的颜色或文件标签及tooltips来展示状态也有助于减少调试的时间；这个可以通过在执行时改变display string来完成（查看cModule’s displayString()）。通过调用bubble(const char*text)方法，可以在模块的上方看到一个短暂出现的“气泡”或“气球”，这也是很有帮助的。另外一种调试的辅助是WATCH(variableName) 宏及其变种 (WATCH_VECTOR, 等)，它能让你在Tkenv GUI中查看变量的值（you’ll find watched variables in the “Contents” tab of the module’s inspector)。
为了记录输出向量，你应当添加一个cOutVector对象到类中作为一个数据成员（或者用new来创建），设置name string作为输出向量的名字，然后持续调用它的record(double value)方法来记录数字。输出标量最好编写在模块（请查看cModule中的 recordScalar(const char *name, double value) 方法）的finish() 函数中，可以基于计数器作为模块的数据成员。也可以计算基本的统计和图表：请参考cStdDev, cDoubleHistogram and cLongHistogram classes 这些类。
NED文件描述静态的拓扑，但也能动态地创建模块和连接。这在网络拓扑不是以NED文件（如普通文本文件，Excel sheet，database等）存储，或者你想在运行时动态地创建和删除模块的情况下很有用。在前一种情况下，一般可以通过Awk,Perl,Python,Ruby,TCL或在文本编辑中一系列的查找/匹配操作来将数据转化为NED。然而，当你决定以C++来创建动态模块时，对一个样本NED文件调用nedtool并查看生成的_n.cc文件会很有帮助。



