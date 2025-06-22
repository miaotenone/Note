#### 模型组成
**Reactor（反应器）**
管理所有感兴趣的事件（通常是网络socket上的可读、可写、错误、关闭等）。
与底层的多路复用接口打交道（如 epoll_wait 、 select 、 poll 等）。
当事件准备好后，它把事件“分发”给相应的处理器（Handler）执行处理逻辑。
**Event Demultiplexer（事件分离器）**
这是操作系统层面的接口或对此的封装，用来“收集”所有socket事件并统一返回就绪事
件给Reactor。
在Linux上，最典型的实现就是 epoll 。你可以视其为Reactor和操作系统之间的中间
件。
**Handler（事件处理器）**
一个Handler对应特定的事件或socket连接。
它会实现若干回调函数，如“当可读时如何处理数据”“当可写时如何发送数据”“当出错或
关闭时该做哪些清理”等。
Handler的实现跟业务逻辑紧密相关，比如EchoHandler、HttpHandler、或者游戏服中的
RoomHandler等等。
**Acceptor（可选）**
对服务器端来说，还需要专门“接受新连接”的角色。
当监听socket上有新连接到来时，就会触发Acceptor的回调，进行 accept() 操作，并把
新的连接socket交给其他Handler进行后续管理。

一个最小化的单Reactor场景大致如下：
1. 初始化：创建Reactor对象并把监听socket注册上；
2. 事件循环：进入死循环，Reactor在 epoll_wait 阻塞；
3. 事件到来：某些fd就绪时，Reactor获取它们；
4. 调用回调：根据fd找到其对应Handler，调用 handleRead() 或 handleWrite() 等函数；
5. 处理完成：Handler可能会更新所关心的事件（例如，从只读变为可写或相反），或者移除/关闭连接；
6. 重复：处理完后回到 epoll_wait ，等待下一轮事件。
![[reactor模型流程.png]]
![[Reactor系统类图.png]]

![[EventDemultiplexer类图.png]]

