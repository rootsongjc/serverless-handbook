# Workflow

> 本文翻译并节选自 [Workflow - Version 0.1](https://github.com/cncf/wg-serverless/blob/master/workflow/spec/spec.md)，在原文的基础上有所改动。

Workflow 是供应商中立的规范，用于定义用户用于指定或描述其 serverless 应用程序流的格式或原语。

许多 serverless 应用程序不是由单个事件触发的简单函数，而是由系列函数执行的多个步骤组成，而函数在不同步骤中由不同事件触发。如果某个步骤涉及多个函数，则该步骤中的函数可能会根据不同的事件触发器依次执行、并行执行或在分支中执行。为了使 serverless 平台正确执行 serverless 应用程序的函数工作流程，应用程序开发人员需要提供工作流程规范。

为了给业界提供一种标准方法，CNCF Serverless WG（Working group）成立了 workflow 子组，供用户指定其 serverless 应用程序工作流，以促进 serverless 应用程序在不同供应商平台之间的可移植性。

为此 CNCF Serverless WG workflow 小组制定了一个完整的协议，使给定的事件时间轴和工作流始终产生相同的作用。

相关使用案例请参考 [Workflow - Version 0.1](https://github.com/cncf/wg-serverless/blob/master/workflow/spec/spec.md) 中给出了一系列 use case：

- 家庭监控
- 借贷审批
- 员工旅行预订
- 流媒体视频点播
- 翻译服务评价

## 功能范围

函数工作流用于将函数编排为协调的微服务应用程序。函数工作流中的每个函数可能由来自各种来源的事件驱动。函数工作流将函数和触发事件分组到一个连贯的单元中，并描述函数的执行和以规定方式传递的信息。具体来说，函数工作流程允许用户：

- 定义 serverless 应用程序中涉及的步骤/状态和工作流程。

- 定义每个步骤中涉及的函数。

- 定义哪个事件或事件组合触发一个或多个函数。

- 定义在触发多个函数时如何安排这些函数依次执行还是并行执行。

- 指定如何在函数或状态之间过滤信息和传递事件。

- 定义在哪种错误状态下需要重试。

- 如果函数是由两个或多个事件触发的，则定义应使用什么标签/键将那些事件与相同的函数工作流实例相关联。

以下是涉及事件和函数的函数工作流的示例。使用这样的函数工作流，用户可以轻松指定事件与函数之间的交互以及如何在工作流程中传递信息。

![Workflow 示例](https://tva1.sinaimg.cn/large/006y8mN6ly1g7nkd73taxj30m90ffgm6.jpg)

使用函数工作流，用户可以定义集合点（状态）以等待预定义的事件，然后再执行一个或多个函数并继续执行函数工作流。

## Workflow 模型

可以将函数工作流（Function Workflow）视为状态的集合以及这些状态之间的转换和分支，并且每个状态可以具有关联的事件和/或功能。函数工作流可以从 CLI 命令调用，也可以在事件从事件源到达时动态触发。来自事件源的事件也可能与函数工作流中的特定状态相关联。函数工作流中的这些状态将等待一个或多个事件源中的一个或多个事件到达，然后再执行其关联的操作并进入下一个状态。其他工作流程功能包括：

- 函数的结果可用于启动重试操作或确定下一个要执行的函数或要转换到的状态。

- 函数工作流提供了一种在事件处理过程中对 JSON 事件有效负载进行过滤和转换的方法。

- 函数工作流为应用程序开发人员提供了一种在事件中指定唯一字段的方法，该字段可用于将事件源中的事件关联到同一函数工作流实例。

我们可以很自然的将函数工作流建模为状态机。以下是函数工作流的定义/规范提供的状态列表。工作流的规范称为工作流模板。工作流模板的实例称为工作流实例（workflow instance）。

- **Event State（事件状态）**：用于等待事件源中的事件，然后调用一个或多个函数以顺序或并行运行。

- **Operation State（操作状态）**：允许一个或多个函数按顺序或并行运行，而无需等待任何事件。

- **Switch State（切换状态）**：允许转换到其他多个状态（例如，不同的函数导致前一个状态触发分支/转换到不同的下一个状态）。

- **Delay State（延迟状态）**：使工作流执行延迟指定的持续时间或直到指定的时间/日期。

- **End State（结束状态）**：失败/成功终止工作流。

- **Parallel State（并行状态）**：允许多个状态并行执行。

函数工作流由工作流规范描述。

## 工作流规范

函数工作流规范定义了函数工作流的行为和操作。函数工作流规范结构应允许用户定义事件到达触发的执行函数。它应具有足够的灵活性，以涵盖从单个函数的简单调用到涉及多个函数和多个事件的复杂应用程序和各种微服务应用程序。

从高层看，函数工作流规范包括两部分：触发器定义（trigger definition）和状态定义（state definition）。

```json
{
  "trigger-defs" : [],
  "states": []
}
```

`trigger-defs` 数组（仅在存在与工作流关联的事件时才需要）是与函数工作流关联的事件触发器的数组。如果应用程序工作流中涉及多个事件，则必须在该事件触发器中指定一个用于将事件与同一工作流实例的其他事件相关联的关联令牌（correlation-token）。

状态数组（必需）是与函数工作流相关联的状态数组。

下面是  JSON 格式的函数工作流示例，其中涉及事件状态和该事件状态的触发器：

```json
{  
   "trigger-defs":[  
      {  
         "name":"OBS-EVENT",
         "source":"CloudEvent source",
         "eventID":"CloudEvent eventID",
         "correlation-token":"A path string to an identification label field in the event message"
      },
      {  
         "name":"TIMER-EVENT",
         "source":"CloudEvent source",
         "eventID":"CloudEvent eventID",
         "correlation-token":"A path string to an identification label field in the event message"
      }
   ],
   "states":[  
      {  
         "name":"STATE-OBS",
         "start":true,
         "type":"EVENT",
         "events":[  
            {  
               "event-expression":"boolean expression 1 of triggering events",
               "action-mode":"Sequential or Parallel",
               "actions":[  
                  {  
                     "function":"function name 1"
                  },
                  {  
                     "function":"function name 2"
                  }
               ],
               "next-state":"STATE-END"
            },
            {  
               "event-expression":"boolean expression 2 of triggering events",
               "action-mode":"Sequential or Parallel",
               "actions":[  
                  {  
                     "function":"function name 3"
                  },
                  {  
                     "function":"function name 4"
                  }
               ],
               "next-state":"STATE-END"
            }
         ]
      },
      {  
         "name":"SATATE-END",
         "type":"END"
      }
   ]
}
```

以下是带有操作状态的函数工作流的另一个示例：

```json
{  
   "states":[  
      {  
         "name":"STATE-ALARM-NOTIFY",
         "start":true,
         "type":"OPERATION",
         "action-mode":"Sequential or Parallel",
         "actions":[  
            {  
               "function":"function name 1"
            },
            {  
               "function":"function name 2"
            }
         ],
         "next-state":"STATE-END"
      }
   ]
}
```

### 触发器定义

`trigger-defs` 数组由一个或多个事件触发器组成。

以 JSON 格式定义的事件触发器示例如下：

```json
{  
   "trigger-defs":[  
      {  
         "name":"EVENT-NAME",
         "source":"CloudEvent source",
         "eventID":"CloudEvent eventID",
         "correlation-token":"A path string to an identification label field in the event message"
      }
   ]
}
```

### 动作定义 {#action-definition}

下面是 JSON 格式的定义。

```json
{  
   "actions":[  
      {  
         "function":"FUNCTION-NAME",
         "timeout":"TIMEOUT-VALUE",
         "retry":[  
            {  
               "match":"RESULT-VALUE",
               "retry-interval":"INTERVAL-VALUE",
               "max-retry":"MAX-RETRY",
               "next-state":"STATE-NAME"
            }
         ]
      }
   ]
}
```

- **function**：指定调用的函数。
- **timeout**：从请求发送给函数起开始计时，等待函数指定完成的时间，单位秒，必须为正整数。
- **retry**：重试策略。
- **match**：匹配的结果值。
- **retry-interval** 和 **max-retry**：当出现错误时使用。
- **next-state**：当超过 `max-retry` 限制后到转移到下一个状态。

### 状态定义

#### 事件状态（Event State）

```json
{  
   "states":[  
      {  
         "name":"STATE-NAME",
         "type":"EVENT",
         "start":true,
         "events":[  
            {  
               "event-expression":"EVENTS-EXPRESSION",
               "timeout":"TIMEOUT-VALUE",
               "action-mode":"ACTION-MODE",
               "actions":[  
               ],
               "next-state":"STATE-NAME"
            }
         ]
      }
   ]
}
```

事件状态必须将 `type` 值指定为 `EVENT`。

- **start**：是否为起始状态。可选的字段。默认为 false。
- **events**：与该事件状态相关的事件数组。
- **event-expression**：这是一个布尔表达式，由一个或多个事件操作数和布尔运算符组成。`EVENTS-EXPRESSION` 可以是 “Event1 or Event2”。到达并匹配 `EVENTS-EXPRESSION` 的第一个事件将导致执行此状态的所有操作，然后转换到下一个状态。
- **timeout**： 指定在 `EVENTS-EXPRESSION` 中等待事件的时间段。如果事件不在超时时间内发生，则工作流将转换为结束状态。
- **action-mode**：指定函数是按顺序执行还是并行执行，并且可以是 `SEQUENTIAL` 或 `PARALLEL`。
- **next-state**：指定在成功执行所有匹配事件的操作之后要转换到的下一个状态的名称。

#### 操作状态（Operation State）

```json
{  
   "states":[  
      {  
         "name":"STATE-NAME",
         "type":"OPERATION",
         "start":true,
         "action-mode":"ACTION-MODE",
         "actions":[  
         ],
         "next-state":"STATE-NAME"
      }
   ]
}
```

- **action-mode**：指定函数是按顺序执行还是并行执行，并且可以是 `SEQUENTIAL` 或 `PARALLEL`。
- **actions**：由一系列动作构成的列表，指定接收到与事件表达式匹配的事件时要执行的函数的列表。
- **next-state**：指定在成功执行所有匹配事件的操作之后要转换到的下一个状态的名称。

#### 分支状态（Switch State）

```json
{  
   "states":[  
      {  
         "name":"STATE-NAME",
         "type":"SWITCH",
         "start":true,
         "choices":[  
            {  
               "path":"PAYLOAD-PATH",
               "value":"VALUE",
               "operator":"COMPARISON-OPERATOR",
               "next-state":"STATE-NAME"
            },
            {  
               "Not":{  
                  "path":"PAYLOAD-PATH",
                  "value":"VALUE",
                  "operator":"COMPARISON-OPERATOR"
               },
               "next-state":"STATE-NAME"
            },
            {  
               "And":[  
                  {  
                     "path":"PAYLOAD-PATH",
                     "value":"VALUE",
                     "operator":"COMPARISON-OPERATOR"
                  },
                  {  
                     "path":"PAYLOAD-PATH",
                     "value":"VALUE",
                     "operator":"COMPARISON-OPERATOR"
                  }
               ],
               "next-state":"STATE-NAME"
            },
            {  
               "Or":[  
                  {  
                     "path":"PAYLOAD-PATH",
                     "value":"VALUE",
                     "operator":"COMPARISON-OPERATOR"
                  },
                  {  
                     "path":"PAYLOAD-PATH",
                     "value":"VALUE",
                     "operator":"COMPARISON-OPERATOR"
                  }
               ],
               "next-state":"STATE-NAME"
            }
         ],
         "default":"STATE-NAME"
      }
   ]
}
```

- **choices**：针对输入数据定义了一个有序的匹配规则集，以使数据进入此状态，并为每个匹配项转换为下一个状态。
- **path**：JSON Path，用于选择要匹配的输入数据的值。
- **value**：匹配值。
- **operator**：指定如何将输入数据与值进行比较，例如 “EQ”、“LT”、“LTEQ”、“GT”、“GTEQ”、“ StrEQ”、“StrLT”、“ StrLTEQ”、“ StrGT”、“StrGTEQ”。
- **next-state**：指定在存在值匹配时要转换到的下一个状态的名称。
- **Not**：必须是单个匹配规则，且不得包含 `next-state` 字段。
- **And 和 Or**：必须是匹配规则的非空数组，它们本身不能包含 `next-state` 字段。
- **default**：如果任何选择值都不匹配，则 `default` 字段将指定下一个状态的名称。
- **关于 next-state**：评估的顺序是从上到下，如果发生匹配，请转到 `next-state`，并忽略其余条件。

#### 延迟状态（Delay State）

```json
{  
   "states":[  
      {  
         "name":"STATE-NAME",
         "type":"DELAY",
         "start":true,
         "time-delay":"TIME-VALUE",
         "next-state":"STATE-NAME"
      }
   ]
}
```

- **time-delay**：指定时间延迟。`TIME-VALUE` 是在此状态下延迟的时间（以秒为单位）。 必须是正整数。
- **next-state**：指定要转换到的下一个状态的名称。`STATE-NAME` 在函数工作流中必须是有效的 State 名称。

#### 结束状态（End State）

```json
{  
   "states":[  
      {  
         "name":"STATE-NAME",
         "type":"END",
         "status": "STATUS"
      }
   ]
}
```

- **status**：该字段必须为 SUCCESS 或 FAILURE，表示工作流结束。

#### 并行状态（Parallel State）

并行状态由多个并行执行的状态组成。并行状态具有多个同时执行的分支。每个分支都有一个状态列表，其中一个状态为开始状态。每个分支继续执行，直到达到该分支内没有下一个状态的状态为止。当所有分支都执行完成后，并行状态将转换为下一个状态。本质上，这是在并行状态内嵌套一组状态。

并行状态由状态类型 “PARALLEL” 定义，并包括一组并行分支，每个分支都有自己的独立状态。每个分支都接收并行状态的输入数据的副本。除 END 状态外，任何类型的状态都可以在分支中使用。

分支内状态的 ”next-state” 转换只能是到该分支内的其他状态。另外，并行状态之外的状态不能转换到并行状态的分支内的状态。

并行状态会生成一个输出数组，其中每个元素都是分支的输出。输出数组的元素不必是同一类型。

```json
{  
   "states":[  
      {  
         "name":"STATE-NAME",
         "type":"PARALLEL",
         "start": true,
         "branches":[  
            {  
               "name":"BRANCH-NAME1",
               "states":[  

               ]
            },
            {  
               "name":"BRANCH-NAME2",
               "states":[  

               ]
            }
         ],
         "next-state":"STATE-NAME"
      }
   ]
}
```

- **branch**：同时执行的分支的列表。每个命名分支都有一个 “states” 列表。分支内每个状态的 “next-state” 字段必须是该分支内的有效状态名称，或者不存在以指示该状态终止该分支的执行。分支执行从分支内具有 “start”：true 的状态开始。
- **next-state**：指定在所有分支完成执行之后要转换到的下一个状态的名称。`STATE-NAME` 在函数工作流中必须是有效的状态名，但不能是并行状态本身中的状态。

## 信息传递

下图显示了通过函数工作流的数据流，该函数工作流包括调用两个 serverless 函数的事件状态。来自一个状态的输出数据作为输入数据传递到下一状态。过滤器用于过滤和转换进入和退出每个状态时的数据。从工作流中的 Operation State 调用时，来自先前状态的输入数据可能会传递到 serverless 函数。

来自 Serverless 函数的响应中包含的数据将作为输出数据发送到下一个状态。如果状态（Operation State 或 Event State）包括一系列顺序操作，则将过滤来自一个 Serverless 函数的响应中包含的数据，然后在请求中将其发送给下一个函数。

在 Event State 下，在将请求从事件源接收到的 CloudEvent 元数据传递到 Serverless 函数之前，可以对其进行转换并将其与从先前状态接收到的数据进行组合。同样，在从 Serverless 函数的响应中接收到的 CloudEvent 元数据可以转换并与从先前状态接收到的数据组合，然后再在发送到事件源的响应中进行传递。

![异步信息传递（图片来自 Workflow - Version 0.1）](https://tva1.sinaimg.cn/large/006y8mN6ly1g7nmzu8mcoj30ml0cmaak.jpg)

在某些情况下，诸如 API 网关之类的事件源希望收到工作流的响应。在这种情况下，可以将在从 Serverless 函数的响应中接收到的 CloudEvent 元数据进行转换，并与从先前状态接收到的数据进行组合，然后再将其在发送到事件源的响应中进行传递，如下所示。

![同步事件消息传递（图片来自 Workflow - Version 0.1）](https://tva1.sinaimg.cn/large/006y8mN6ly1g7nn23l4jfj30ml09lt93.jpg)

### 过滤器机制

状态机维护一个隐式 JSON 数据，该数据可以从每个过滤器作为 JSONPath 表达式 `$` 进行访问。

过滤器共有三种：

- **事件过滤器（Event Filter）**
  - 当数据从事件传递到当前状态时调用。
- **状态过滤器（State Filter）**
  - 当数据从先前状态传递到当前状态时调用
  - 当数据从当前状态传递到下一个状态时调用
- **动作过滤器**（Action 是指定义 Serverless 函数的[动作](#action-definition)定义）
  - 当数据从当前状态传递到第一个操作时调用
  - 当数据从一个动作传递到另一个动作时调用
  - 当数据从最后一个动作传递到当前状态时调用

每个过滤器都有三种路径过滤器：

- **InputPath**
  - 选择事件、状态或操作的输入数据作为 JSONPath
  - 默认值为 `$`
- **ResultPath**
  - 将 Action 输出的结果 JSON 节点指定为 JSONPath
  - 默认值为 `$`
- **OutputPath**
  - 将 State 或 Action 的输出数据指定为 JSONPath
  - 默认值为 `$`

![顺序过滤器示意图（图片来自 Workflow - Version 0.1）](https://tva1.sinaimg.cn/large/006y8mN6ly1g7nncficq1j30qo0f00tj.jpg)

![并行过滤器示意图（图片来自 Workflow - Version 0.1））](https://tva1.sinaimg.cn/large/006y8mN6ly1g7nncpg5yrj30qo0f0js7.jpg)

### 示例

组成以下两个函数 “hello” 和 “save_result”。 第二个函数 “save_result” 将第一个函数 “ hello” 的输出作为输入。

- Function "hello":
  input {"name": String}
  output {"payload": String}
- Function "save_result":
  input {"value1: String}
  output String

CNCF Function Workflow 语言：

```json
{  
   "states":[  
      {  
         "name":"HelloWorld",
         "type":"OPERATION",
         "start":true,
         "action-mode":"Sequential",
         "actions":[  
            {  
               "function":"hello"
            }
         ],
         "next-state":"UpdateArg"
      },
      {  
         "name":"UpdateArg",
         "type":"OPERATION",
         "start":false,
         "action-mode":"Sequential",
         "InputPath":"$.payload",
         "ResultPath":"$.ifttt.value1",
         "OutputPath":"$.ifttt",
         "actions":[  

         ],
         "next-state":"SaveResult"
      },
      {  
         "name":"SaveResult",
         "type":"OPERATION",
         "start":false,
         "action-mode":"Sequential",
         "actions":[  
            {  
               "function":"save_resut"
            }
         ],
         "next-state":"STATE_END"
      },
      {  
         "name":"STATE-END",
         "type":"END"
      }
   ]
}
```

![Hello world 示意图（图片来自 Workflow - Version 0.1）](https://tva1.sinaimg.cn/large/006y8mN6ly1g7nnhjqx4zj30qo0f074x.jpg)



### 错误

状态机在运行时返回以下预定义的错误代码。通常，它在[动作定义](#action-definition)的 `retry` 字段中使用。

- SYS.Timeout
- SYS.Fail
- SYS.MatchAny
- SYS.Permission
- SYS.InvalidParameter
- SYS.FilterError

## 参考

- [Workflow - Version 0.1 - github.com](https://github.com/cncf/wg-serverless/blob/master/workflow/spec/spec.md)
