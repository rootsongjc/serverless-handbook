# CloudEvents

> 本文翻译并节选自 [CloudEvents - Version 1.0-rc1](https://github.com/cloudevents/spec/blob/master/spec.md)，在原文的基础上有所改动。

CloudEvents 是供应商中立的事件数据定义规范。

## 概览

事件无处不在。但是，事件产生者倾向于以不同的方式描述事件。

缺少通用的事件描述方式意味着开发人员需要不断重新学习如何使用事件。这也限制了库、工具和基础架构帮助跨环境（例如 SDK、事件路由器或跟踪系统）传递事件数据的潜力。以事件数据实现的可移植性和生产率上受到阻碍。

CloudEvents 是以通用格式描述事件数据的规范，以提供跨服务、平台和系统的互操作性。

事件格式指定如何使用某些编码格式序列化 CloudEvent。支持这些编码兼容 CloudEvents 的实现必须遵守相应事件格式中指定的编码规则。所有实现都**必须**支持 [JSON](https://github.com/cloudevents/spec/blob/master/json-format.md) 格式。

有关该规范背后的历史、开发和设计原理的更多信息，请参阅 [CloudEvents Primer](https://github.com/cloudevents/spec/blob/master/primer.md) 文档。

## 术语

下图是对 CloudEvents 中的部分术语的关系描述。

![CloudEvents 部分术语](https://tva1.sinaimg.cn/large/006y8mN6ly1g7n7176el8j317g0luaai.jpg)

**Occurrence（发生）**

“Occurrence（发生）”是在软件系统运行期间捕获的事实陈述。这可能是由于系统发出的信号或系统正在观察的信号，由于状态变化，计时器完成或任何其他值得注意的活动而发生的。 例如，由于电池电量不足或虚拟机将要执行重启计划，设备可能会进入警报状态。

**Event（事件）**

“Event（发生）”是表示发生（Occurrence）及其上下文的数据记录。Event 从 Event 生产者（source）路由到感兴趣的 Event 使用者。Event 路由可以基于 Event 中包含的信息，但是 Event 不会标识特定的路由目的地。Event 包含两种类型的信息：表示 Occurrence 的 Event Data 和提供有关 Occurrence 上下文信息的 Context 元数据。一次发生可能会导致多个事件。

**Producer（生产者）**

“Producer（生产者）”是创建描述 CloudEvent 的数据结构的特定实例、过程或设备。

**Source（源）**

“Source（源）”是发生事件的上下文。在分布式系统中，Source 可能包含多个 Producer。 如果 Source 不知道 CloudEvent，则外部 Producer 将代表 Source 创建 CloudEvent。

**Consumer（消费者）**

“消费者”接收事件并对其采取行动。Consumer 使用上下文和数据执行某些逻辑，这可能导致新 Event 的发生。

**Intermediary（中介）**

“Intermediary（中介）”接收包含 Event 的消息，目的是将 Event 转发到下一个接收者，该接收者可能是另一个 Intermediary 或 Consumer。Intermediary 的典型任务是根据 Context 中的信息将 Event 路由到接收者。

**Context（上下文）**

Context（上下文）元数据封装在“Context Attributes（上下文属性）”中。 工具和应用程序代码可以使用此信息来标识 Event 与系统各方面或其他 Event 的关系。

**Data（数据）**

有关事件的特定于域的信息（即有效负载）。这可能包括有关 Occurrence 的信息，有关已更改数据的详细信息或更多信息。

**Message（消息）**

Event 是通过消息从源传递到目的地。

**Protocol（协议）**

可以通过各种行业标准协议（例如 HTTP、AMQP、MQTT、SMTP），开源协议（如 Kafka、NATS）或特定于平台/供应商的协议（AWS Kinesis、Azure Event Grid）传递 Message。

## Context Attributes（上下文属性）

每个符合此规范的 CloudEvent **必须**包含指定为 **REQUIRED** 的上下文属性，并且**可以**包括一个或多个 **OPTIONAL** 上下文属性。

这些属性虽然描述了事件，但设计为可以独立于事件数据进行序列化。这样就可以在目的地对它们进行检查，而不必对事件数据进行反序列化。

### 属性命名规范

CloudEvents 规范定义了到各种协议和编码的映射，随附的 CloudEvents SDK 针对各种运行时和语言。其中一些将元数据元素视为区分大小写，而其他元素则不区分大小写，并且单个 CloudEvent 可能会通过涉及协议、编码和运行时混合的多个跃点进行路由。因此，本规范限制了所有属性的可用字符集，以防止区分大小写问题或与通用语言中标识符的允许字符集冲突。

CloudEvents 属性名称**必须**由 ASCII 字符集的小写字母（“ a”至“ z”）或数字（“ 0”至“ 9”）组成，并且必须以小写字母开头。属性名称应具有描述性和简洁性，长度不得超过 20 个字符。

### 类型系统

以下抽象数据类型可用于属性。这些类型中的每一个**可以**通过不同的事件格式和在传输元数据字段中以不同的方式表示。该规范为所有实现**必须**支持的每种类型定义了规范的字符串编码。

#### REQUIRED 属性

以下属性必须出现在所有 CloudEvents 中：

- id：String
- source：URI-reference
- specversion：String
- type：String

#### OPTIONAL 属性

以下属性是可选的，**可以**出现在 CloudEvents 中。 

- datacontenttype：String
- dataschema：URI
- subject：String
- time：Timestamp

#### 扩展 Context Attributes

CloudEvents Producer **可以**在 Event 中包含其他 Context Attribute，这些属性可以在与 Event 处理相关的辅助操作中使用。

该规范对扩展属性的语义没有任何限制，但**必须**使用类型系统中定义的类型。扩展的每个定义都**应**完全定义属性的所有方面，例如属性的名称，语义含义和可能的值，甚至表明它对其值没有任何限制。新的扩展名定义**应该**使用具有足够描述性的名称，以减少与其他扩展名发生名称冲突的几率。特别是，扩展作者**应该**检查扩展文档中的已知扩展集，不仅是可能的名称冲突，还是可能感兴趣的扩展。

每个定义如何序列化 CloudEvent 的规范都将定义扩展属性的显示方式。

扩展属性必须使用与所有 CloudEvents 上下文属性相同的常规模式进行序列化。例如，在二进制 HTTP 中，这意味着它们**必须**显示为带有 `ce-` 前缀的HTTP标头。属性的规范**可以**定义一个二级序列化，其中数据在消息中的其他位置重复。

在定义了二级序列化的情况下，扩展规范还**必须**说明如果两个序列化位置的数据不同，CloudEvent 的接收者将要做什么。另外，发送者需要为 intermediary 和接收者不知道其扩展的情况做好准备，因此专用序列化版本很可能不会作为 CloudEvent 扩展属性进行处理。

许多传输支持发送者包括附加元数据的功能，例如作为 HTTP 标头。虽然未强制要求 CloudEvents 接收器处理和传递它们，但建议通过某种机制来进行处理，以使其清楚地知道它们不是 CloudEvents 的元数据。

这是一个说明需要其他属性的示例。在许多物联网和企业用例中，Event 可以在无服务器应用程序中使用，该应用程序跨多种 Event 类型执行操作。为了支持这种用例，Event Producer 将需要向 “Context Attributes” 添加其他标识属性，Event 使用者可以使用这些属性将这个事件与其他事件相关联。如果此类身份属性恰好是事件 “Data” 的一部分，则 Event 生成器还将身份属性添加到 “Context Attributes”，Event 用者可以轻松访问此信息，而无需解码和检查 Event Data。此类身份属性还可用于帮助中间网关确定如何路由 Event。

#### Event Data（事件数据）

按照术语 Data（数据）的定义，CloudEvents 可以包含有关事件的特定于域的信息。 如果存在，此信息将封装在数据中。

### Size Limits（大小限制）

在许多情况下，CloudEvents 将通过一个或多个通用 intermediary 进行转发，每个 intermediary 都可能会对转发 Event 的大小施加限制。CloudEvents 也可能会路由到受存储或内存限制的 Consumer（如嵌入式设备），因此会遇到大型单一事件。

Event 的 “Size（大小）”是其线路大小，并包括针对该 Event 在线路上传输的每个位：根据所选的 Event 格式和所选的协议绑定，传输 frame-metadata（帧元数据），event metadata（事件元数据）和 event data（事件数据）。

如果应用程序配置要求 Event 跨不同的传输进行路由或 Event 进行重新编码，则**应该**考虑应用程序使用的效率最低的传输和编码应符合以下大小限制：

- Intermediary **必须**转发大小为 64 KB 或更小的事件。
- Consumer 应该接受至少 64 KB 的事件。

实际上，这些规则将允许 Producer 安全地发布最大为 64KB 的 Event。此处的安全是指通常合理的做法是，期望所有 Intermediary 都接受并转发该事件。无论是出于本地考虑，还是要接受或拒绝该大小的事件，它都在任何特定的 Consumer 控制之下。

通常，CloudEvents 发布者**应该**通过避免将大型数据项嵌入 Event 有效负载中来保持事件紧凑，而是将 Event 有效负载链接到此类数据项。从访问控制的角度来看，此方法还允许 Event 的更广泛分布，因为通过解析链接访问 Event 相关的详细信息可实现差异化的访问控制和选择性公开，而不是将敏感的详细信息直接嵌入事件中。

## 隐私与安全

互操作性是此规范的主要推动力，要使这种行为成为可能，就需要明确提供一些信息，从而可能导致信息泄漏。

请考虑以下事项，以防止意外泄漏，尤其是在利用第三方平台和通信网络时：

**Context Attributes**

敏感信息**不应**携带和表示在上下文属性中。

CloudEvent Producer、Consumer 和 Intermediary **可以**内省并记录 Context Attributes。

**数据**

特定于域的 Event 数据应该被加密以限制对可信方的可见性。用于这种加密的机制是 Producer 和 Consumer 之间的协议，因此不在本规范的范围之内。

**传输绑定**

**应当**采用传输级安全性来确保 CloudEvents 的可信和安全交换。

## 示例

下面时候使用 Json 序列化 CloudEvent 的示例。

```json
{
    "specversion" : "1.0-rc1",
    "type" : "com.github.pull.create",
    "source" : "https://github.com/cloudevents/spec/pull",
    "subject" : "123",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "text/xml",
    "data" : "<much wow=\"xml\"/>"
}
```

## 参考

- [CloudEvents - Version 1.0-rc1 - github.com](https://github.com/cloudevents/spec/blob/master/spec.md)