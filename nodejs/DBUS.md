## DBUS

### 什么是DBUS

D-Bus是一种消息总线系统，是进程间通信(IPC)的系统。DBUS实质上是一个`适用于桌面应用`的进程间通信机制，即IPC-适用于在同一台机器，不适合于Internet的IPC机制。

从总体结构上讲，它分为三层：

- libdbus: 运行库，或称为协议的实现。它准许两个应用程序互相连接并交换消息。
- 一个消息总线守护程序dbus-daemon,建立在libdbus 之上，多个应用程序可以连接。守护程序可以将消息从一个应用程序路由到零个或多个其他应用程序。
- 基于特定应用程序框架的绑定或包装程序库。例如 `libdbus-glib` & `libdbus-qt` & `dubs-python`... 这些包装库是大家使用的API，因为它们简化了D-Bus编程的细节。libdbus旨在作为高级绑定的低级后端。libdbus API的大部分仅对绑定实现有用。

Libdbus 仅支持一对一连接，就像原始网络套接字一样。但是不是通过连接发送字节流，而是发送message。消息具有标识消息类型的标头和包含数据有效负载的正文。libdbus还抽象出所使用的数据传输方式（套接字与其他方式），并处理诸如身份验证之类的细节。



##  D-Bus 架构

![D-Bus 性能分析- daw1213 - 博客园](https://img2018.cnblogs.com/blog/1077750/201809/1077750-20180918155146337-1175366103.png)

D-Bus的拓扑结构看起来像是与其连接的每个进程共享的软件总线，因此任何进程都可以与另一个进程进行通信，或者组播/广播事件。 该拓扑以中心辐射方式实现，即所有进程都连接到守护程序。 在D-Bus上运行的任何进程都有一个具有唯一总线地址的总线连接。

上述段落表达了：D-Bus的拓扑结构- 所有进行都需要连接到daemon上，获取D-Bus消息。

D-Bus support two basic IPC operations - signal and method. Signal implements a publisher / subscriber model and it is usually used to deliver certain event to other application. Method implements an one-to-one caller / callee model, which is the same as **RPC**. All operations will eventually generates bus messages and be transported through Unix domain socket. Message is **routed** by the daemon. For method call, routing is based on source and destination. For signal, routing is based on various match rules on the attributes of message, such as message type, bus name, object path, interface name, etc.

#### 原生对象和对象路径

所有使用D-Bus 的应用程序都包含一些对象，当一个D-Bus 连接收到了一条消息时，该消息是被发往一个对象而不是整个应用程序。在开发中程序框架定义着这样的对象，例如：JAVA,GObject,等。在D-Bus中成为native object。

对于底层的D-Bus 协议，并不理会这些native object,它们使用的是一个叫做object path 的概念。通过object path, 高层编程可以为对象实例进行命名，并准许远程应用引用它们。这些名字看起来像文件系统路径，例如一个对象可能叫做"/org/kde/kspread/sheets/3/cells/4/5"。简单的说： 一个应用创建对象实例进行D-Bus 的通信，这些对象实例都有一个名字，命名方式类似于路径,例如 /com/mycompany,这个名字在全局(session或者system)是唯一的，**用于消息的路由**。

#### 方法和信号

每一个对象有两类成员： 方法和信号。方法和Java中方法是相同概念，方法是一段函数代码，带有输入和输出。信号是广播给所有兴趣的其他实体，信号可以带有数据payload。

在D-Bus中有四种类型的消息： 方法调用(method calls)、方法返回(method returns)、信号(signals)和错误(errors)。要执行D-BUS对象的方法，您需要像对象发送一个方法调用消息。它将完成一些处理（就是执行了对象中的Method，Method是可以带有输入参数的。）并返回，返回消息或者错误消息。信号的不同之处在于它们不返回任何内容：既没有“信号返回”消息，也没有任何类型的错误消息。



#### 接口Interface

每一个对象支持一个或者多个接口，接口是一组方法和信号，接口定义一个对象实体的类型。D-Bus 对接口的命名方式，类似： org.freedesktop.introspectable。开发人员通常使用编程语言中类的名字作为接口名字。











#### 参考

[D-Bus性能分析](https://www.cnblogs.com/brt3/p/9614632.html)















































