# CNCF Serverless WG

CNCF Serverless WG 成立于 2017 年中。当前的主要成果有：

- [CNCF Serverless Whitepaper V1.0](https://github.com/cncf/wg-serverless/tree/master/whitepapers/serverless-overview)：阐明 serverless 技术概况、生态系统状态、为 CNCF 的下一步动作做指导。
- [Serverless Landscape](https://github.com/cncf/landscape#serverless)：展示目前的 serverless 生态系统。
- [CloudEvents 规范](https://cloudevents.io)：定义最小化的事件通用属性，于 2017 年 12 月启动，2018 年 5 月进入 CNCF Sandbox
- [Workflow 规范](https://github.com/cncf/wg-serverless/blob/master/workflow/spec/spec.md)：函数编排

关于该工作组的详细信息请访问 <https://github.com/cncf/wg-serverless>。

## 下一步工作

CNCF 技术监督委员会（TOC）会考虑：

- **鼓励更多的 serverless 技术供应商和开源开发人员加入 CNCF**，以共享想法并相互借鉴创新。例如，保持 “Serverless Landscape” 文档中列出的开源项目更新，并维护功能矩阵。
- **通过建立可互操作的 API 来培育开放的生态系统**，并通过供应商承诺和开源工具确保可互操作的实施。在平台提供商和第三方开发人员库创建者的帮助下，从事类似于 CSI 和 CNI 新的互操作性和可移植性工作。其中一些可能值得创建自己的 CNCF 工作组，或者可以作为 Serverless 工作组的倡议而继续。例如：
  - **事件**：定义通用事件格式和 API 以及元数据。可以在 [Serverless WG  Github 存储库](https://github.com/cncf/wg-serverless/tree/master/proposals)中找到一些初始建议。
  - **部署**：利用既是 Serverless 提供者又是现有 CNCF 成员，成立一个新的工作组，探索可能采取的小步骤，以协调一组通用的功能定义元数据。例如：应用程序定义清单，例如 [AWS SAM](https://github.com/awslabs/serverless-application-model) 和 [OpenWhisk Packaging Specification](https://github.com/apache/incubator-openwhisk-wskdeploy/tree/master/specification#openwhisk-packaging-specification)。

- **跨不同提供商的 Serverless 平台运行函数 WorkFlow**。有许多使用场景，它们超出了触发单个函数的单个事件的范围，并且涉及到依次执行或并行执行，并由事件的不同组合 + 函数的返回值触发的多个函数的工作流。在工作流的上一步中。如果我们可以定义一组通用的构造，开发人员可以使用它们来定义其用例工作流，那么他们将是
  能够创建可在不同的 Serverless 平台上使用的工具。这些构造指定事件和函数之间的关系/交互作用，工作流中的函数之间的关系/交互作用以及如何将信息从一个函数传递给下一步骤的函数等。例如 AWS Step Function Constructs 和 Huawei Function Graph/Workflow Constructs。
- **建立开源工具的生态系统**，以加速开发人员采用的速度，探索令人关注的领域，例如：
  - 仪器仪表（Instrumentation）
  - 可调试性（Debugability）
- **教育**：提供一组设计模式，参考架构以及通用的新用户词汇表。
  - **术语表**：以发布的形式维护术语表，并确保工作组文档始终使用这些术语。
  - **用例**：维护用例列表，按通用模式分组，以创建共享的高级词汇表。支持以下目标：
    - 对于不使用 Serverless 平台的开发人员：加深对常见用例的了解，确定良好的切入点。
    - 对于 Serverless 提供者和库/框架作者，促进他们考虑共同需求。
  - CNCF GitHub 存储库中的示例应用程序和开源工具，偏重于强调互操作性方面或链接到每个提供商的外部资源。

- 提供有关如何评估 Serverless 架构相对于 CaaS 或 PaaS 的功能和非功能特性的指南。可以采用决策树的形式，也可以从 CNCF 项目系列中推荐一套工具。
- 开始一个流程，例如来自 Serverless WG 和 Storage WG 的 CNCF 输出（针对上面引用的建议文档），在 GitHub 中以 Markdown 文件保存，可以随着时间的推移对其进行协作维护，鉴于此领域的创新速度这一点尤其重要。