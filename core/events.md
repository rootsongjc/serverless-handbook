# 事件驱动

Serverless 应用是由事件驱动的，当应用观察的事件源中有情况发生时会触发响应的函数执行，函数执行后会到达某个状态，就像状态机一样，随着一系列的事件发生会触发函数顺序或并行运行，这里就会涉及到事件数据结构、传递与工作流的规范。

由 CNCF Serverless 工作组制定的 [CloudEvents](https://cloudevents.io) 是以通用格式描述事件数据的规范，以提供跨服务、平台和系统的互操作性。

Workflow 用于表示一系列事件或函数运行结果触发状态变更并运行相应函数的流程，该流程可能看起来如下图所示。

![Workflow 示例（图片来自 Workflow - Version 0.1）](https://tva1.sinaimg.cn/large/006y8mN6ly1g7nkd73taxj30m90ffgm6.jpg)

CNCF Serverless 工作组的 Workflow 子组制定了供应商中立的 Workflow 规范，用于定义用户用于指定或描述其 serverless 应用程序流的格式或原语。
