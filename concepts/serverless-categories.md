# Serverless 的分类

Serverless 不如 IaaS 和 PaaS 那么好理解，因为它通常包含了两个领域 BaaS（Backend as a Service）和 FaaS（Function as a Service）。

### BaaS

BaaS（Backend as a Service）后端即服务，一般是一个个的 API 调用后端或别人已经实现好的程序逻辑，比如身份验证服务 Auth0，这些 BaaS 通常会用来管理数据，还有很多公有云上提供的我们常用的开源软件的商用服务，比如亚马逊的 RDS 可以替代我们自己部署的 MySQL，还有各种其它数据库和存储服务。

### FaaS

FaaS（Functions as a Service）函数即服务，FaaS 是无服务器计算的一种形式，当前使用最广泛的是 AWS 的 Lambada。

现在当大家讨论 Serverless 的时候首先想到的就是 FaaS，有点甚嚣尘上了。FaaS 本质上是一种事件驱动的由消息触发的服务，FaaS 供应商一般会集成各种同步和异步的事件源，通过订阅这些事件源，可以突发或者定期的触发函数运行。

![服务端软件的运行环境](https://tva1.sinaimg.cn/large/006y8mN6ly1g7pfbgktr9j30n60bower.jpg)

传统的服务器端软件不同是经应用程序部署到拥有操作系统的虚拟机或者容器中，一般需要长时间驻留在操作系统中运行，而 FaaS 是直接将程序部署上到平台上即可，当有事件到来时触发执行，执行完了就可以卸载掉。

下面是 Function-as-a-Service 全景图（图片来自 [Github](https://github.com/amyers1793/FunctionasaServiceLandscape))

![FaaS Landscape](https://tva1.sinaimg.cn/large/006y8mN6ly1g7pfbwfjfvj30zk0k0mzk.jpg)

### Serverless Landscape

CNCF Serverless WG 维护了一份 Serverless Landscape，可以帮助大家快速直观的了解 Serverless 生态系统。

![CNCF Serverless Landscape](https://tva1.sinaimg.cn/large/006y8mN6ly1g7oqya95mzj31b50u0n99.jpg)

此图的详细信息请访问 <https://s.cncf.io> 或 <https://github.com/cncf/wg-serverless>。