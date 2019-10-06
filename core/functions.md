# 函数计算

下图是 Serverless 中的（FaaS）函数定义，从图中可以看出与容器、12 要素及 Kubernetes 的运行时设计十分契合。

![Serverless 中的函数定义](https://tva1.sinaimg.cn/large/006y8mN6ly1g7ldey3l7gj31ti0mwta9.jpg)

下图是 FaaS 中函数输入、context 及输出。

![FaaS 中的函数](https://tva1.sinaimg.cn/large/006y8mN6ly1g7ldhm7bxyj31040u0q5n.jpg)

当函数创建完成后通过函数部署流水线部署运行。

![函数部署流水线](https://tva1.sinaimg.cn/large/006y8mN6ly1g7ocpnzg6cj31xo0kcgme.jpg)

以上图片根据 CNCF Serverless Whitepaper v1.0 绘制。