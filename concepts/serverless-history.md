# Serverless 的历史

Serverless 架构是云的自然延伸，为了理解 serverless，我们有必要回顾一下云计算的发展。

## IaaS

2006 年 AWS 推出 EC2（Elastic Compute Cloud），作为第一代 IaaS（Infrastructure as a Service），用户可以通过 AWS 快速的申请到计算资源，并在上面部署自己的互联网服务。IaaS 从本质上讲是服务器租赁并提供基础设施外包服务。就比如我们用的水和电一样，我们不会自己去引入自来水和发电，而是直接从自来水公司和电网公司购入，并根据实际使用付费。

EC2 真正对 IT 的改变是硬件的虚拟化（更细粒度的虚拟化），而 EC2 给用户带来了以下五个好处：

- **降低劳动力成本**：减少了企业本身雇佣IT人员的成本
- **降低风险**：不用再像自己运维物理机那样，担心各种意外风险，EC2 有主机损坏，再申请一个就好了。
- **降低基础设施成本**：可以按小时、周、月或者年为周期租用 EC2。
- **扩展性**：不必过早的预期基础设施采购，因为通过云厂商可以很快的获取。
- **节约时间成本**：快速的获取资源开展业务实验。

以上说了是 IaaS 或者说基础设施外包的好处，当然其中也有弊端，我们将在后面讨论。

以上是 AWS 为代表的公有云 IaaS，还有使用 [OpenStack](https://www.openstack.org/) 构建的私有云也能够提供 IaaS 能力。

## PaaS

PaaS（Platform as a Service）是构建在 IaaS 之上的一种平台服务，提供操作系统安装、监控和服务发现等功能，用户只需要部署自己的应用即可，最早的一代是 Heroku。Heroko 是商业的 PaaS，还有一个开源的 PaaS——[Cloud Foundry](https://www.cloudfoundry.org/)，用户可以基于它来构建私有 PaaS，如果同时使用公有云和私有云，如果能在两者之间构建一个统一的 PaaS，那就是“混合云”了。

在 PaaS 上最广泛使用的技术就要数 [Docker](https://www.docker.com/) 了，因为使用容器可以很清晰的描述应用程序，并保证环境一致性。管理云上的容器，可以称为是 CaaS（Container as a Service），如 [GCE（Google Container Engine）](https://cloud.google.com/container-engine/)。也可以基于 [Kubernetes](https://kubernetes.io)、[Mesos](http://mesos.apache.org/) 这类开源软件构件自己的 CaaS，不论是直接在 IaaS 构建还是基于 PaaS。

PaaS 是对软件的一个更高的抽象层次，已经接触到应用程序的运行环境本身，可以由开发者自定义，而不必接触更底层的操作系统。

## Serverless

![Serverless 历史](https://tva1.sinaimg.cn/large/006y8mN6ly1g7m8s6kob0j30z40e0756.jpg)

上图来自 [CNCF Serverless Whitepaper v1.0](https://gw.alipayobjects.com/os/basement_prod/24ec4498-71d4-4a60-b785-fa530456c65b.pdf)。

Serverless 的商业化产品有：
- AWS lambda
- Google Cloud Function
- Microsoft Azure Cloud Functions
- Huawei Function Stage
- 其他

### 第一个 Serverless 平台

历史上第一个 Serverless 平台可以追溯到 2006 年，名为 Zimki（该公司已倒闭），这个平台提供服务端 JavaScript 应用，虽然他们没有使用Serverless 这个名词，但是他们是第一个“按照实际调用付费”的平台。第一个使用 Serverless 名词的是 [iron.io](https://iron.io/)。

## 总结

Serverless 实际发展已经有 10 年之久，而随着以 Kubernetes 为基础的的云原生应用平台的兴起，serverless 再度成为人民追逐的焦点。