![logo](./assets/logo.png)

# 自动导引车（AGV）与主控系统之间的通信接口

## VDA 5050

## 版本 2.1.0

![控制系统和自动导引车](./assets/csagv.png)

### 简要信息

定义无人运输系统（DTS）的通信接口。本建议描述了在中央主控系统和自动导引车（AGV）之间交换命令和状态数据的通信接口，用于内部物流过程。

### 免责声明

以下说明作为执行自动导引车（AGV）与主控系统之间通信接口的指引，适用于所有人且不具约束力。应用者应确保在具体情况下正确应用。

他们应考虑到每次发布时的最新技术状态。通过应用这些建议，没有人可以逃避对自己行为的责任。声明不声称是详尽的或对现有法律的准确解释。它们不能替代对相关政策、法律和法规的研究。此外，还应考虑各自产品的特殊性及其不同的可能应用。每个人在这方面都需自担风险。VDA及参与开发或应用这些建议的人不承担责任。

如果您在应用这些建议时发现任何不准确之处或可能的错误解释，请立即通知VDA，以便可以纠正任何缺陷。

**出版者**
德国汽车工业协会 (VDA)
Behrenstraße 35, 10117 Berlin,
德国
www.vda.de

**版权**
汽车工业协会 (VDA)
复制和任何其他形式的复制仅在注明来源的情况下允许。

版本 2.1.0

## 目录

[1 前言](#1-foreword)<br>
[2 文件目标](#2-objective-of-the-document)<br>
[3 范围](#3-scope)<br>
[3.1 其他适用文件](#31-other-applicable-documents)<br>
[4 要求和协议定义](#4-requirements-and-protocol-definition)<br>
[5 通信的过程和内容](#5-process-and-content-of-communication)<br>
[6 协议规范](#6-protocol-specification)<br>
[6.1 表格符号和格式含义](#61-symbols-of-the-tables-and-meaning-of-formatting)<br>
[6.1.1 可选字段](#611-optional-fields)<br>
[6.1.2 允许的字符和字段长度](#612-permitted-characters-and-field-lengths)<br>
[6.1.3 枚举的表示法](#613-notation-of-fields-topics-and-enumerations) <br>
[6.1.4 JSON 数据类型](#614-json-data-types)<br>
[6.2 MQTT 连接处理、安全性和QoS](#62-mqtt-connection-handling-security-and-qos)<br>
[6.3 MQTT 主题级别](#63-mqtt-topic-levels)<br>
[6.4 协议头](#64-protocol-header)<br>
[6.5 通信主题](#65-topics-for-communication)<br>
[6.6 主题："命令"（从主控系统到AGV）](#66-topic-order-from-master-control-to-agv)<br>
[6.6.1 概念和逻辑](#661-concept-and-logic)<br>
[6.6.2 命令和命令更新](#662-orders-and-order-update)<br>
[6.6.3 命令取消（由主控系统）](#663-order-cancellation-by-master-control)<br>
[6.6.4 命令拒绝](#664-order-rejection)<br>
[6.6.5 走廊](#665-corridors)<br>
[6.6.6 命令消息的实现](#666-implementation-of-the-order-message)<br>
[6.7 地图](#67-maps)<br>
[6.7.1 地图分发](#671-map-distribution)<br>
[6.7.2 车辆状态中的地图](#672-maps-in-the-vehicle-state)<br>
[6.7.3 地图下载](#673-map-download)<br>
[6.7.4 启用下载的地图](#674-enable-downloaded-maps)<br>
[6.7.5 删除车辆上的地图](#675-delete-maps-on-vehicle)<br>
[6.8 动作](#68-actions)<br>
[6.8.1 预定义动作的定义、参数、效果和范围](#681-definition-parameters-effects-and-scope-of-predefined-actions)<br>
[6.8.2 预定义动作的状态](#682-states-of-predefined-actions)<br>
[6.9 主题："即时动作"（从主控系统到AGV）](#69-topic-instantactions-from-master-control-to-agv)<br>
[6.10 主题："状态"（从AGV到主控系统）](#610-topic-state-from-agv-to-master-control)<br>
[6.10.1 概念和逻辑](#6101-concept-and-logic)<br>
[6.10.2 节点的遍历和进入/离开边缘，触发动作](#6102-traversal-of-nodes-and-enteringleaving-edges-triggering-of-actions)<br>
[6.10.3 基础请求](#6103-base-request)<br>
[6.10.4 信息](#6104-information)<br>
[6.10.5 错误](#6105-errors)<br>
[6.10.6 状态消息的实现](#6106-implementation-of-the-state-message)<br>
[6.11 动作状态](#611-action-states)<br>
[6.12 动作阻塞类型和顺序](#612-action-blocking-types-and-sequence)<br>
[6.13 主题 "可视化"](#613-topic-visualization)<br>
[6.14 主题 "连接"](#614-topic-connection)<br>
[6.15 主题 "事实表"](#615-topic-factsheet)<br>
[6.15.1 事实表 JSON 结构](#6151-factsheet-json-structure)<br>
[7 最佳实践](#7-best-practice)<br>
[7.1 错误参考](#71-error-reference)<br>
[7.2 参数格式](#72-format-of-parameters)<br>
[8 术语表](#8-glossary)<br>
[8.1 定义](#81-definition)<br>

# 1 foreword
# 1 前言

该接口是由德国汽车工业协会 (VDA) 和德国机械设备制造业联合会 (VDMA) 合作建立的。双方的目标是创建一个普遍适用的接口。对接口的更改建议应提交给VDA，并与VDMA共同评估，在积极决定的情况下纳入新版本状态。通过GitHub对本文档的贡献非常感谢。可以在以下链接找到存储库：https://github.com/vda5050/vda5050。

# 2 objective-of-the-document
# 2 文件目标

本建议的目标是简化新车辆与现有主控系统的连接，并在同一工作环境中实现来自不同制造商的AGV和传统系统（库存系统）的并行操作。

应定义主控系统与AGV之间的统一接口。应通过以下几点实现：

- 描述AGV与主控系统之间通信的标准，从而为使用协作运输车辆将运输系统集成到连续过程自动化中提供基础。
- 通过增加车辆自主性、过程模块和接口等提高灵活性，最好是分离事件控制命令链的刚性顺序。
- 由于高“即插即用”能力，减少实施时间，因为所需信息（例如命令信息）由中央服务提供并普遍有效。车辆应能够在不考虑制造商的情况下以相同的实施努力投入运行，同时考虑到职业安全的要求。
- 通过使用统一的、跨越的协调与相应的逻辑来减少复杂性并提高系统的“即插即用”能力，适用于所有运输车辆、车辆模型和制造商。
- 通过在车辆控制和协调层之间使用通用接口来提高制造商的独立性。
- 通过实现专有主控系统与上级主控系统之间的垂直通信来集成专有DTS库存系统（参见图1）。

![图1 DTS库存系统的集成](./assets/concept_DTS.png)
>图1 DTS库存系统的集成

为了实现上述目标，本文档描述了AGV与主控系统之间命令和状态信息通信的接口。

本文档中未初步包含AGV与主控系统之间操作所需的其他接口（例如，考虑到路径规划等方面的特殊技能）或与其他系统组件（例如，外部外围设备、消防门等）通信的接口。

# 3 Scope
# 3 范围

本建议包含关于自动导引车（AGV）与主控系统之间通信的定义和最佳实践。目标是允许具有不同特征的AGV（例如，底盘牵引车或叉车AGV）以统一的语言与主控系统通信。这为在主控系统中操作任何组合的AGV奠定了基础。主控系统提供命令并协调AGV交通。

接口基于汽车行业生产和工厂物流的要求。根据制定的要求，内部物流的要求涵盖了物流部门的要求，即从货物接收到生产供应到货物发出的物流过程，通过控制自由导航车辆和引导车辆。

与自动化车辆不同，自主车辆能够根据相应的传感器系统和算法独立解决出现的问题，并能够相应地对动态环境的变化做出反应或在不久之后适应它们。自主属性，例如独立绕过障碍物，可以由自由导航车辆和引导车辆实现。然而，一旦路径规划在车辆本身上进行，本文档描述了自由导航车辆（参见术语表）。自主系统不是完全分散的（群体智能），并通过预定义的规则具有定义的行为。

为了实现可持续的解决方案，下面描述了一个可以在其结构中扩展的接口。这应能够完全覆盖引导车辆的主控系统。自由导航车辆可以集成到结构中；为此所需的详细规范不属于本建议的一部分。

对于专有库存系统的集成，可能需要对接口进行单独定义，这不被视为本建议的一部分。

## 3.1 Other applicable documents
## 3.1 其他适用文件

文件 | 版本 | 描述
---|---|---
VDI 指南 2510 | 2005年10月 | 无人运输系统（DTS）
VDI 指南 4451 第7页 | 2005年10月 | 无人运输系统（DTS）的兼容性 - DTS主控系统
DIN EN ISO 3691-4 | 2023年12月 | 工业卡车安全要求和验证-第4部分：无人驾驶卡车及其系统
LIF – 布局交换格式| 2024年3月 | 定义无人运输车辆集成商与（第三方）主控系统之间轨道布局交换的格式。

# 4 Requirements and protocol definition
# 4 要求和协议定义

通信接口旨在支持以下要求：

- 控制至少1000辆车
- 允许集成具有不同自主程度的车辆
- 允许决策，例如，关于路线选择或交叉口的行为

车辆应在定期间隔或状态变化时传输其状态。

通信通过无线网络进行，考虑到连接故障和消息丢失的影响。

消息协议是消息队列遥测传输（MQTT），应与JSON结构一起使用。在开发此协议期间测试了MQTT 3.1.1，并且是兼容性所需的最低版本。MQTT允许将消息分发到子频道，这些子频道称为“主题”。MQTT网络中的参与者订阅这些主题并接收与他们相关或感兴趣的信息。

JSON结构允许将来通过附加参数扩展协议。参数用英语描述，以确保协议在德语区以外可读、可理解和可应用。

# 5 Process and content of communication
# 5 通信的过程和内容

AGV的操作至少有以下参与者：

- AGV系统的操作员提供基本信息
- 主控系统组织和管理操作
- AGV执行命令

图2描述了操作阶段的通信内容。在实施或修改期间，AGV和主控系统手动配置。

![图2 信息流结构](./assets/information_flow_VDA5050.png)
>图2 信息流结构

在实施阶段，由主控系统和AGV组成的无人运输系统（DTS）被设置。操作员定义必要的框架条件，并手动输入所需信息或通过从其他系统导入将其存储在主控系统中。基本上，这涉及以下内容：

- 路线定义：使用CAD导入，路线可以导入到主控系统。或者，操作员也可以在主控系统中手动实现路线。路线可以是单行道，限制某些车辆组使用（基于尺寸比例）等。
- 路线网络配置：在路线内，定义了装卸站、充电站、外围环境（门、电梯、障碍物）、等待位置、缓冲站等。
- 车辆配置：操作员存储AGV的物理属性（尺寸、可用的载货架等）。AGV应通过主题`factsheet`以特定方式传达此信息，该方式在本文档的[6.15 主题 "事实表"](#615-topic-factsheet)部分中定义。

上述路线和路线网络的配置不属于本文档的一部分。它们构成了在主控系统基于此信息和要完成的运输要求进行命令控制和驾驶课程分配的基础。然后，AGV的命令通过MQTT消息代理传输到车辆。车辆在执行任务的同时持续向主控系统报告其状态。这也是通过MQTT消息代理完成的。

主控系统的功能包括：

- 向AGV分配命令
- 路线计算和AGV的引导（考虑到每个AGV的个体物理属性的限制，例如尺寸、机动性等）
- 检测和解决阻塞（“死锁”）
- 能源管理：充电命令可以中断传输命令
- 交通控制：缓冲路线和等待位置
- 环境的（临时）变化，例如释放某些区域或改变最大速度
- 与外围系统（如门、门、升降机等）的通信
- 检测和解决通信错误

AGV的功能包括：

- 定位
- 沿关联路线导航（引导或自主）
- 执行动作
- 持续传输车辆状态

此外，集成商在配置整个系统时应考虑以下内容（不完整列表）：

- 地图配置：主控系统和AGV的坐标系应匹配。
- 枢轴点：使用AGV的不同点或充电点作为枢轴点会导致车辆的不同包络。参考点可能因情况而异，例如，对于携带负载的AGV和不携带负载的AGV可能不同。

# 6 Protocol specification
# 6 协议规范

以下部分描述了通信协议的详细信息。协议规定了主控系统和AGV之间的通信。AGV与外围设备之间的通信，例如AGV与门之间的通信，被排除在外。

不同的消息以表格形式呈现，描述了作为命令、状态等发送的JSON字段的内容。

此外，JSON模式可用于在公共git存储库中进行验证（https://github.com/VDA5050/VDA5050）。JSON模式会随着VDA5050的每次发布而更新。如果JSON模式与本文档之间存在差异，则以本文档中的变体为准。

## 6.1 Symbols of the tables and meaning of formatting
## 6.1 表格符号和格式含义

表格包含标识符的名称、其单位、其数据类型以及描述（如果有）。

标识 | 描述
---|---
standard | 变量是基本数据类型
**bold** | 变量是非基本数据类型（例如，JSON对象或数组）并单独定义
*italic* | 变量是可选的
***italic and bold***| 变量是可选的且是非基本数据类型
arrayName[arrayDataType] | 变量（此处为arrayName）是方括号中包含的数据类型的数组（此处数据类型为arrayDataType）

所有关键字区分大小写。所有字段名称均为驼峰命名法。所有枚举均为大写且无下划线。

### 6.1.1 Optional fields
### 6.1.1 可选字段

如果变量被标记为可选，则意味着对于发送方来说是可选的，因为在某些情况下变量可能不适用（例如，当主控系统向AGV发送命令时，某些AGV自行规划其轨迹，命令中的`trajectory`字段可以省略）。

如果AGV接收到包含在此协议中标记为可选的字段的消息，AGV应相应地采取行动，不能忽略该字段。如果AGV无法相应地处理消息，则预期行为是通过错误消息进行通信并拒绝命令。

主控系统应仅发送AGV支持的可选信息。

示例：轨迹是可选的。如果AGV无法处理轨迹，主控系统不应向车辆发送轨迹。

AGV应通过AGV`factsheet`消息传达其需要的可选参数。

### 6.1.2 Permitted characters and field lengths
### 6.1.2 允许的字符和字段长度

所有通信均以UTF-8编码，以实现描述的国际适应性。建议ID仅使用以下字符：

A-Z a-z 0-9 _ - . :

未定义最大消息长度，但受MQTT协议规范的限制，可能还受事实表中定义的技术约束的限制。如果AGV的内存不足以处理传入的命令，则应拒绝命令。最大字段长度、字符串长度或值范围的匹配由集成商负责。为了便于集成，AGV供应商应提供在[事实表部分](#616-topic-factsheet)中详细说明的AGV事实表。

### 6.1.3 Notation of fields, topics and enumerations
### 6.1.3 枚举的表示法

本文档中的主题和字段以以下样式突出显示：`exampleField`和`exampleTopic`。枚举应以大写书写。这些值在文档中用单引号括起来。这包括`actionStatus`字段中的关键字（'WAITING'，'FINISHED'等）。

### 6.1.4 JSON data types
### 6.1.4 JSON 数据类型

尽可能使用JSON数据类型。因此，布尔值通过“true”或“false”进行编码，而不是使用枚举（'TRUE'，'FALSE'）或魔术数字。数值数据类型以类型和精度指定，例如，float64或uint32。不支持IEEE 754的特殊数值，如NaN和无穷大。

## 6.2 MQTT connection handling, security and QoS
## 6.2 MQTT 连接处理、安全性和QoS

MQTT协议为客户端提供了设置最后遗言消息的选项。如果客户端由于任何原因意外断开连接，最后遗言将由代理分发给其他订阅的客户端。此功能的使用在[6.14 主题 "连接"](#614-topic-connection)部分中进行了描述。

如果AGV与代理断开连接，它会保留所有命令信息并执行命令直到最后释放的节点。

协议安全性需要通过代理配置来考虑。

为了减少通信开销，MQTT QoS级别0（尽力而为）用于主题`order`、`instantActions`、`state`、`factsheet`和`visualization`。主题`connection`应使用QoS级别1（至少一次）。

## 6.3 MQTT topic levels
## 6.3 MQTT 主题级别

由于云提供商的强制性主题结构，MQTT主题结构没有严格定义。对于基于云的MQTT代理，主题结构必须单独适应以匹配本协议中定义的主题。这意味着以下部分中定义的主题名称是强制性的。

对于本地代理，建议的MQTT主题级别如下：

**interfaceName/majorVersion/manufacturer/serialNumber/topic**

示例：
```
uagv/v2/KIT/0001/order
```

MQTT 主题级别 | 数据类型 | 描述
---|---|---
interfaceName | string | 使用的接口名称
majorVersion | string | VDA 5050建议的主要版本号，前面加“v”
manufacturer | string | AGV的制造商。
serialNumber | string | 唯一的AGV序列号，由以下字符组成：<br>A-Z <br>a-z <br>0-9 <br>_ <br>. <br>: <br>-
topic | string | 主题（例如，命令或状态）参见[6.5 通信主题](#65-topics-for-communication)部分

注意：由于“/”字符用于定义主题层次结构，因此不应在上述任何字段中使用。某些MQTT代理还使用“$”字符用于特殊的内部主题，因此也不应使用。

## 6.4 Protocol header
## 6.4 协议头

每个JSON消息都以头开始。在以下部分中，出于可读性目的，将引用以下字段作为头。头由以下各个元素组成。头不是JSON对象。

对象结构/标识符 | 数据类型 | 描述
---|---|---
headerId | uint32 | 消息的头ID。<br>头ID是每个主题定义的，并随着每个发送的（但不一定接收的）消息递增1。
timestamp | string | 时间戳（ISO 8601，UTC）；YYYY-MM-DDTHH:mm:ss.ffZ（例如，“2017-04-15T11:40:03.12Z”）。
version | string | 协议的版本[Major].[Minor].[Patch]（例如，1.3.2）。
manufacturer | string | AGV的制造商。
serialNumber | string | AGV的序列号。

### 协议版本

协议版本使用语义版本控制作为版本控制方案。

主要版本更改的示例：

- 重大更改，例如新的非可选字段

次要版本更改的示例：

- 新功能，例如附加的可视化主题

补丁版本的示例：

- 更高的电池充电精度

## 6.5 Topics for communication
## 6.5 通信主题

AGV协议使用以下主题在主控系统和AGV之间进行信息交换。

主题名称 | 发布者 | 订阅者 | 用于 | 实现 | 模式
---|---|---|---|---|---
order | 主控系统 | AGV | 从主控系统到AGV的驾驶命令通信 | 强制性 | order.schema
instantActions | 主控系统 | AGV | 需要立即执行的动作的通信 | 强制性 | instantActions.schema
state | AGV | 主控系统 | AGV状态的通信 | 强制性 | state.schema
visualization | AGV | 可视化系统 | 仅用于可视化目的的高频位置主题 | 可选 | visualization.schema
connection | 代理/AGV | 主控系统 | 指示AGV连接丢失时，不用于主控系统检查车辆健康状况，添加用于MQTT协议级别的连接检查 | 强制性 | connection.schema 
factsheet | AGV | 主控系统 | 参数或供应商特定信息，以协助在主控系统中设置AGV | 强制性 | factsheet.schema

## 6.6 Topic: "order" (from master control to AGV)
## 6.6 主题："命令"（从主控系统到AGV）

主题“命令”是AGV通过其接收JSON封装命令的MQTT主题。

### 6.6.1 Concept and logic
### 6.6.1 概念和逻辑

命令的基本结构是节点和边缘的图。AGV应遍历节点和边缘以完成命令。所有连接节点和边缘的完整图由主控系统持有。

主控系统中的图表示包含限制，例如，哪个AGV被允许遍历哪个边缘。这些限制不会传达给AGV。主控系统仅在AGV命令中包含AGV被允许遍历的边缘。

![图3 主控系统中的图表示和命令中传输的图](./assets/graph_representation_transmission.png)
>图3 主控系统中的图表示和命令中传输的图

节点和边缘在命令消息中作为两个列表传递。节点和边缘在这些列表中的顺序也决定了节点和边缘应以何种顺序遍历。

对于有效的命令，至少应有一个节点，并且边缘的数量应等于节点的数量减去一个。

命令的第一个节点应对AGV来说是显而易见的。这意味着AGV已经站在节点上，或者AGV在节点的偏差范围内。

节点和边缘都有一个布尔属性`released`。如果节点或边缘被释放，AGV应遍历它。如果节点或边缘未被释放，AGV不应遍历它。

边缘只能在其起始节点和结束节点都被释放时释放。

在未释放的边缘之后，序列中不能跟随已释放的节点或边缘。

已释放的节点和边缘的集合称为“基础”。未释放的节点和边缘的集合称为“视野”。

发送不带视野的命令是有效的。

命令消息不一定描述完整的运输命令。为了交通控制和适应资源受限的车辆，完整的运输命令（可能由许多节点和边缘组成）可以分成许多子命令，这些子命令通过其`orderId`和`orderUpdateId`连接。更新命令的过程在下一节中描述。

### 6.6.2 Orders and order update
### 6.6.2 命令和命令更新

为了支持交通管理，主控系统可以将通过命令传达的路径分为两部分：

- *“基础”*：这是AGV被允许行驶的定义路线。基础路线的所有节点和边缘已由主控系统为车辆释放。基础的最后一个节点称为决策点。
- *“视野”*：这是主控系统当前为AGV计划在决策点之后行驶的路线。视野路线尚未由主控系统释放。

如果没有进一步的节点和边缘添加到基础中，AGV应在决策点停下。为了确保流畅的运动，如果交通情况允许，主控系统应在AGV到达决策点之前扩展基础。

由于MQTT是异步协议，并且通过无线网络的传输不可靠，基础不能更改。因此，主控系统应假设基础已由AGV执行。后面部分描述了一种取消命令的程序，但由于上述通信限制，这也被认为是不可靠的。

主控系统可以通过向AGV发送更新的路线来更改视野，其中包括更改的节点和边缘列表。更改视野路线的过程如图4所示。

![图4 更改驾驶路线“视野”的过程](./assets/driving_route_horizon.png)
>图4 更改驾驶路线“视野”的过程

在图4中，初始作业首先由控制面板在时间t = 1发送。图5显示了可能作业的伪代码。为了便于阅读，此处省略了完整的JSON示例。

```
{
	orderId: "1234"
	orderUpdateId:0,
	nodes: [
	 	 f {released: True},
	 	 d {released: True},
	 	 g {released: True},
	 	 b {released: False},
	 	 h {released: False}
	],
	edges: [
		e1 {released: True},
		e3 {released: True},
		e8 {released: False},
		e9 {released: False}
	]
}
```
>图5 命令的伪代码。

在时间t = 3，命令通过发送命令的扩展进行更新（参见图6中的示例）。请注意，`orderUpdateId`递增，并且命令更新的第一个节点对应于先前命令消息的最后共享基础节点。

这确保了AGV也可以执行命令更新，即，通过执行AGV已知的边缘可以到达作业更新的第一个节点。

```
{
	orderId: 1234,
	orderUpdateId: 1,
	nodes: [
		g {released: True},
		b {released: True},
		h {released: True},
		i {released: False}
	],
	edges: [
		e8 {released: True},
		e9 {released: True},
		e10 {released: False}
	]
}
```
>图6 命令更新的伪代码。请注意`orderUpdateId`的更改。

这也有助于在命令更新丢失的情况下（例如，由于不可靠的无线网络）。AGV始终可以检查最后已知的基础节点是否与第一个新基础节点具有相同的`nodeId`（和`nodeSequenceId`，稍后会详细介绍）。

还要注意，节点g是唯一再次发送的基础节点。由于基础不能更改，因此不允许重新传输节点f和d。

重要的是，缝合节点（示例中的节点g）的内容不能更改。对于动作、偏差范围等，AGV应使用第一个命令中提供的指令（图5，orderUpdateId 0）。

![图7 常规更新过程 - 命令扩展](./assets/update_order_extension.png)
>图7 常规更新过程 - 命令扩展。

图7描述了命令应如何扩展。它显示了AGV当前可用的信息。`orderId`保持不变，`orderUpdateId`递增。

先前基础的最后一个节点是更新命令中的第一个基础节点。通过此节点，AGV可以将更新的命令添加到当前命令中（缝合）。先前基础的其他节点和边缘不会重新发送。

主控系统可以通过发送完全不同的节点作为新基础来更改视野。视野也可以被删除。

为了允许命令中的循环（例如，从节点a到b然后返回到a），为节点和边缘对象分配了`sequenceId`。此`sequenceId`在节点和边缘上运行（命令的第一个节点接收0，第一个边缘接收1，第二个节点接收2，依此类推）。这使得更容易跟踪命令进度。

一旦分配了`sequenceId`，它不会随着命令更新而更改（参见图7）。这对于在AGV侧确定主控系统引用哪个节点是必要的。

图8描述了接受命令或命令更新的过程。

![图8 接受命令或命令更新的过程](./assets/process_order_update.png)
>图8 接受命令或命令更新的过程。

1)	**接收到的命令有效吗？**：
所有格式和JSON数据类型是否正确？

2)	**接收到的命令是新命令还是当前命令的更新？**：
接收到的命令的`orderId`是否与车辆当前持有的命令的`orderId`不同？

3)	**车辆仍在执行命令还是在等待更新？**：
`nodeStates`是否不为空或`actionStates`是否包含既不是“FAILED”也不是“FINISHED”的状态？命令视野的节点和边缘及其相关的动作状态也包含在状态中。车辆可能仍有视野，因此正在等待更新并执行命令。

4) **新命令的开始是否足够接近当前位置？**：车辆是否已经站在节点上，或者是否在节点的偏差范围内（参见[6.6.1 概念和逻辑](#661-concept-and-logic)部分）？

5) **接收到的命令更新是否已过时？**：`orderUpdateId`是否小于车辆当前的`orderUpdateId`？

6)	**接收到的命令更新当前在车辆上吗？**：`orderUpdateId`是否等于车辆当前的`orderUpdateId`？

7)	**接收到的更新是否是当前仍在运行的命令的有效延续？**：接收到的命令的第一个节点是否等于当前基础的决策点（先前基础的最后一个节点）？车辆仍在移动或执行与先前命令更新中释放的基础相关的动作，或者仍有视野，因此正在等待命令的延续。在这种情况下，只有当新基础的第一个节点等于先前基础的最后一个节点时，才接受命令更新。

8)	**接收到的更新是否是先前完成的命令的有效延续？**：接收到的命令更新的第一个节点的`nodeId`和`sequenceId`是否等于`lastNodeId`和`lastNodeSequenceId`？车辆不再执行任何动作，也不再等待命令的延续（意味着它已完成其基础及其相关动作，并且没有视野）。现在接受命令更新，如果它从最后遍历的节点继续，因此新基础的第一个节点需要匹配车辆的`lastNodeId`和`lastNodeSequenceId`。

9)	填充/附加状态指的是`actionStates`/`nodeStates`/`edgeStates`。

### 6.6.3 Order cancellation (by master control)
### 6.6.3 命令取消（由主控系统）

在基础节点发生计划外更改的情况下，应使用即时动作`cancelOrder`取消命令。

在接收到即时动作`cancelOrder`后，车辆停止（根据其能力，例如，立即停止或在下一个节点上停止）。

如果有计划的动作，这些动作应被取消并报告为“FAILED”状态。如果有正在运行的动作，这些动作应被取消并报告为“FAILED”状态。如果动作无法中断，动作的`actionState`应反映这一点，通过报告“RUNNING”状态，而后是相应的状态（如果成功则为“FINISHED”，如果不成功则为“FAILED”）。在所有车辆运动和所有动作停止后，`cancelOrder`动作状态应报告为“FINISHED”。

`orderId`和`orderUpdateId`保持不变。

图9显示了不同AGV能力的预期行为。

![图9 取消命令后的预期行为](./assets/process_cancel_order.png)
>图9 取消命令后的预期行为。

#### 6.6.3.1 取消后接收新命令

在命令取消后，车辆应处于接收新命令的状态。

对于通过标签在节点上定位的AGV，新命令必须从AGV现在站立的节点开始（另见图5）。

对于可以在节点之间停止的AGV，主控系统可以选择如何开始下一个命令。AGV应接受这两种方法。

有两种选择：

- 发送一个命令，其中第一个节点是一个临时节点，位于AGV当前站立的位置。然后，AGV应意识到该节点是显而易见的并接受命令。
- 发送一个命令，其中第一个节点是先前命令的最后遍历节点，但设置偏差范围如此之大，以至于AGV在此范围内。因此，AGV应意识到该节点应被视为已遍历并接受命令。

#### 6.6.3.2 当AGV没有命令时接收取消命令动作

如果AGV接收到`cancelOrder`动作，但AGV当前没有命令，或者先前的命令已被取消，则`cancelOrder`动作应报告为“FAILED”。

AGV应报告“noOrderToCancel”错误，并将`errorLevel`设置为“WARNING”。`instantAction`的`actionId`应作为`errorReference`传递。

### 6.6.4 Order rejection
### 6.6.4 命令拒绝

在以下几种情况下，应拒绝命令。这些场景在图8中显示并在下文中描述。

#### 6.6.4.1 车辆收到格式错误的新命令

解决方案：

1. 车辆不在其内部缓冲区中接管新命令。
2. 车辆报告警告“validationError”
3. 警告应报告，直到车辆接受新命令。

#### 6.6.4.2 车辆收到包含其无法执行的动作或无法使用的字段的命令

示例：

- 不可执行的动作：提升高度高于最大提升高度，尽管没有安装行程的提升动作等。
- 无法使用的字段：轨迹等。

解决方案：

1. 车辆不在其内部缓冲区中接管新命令
2. 车辆报告警告“orderError”，错误字段作为错误引用
3. 警告应报告，直到车辆接受新命令。

#### 6.6.4.3 车辆收到具有相同orderId但orderUpdateId低于当前orderUpdateId的新命令

解决方案：

1. 车辆不在其内部缓冲区中接管新命令。
2. 车辆保留先前命令在其缓冲区中。
3. 车辆报告警告“orderUpdateError”
4. 车辆继续执行先前命令。

如果AGV两次接收到具有相同`orderId`和`orderUpdateId`的命令，则第二个命令将被忽略。如果主控系统重新发送命令，因为状态消息接收得太晚，主控系统无法验证第一个命令已被接收，这可能会发生。

### 6.6.5 Corridors
### 6.6.5 走廊

可选的`corridor`边缘属性允许车辆偏离边缘轨迹以避开障碍物，并定义车辆被允许操作的边界。要使用`corridor`属性，需要预定义的轨迹，如果没有定义`corridor`属性，车辆将遵循该轨迹。这可以是主控系统已知的车辆上定义的轨迹，也可以是命令中发送的轨迹。使用`corridor`属性的车辆的行为仍然是线引导车辆的行为，只是允许暂时偏离轨迹以避开障碍物。

*备注：
命令中的边缘定义了两个节点之间的逻辑连接，而不一定是车辆在从起始节点到结束节点时遵循的（真实）轨迹。根据车辆类型，车辆在起始节点和结束节点之间采取的轨迹由主控系统通过轨迹边缘属性定义，或分配给车辆作为预定义轨迹。根据车辆的内部状态，选择的轨迹可能会有所不同。*

![图10 具有走廊属性的边缘。](./assets/edges_with_corridors.png)
>图10 具有`corridor`属性的边缘，定义了车辆被允许偏离其预定义轨迹以避开障碍物的左边界和右边界。在左侧，运动中心定义了允许的偏差，而在右侧，车辆的轮廓（可能由负载扩展）定义了允许的偏差。这由`corridorRefPoint`参数定义。

车辆被允许独立导航（并偏离原始边缘轨迹）的区域由左边界和右边界定义。可选的`corridorRefPoint`字段指定车辆控制点或车辆轮廓是否应在定义的边界内。边缘的边界应定义为车辆在通过节点时位于新边缘和当前边缘的边界内。主控系统不应使用`corridor`属性，如果车辆不应偏离轨迹，则将走廊边界设置为零。

车辆的运动控制软件应不断检查车辆是否在定义的边界内。如果没有，车辆应停止，因为它超出了允许的导航空间并报告错误。主控系统可以决定是否需要用户交互，或者车辆是否可以通过取消当前命令并向车辆发送新的命令来继续，该命令的走廊信息允许车辆再次移动。

*备注：允许车辆偏离轨迹会增加车辆在驾驶时的可能占地面积。在初始操作期间应考虑到这一情况，如果主控系统根据车辆的占地面积做出交通控制决策。*

有关更多信息，请参见[6.10.2 节点的遍历和进入/离开边缘](#6102-traversal-of-nodes-and-enteringleaving-edges-triggering-of-actions)部分。

## 6.6.6 Implementation of the order message
## 6.6.6 命令消息的实现

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
headerId | | uint32 | 消息的头ID。<br>头ID是每个主题定义的，并随着每个发送的（但不一定接收的）消息递增1。
timestamp | | string | 时间戳（ISO 8601，UTC）；YYYY-MM-DDTHH:mm:ss.ffZ（例如，“2017-04-15T11:40:03.12Z”）
version | | string | 协议的版本[Major].[Minor].[Patch]（例如，1.3.2）
manufacturer | | string | AGV的制造商
serialNumber | | string | AGV的序列号
orderId | | string | 命令标识。<br>用于标识属于同一命令的多个命令消息。
orderUpdateId | | uint32 | 命令更新标识。<br>在orderId中是唯一的。<br>如果命令更新被拒绝，则在拒绝消息中传递此字段。
*zoneSetId* | | string | AGV必须用于导航的区域集的唯一标识符，或主控系统用于规划的区域集。<br><br>可选：某些主控系统不使用区域。<br>某些AGV不理解区域。<br>如果不使用区域，请不要添加到消息中。
**nodes [node]** | | array | 为完成命令而需要遍历的节点对象数组。<br>一个节点足以构成有效命令。<br>在这种情况下，边缘数组留空。
**edges [edge]** | | array | 为完成命令而需要遍历的边缘对象数组。<br>一个节点足以构成有效命令。<br>在这种情况下，边缘数组留空。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**node** { | | JSON 对象 |
nodeId | | string | 唯一的节点标识
sequenceId | | uint32 | 用于跟踪命令中节点和边缘的顺序并简化命令更新的编号。<br>主要目的是区分在一个orderId中多次通过的节点。<br>变量sequenceId在同一orderId的所有节点和边缘上运行，并在发出新orderId时重置。
*nodeDescription* | | string | 节点的附加信息
released | | boolean | "true"表示节点是基础的一部分。<br>"false"表示节点是视野的一部分。
***nodePosition*** | | JSON 对象 | 节点位置。<br>对于不需要节点位置的车辆类型（例如，线引导车辆）是可选的。
**actions [action]** <br> } | | array | 在节点上执行的动作数组。<br>如果不需要动作，则为空数组。

对象结构 | 单位 | 数据类型 | 描述
---| --- |--- | ---
**nodePosition** { | | JSON 对象 | 在全球项目特定的世界坐标系中定义地图上的位置。<br>每个楼层都有自己的地图。<br>所有地图应使用相同的项目特定全球原点。
x | m | float64 | 相对于地图坐标系的X位置。<br>精度取决于具体实现。
y | m | float64 | 相对于地图坐标系的Y位置。<br>精度取决于具体实现。
*theta* | rad | float64 | 范围：[-Pi ... Pi]<br><br>AGV在节点上的绝对方向。<br>可选：车辆可以自行规划路径。<br>如果定义，AGV必须在此节点上假设theta角度。<br>如果先前的边缘不允许旋转，AGV应在节点上旋转。<br>如果后续边缘定义了不同的方向但不允许旋转，AGV应在进入边缘之前在节点上旋转到边缘所需的方向。
*allowedDeviationXY* | m | float64 | 指示AGV应如何精确匹配节点位置以将其视为已遍历。<br><br>如果=0.0：不允许偏差（不允许偏差意味着在AGV制造商的正常公差范围内）。<br><br>如果>0.0：允许的偏差半径（以米为单位）。<br>如果AGV在偏差半径内通过节点，则可以将节点视为已遍历。
*allowedDeviationTheta* | rad | float64 | 范围：[0.0 ... Pi]<br><br>指示AGV在节点上定义的theta方向的精确程度。<br>最低可接受角度为theta - allowedDeviationTheta，最高可接受角度为theta + allowedDeviationTheta。
mapId | | string | 位置所引用的地图的唯一标识。<br>每个地图都有相同的项目特定全球原点。<br>当AGV使用电梯时，例如，从出发楼层到目标楼层，它将从出发楼层的地图上消失，并在目标楼层的相关电梯节点上生成。
*mapDescription* <br> } | | string | 地图的附加信息。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**action** { | | JSON 对象 | 描述AGV可以执行的动作。
actionType | | string | 动作的名称，如“动作和参数”第一列中所述。<br>标识动作的功能。
actionId | | string | 唯一的ID，用于标识动作并将其映射到状态中的actionState。<br>建议：使用UUID。
*actionDescription* | | string | 动作的附加信息
blockingType | | string | 枚举{'NONE', 'SOFT', 'HARD'}：<br>'NONE'：允许驾驶和其他动作；<br>'SOFT'：允许其他动作但不允许驾驶；<br>'HARD'：此时唯一允许的动作。
***actionParameters [actionParameter]*** <br><br> } | | array | 指定动作的actionParameter对象数组，例如“deviceId”、“loadId”、“外部触发器”。<br><br>可以在[7.2 参数格式](#72-format-of-parameters)中找到示例实现。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**edge** { | | JSON 对象 | 两个节点之间的方向连接。
edgeId | | string | 唯一的边缘标识。
sequenceId | | uint32 | 用于跟踪命令中节点和边缘的顺序并简化命令更新的编号。<br>变量sequenceId在同一orderId的所有节点和边缘上运行，并在发出新orderId时重置。
*edgeDescription* | | string | 边缘的附加信息。
released | | boolean | "true"表示边缘是基础的一部分。<br>"false"表示边缘是视野的一部分。
startNodeId | | string | 命令中第一个节点的nodeId。
endNodeId | | string | 命令中最后一个节点的nodeId。
*maxSpeed* | m/s | float64 | 边缘上的允许最大速度。<br>速度由车辆的最快测量值定义。
*maxHeight* | m | float64 | 边缘上车辆的允许最大高度，包括负载。
*minHeight* | m | float64 | 边缘上负载处理设备的允许最小高度。
*orientation* | rad | float64 | AGV在边缘上的方向。值`orientationType`定义了它是相对于全球项目特定地图坐标系解释的还是相对于边缘切线解释的。在相对于边缘切线解释的情况下，0.0表示向前驾驶，PI表示向后驾驶。<br>示例：方向Pi/2 rad将导致旋转90度。<br><br>如果AGV以不同的方向开始，请在边缘上旋转车辆到所需方向，如果`rotationAllowed`设置为“true”。<br>如果`rotationAllowed`为“false”，请在进入边缘之前旋转。<br>如果无法做到这一点，请拒绝命令。<br><br>如果没有定义轨迹，请将旋转应用于边缘的两个连接节点之间的直接路径。<br>如果为边缘定义了轨迹，请将方向应用于轨迹。
*orientationType* | | string | 枚举{'GLOBAL', 'TANGENTIAL'}：<br>'GLOBAL'：相对于全球项目特定地图坐标系；<br>'TANGENTIAL'：相对于边缘切线。<br><br>如果未定义，默认值为'TANGENTIAL'。
*direction* | | string | 在交叉口为线引导或线圈引导车辆设置方向，最初定义（车辆个体）。<br>示例：“左”、“右”、“直行”。
*rotationAllowed* | | boolean | "true"：允许在边缘上旋转。<br>"false"：不允许在边缘上旋转。<br><br>可选：<br>如果未设置，则无限制。
*maxRotationSpeed* | rad/s | float64| 最大旋转速度<br><br>可选：<br>如果未设置，则无限制。
***trajectory*** | | JSON 对象 | 此边缘的轨迹JSON对象为NURBS。<br>定义AGV应在边缘的起始节点和结束节点之间移动的路径。<br><br>可选：<br>如果AGV无法处理轨迹或AGV自行规划轨迹，可以省略。
*length* | m | float64 | 从起始节点到结束节点的路径长度<br><br>可选：<br>此值由线引导AGV用于在到达停止位置之前降低速度。
***corridor*** | | JSON 对象 | 定义车辆可以偏离其轨迹的边界，例如，避开障碍物。<br>
**action [action]**<br><br><br> } | | array | 在边缘上执行的动作数组。<br>如果不需要动作，则为空数组。<br>由边缘触发的动作仅在AGV遍历触发动作的边缘时才会激活。<br>当AGV离开边缘时，动作将停止并恢复进入边缘之前的状态。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**trajectory** { | | JSON 对象 |
degree | | float64 | 范围：[1.0 ... float64.max]<br><br>定义轨迹的NURBS曲线的度数。<br><br>如果未定义，默认值为1。
**knotVector [float64]** | | array | 范围：[0.0 ... 1.0]<br><br>NURBS的节点值数组。<br><br>knotVector的大小为控制点数量+度数+1。
**controlPoints [controlPoint]**<br><br> } | | array | 定义NURBS控制点的controlPoint对象数组，显式包括起始点和结束点。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**controlPoint** { | | JSON 对象 |
x | | float64 | 在世界坐标系中描述的X坐标。
y | | float64 | 在世界坐标系中描述的Y坐标。
*weight* | | float64 | 范围：[0.0 ... float64.max]<br><br>控制点在曲线上的权重。<br>如果未定义，默认值为1.0。
} | | |

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
***corridor*** { | | JSON 对象 |
leftWidth | m | float64 | 范围：[0.0 ... float64.max]<br>定义相对于车辆轨迹的左侧走廊的宽度（以米为单位）（见图13）。
rightWidth | m | float64 | 范围：[0.0 ... float64.max]<br>定义相对于车辆轨迹的右侧走廊的宽度（以米为单位）（见图13）。
*corridorRefPoint* <br><br>**}**| | string | 定义边界是对运动中心还是车辆轮廓有效。如果未指定，边界对车辆的运动中心有效。<br>枚举{'KINEMATICCENTER'，'CONTOUR'}

### 6.7 Maps
### 6.7 地图

为了确保不同类型AGV之间的一致导航，位置始终相对于项目特定的坐标系指定（见图11）。为了区分站点或位置的不同级别，使用唯一的`mapId`。地图坐标系应指定为右手坐标系，z轴指向天空。因此，正旋转应理解为逆时针旋转。车辆坐标系也指定为右手坐标系，x轴指向车辆的前进方向，z轴指向上方。车辆参考点在车辆参考框架中定义为（0,0,0），除非另有说明。这符合DIN ISO 8855第2.11节。

![图11 带有示例AGV和方向的坐标系](./assets/coordinate_system_vehicle_orientation.png)
>图11 带有示例AGV和方向的坐标系

X、Y和Z坐标应以米为单位给出。方向应以弧度为单位，并应在+Pi和-Pi之间。

![图12 地图和车辆的坐标系](./assets/coordinate_system_vehicle_map.png)
>图12 地图和车辆的坐标系

### 6.7.1 Map distribution
### 6.7.1 地图分发

为了实现自动地图分发和智能管理车辆的重启，介绍了一种标准化的地图分发方式。

要分发的地图文件存储在车辆可访问的专用地图服务器上。为了确保高效传输，每次传输应由单个文件组成。如果需要多个地图或文件，应将它们打包或打包成一个文件。从地图服务器到车辆的地图传输是一个拉操作，由主控系统使用`instantAction`触发下载命令。

每个地图由地图标识符（字段`mapId`）和地图版本（字段`mapVersion`）的组合唯一标识。地图标识符描述车辆物理工作空间的特定区域，地图版本指示对先前版本的更新。在接受新命令之前，车辆应检查请求命令中的每个地图标识符在车辆上是否有地图。主控系统负责确保激活正确的地图以操作车辆。

为了最大限度地减少停机时间并使主控系统更容易同步新地图的激活，必须在车辆上预加载或缓冲地图。可以通过车辆状态通道访问车辆上的地图状态。需要注意的是，将地图传输到AGV然后激活地图是不同的过程。要在车辆上激活预加载的地图，主控系统发送即时动作。在这种情况下，任何其他具有相同地图标识符但不同地图版本的地图将自动禁用。主控系统可以通过另一个即时动作删除地图。此过程的结果在车辆状态中显示。

地图分发过程如图13所示。

![图13 地图分发过程](./assets/map_distribution_process.png)
>图13 下载、启用和删除地图所需的主控系统、AGV和地图服务器之间的通信。

#### 6.7.2 Maps in the vehicle state
#### 6.7.2 车辆状态中的地图

状态中的`agvPosition`字段中的`mapId`表示当前活动的地图。车辆上可用地图的信息显示在状态消息的`maps`数组中。此数组中的每个条目都是一个JSON对象，由必填字段`mapId`、`mapVersion`和`mapStatus`组成，后者可以是'ENABLED'或'DISABLED'。'ENABLED'地图可以在需要时由车辆使用。'DISABLED'地图不应使用。下载过程的状态由当前动作未完成指示。错误也在状态中报告。

请注意，具有不同`mapId`的多个地图可以同时启用。一次只能启用一个具有相同`mapId`的地图版本。如果`maps`数组为空，这意味着车辆上当前没有可用的地图。

#### 6.7.3 Map download
#### 6.7.3 地图下载

地图下载由主控系统的`downloadMap`即时动作触发。此命令包含存储在地图服务器上的地图的必填参数`mapId`和`mapDownloadLink`，车辆可以访问这些参数。

AGV在开始下载地图文件时将`actionStatus`设置为“RUNNING”。如果下载成功，`actionStatus`更新为“FINISHED”。如果下载不成功，状态设置为“FAILED”。下载成功完成后，地图应添加到状态中的`maps`数组中。在地图准备好启用之前，不应在状态中报告地图。

确保下载地图的过程不会修改、删除、启用或禁用车辆上的任何现有地图非常重要。
车辆应拒绝下载已在车辆上的`mapId`和`mapVersion`的地图。应报告错误，并将即时动作的状态设置为“FAILED”。主控系统应首先删除车辆上的地图，然后重新启动下载。

#### 6.7.4 Enable downloaded maps
#### 6.7.4 启用下载的地图

有两种方法可以在车辆上启用地图：

1. **主控系统启用地图**：使用`enableMap`即时动作在车辆上将地图设置为“ENABLED”。其他具有相同`mapId`但不同`mapVersion`的版本设置为“DISABLED”。
2. **手动在车辆上启用地图**：在某些情况下，可能需要直接在车辆上启用地图。结果应在车辆状态中报告。

主控系统负责确保在向车辆发送命令中的`nodePosition`的一部分时激活车辆上的正确地图。
如果车辆要在新地图上设置为特定位置，则使用`initPosition`即时动作。

#### 6.7.5 Delete maps on vehicle
#### 6.7.5 删除车辆上的地图

主控系统可以请求从车辆中删除特定地图。这是通过即时动作`deleteMap`完成的。当车辆内存不足时，应报告给主控系统，主控系统可以启动地图的删除。车辆本身不允许删除地图。
成功删除地图后，重要的是从车辆状态中的地图数组中删除该地图的条目。

## 6.8 Actions
## 6.8 动作

如果AGV支持驾驶以外的动作，这些动作通过附加到节点或边缘的动作字段执行，或通过单独的主题`instantActions`发送（参见[6.9 主题 "即时动作"](#69-topic-instantactions-from-master-control-to-agv)部分）。

在边缘上执行的动作应仅在AGV在边缘上时运行（参见[6.11.2 节点的遍历和进入/离开边缘](#6102-traversal-of-nodes-and-enteringleaving-edges-triggering-of-actions)部分）。

在节点上触发的动作可以运行，只要它们需要运行，并且应是自终止的（例如，持续五秒的音频信号或拾取动作，完成后拾取负载）或成对制定（例如，“activateWarningLights”和“deactivateWarningLights”），尽管可能有例外。

以下部分介绍了AGV应使用的预定义动作，如果AGV的能力映射到动作描述。如果有合理的方法使用定义的参数，则应使用它们。如果需要成功执行动作，可以定义附加参数。

如果没有办法将某些动作映射到以下部分的动作之一，AGV制造商可以定义主控系统应使用的附加动作。

### 6.8.1 Definition, parameters, effects and scope of predefined actions
### 6.8.1 预定义动作的定义、参数、效果和范围

general | | scope
:---:|--- | :---:
action, counter action, description, idempotent, parameters | linked state | instant, node, edge

action | counter action | description | idempotent | parameters | linked state | instant | node | edge
---|---|---|---|---|---|---|---|---
startPause | stopPause | 激活暂停模式。<br>需要链接状态，因为许多AGV可以通过使用硬件开关暂停。<br>不再有AGV驾驶运动 - 不需要到达下一个节点。<br>动作可以继续。<br>命令是可恢复的。 | 是 | - | paused | 是 | 否 | 否
stopPause | startPause | 停用暂停模式。<br>运动和所有其他动作将恢复（如果有）。<br>需要链接状态，因为许多AGV可以通过使用硬件开关暂停。<br>stopPause还可以重新启动通过硬件按钮触发startPause的车辆（如果已配置）。 | 是 | - | paused | 是 | 否 | 否
startCharging | stopCharging | 激活充电过程。<br>充电可以在充电点（车辆站立）或充电车道（驾驶时）进行。<br>过充保护是车辆的责任。 | 是 | - | .batteryState.charging | 是 | 是 | 否
stopCharging | startCharging | 停用充电过程以发送新命令。<br>充电过程也可以由车辆/充电站中断，例如，如果电池已满。<br>电池状态仅允许为“false”，当AGV准备接收命令时。 | 是 | - |.batteryState.charging | 是 | 是 | 否
initPosition | - | 使用给定参数重置（覆盖）AGV的姿态。 | 是 | x (float64)<br>y (float64)<br>theta (float64)<br>mapId (string)<br>lastNodeId (string) | .agvPosition.x<br>.agvPosition.y<br>.agvPosition.theta<br>.agvPosition.mapId<br>.lastNodeId<br>.maps | 是 | 是<br>(电梯) | 否
enableMap | - | 显式启用先前下载的地图以在命令中使用，而无需初始化新位置。 | 是 | mapId (string)<br>mapVersion (string) | .maps | 是 | 是 | 否
downloadMap | - | 触发新地图的下载。在下载期间处于活动状态。错误在车辆状态中报告。完成后验证成功下载，准备地图以供使用并在状态中设置地图。 | 是 | mapId (string)<br>mapVersion (string)<br>mapDownloadLink (string)<br>mapHash (string, optional) | .maps | 是 | 否 | 否
deleteMap | - | 触发从车辆内存中删除地图。 | 是 | mapId (string)<br>mapVersion (string) | .maps | 是 | 否 | 否
stateRequest | - | 请求AGV发送新的状态报告。 | 是 | - | - | 是 | 否 | 否
logReport | - | 请求AGV生成并存储日志报告。 | 是 | reason<br>(string) | - | 是 | 否 | 否
pick | drop<br><br>(如果自动化) | 请求AGV拾取负载。<br>具有多个负载处理设备的AGV可以并行处理多个拾取操作。<br>在这种情况下，参数lhd需要存在（例如，LHD1）。<br>参数stationType通知如何详细处理拾取操作（例如，地面位置、货架位置、被动输送机、主动输送机等）。<br>负载类型通知负载单元，并可用于切换字段（例如，EPAL，INDU等）。<br>为了准备负载处理设备（例如，基于高度参数的预提升操作），可以提前在视野中宣布动作。<br>但是，预提升操作等不在AGV状态中报告为“RUNNING”，因为相关节点尚未释放。<br>如果在边缘上，车辆可以使用其传感设备检测节点的拾取位置。 | 否 |lhd (string, optional)<br>stationType (string)<br>stationName(string, optional)<br>loadType (string) <br>loadId(string, optional)<br>height (float64) (optional)<br>定义与地板相关的负载底部<br>depth (float64) (optional) 用于叉车<br>side(string) (optional) 例如，输送机 | .load | 否 | 是 | 是
drop | pick<br><br>(如果自动化) | 请求AGV放下负载。<br>有关更多详细信息，请参见动作拾取。 | 否 | lhd (string, optional)<br>stationType (string, optional)<br>stationName (string, optional)<br>loadType (string, optional)<br>loadId(string, optional)<br>height (float64, optional)<br>depth (float64, optional) <br>… | .load | 否 | 是 | 是
detectObject | - | AGV检测对象（例如，负载、充电点、空闲停车位置）。 | 是 | objectType(string, optional) | - | 否 | 是 | 是
finePositioning | - | 在节点上，AGV将精确定位在目标上。<br>AGV被允许偏离其节点位置。<br>在边缘上，AGV将在遍历边缘时对齐固定设备。<br>即时动作：AGV开始精确定位在目标上。 | 是 | stationType(string, optional)<br>stationName(string, optional) | - | 否 | 是 | 是
waitForTrigger | - | AGV必须等待AGV上的触发器（例如，按钮按下，手动加载）。<br>主控系统负责处理超时，并在必要时取消命令。 | 是 | triggerType(string) | - | 否 | 是 | 否
cancelOrder | - | AGV尽快停止。<br>这可能是立即或在下一个节点上。<br>然后命令被删除。所有动作被取消。 | 是 | - | - | 是 | 否 | 否
factsheetRequest | - | 请求AGV发送事实表 | 是 | - | - | 是 | 否 | 否

### 6.8.2 States of predefined actions
### 6.8.2 预定义动作的状态

action | action states
---|:---:
 | | 'INITIALIZING', 'RUNNING', 'PAUSED', 'FINISHED', 'FAILED' |

action | 'INITIALIZING' | 'RUNNING' | 'PAUSED' | 'FINISHED' | 'FAILED'
---|---|---|---|---|---
startPause | - | 正在准备激活模式。<br>如果AGV支持即时过渡，可以省略此状态。 | - | 车辆静止。<br>所有动作将暂停。<br>暂停模式已激活。<br>AGV报告.paused: "true"。 | 由于某种原因无法激活暂停模式（例如，被硬件开关覆盖）。
stopPause | - | 正在准备停用模式。<br>如果AGV支持即时过渡，可以省略此状态。 | - | 暂停模式已停用。<br>所有暂停的动作将恢复。<br>AGV报告.paused: "false"。 | 由于某种原因无法停用暂停模式（例如，被硬件开关覆盖）。
startCharging | - | 正在进行充电过程的激活（与充电器的通信正在进行）。<br>如果AGV支持即时过渡，可以省略此状态。 | - | 充电过程已开始。<br>AGV报告.batteryState.charging: "true"。 | 由于某种原因无法启动充电过程（例如，与充电器未对齐）。充电问题应与错误相对应。
stopCharging | - | 正在进行充电过程的停用（与充电器的通信正在进行）。<br>如果AGV支持即时过渡，可以省略此状态。 | - | 充电过程已停止。<br>AGV报告.batteryState.charging: "false" | 由于某种原因无法停止充电过程（例如，与充电器未对齐）。<br>充电问题应与错误相对应。
initPosition | - | 正在进行新姿态的初始化（置信度检查等）。<br>如果AGV支持即时过渡，可以省略此状态。 | - | 姿态已重置。<br>AGV报告<br>.agvPosition.x = x,<br>.agvPosition.y = y,<br>.agvPosition.theta = theta<br>.agvPosition.mapId = mapId<br>.agvPosition.lastNodeId = lastNodeId | 姿态无效或无法重置。<br>一般定位问题应与错误相对应。
| downloadMap | 初始化与地图服务器的连接。 | AGV正在下载地图，直到下载完成。 | - | AGV通过设置mapId/mapVersion和相应的mapStatus为'DISABLED'来更新其状态。 | 下载失败，在车辆状态中更新（例如，连接丢失，地图服务器无法访问，地图ID/版本在地图服务器上不存在）。 |
| enableMap | - | AGV启用请求的mapId和mapVersion的地图，同时禁用具有相同mapId的其他版本。 | - | AGV将请求地图的相应mapStatus更新为'ENABLED'，并将具有相同mapId的其他版本更新为'DISABLED'。 | 请求的mapId/mapVersion组合不存在。|
| deleteMap | - | AGV从其内部内存中删除请求的mapId和mapVersion的地图。 | - | AGV从其状态中删除mapId/mapVersion。 | 如果地图当前正在使用，则无法删除地图。请求的mapId/mapVersion组合已被删除。 |
stateRequest | - | - | - | 状态已被传达 | -
logReport | - | 报告正在生成。<br>如果AGV支持即时生成，可以省略此状态。 | - | 报告已存储。<br>日志的名称将在状态中报告。 | 无法存储报告（例如，没有空间）。
pick | 正在初始化拾取过程，例如，未完成的提升操作。 | 拾取过程正在进行中（AGV正在进入站点，负载处理设备正在忙碌，与站点的通信正在进行等）。 | 拾取过程正在暂停，例如，如果安全区域被违反。<br>在移除违规后，拾取过程继续。 | 拾取已完成。<br>负载已进入AGV，AGV报告新的负载状态。 | 拾取失败，例如，站点意外为空。<br>失败的拾取操作应与错误相对应。
drop | 正在初始化放下过程，例如，未完成的提升操作。 | 放下过程正在进行中（AGV正在进入站点，负载处理设备正在忙碌，与站点的通信正在进行等）。 | 放下过程正在暂停，例如，如果安全区域被违反。<br>在移除违规后，放下过程继续。 | 放下已完成。<br>负载已离开AGV，AGV报告新的负载状态。 | 放下失败，例如，站点意外被占用。<br>失败的放下操作应与错误相对应。
detectObject | - | 对象检测正在进行中。 | - | 对象已被检测到。 | AGV无法检测到对象。
finePositioning | - | AGV精确定位在目标上。 | 精确定位过程正在暂停，例如，如果安全区域被违反。<br>在移除违规后，精确定位继续。 | 目标位置相对于站点已达到。 | 目标位置相对于站点无法达到。
waitForTrigger | - | AGV正在等待触发器 | - | 触发器已被触发。 | waitForTrigger失败，如果命令已被取消。
cancelOrder | - | AGV正在停止或驾驶，直到到达下一个节点。 | - | AGV静止并已取消命令。 | -
factsheetRequest | - | - | - | 事实表已被传达 | -

## 6.9 Topic: "instantActions" (from master control to AGV)
## 6.9 主题："即时动作"（从主控系统到AGV）

在某些情况下，需要向AGV发送需要立即执行的动作。这可以通过将`instantAction`消息发布到主题`instantActions`来实现。这些不应与AGV当前命令的内容冲突（例如，`instantAction`降低叉子，而命令说要提升叉子）。

一些即时动作可能相关的示例包括：
- 暂停AGV而不更改当前命令中的任何内容；
- 暂停后恢复命令；
- 激活信号（光学、音频等）。

有关更多信息，请参见[7 最佳实践](#7-best-practice)。

对象结构 | 数据类型 | 描述
---|---|---
headerId | uint32 | 消息的头ID。<br>头ID是每个主题定义的，并随着每个发送的（但不一定接收的）消息递增1。
timestamp | string | 时间戳（ISO 8601，UTC）；YYYY-MM-DDTHH:mm:ss.ffZ（例如，“2017-04-15T11:40:03.12Z”）
version | string | 协议的版本[Major].[Minor].[Patch]（例如，1.3.2）
manufacturer | string | AGV的制造商
serialNumber | string | AGV的序列号
actions [action] | array | 需要立即执行的动作数组，不属于常规命令的一部分。

当AGV接收到`instantAction`时，会在AGV的状态中添加一个适当的`actionStatus`到`actionStates`数组中。`actionStatus`会根据动作的进展进行更新。有关`actionStatus`的不同转换，请参见图16。

## 6.10 Topic: "state" (from AGV to master control)
## 6.10 主题："状态"（从AGV到主控系统）

AGV状态将仅在一个主题上进行传输。与单独的消息（例如，命令、电池状态和错误）相比，使用一个主题将减少代理和主控系统处理消息的工作量，同时也保持AGV状态信息的同步。

AGV状态消息将在相关事件发生时或最迟每30秒通过MQTT代理发布到主控系统。

触发状态消息传输的事件包括：
- 接收到命令
- 接收到命令更新
- 负载状态的变化
- 错误或警告
- 驶过节点
- 切换操作模式
- `driving`字段的变化
- `nodeStates`、`edgeStates`或`actionStates`的变化
- `maps`字段的变化

应努力减少通信量。如果两个事件相互关联（例如，接收新命令通常会强制更新`nodeStates`和`edgeStates`；驶过节点也是如此），则应触发一次状态更新而不是多次。

### 6.10.1 Concept and logic
### 6.10.1 概念和逻辑

命令进度由`nodeStates`和`edgeStates`跟踪。此外，如果AGV能够推导出其当前位置，它可以通过`position`字段发布其位置。

如果AGV自行规划路径，它应通过状态消息中的`trajectory`对象以NURBS的形式传达其计算的轨迹（包括基础和视野），除非主控系统无法使用此字段，并且在集成期间已同意不发送此字段。在节点被主控系统释放后，AGV不允许更改其轨迹。

`nodeStates`和`edgeStates`包括AGV仍需遍历的所有节点/边缘。

![图14 状态主题提供的命令信息。仅传输最后一个节点的ID和剩余的节点和边缘](./assets/order_information_state_topic.png)
>图14 状态主题提供的命令信息。仅传输最后一个节点的ID和剩余的节点和边缘

### 6.10.2 Traversal of nodes and entering/leaving edges, triggering of actions
### 6.10.2 节点的遍历和进入/离开边缘，动作的触发

AGV自行决定何时将节点视为已遍历。通常，AGV的控制点应在节点的`allowedDeviationXY`范围内，其方向应在`allowedDeviationTheta`范围内。如果后续边缘的边缘属性`corridor`被设置，则还应满足这些边界。

AGV通过从`nodeStates`数组中移除其`nodeState`并将`lastNodeId`、`lastNodeSequenceId`设置为已遍历节点的值来报告节点的遍历。

一旦AGV报告节点已遍历，AGV应触发与节点相关联的动作（如果有）。

节点的遍历也标志着离开通向该节点的边缘。然后应从`edgeStates`中移除该边缘，并完成在边缘上活动的动作。

节点的遍历还标志着AGV进入下一个边缘的时刻（如果有）。此时应触发边缘的动作。此规则的一个例外是，如果AGV必须在边缘上暂停（由于软或硬阻塞边缘或其他原因）——那么AGV在再次开始移动后进入边缘。

![图15 在命令处理期间的nodeStates、edgeStates和actionStates的描述](./assets/states_during_order_handling.png)
>图15 在命令处理期间的`nodeStates`、`edgeStates`和`actionStates`的描述

### 6.10.3 Base request
### 6.10.3 基础请求

如果AGV检测到其基础即将耗尽，它可以将`newBaseRequest`标志设置为“true”以防止不必要的制动。

### 6.10.4 Information
### 6.10.4 信息

AGV可以通过`information`数组向主控系统提交任意附加信息。由AGV决定报告信息的时间长度。

主控系统不应使用信息消息进行逻辑处理，仅应用于可视化和调试目的。

### 6.10.5 Errors
### 6.10.5 错误

AGV通过`errors`数组报告错误。错误有两个级别：“警告”和“致命”。“警告”是自我解决的错误，例如，字段违规。“致命”错误需要人工干预。错误可以通过`errorReferences`数组传递帮助找到错误原因的引用。

### 6.10.6 Implementation of the state message
### 6.10.6 状态消息的实现

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
headerId | | uint32 | 消息的头ID。<br>头ID是每个主题定义的，并随着每个发送的（但不一定接收的）消息递增1。
timestamp | | string | 时间戳（ISO 8601，UTC）；YYYY-MM-DDTHH:mm:ss.ffZ（例如，“2017-04-15T11:40:03.12Z”）。
version | | string | 协议的版本[Major].[Minor].[Patch]（例如，1.3.2）。
manufacturer | | string | AGV的制造商。
serialNumber | | string | AGV的序列号。
*maps[map]* | | array | 当前存储在车辆上的地图对象数组。
orderId| | string | 当前命令或先前完成命令的唯一命令标识。<br>orderId保持不变，直到接收到新命令。<br>如果没有可用的先前orderId，则为空字符串（""）。
orderUpdateId | | uint32 | 命令更新标识，以识别AGV已接受的命令更新。<br>如果没有可用的先前orderUpdateId，则为“0”。
*zoneSetId* | |string | AGV当前用于路径规划的区域集的唯一ID。<br>应与命令中使用的相同。<br><br>可选：如果AGV不使用区域，则可以省略此字段。
lastNodeId | | string | 最后到达节点的节点ID，或者如果AGV当前在节点上，则为当前节点（例如，“node7”）。如果没有可用的`lastNodeId`，则为空字符串（""）。
lastNodeSequenceId | | uint32 | 最后到达节点的序列ID，或者如果AGV当前在节点上，则为当前节点的序列ID。<br>如果没有可用的`lastNodeSequenceId`，则为“0”。
**nodeStates [nodeState]** | |array | 需要遍历以完成命令的nodeState对象数组<br>（如果空闲则为空数组）
**edgeStates [edgeState]** | |array | 需要遍历以完成命令的edgeState对象数组<br>（如果空闲则为空数组）
***agvPosition*** | | JSON对象 | AGV在地图上的当前位置。<br><br>可选：仅适用于没有自我定位能力的AGV，例如，线导向AGV。
***velocity*** | | JSON对象 | AGV在车辆坐标中的速度。
***loads [load]*** | | array | AGV当前处理的负载。<br><br>可选：如果AGV无法确定负载状态，则应完全省略此字段，而不是报告为空数组。<br>如果AGV可以确定负载状态，但数组为空，则认为AGV未加载。
driving | | boolean | “true”：表示AGV正在驾驶和/或旋转。AGV的其他运动（例如，提升运动）不包括在内。<br>“false”：表示AGV既不驾驶也不旋转。
*paused* | | boolean | “true”：AGV当前处于暂停状态，可能是由于AGV上的物理按钮或即时动作。<br>AGV可以恢复命令。<br><br>“false”：AGV当前不处于暂停状态。
*newBaseRequest* | | boolean | “true”：AGV几乎到达基础的末端，如果没有传输新的基础，将减速。<br>触发主控系统发送新的基础。<br><br>“false”：不需要基础更新。
*distanceSinceLastNode* | meter | float64 | 用于线导向车辆，指示其在lastNodeId之后行驶的距离。<br>距离以米为单位。
**actionStates [actionState]** | | array | 包含当前命令的所有动作和自上次命令以来接收到的所有即时动作的数组。动作状态在接收到新命令时保留。动作状态，除了正在运行的即时动作，在接收到新命令时被移除。<br>这可能包括仍在进行中的先前节点的动作。<br><br>当动作完成时，发布更新的状态消息，actionStatus设置为“FINISHED”，如果适用，带有相应的resultDescription。
**batteryState** | | JSON对象 | 包含所有与电池相关的信息。
operatingMode | | string | 枚举{'AUTOMATIC', 'SEMIAUTOMATIC', 'MANUAL', 'SERVICE', 'TEACHIN'}<br>有关更多信息，请参见[6.10.6 状态消息的实现](#6106-implementation-of-the-state-message)中的表1。
**errors [error]** | | array | 错误对象数组。<br>AGV的所有活动错误应在数组中。<br>空数组表示AGV没有活动错误。
***information [info]*** | | array | 信息对象数组。<br>空数组表示AGV没有信息。<br>这仅应用于可视化或调试——不应用于主控系统中的逻辑。
**safetyState** | | JSON对象 | 包含所有与安全相关的信息。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**map**{ | | JSON对象|
mapId | | string | 描述车辆工作空间定义区域的地图ID。
mapVersion | | string | 地图的版本。
*mapDescription* | | string | 地图的附加信息。
mapStatus <br>}| | string | 枚举{'ENABLED', 'DISABLED'}<br>'ENABLED'：表示此地图当前在AGV上处于活动状态/使用中。最多只有一个具有相同mapId的地图可以将其状态设置为'ENABLED'。<br>'DISABLED'：表示此地图版本当前未在AGV上启用，因此可以通过请求启用或删除。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**nodeState** { | JSON对象 | |
nodeId | | string | 唯一的节点标识。
sequenceId | | uint32 | 区分具有相同nodeId的多个节点的序列ID。
*nodeDescription* | | string | 节点的附加信息。
released| | boolean | “true”表示节点是基础的一部分。<br>“false”表示节点是视野的一部分。
***nodePosition***<br><br>}| | JSON对象 | 节点位置。<br>该对象在[6.6 主题“命令”](#66-topic-order-from-master-control-to-agv)中定义<br>可选：<br>主控系统有此信息。<br>可以额外发送，例如，用于调试目的。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**edgeState** { | | JSON对象 | |
edgeId | | string | 唯一的边缘标识。
sequenceId | | uint32 | 区分具有相同edgeId的多个边缘的序列ID。
*edgeDescription* | | string | 边缘的附加信息。
released | | boolean | “true”表示边缘是基础的一部分。<br>“false”表示边缘是视野的一部分。
***trajectory*** <br><br>} | | JSON对象 | 轨迹以NURBS的形式进行通信，并在[6.6.6 命令消息的实现](#666-implementation-of-the-order-message)中定义<br><br>轨迹段从车辆进入边缘的点开始，并在车辆报告终点节点已遍历时终止。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**agvPosition** { | | JSON对象 | 在世界坐标中定义地图上的位置。每个楼层都有自己的地图。
positionInitialized | | boolean | “true”：位置已初始化。<br>“false”：位置未初始化。
*localizationScore* | | float64 | 范围：[0.0 ... 1.0]<br><br>描述定位的质量，因此可以用于，例如，SLAM AGV描述当前位置信息的准确性。<br><br>0.0：位置未知<br>1.0：位置已知<br><br>可选：对于无法估计其定位分数的车辆。<br><br>仅用于日志记录和可视化目的。
*deviationRange* | m | float64 | 位置偏差范围的值，以米为单位。<br><br>可选：对于无法估计其偏差的车辆，例如，基于网格的定位。<br><br>仅用于日志记录和可视化目的。
x | m | float64 | 参考地图坐标系的地图上的X位置。<br>精度取决于具体实现。
y | m | float64 | 参考地图坐标系的地图上的Y位置。<br>精度取决于具体实现。
theta | | float64 | 范围：[-Pi ... Pi]<br><br>AGV的方向。
mapId | | string | 参考位置的地图的唯一标识。<br><br>每个地图都有相同的坐标原点。<br>当AGV使用电梯从出发楼层到目标楼层时，它会离开出发楼层的地图，并在目标楼层的相应电梯节点上生成。
*mapDescription*<br>} | | string | 地图的附加信息。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**velocity** { | | JSON对象 |
*vx* | m/s | float64 | AGV在其X方向的速度。
*vy* | m/s | float64 | AGV在其Y方向的速度。
*omega*<br>}| Rad/s | float64 | AGV围绕其Z轴的转速。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**load** { | | JSON对象 |
*loadId* | | string | 负载的唯一标识（例如，条形码或RFID）。<br><br>如果AGV可以识别负载但尚未识别负载，则为空字段。<br><br>可选：如果AGV无法识别负载。
*loadType* | | string | 负载类型。
*loadPosition* | | string | 指示AGV使用的负载处理/携带单元，例如，在AGV有多个位置/位置携带负载的情况下。<br><br>例如：“前”，“后”，“位置C1”等。<br><br>可选：对于只有一个loadPosition的车辆
***boundingBoxReference*** | | JSON对象 | 用于定位边界框的位置的参考点。<br>参考点始终是边界框底面（在高度=0）的中心，并在AGV的坐标系中描述。
***loadDimensions*** | | JSON对象 | 负载边界框的尺寸，以米为单位。
*weight*<br>} | kg | float64 | 范围：[0.0 ... float64.max]<br><br>以千克为单位测量的负载的绝对重量。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**boundingBoxReference** { | | JSON对象 | 用于定位边界框的位置的参考点。<br>参考点始终是边界框底面（在高度=0）的中心，并在AGV的坐标系中描述。
x | | float64 | 参考点的X坐标。
y | | float64 | 参考点的Y坐标。
z | | float 64 | 参考点的Z坐标。
*theta*<br> } | | float64 | 负载边界框的方向。<br>对于拖车、列车等很重要。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**loadDimensions** { | | JSON对象 | 负载边界框的尺寸，以米为单位。
length | m | float64 | 负载边界框的绝对长度。
width | m | float64 | 负载边界框的绝对宽度。
*height* <br>}| m | float64 | 负载边界框的绝对高度。<br><br>可选：<br><br>仅在已知时设置值。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**actionState** { | | JSON对象 |
actionId | |string | 动作的唯一标识符。
*actionType* | | string | 动作的类型。<br><br>可选：仅用于信息或可视化目的。主控系统知道在命令中分派的动作类型。
*actionDescription* | | string | 当前动作的附加信息。
actionStatus | | string | 枚举{'WAITING', 'INITIALIZING', 'RUNNING', 'PAUSED', 'FINISHED', 'FAILED'}<br><br>参见[6.11 actionStates](#611-actionstates)。
*resultDescription*<br>} | | string | 结果的描述，例如，RFID读取的结果。<br><br>错误将在错误中传输。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**batteryState** { | | JSON对象 | 
batteryCharge | % | float64 | 电量状态：<br>如果AGV仅提供良好或不良电池电量的值，这些将分别表示为20%（不良）和80%（良好）。
*batteryVoltage* | V | float64 | 电池电压。
*batteryHealth* | % | int8 | 范围：[0 ... 100]<br><br>描述电池健康状况的状态。 
charging | | boolean | “true”：正在充电。<br>“false”：AGV当前未充电。
*reach* <br>}| m | uint32 | 范围：[0 ... uint32.max]<br><br>当前电量状态下的估计可达范围。 

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**error** { | | JSON对象 |
errorType | | string | 错误的类型/名称
***errorReferences [errorReference]*** | | array | 引用数组（例如，nodeId、edgeId、orderId、actionId等）以提供与错误相关的更多信息。<br>有关更多信息，请参见[7 最佳实践](#7-best-practice)。
*errorDescription* | | string | 提供详细信息和可能原因的详细描述。
*errorHint* | | string | 关于如何处理或解决报告错误的提示。
errorLevel <br> }| | string | 枚举{'WARNING', 'FATAL'}<br><br>'WARNING'：AGV准备启动（例如，维护周期到期警告）。<br>'FATAL'：AGV不在运行状态，需要用户干预（例如，激光扫描仪被污染）。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**errorReference** { | | JSON对象 |
referenceKey | | string | 指定使用的引用类型（例如，nodeId、edgeId、orderId、actionId等）。
referenceValue <br>} | | string | 属于引用键的值。例如，发生错误的节点的ID。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**info** { | | JSON对象 |
infoType | | string | 信息的类型/名称。
*infoReferences [infoReference]* | | array | 引用数组。
*infoDescription* | | string | 信息的描述。
infoLevel <br>}| | string | 枚举{'DEBUG', 'INFO'}<br><br>'DEBUG'：用于调试。<br> 'INFO'：用于可视化。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**infoReference** { | | JSON对象 |
referenceKey | | string | 引用的类型（例如，headerId、orderId、actionId等）。
referenceValue <br>} | | string | 引用的值，属于引用键。

对象结构 | 单位 | 数据类型 | 描述
---|---|---|---
**safetyState** { | | JSON对象 |
eStop | | string | 枚举{'AUTOACK', 'MANUAL', 'REMOTE', 'NONE'}<br><br>紧急停止的确认类型：<br>'AUTOACK'：自动确认的紧急停止已激活，例如，由保险杠或保护区域触发。<br>'MANUAL'：紧急停止必须在车辆上手动确认。<br>'REMOTE'：设施的紧急停止必须远程确认。<br>'NONE'：没有激活紧急停止。
fieldViolation<br>} | | boolean | 保护区域违规。<br>"true"：区域被违反<br>"false"：区域未被违反。

#### 操作模式描述
以下描述列出了主题“状态”的操作模式。

标识符 | 描述
---|---
AUTOMATIC | AGV完全由主控系统控制。<br>AGV根据主控系统的命令进行驾驶和执行动作。
SEMIAUTOMATIC | AGV由主控系统控制。<br>AGV根据主控系统的命令进行驾驶和执行动作。<br>驾驶速度由HMI控制（速度不能超过自动模式的速度）。<br>转向由自动控制（非安全HMI可能）。
MANUAL | 主控系统不控制AGV。<br>主管不向AGV发送驾驶命令或动作。<br>HMI可用于控制AGV的转向和速度以及处理设备。<br>AGV的位置发送到主控系统。<br>当AGV进入或离开此模式时，它会立即清除所有命令（需要安全HMI）。
SERVICE | 主控系统不控制AGV。<br>主控系统不向AGV发送驾驶命令或动作。<br>授权人员可以重新配置AGV。
TEACHIN | 主控系统不控制AGV。<br>主管不向AGV发送驾驶命令或动作。<br>AGV正在被教导，例如，映射由主控系统完成。

>表1 操作模式及其含义

## 6.11 Action states
## 6.11 动作状态

当AGV接收到一个`action`（无论是附加到`node`或`edge`还是通过`instantAction`），它应在其`actionStates`数组中用一个`actionState`表示此`action`。

`actionStates`在字段`actionStatus`中描述动作生命周期的哪个阶段。

表2描述了枚举`actionStatus`可以持有的值。

actionStatus | 描述
---|---
'WAITING' | 动作已被AGV接收，但触发的节点尚未到达或活动的边缘尚未进入。
'INITIALIZING' | 动作已触发，准备措施已启动。
'RUNNING' | 动作正在运行。
'PAUSED' | 动作因暂停即时动作或外部触发（AGV上的暂停按钮）而暂停。
'FINISHED' | 动作已完成。<br>结果通过resultDescription报告。
'FAILED' | 动作因某种原因无法完成。

>表2 actionStatus字段的可接受值

状态转换图如图16所示。

![图16 actionStates的所有可能状态转换](./assets/action_state_transition.png)
>图16 actionStates的所有可能状态转换

## 6.12 Action blocking types and sequence
## 6.12 动作阻塞类型和顺序

多个动作在列表中的顺序定义了这些动作的执行顺序。动作的并行执行由其各自的`blockingType`控制。

动作可以有三种不同的阻塞类型，如表3所述。

blockingType | 描述
---|---
NONE | 动作可以与其他动作并行执行，并且车辆可以驾驶。
SOFT | 动作可以与其他动作并行执行。车辆不得驾驶。
HARD | 动作不得与其他动作并行执行。车辆不得驾驶。

>表3 动作阻塞类型

如果同一节点上有多个动作具有不同的阻塞类型，图17描述了AGV应如何处理这些动作。

![图17 处理多个动作](./assets/handling_multiple_actions.png)
>图17 处理多个动作

## 6.13 Topic "visualization"
## 6.13 主题"可视化"

为了实现接近实时的位置更新，AGV可以在主题`visualization`上广播其位置和速度。

位置对象的结构与状态中的位置和速度对象相同。有关更多信息，请参见[6.10.6 状态消息的实现](#6106-implementation-of-the-state-message)中的车辆状态。此主题的更新速率由集成商定义。

## 6.14 Topic "connection"
## 6.14 主题"连接"

在AGV客户端连接到代理期间，可以设置最后遗嘱主题和消息，当AGV客户端与代理断开连接时由代理发布。因此，主控系统可以通过订阅所有AGV的连接主题来检测断开事件。通过代理和客户端之间交换的心跳检测断开连接。大多数代理中可以配置间隔，应该设置为大约15秒。`connection`主题的服务质量级别应为1 - 至少一次。

建议的最后遗嘱主题结构为：

**uagv/v2/manufacturer/SN/connection**

最后遗嘱消息定义为具有以下字段的JSON封装消息：

标识符 | 数据类型 | 描述
---|---|---
headerId | uint32 | 消息的头ID。<br>头ID是每个主题定义的，并随着每个发送的（但不一定接收的）消息递增1。
timestamp | string | 时间戳（ISO8601，UTC）；YYYY-MM-DDTHH:mm:ss.ffZ（例如，“2017-04-15T11:40:03.12Z”）。
version | string | 协议的版本[Major].[Minor].[Patch]（例如，1.3.2）。
manufacturer | string | AGV的制造商。
serialNumber | string | AGV的序列号。
connectionState | string | 枚举{'ONLINE', 'OFFLINE', 'CONNECTIONBROKEN'}<br><br>'ONLINE'：AGV与代理的连接处于活动状态。<br><br>'OFFLINE'：AGV与代理的连接已以协调方式离线。<br><br> 'CONNECTIONBROKEN'：AGV与代理的连接意外中断。

当使用MQTT断开连接命令以优雅的方式结束连接时，不会发送最后遗嘱消息。只有在连接意外中断时，代理才会发送最后遗嘱消息。

**注意**：由于MQTT中最后遗嘱功能的性质，最后遗嘱消息在AGV与MQTT代理之间的连接阶段定义。因此，时间戳和headerId字段将始终过时。

AGV想要优雅地断开连接：

1. AGV发送“uagv/v2/manufacturer/SN/connection”，`connectionState`设置为`OFFLINE`。
2. 使用断开连接命令断开MQTT连接。

AGV上线：

1. 在创建MQTT连接时，将最后遗嘱设置为“uagv/v2/manufacturer/SN/connection”，字段`connectionState`设置为`CONNECTIONBROKEN`。
2. 发送主题“uagv/v2/manufacturer/SN/connection”，`connectionState`设置为`ONLINE`。

此主题上的所有消息应以保留标志发送。

当AGV与代理之间的连接意外停止时，代理将发送最后遗嘱主题：“uagv/v2/manufacturer/SN/connection”，字段`connectionState`设置为`CONNECTIONBROKEN`。

## 6.15 Topic "factsheet"
## 6.15 主题"事实表"

事实表提供有关特定AGV类型系列的基本信息。此信息允许比较不同的AGV类型，并可用于AGV系统的规划、尺寸和模拟。事实表还包括AGV通信接口的信息，这些信息是将AGV类型系列集成到符合VDA-5050的主控系统中所需的。

AGV事实表中的某些字段的值只能在系统集成期间指定，例如项目特定负载和站类型的分配，以及此AGV支持的站和负载类型列表。

事实表既是人类可读的文档，也是机器处理的文档，例如，主控系统应用程序的导入，因此被指定为JSON文档。

主控系统可以通过发送即时动作`factsheetRequest`从AGV请求事实表。

此主题上的所有消息应以保留标志发送。

### 6.15.1 Factsheet JSON structure
### 6.15.1 事实表JSON结构
事实表由下表中列出的JSON对象组成。

| **字段** | **数据类型** | **描述** |
| --- | --- | --- |
| headerId | uint32 | 消息的头ID。<br>头ID是每个主题定义的，并随着每个发送的（但不一定接收的）消息递增1。 |
| timestamp | string | 时间戳（ISO8601，UTC）；YYYY-MM-DDTHH:mm:ss.ffZ（例如，“2017-04-15T11:40:03.12Z”）。 |
| version | string | 协议的版本[Major].[Minor].[Patch]（例如，1.3.2）。 |
| manufacturer | string | AGV的制造商。 |
| serialNumber | string | AGV的序列号。 |
| **typeSpecification** | JSON对象 | 这些参数通常指定AGV的类别和能力。 |
| **physicalParameters** | JSON对象 | 这些参数指定AGV的基本物理属性。 |
| **protocolLimits** | JSON对象 | MQTT通信中标识符、数组、字符串等的长度限制。 |
| **protocolFeatures** | JSON对象 | 支持的VDA5050协议功能。 |
| **agvGeometry** | JSON对象 | AGV几何的详细定义。 |
| **loadSpecification** | JSON对象 | 负载能力的抽象规范。 |
| ***vehicleConfig*** | JSON对象 | 车辆上当前软件和硬件版本的摘要以及可选的网络信息。 |

#### typeSpecification

此JSON对象描述AGV类型的一般属性。

| **字段** | **数据类型** | **描述** |
|---|---|---|
| seriesName | string | 制造商指定的自由文本通用系列名称。 |
| *seriesDescription* | string | AGV类型系列的人类可读的自由文本描述。 |
| agvKinematic | string | AGV运动学类型的简化描述。<br/> [DIFF, OMNI, THREEWHEEL]<br/>DIFF：差动驱动，<br/>OMNI：全向车辆，<br/>THREEWHEEL：三轮驱动车辆或具有类似运动学的车辆。 |
| agvClass | string | AGV类别的简化描述。<br/>[FORKLIFT, CONVEYOR, TUGGER, CARRIER]<br/>FORKLIFT：叉车，<br/>CONVEYOR：带有传送带的AGV，<br/>TUGGER：拖车，<br/>CARRIER：带或不带提升单元的负载承载器。 |
| maxLoadMass | float64 | [kg]，最大可负载质量。 |
| localizationTypes | array of string | 定位类型的简化描述。<br/>示例值：<br/>NATURAL：自然地标，<br/>REFLECTOR：激光反射器，<br/>RFID：RFID标签，<br/>DMC：数据矩阵码，<br/>SPOT：磁点，<br/>GRID：磁网格。<br/>
| navigationTypes | array of string | AGV支持的路径规划类型数组，按优先级排序。<br/>示例值：<br/>PHYSICAL_LINE_GUIDED：无路径规划，AGV遵循物理安装的路径，<br/>VIRTUAL_LINE_GUIDED：AGV遵循固定的（虚拟）路径，<br/>AUTONOMOUS：AGV自主规划其路径。|

#### physicalParameters

此JSON对象描述AGV的物理属性。

| **字段** | **数据类型** | **描述** |
|---|---|---|
| speedMin | float64 | [m/s] AGV的最小控制连续速度。 |
| speedMax | float64 | [m/s] AGV的最大速度。 |
| *angularSpeedMin* | float64 | [Rad/s] AGV的最小控制连续旋转速度。 |
| *angularSpeedMax* | float64 | [Rad/s] AGV的最大旋转速度。 |
| accelerationMax | float64 | [m/s²] 最大负载下的最大加速度。 |
| decelerationMax | float64 | [m/s²] 最大负载下的最大减速度。 |
| heightMin | float64 | [m] AGV的最小高度。 |
| heightMax | float64 | [m] AGV的最大高度。 |
| width | float64 | [m] AGV的宽度。 |
| length | float64 | [m] AGV的长度。 |

#### protocolLimits

此JSON对象描述AGV的协议限制。
如果未定义参数或设置为零，则该参数没有明确限制。

| **字段** | **数据类型** | **描述** |
|---|---|---|
| **maxStringLens** { | JSON对象 | 字符串的最大长度。 |
| &emsp;*msgLen* | uint32 | 最大MQTT消息长度。 |
| &emsp;*topicSerialLen* | uint32 | MQTT主题中序列号部分的最大长度。<br/><br/>受影响的参数：<br/>order.serialNumber<br/>instantActions.serialNumber<br/>state.SerialNumber<br/>visualization.serialNumber<br/>connection.serialNumber |
| &emsp;*topicElemLen* | uint32 | MQTT主题中所有其他部分的最大长度。<br/><br/>受影响的参数：<br/>order.timestamp<br/>order.version<br/>order.manufacturer<br/>instantActions.timestamp<br/>instantActions.version<br/>instantActions.manufacturer<br/>state.timestamp<br/>state.version<br/>state.manufacturer<br/>visualization.timestamp<br/>visualization.version<br/>visualization.manufacturer<br/>connection.timestamp<br/>connection.version<br/>connection.manufacturer |
| &emsp;*idLen* | uint32 | ID字符串的最大长度。<br/><br/>受影响的参数：<br/>order.orderId<br/>order.zoneSetId<br/>node.nodeId<br/>nodePosition.mapId<br/>action.actionId<br/>edge.edgeId<br/>edge.startNodeId<br/>edge.endNodeId |
| &emsp;*idNumericalOnly* | boolean | 如果为“true”，ID字符串需要仅包含数字值。 |
| &emsp;*enumLen* | uint32 | 枚举和键字符串的最大长度。<br/><br/>受影响的参数：<br/>action.actionType action.blockingType<br/>edge.direction<br/>actionParameter.key<br/>state.operatingMode<br/>load.loadPosition<br/>load.loadType<br/>actionState.actionStatus<br/>error.errorType<br/>error.errorLevel<br/>errorReference.referenceKey<br/>info.infoType<br/>info.infoLevel<br/>safetyState.eStop<br/>connection.connectionState |
| &emsp;*loadIdLen* | uint32 | loadId字符串的最大长度。 |
| } | | |
| **maxArrayLens** { | JSON对象 | 数组的最大长度。 |
| &emsp;*order.nodes* | uint32 | AGV可处理的每个命令的最大节点数。 |
| &emsp;*order.edges* | uint32 | AGV可处理的每个命令的最大边缘数。 |
| &emsp;*node.actions* | uint32 | AGV可处理的每个节点的最大动作数。 |
| &emsp;*edge.actions* | uint32 | AGV可处理的每个边缘的最大动作数。 |
| &emsp;*actions.actionsParameters* | uint32 | AGV可处理的每个动作的最大参数数。 |
| &emsp;*instantActions* | uint32 | AGV可处理的每条消息的最大即时动作数。 |
| &emsp;*trajectory.knotVector* | uint32 | AGV可处理的每个轨迹的最大节点数。 |
| &emsp;*trajectory.controlPoints* | uint32 | AGV可处理的每个轨迹的最大控制点数。 |
| &emsp;*state.nodeStates* | uint32 | AGV发送的最大nodeStates数，AGV基础中的最大节点数。 |
| &emsp;*state.edgeStates* | uint32 | AGV发送的最大edgeStates数，AGV基础中的最大边缘数。 |
| &emsp;*state.loads* | uint32 | AGV发送的最大负载对象数。 |
| &emsp;*state.actionStates* | uint32 | AGV发送的最大actionStates数。 |
| &emsp;*state.errors* | uint32 | AGV在一条状态消息中发送的最大错误数。 |
| &emsp;*state.information* | uint32 | AGV在一条状态消息中发送的最大信息数。 |
| &emsp;*error.errorReferences* | uint32 | AGV为每个错误发送的最大错误引用数。 |
| &emsp;*information.infoReferences* | uint32 | AGV为每个信息发送的最大信息引用数。 |
| } | | |
| **timing** { | JSON对象 | 时间信息。 |
| &emsp;minOrderInterval | float32 | [s]，发送命令消息到AGV的最小间隔。 |
| &emsp;minStateInterval | float32 | [s]，发送状态消息的最小间隔。 |
| &emsp;*defaultStateInterval* | float32 | [s]，发送状态消息的默认间隔，*如果未定义，则使用主文档中的默认值*。 |
| &emsp;*visualizationInterval* | float32 | [s]，在可视化主题上发送消息的默认间隔。 |
| } | | |

#### protocolFeatures

此JSON对象定义AGV支持的动作和参数。

| **字段** | **数据类型** | **描述** |
|---|---|---|
| **optionalParameters** [**optionalParameter**] | JSON对象数组 | 支持和/或需要的可选参数数组。<br/>未在此列出的可选参数被假定为AGV不支持。 |
| { | | |
| &emsp;parameter | string | 可选参数的全名，例如，"*order.nodes.nodePosition.allowedDeviationTheta"*.|
| &emsp;support | 枚举 | 可选参数的支持类型，可能的值如下：<br/>'SUPPORTED'：可选参数按指定支持。<br/>'REQUIRED'：可选参数是AGV正常运行所必需的。 |
| &emsp;*description*| string | 自由格式文本：可选参数的描述，例如，<ul><li>为什么此AGV类型需要可选参数方向及其可能包含的值。</li><li>参数nodeMarker应仅包含无符号整数。</li><li>NURBS支持仅限于直线和圆弧段。</li>|
| } | | |
| **agvActions** [**agvAction**] | JSON对象数组 | 此AGV支持的所有动作及其参数的数组。这包括VDA5050中指定的标准动作和制造商特定的动作。 |
| { | | |
| &emsp;actionType | string | 对应于action.actionType的动作类型。 |
| &emsp;*actionDescription* | string | 自由格式文本：动作的描述。 |
| &emsp;actionScopes | 枚举数组 | 使用此动作类型的允许范围数组。<br/><br/>'INSTANT'：可用作即时动作。<br/>'NODE'：可用于节点。<br/>'EDGE'：可用于边缘。<br/><br/>例如：['INSTANT', 'NODE']|
| &emsp;***actionParameters** [**actionParameter**]* | JSON对象数组 | 动作具有的参数数组。<br/>如果未定义，则动作没有参数。<br/>此处定义的JSON对象与[6.6.6 命令消息的实现](#666-implementation-of-the-order-message)中节点和边缘内使用的JSON对象不同。|
|&emsp;*{* | | |
|&emsp;&emsp;key | string | 参数的键字符串。 |
|&emsp;&emsp;valueDataType | 枚举 | 值的数据类型，可能的数据类型为：'BOOL'，'NUMBER'，'INTEGER'，'FLOAT'，'STRING'，'OBJECT'，'ARRAY'。 |
|&emsp;&emsp;*description* | string | 自由格式文本：参数的描述。 |
|&emsp;&emsp;*isOptional* | boolean | “true”：可选参数。 |
|&emsp;*}* | | |
|*resultDescription* | string | 自由格式文本：结果的描述。 |
|*blockingTypes* | 枚举数组 | 为定义的动作可能的阻塞类型数组。 </br> 枚举{'NONE', 'SOFT', 'HARD'} |
|*}* | | |

### agvGeometry

此JSON对象定义AGV的几何属性，例如，轮廓和车轮位置。

| **字段** | **数据类型** | **描述** |
|---|---|---|
| ***wheelDefinitions** [**wheelDefinition**]* | JSON对象数组 | 车轮的数组，包含车轮布置和几何。 |
| { | | |
| &emsp;type | 枚举 | 车轮类型<br/> 枚举{'DRIVE', 'CASTER', 'FIXED', 'MECANUM'}。 |
| &emsp;isActiveDriven | boolean | “true”：车轮是主动驱动的。 |
| &emsp;isActiveSteered | boolean | “true”：车轮是主动转向的。 |
| &emsp;**position** { | JSON对象 | |
|&emsp;&emsp; x | float64 | [m]，AGV坐标系中的x位置。 |
|&emsp;&emsp; y | float64 | [m]，AGV坐标系中的y位置。 |
|&emsp;&emsp; *theta* | float64 | [rad]，AGV坐标系中车轮的方向。对于固定车轮是必要的。 |
| &emsp;} | | |
| &emsp;diameter | float64 | [m]，车轮的标称直径。 |
| &emsp;width | float64 | [m]，车轮的标称宽度。 |
| &emsp;*centerDisplacement* | float64 | [m]，车轮中心到旋转点的标称位移（对于脚轮是必要的）。<br/> 如果未定义此参数，则假定为0。 |
| &emsp;*constraints* | string | 自由格式文本：可由制造商用于定义约束。 |
| } | | |
| ***envelopes2d** [**envelope2d**]* | JSON对象数组 | AGV在2D中的包络曲线数组，例如，未加载和加载状态的机械包络，不同速度情况下的安全区域。 |
| { | | |
| &emsp;set | string | 包络曲线集的名称。 |
| &emsp;**polygonPoints** **[polygonPoint]** | JSON对象数组 | 包络曲线作为x/y多边形，多边形假定为闭合且不应自相交。 |
| &emsp;{ | | |
|&emsp;&emsp; x | float64 | [m]，多边形点的X位置。 |
|&emsp;&emsp; y | float64 | [m]，多边形点的Y位置。 |
| &emsp;} | | |
| &emsp;*description* | string | 自由格式文本：包络曲线集的描述。 |
| *}* | | |
| ***envelopes3d [envelope3d]*** | JSON对象数组 | AGV在3D中的包络曲线数组。 |
| *{* | | |
| &emsp;set | string | 包络曲线集的名称。 |
| &emsp;format | string | 数据的格式，例如，DXF。 |
| &emsp;***data*** | JSON对象 | 3D包络曲线数据，格式在'format'中指定。 |
| &emsp;*url* | string | 下载3D包络曲线数据的协议和URL定义，例如，<ftp://xxx.yyy.com/ac4dgvhoif5tghji>。 |
| &emsp;*description* | string | 自由格式文本：包络曲线集的描述 |
| *}* | | |

#### loadSpecification

此JSON对象指定AGV的负载处理和支持的负载类型。

| **字段** | **数据类型** | **描述** |
|---|---|---|
| *loadPositions* | 字符串数组 | 负载位置/负载处理设备的数组。<br/>此数组包含参数"state.loads[].loadPosition"和动作参数"lhd"的有效值，用于动作拾取和放置。<br/>*如果此数组不存在或为空，则AGV没有负载处理设备。* |
| ***loadSets [loadSet]*** | JSON对象数组 | AGV可以处理的负载集数组 |
| { | | |
|&emsp; setName | string | 负载集的唯一名称，例如，DEFAULT，SET1等。 |
|&emsp; loadType | string | 负载类型，例如，EPAL，XLT1200等。 |
|&emsp; *loadPositions* | 字符串数组 | 负载位置之间的负载处理设备数组，此负载集对其有效。<br/>*如果此参数不存在或为空，则此负载集对AGV上的所有负载处理设备有效。* |
|&emsp; ***boundingBoxReference*** | JSON对象 | 在状态消息中参数loads[]中定义的边界框参考。 |
|&emsp; ***loadDimensions*** | JSON对象 | 在状态消息中参数loads[]中定义的负载尺寸。 |
|&emsp; *maxWeight* | float64 | [kg]，负载类型的最大重量。 |
|&emsp; *minLoadhandlingHeight* | float64 | [m]，此负载类型和重量的最小允许高度<br/>参考边界框参考。 |
|&emsp; *maxLoadhandlingHeight* | float64 | [m]，此负载类型和重量的最大允许高度<br/>参考边界框参考。 |
|&emsp; *minLoadhandlingDepth* | float64 | [m]，此负载类型和重量的最小允许深度<br/>参考边界框参考。 |
|&emsp; *maxLoadhandlingDepth* | float64 | [m]，此负载类型和重量的最大允许深度<br/>参考边界框参考。 |
|&emsp; *minLoadhandlingTilt* | float64 | [rad]，此负载类型和重量的最小允许倾斜。 |
|&emsp; *maxLoadhandlingTilt* | float64 | [rad]，此负载类型和重量的最大允许倾斜。 |
|&emsp; *agvSpeedLimit* | float64 | [m/s]，此负载类型和重量的最大允许速度。 |
|&emsp; *agvAccelerationLimit* | float64 | [m/s²]，此负载类型和重量的最大允许加速度。 |
|&emsp; *agvDecelerationLimit* | float64 | [m/s²]，此负载类型和重量的最大允许减速度。 |
|&emsp; *pickTime* | float64 | [s]，拾取负载的近似时间 |
|&emsp; *dropTime* | float64 | [s]，放下负载的近似时间。 |
|&emsp; *description* | string | 自由格式文本：负载处理集的描述。 |
| } | | |

#### vehicleConfig

此JSON对象详细说明了车辆上运行的软件和硬件版本，以及网络信息的简要摘要。

| **字段** | **数据类型** | **描述** |
|---|---|---|
| *versions[versionInfo]* | JSON对象数组 | 包含软件和硬件信息的键值对对象数组。| | { | | |
|&emsp; key | string | 使用的软件/硬件版本的键。（例如，softwareVersion） |
|&emsp; value | string | 与键对应的版本。（例如，v1.12.4-beta） |
| } | | |
| *network* { | JSON对象 | 关于车辆网络连接的信息。列出的信息在车辆运行时不应更新。 |
|&emsp;&emsp; *dnsServers* | 字符串数组 | 车辆使用的域名服务器（DNS）数组。 |
|&emsp;&emsp; *ntpServers* | 字符串数组 | 车辆使用的网络时间协议（NTP）服务器数组。 |
|&emsp;&emsp; *localIpAddress* | string | 用于与MQTT代理通信的预先分配的IP地址。请注意，此IP地址在操作期间不应修改/更改。 |
|&emsp;&emsp; *netmask* | string | 与本地IP地址对应的网络配置中使用的子网掩码。|
|&emsp;&emsp; *defaultGateway* | string | 车辆使用的默认网关，与本地IP地址对应。 |
| &emsp;} | | |

# 7 Best practice
# 7 最佳实践

本节包括有助于促进与协议逻辑一致的共同理解的附加信息。

## 7.1 Error reference
## 7.1 错误引用

如果由于错误的命令导致错误，AGV应在状态主题的errorReferences字段中返回有意义的错误引用（参见[6.10.6 状态消息的实现](#6106-implementation-of-the-state-message)）。这可以包括以下信息：

- `headerId`
- 主题（`order`或`instantAction`）
- 如果错误是由命令更新引起的，则为`orderId`和`orderUpdateId`
- 如果错误是由动作引起的，则为`actionId`
- 如果错误是由错误的动作参数引起的，则为参数列表

如果由于外部因素（例如，预期位置没有负载）导致动作无法完成，则应引用actionId。

## 7.2 Format of parameters
## 7.2 参数格式

错误、信息和动作的参数设计为具有键值对的JSON对象数组。

| **字段** | **数据类型** | **描述** |
|---|---|---|
**actionParameter** { | JSON对象 | 指定动作的actionParameter，例如，deviceId、loadId、外部触发器。
key | string | 参数的键。
value</br>} | 可能的值：</br>array,</br>boolean,</br>number,</br>string,</br>object | 属于键的参数值。

动作“someAction”的`actionParameter`示例，具有stationType和loadType的键值对：

"actionParameters":[
{"key":"stationType", "value": "floor"},
{"key":"weight", "value": 8.5},
{"key": "loadType", "value": "pallet_eu"}
]

使用建议的“key”: “actualKey”, “value”: “actualValue”方案的原因是为了保持实现的通用性。“actualValue”可以是任何可能的JSON数据类型，例如float、bool，甚至是对象。

# 8 Glossary
# 8 术语表

## 8.1 Definition
## 8.1 定义

概念 | 描述
---|---
自由导航AGV | 使用地图规划自己路径的车辆。<br>主控系统仅发送起点和终点坐标。<br>车辆将其路径发送到主控系统。<br>当与主控系统的连接中断时，车辆能够继续其行程。<br>自由导航车辆可能被允许绕过局部障碍物。<br>车辆本身也可能进行接收/分配位置的微调。
引导车辆（物理或虚拟） | 由主控系统发送路径的车辆。<br>路径的计算在主控系统中进行。<br>当与主控系统的通信中断时，车辆终止其释放的节点和边缘（“基础”）然后停止。<br>引导车辆可能被允许绕过局部障碍物。<br>车辆本身也可能进行接收/分配位置的微调。
中央地图 | 将在主控系统中集中保存的地图。<br>这最初是创建的，然后被使用。
