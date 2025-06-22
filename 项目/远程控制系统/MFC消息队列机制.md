MFC（Microsoft Foundation Classes）消息队列机制是Windows应用程序开发中处理用户输入和系统事件的核心部分。以下是MFC消息队列机制的概述：
https://developer.aliyun.com/article/1526411
https://zhuanlan.zhihu.com/p/489889991
https://www.cnblogs.com/bcfx/articles/2932134.html
#### **消息类型**

- **队列消息**：先保存在消息队列中，由消息循环取出并分发到各窗口处理。如：WM_PAINT、WM_TIMER、WM_CREATE、WM_QUIT，以及鼠标、键盘消息等。
- **非队列消息**：绕过系统消息队列和线程消息队列，直接发送到窗口过程进行处理。如：WM_ACTIVATE、WM_SETFOCUS、WM_SETCURSOR、WM_WINDOWPOSCHANGED等。

#### **消息处理流程**

1. **消息产生**：由用户操作（如键盘输入、鼠标点击）或系统事件（如定时器触发）产生。
2. **消息入队**：消息被放入相应线程的消息队列中。
3. **消息循环**：应用程序的主线程通过消息循环（如`GetMessage`和`PeekMessage`函数）从消息队列中获取消息。
4. **消息翻译**：某些消息（如虚拟键消息）通过`TranslateMessage`函数进行翻译，转换为更具体的消息（如字符消息）。
5. **消息分发**：`DispatchMessage`函数将消息发送到相应的窗口过程函数（`WindowProc`）进行处理。
6. **消息映射**：MFC通过消息映射表将消息与特定的成员函数关联起来。当消息到达时，MFC查找消息映射表，调用相应的处理函数。
7. **消息过滤**：MFC提供了`PreTranslateMessage`函数，允许在消息分发给窗口过程之前进行预处理，用于全局快捷键的处理和特殊消息的拦截。

#### **消息队列与线程**

- 每个UI线程都有自己的消息队列，MFC的文档视图结构利用这一特性，确保消息在正确的线程上下文中处理。

#### **消息结构**

- 消息通常包含一个消息ID、窗口句柄、以及两个32位参数（`wParam`和`lParam`），用于传递与消息相关的额外信息。

通过上述机制，MFC简化了Windows消息的处理，使开发者能够专注于业务逻辑，而无需关注底层消息传递的细节。理解MFC消息机制对于编写高效、健壮的MFC应用至关重要。