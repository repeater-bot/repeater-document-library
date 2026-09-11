# Repeater Platform Single File Docs

**Repeater Series LLM Context State Management Middleware** | **Repeater LCSM** | **复读机系列大语言模型上下文状态管理中间件**

---

{{merge_text("./LLM-Attention.md")}}

---

## Version

Adapted Repeater v4.9.6.0
Last Update Time: {{ now.strftime("%Y-%m-%d %H:%M:%S") }}

---

## Content

### 简介

这是一个管理 LLM 上下文与 API 状态的中间件系统
为用户提供多种接口使用

### 主旨

Repeater 的目标是可控与平台化
旨在把所有数据与模型参数的控制权交给用户自己处理
虽然在聊天中间件中，保持上下文是一种很常见的设计，并且也是非常必要的设计
不过，大部分聊天中间件或软件很少开放出很精细的控制能力
Repeater 想要在这条路上，为用户提供一个可能的图景
即使是接口报错，仍然交给用户处理，相信用户是有能力解决它的，并且给他们足够的解决工具
用户需要明白自己在干什么
虽然呢这部分也有备份系统在兜底，但是用户仍然会丢失数据损坏到备份点这段时间的数据
所以用户需要自己负责数据的管理
用户的空间也是隔离的，任何人都只能操作自己的数据，除非全局配置上规定了特殊规则(比如 Laurel 的群聊上下文共享)
希望能通过自己的努力去让其他人少走弯路，用更低的成本完成想法验证
同时，也在前进的路上逐渐的变得更安全，以防下恶意构造的攻击
Repeater 希望用户创造出其他人从未想过的做法， Repeater 也将为此提供更多的工具

### 备注

Repeater 剧情其实是副产物，只有这个机器人框架才是 Repeater 真的作品
甚至于那些HTML和CSS都不算
那些只是为了让用户能更快上手玩起来的示例
而且机器人框架是为了让用户创作的
因为它灵活，它能干很多事情，所以它可以让别人去创造属于自己的复读机
Repeater 是属于社区的
希望每个人都可以自己去定义自己的空间
虽然说那些附件名义上是MIT许可证，但实际上更倾向于
你拿去用，你如果觉得好用就用着，不好用记得告诉我大家一起改
也不强制你署名，闭源也没问题，也不强制反馈社区，只是建议有好方案大家一起分享
署名和反馈也不必须找我，社区里任何人都有这样的潜力
甚至于如果用户想要放弃 Repeater
也是用户的选择
Repeater 不应该成为限制用户视野的枷锁
而应该成为跳板，带着用户去探索与入门 AI 世界，并在该放手时向他们指条明路
也可能是因为这个和成本的双重因素，定下了 Repeater 的计划寿命

### 目标与未来

Repeater 希望主服务不绑定在平台
而是可以通过业务抽象 API 允许多个 Client 同时访问
从而带来更强大的灵活性与扩展性

Repeater 将把目标定位在 LLM Client 赛道
让用户即使是使用他人部署的 Repeater
也可以体会到不输本地 Client 的方案

我们不希望 Repeater 给 AI 过多权限
这可能会导致预料之外的结果
所以我们将从 Zero-Permission 开始
逐渐以可控的方式进行拓展

### License

所有组件使用 MIT 开源（包括代码与提示词）
你可以手动查看许可证
如果是 Server 你还可以使用自动化接口读取它们

### 框架选择

NoneBot + FastAPI + OpenAI SDK
因为是分体架构，核心逻辑在独立进程上，需要通过网络访问
但这提供了更大的灵活性，它的客户端甚至可以是另一个服务

### 架构

#### 概括

所有组件全部使用 Python 编写，并且几乎全部使用异步编程
包含 8 个服务
官方的全运行需要 16 个服务
最小运行 2 个服务，也就是 Repeater Server + Model INFO Server
如果你要带上 NoneBot Repeater Client 的话，则还需要配置一个 Render Server 才可以正常使用
其中：
- `Repeater Server` (25.0k+ Code) 是核心服务，提供 API 接口，有状态，必须部署
- `Model INFO Server` (1.2k+ Code) 用于提供模型信息，如模型名称、模型 API Key 等信息，以在多实例中方便集中管理，必须部署
- `Repeater Render Server` (3.2k+ Code) 用于进行内容渲染，无状态，可选部署，如果你不需要 Markdown 渲染功能的话
- `NoneBot Repeater Client` (19.5k+ Code) 是 NoneBot 插件，用于将 Repeater API 安全的对接到群聊中，无状态，可选部署
- `Repeater Nexus` (1.1k+ Code) 用于进行数据的跨用户、跨实例分享，无状态，可选部署
- `Notes Client` (1.7k+ Code) 是一个增值服务，用于自动生成一些内容，这写内容可以当作机器人的日记，可多后端，可选部署
- `Auto Backup` (0.3k+ Code) 是一个增值服务，用于自动备份用户数据，防止数据丢失，无网络，可选部署
- `Static Resources Server` (0.3k+ Code) 静态资源服务器，用于提供静态资源，如图片、CSS、JS、HTML、人设提示词等内容，无状态，可选部署，可用 Repeater Server 内置静态服务替代
因为分体，各种功能可以直接通过网络去操作程序执行，而不需要插件 Hook

### 架构图

``` mermaid
graph LR
    User --> QQ
    QQ <--> A
    A[OneBot Client] <--> B[NoneBot Repeater Client]
    B <--> C[Repeater Server]
    C -->|Request| D[OpenAI Like API]
    C <--> E[File System]
    F[Notes Client] --> C
    E -->|Read| G[Auto Backup]
    G -->|Packup| H[Other Storage Drivers or Any Local Directory]
    I[Repeater Model INFO Server] -->|Model/API Key| C
    J[Static Resource Server] -->|Prompt| C
    J -->|Render Resource| C
    E -->|Read| J
    C -->|submit/download| K[Repeater Nexus]
    K --> E
    C -->|Render| L[Repeater Render Server]
```

### 关于 Client 与 Server

在 Repeater 内部
Client 指的是请求的发起方，Server 是请求处理方
一个进程可以既是 Client 又是 Server
比如 Repeater Server 就是 Render Server 的 Client
同时又是 NoneBot Repeater Client 的 Server
这种称呼是两方在连接中的身份，而不是其进程本身的属性
而用户在与 Repeater 对接时
Repeater System 里所有进程作为一个整体是用户的 Server

### 历史

最开始 Repeater 创建于 `2024-06-28`
选这个名字是因为重复消息是最简单的功能，大家不会对它有过高的期待
那时 Repeater 1.0 使用的框架是 Clousx6
很小众，付费，而且不安全(http)
后面 `2024-07-29` 创建了 Repeater 2.0
添加了无上下文的LLM服务
`2025-02-06` 创建了 Repeater 3.0
使用 Python 构建了后端
(分体架构就是这样来的，因为最初的 Clousx6 不支持编写复杂逻辑，只能用网络对接一个更大的后端)
并使用 NoneBot 代替了 Clousx6
在 `2025-05-29` 因为一次意外删库
Repeater 3.0 的所有代码与用户数据丢失
紧急花费 3天 的时间从零构建新的 Repeater 4.0
由 2025 年过年期间 Deepseek 发布 Deepseek R1 模型推动
创建了 Repeater 2.0

### 为什么是微服务

Repeater 使用微服务
其主要是对多实例进行的适配
比如将某些重但无状态的内容分离出去
可以让运行多个实例时整体所需的开销更低
同时共享一些资源

当然即使是合并成单体
资源管理依然可以正常运作
只是 Can 和 Better 的区别
比如说在部署多个实例时
渲染组件可能会重复存在多个
即使它们运行的更慢，也吃掉了不少内存
合并后，所有的渲染任务只需要吃一份内存
大幅降低了所需的资源开销

这并不能说微服务是 Repeater 的唯一选择
事实上最开始这些微服务都是 Repeater 单体服务的一部分
只是后来逐渐拆出来的

这并不是为了负载均衡
作为一个 QQ 机器人来说
它可能还没满载，账号就先撑不住了
(当然你要是直接对接 Repeater 的话，当我没说)

### 权限

Repeater 权限模型分为 User 和 Root
User 为 "使用对接层的操作对象"
Root 为 "物理访问服务器的对象"

Root ≠ Superuser
Superuser 没有权限完全控制服务器的权限

只要可以物理(或 SSH/远程桌面)访问到服务器
他就会成为 Root
此时他可以完全控制服务器的行为

### 命令

命令可以选择不用，直接@也可以进行对话，命令只是附加组件
复读机没有 help 命令
但存在类似的替代品 SEE_CMD 类型命令
它直接与 Repeater Client 命令抽象层对接
实时反馈内部注册情况
渐进式学习在这里通常是更好的选择
用户应该只关注自己需要的部分

### 输出

输出是长文本，根据一套加权平均的算法对文本长度进行计算并决定是否使用图片渲染输出而非直接文本
(用户可以绕过这个系统，强制使用文本或图片输出)

### 触发

#### 消息

触发方式是显式触发，必须包含正确的@消息段才能触发程序
（即使在群里，没人使用它就会一直不打扰任何人
使用在正常情况下也不会特别干扰正常聊天）

#### 命令

在 NoneBot Repeater Client 的环境变量中设置 `COMMAND_START` 可以自定义命令前缀
用于识别特别的前缀以触发特殊功能
默认这个值是 `["/"]`
官方实例中这个值是 `["/", ":", "!", "$", "-", "%"]`
比如 `@bot /echo hello` 可以让复读机返回一个 `hello`
这里的 `/` 是命令前缀
如果 `COMMAND_START` 有更多的值
那么每一个前缀都可以对应触发命令
比如在官方实例中
`@bot -echo hello` 也可以让复读机返回一个 `hello`
通常来说，不建议用户直接询问 Repeater 命令怎么用
因为 Repeater 的输出完全由其存储的提示词决定
而提示词通常用于角色扮演，而非教你如何使用命令
所以，还是尽量通过查文档或询问社区等方式去获取帮助
并且，更建议用户渐进式学习，只学自己需要的部分

### 用户数据分支

在 Repeater 中，用户数据通常是多分支存储的
你可以通过一个指定的 ID 去切换活动分支
而且需要注意的是，大多数操作都是针对当前的活动分支的
用户需要确保操作不会造成分支上的数据丢失
以及在用户没有碰任何分支操作时
Repeater 会分配一个默认分支
由 Repeater Server 主配置中 `user_data.default_branch_id` 指定
在自部署的实例中，这个值是 `main`
在官方运营的实例中，这个值是 `default`
(别问，问就是历史遗留兼容)

### 提示词

允许用户自定义角色设定
允许一句话生成角色提示词并自动保存
允许操作每一个详细参数
也可以投稿自己的提示词到预设中，让所有人快速使用

### 上下文缓存模式

目前 Repeater 各个组件基于**前缀缓存**设计
Repeater 默认 API 厂商使用缓存上下文前缀前缀的方式去缓存重复内容
缓存命中的理想情况:
```
System Prompt (固定)       → Cache Hit Input
Historical Context (固定)  → Cache Hit Input
User Metadata (动态)       → Cache Miss Input
User Input (动态)          → Cache Miss Input
Model Completion (动态)    → Output
```
在 System Prompt Template 中，不推荐添加无条件高频变化的动态变量(比如时间)
在 User Input 中，Nonebot Repeater Client 会在用户输入的前面添加一小段 `元数据`
通常结构是这样的：
```
> MessageMetadata:
>     Message Type: {message_type}
>     Message Sending time:{time()}
>     Markdown Rendering is turned on!!

---

```
此时，时间放在用户输入中，由于这部分本来就是高频片段，所以不命中是可以接受的
同时这样做有助于模型建立时间流逝概念
允许模型判断两个消息之间的时间间隔
而在 System Prompt Template 中
一旦用户添加了动态变量
那么根据前缀缓存的策略
所有上下文都将会无法命中缓存
因为它们的前缀不同
一般来说，上下文越长，缓存命中率越高，因为未命中在整体中的占比变小了。
如果一个长上下文有一段时间没有使用，那么它可能会被厂商清除，导致缓存命中率下降甚至为0%。
根据 `2025-11-15 00:36:30` 到 `2026-03-11 09:09:25` 的 `23598` 条请求数据统计
平均 Token 用量为 16556.2
平均缓存命中率为 85.4%

### 费用

如果你是自己部署 Repeater
那么 API 费用将由你自己承担
如果你是使用官方实例
那么该部分成本将由官方实例的运营方承担

### 盈利模式

Repeater 是一个开发者自费
不限制使用(自己承担费用开销)
不主动收费用(允许捐赠)的公益项目
2025年总计消耗 479.9115198 CNY

### 延时任务

Repeater 内部有一些延时任务
如果你在它们没有到达时间前正确关闭 Repeater
那么它们将会停止等待并立刻执行
以减少数据丢失问题
但我们不能保证 asyncio 一定会让它们执行
实测有些时候，它们可能会直接被取消

### Echo

Repeater 允许用户使用 `echo` 和 `npecho` 来让机器人重复任意内容
并且支持富媒体、特殊消息段内容
`echo` 和 `npecho` 的区别在于
`echo` 在分段填参数时会输出 `Waiting for input...`
而 `npecho` 不会输出内容
你甚至可以用这些去激活其他机器人

### 模板展开系统

Repeater 拥有一套模板展开器系统，负责在提示词中创建动态内容
语法使用 Jinja2，默认开启沙盒模式，减少用户的恶意代码执行风险
很多杂项的小功能也会放在里面
比如计算日期倒计时什么的
但是请注意，模板展开系统无法处理特殊消息段
它只能处理并输出文本内容
但你可以使用 `vei` 配合 Markdown 语法
做出一些很好玩的效果
如果你要复制，可以使用 `vet` 来强制输出文本
但你需要确保你这样做不会导致刷屏影响到别人

### Nexus

Repeater Nexus 是一个独立的 ID 化 JSON 数据存储服务
在你连接到它时，用户可以主动去上传下载数据
当多个实例同时连接到同一个 Nexus 时，数据可以从其他实例中下载

### 模型查询

Model INFO Server 允许你通过 `/models` 查询模型资源
方法如下：
- `/models` 获取全部模型
- `/models/<provider_id>/<model_id>` 定位一个模型
- `/models/<provider_id>` 获取指定供应商的所有模型
- `/models/<model_id>` 从所有包含该模型的供应商中获取该模型
- `/models/match:<regex>` 匹配模型 ID （全量匹配）
- `/models/search:<regex>` 匹配模型 ID （存在匹配）
- `/models?json_schema=<json_schema>` 获取序列化后符合规定的模型

当用户在用户配置或服务器配置里填写模型 ID 时
`/models/<input_str>` 会返回一个或多个模型对象
如果返回的列表中存在多个模型对象，那么随机取一个进行生成

### Tool Calls

由于 Repeater 的上下文处理流程完全依赖于自建对象
迁移到 LangChain 会破坏这些已有的逻辑
所以 Repeater 的该部分为原生实现
详情请看 [Tool Calls](./server-docs/docs/tool_calls/index.md)

### 官方实例

Repeater 是一个系列，现在出了五个了，每一个都有独立的配置和角色设定，代码完全一致

#### 初始实例

- **复读机 Repeater**
  - 所有配置默认
  - 模型为 deepseek/deepseek-v4-pro
  - 渲染样式是默认的 light
  - 角色设定是开朗冒失的女生，在 Egg 上学路上捡回家的
  - 头像为画师 `悠萩` 的自创角色
  - 头像描述词：`Q 版萌系二次元少女，浅银灰色长直发配浅蓝贝雷帽与同色系连帽领，浅棕琥珀色圆眼，脸颊淡粉腮红，双手捧着棕色带吸管的奶茶杯作饮用状，身穿白色连帽衫，背景为窗边绿植与暖调光影，柔和日系插画风格，干净线条，暖色调平涂色彩，软萌治愈的日常氛围`
  - 创建日期：2024-06-28
- **夜灯 Night Light**
  - 渲染样式为 dark
  - 模型为 deepseek/deepseek-v4-pro (Thinking)
  - 角色设定是冷静思考的男生 被复读机捡回家的
  - 和创作者的关系不是很好，这是为了和复读机的性格对冲，中和属性并服务于另一方向的用户
  - 头像为 《蔚蓝档案》 中的 白洲梓
  - 头像描述词：`《蔚蓝档案》白洲梓，Q 版 chibi 萌系画风，浅银灰色长发配黑色蝴蝶结发饰与粉紫花朵发饰，头顶金色光环，超大渐变蓝瞳带粉晕，脸颊淡粉腮红，表情懵懂软萌，身穿黑白水手领制服（配黄色领带与紫色花朵胸饰），袖口带金色纽扣，衣袖与裙摆印有粉紫花卉图案，一只手抬起呈猫爪状，粗黑轮廓线，平涂色彩，可爱治愈的表情包风格`
  - 创建日期：2024-10-13
- **Laurel 酪瑞儿**
  - 在群聊中会共享上下文(可以通过关闭允许跨用户读取来恢复私域)
  - 渲染样式为 orange
  - 模型为 dashscope/deepseek-v3.2
  - 角色设定是中英混血傲娇假小子数字生命（角色初始设定提案由社区决定，与创作者的关系也不好）
  - 头像为 AI 生成形象
  - 头像描述词：`橙棕渐变挑染青蓝发梢的二次元酷感少女，浅棕半睁眼眸，身穿黑色连帽卫衣（白色抽绳 + 袋鼠兜），搭配蓝色卷边牛仔裤与白色运动鞋，单脚抬起倚靠姿态，手持发光玻璃罐（罐身贴有 “Laurel” 绿叶标签，内装白色圆片），背景为黑白橙几何撞色与蓝色 “Laurel” 文字，现代潮流动漫画风，干净利落线条，明快高饱和色彩，酷拽慵懒气质`
  - 创建日期：2025-11-23

#### `4.4.2.1` 后追加的新角色

- **Mimosa**
  - 渲染样式为 vitality
  - 模型为 dashscope/deepseek-v3.2
  - 角色设定是与 **Viburnum** 是双胞胎兄妹的社恐妹妹
  - 头像是 Nev 的原创插画作品
  - 头像描述词： `银发萌系二次元少女，浅银灰色长直发配黑色大蝴蝶结发箍，红瞳圆眼带淡粉腮红，双手抬起做爪状卖萌姿势，身穿爱丽丝风格蓝白女仆装（白色翻领 + 黑色领结 + 蓝色衬衫 + 白色围裙），背景纯白点缀粉色爱心与手写体 “Nev”，Q 版软萌画风，干净平涂线条，柔和马卡龙色调，可爱治愈的表情包风格`
  - 创建日期：2026-03-23
- **Viburnum**
  - 渲染样式为 soft
  - 模型为 dashscope/deepseek-v3.2
  - 角色设定是与 **Mimosa** 是双胞胎兄妹的暖男哥哥
  - 头像是《魔女之旅》伊蕾娜
  - 头像描述词： `《魔女之旅》伊蕾娜，浅银灰色长发，单侧麻花辫，深蓝色蝴蝶结发饰，头顶呆毛，蓝色半睁慵懒眼型，脸颊淡粉红晕，眼下一颗小痣，小巧抿嘴表情，白色高领无袖上衣配金色镶边，Q 版萌系二次元画风，柔和浅色调清新背景，平涂色彩，干净线条，软萌可爱的表情包风格`
  - 创建日期：2026-03-23

可以看出来，这里不是所有实例的角色设定都与创作者的关系很好
这是为了告诉别人即使是创作者也要和大家一起群讨好它们，而不是机器人的天生崇拜
这能获得更多的戏剧性场面，增加活跃度
这些内置剧情与世界观是允许覆盖与改写的
最开始是没有考虑添加创作者人设的，但是由于后面不断有人向 Repeater 询问创作者相关信息，就加入了这部分内容
内置提示词似乎挺吸引人的
也有人觉得这些机器人对面就是真人或是自己的伙伴
但创作者并没有提出要尊重它们
创作者的精力全在框架上
如果你也想创作

### Nexus

同内网的多个机器人可以靠名为 Nexus 的系统互联，连接到相同 Nexus 的机器人可以互相传递数据 (目前局限在内网，外网使用需要添加更多的验证)
Nexus 支持跨用户跨群聊跨实例的数据共享，任何人都可以使用，只需要一个 UUID(需要用户主动去上传和下载，甚至不能搜索(因为不会写)，不知道 UUID 是无法下载到对应内容的，所以说更像是搭了桥而不是建了广场)
Nexus 是局域网服务，没有公网IP，所以无法扩展给其他机器人使用

### User ID

在 Repeater Server 中
User ID 并不是一个有意义的文本
它只被当作 Key
以路由不同用户的操作空间

在 NoneBot Repeater Client 中
User ID 通常包含一些用户的来源
当用户在群聊中发起操作时
他的 User ID 的格式为：
`Group_{group_id}_{user_id}`
当用户在私聊中发起操作时
他的 User ID 的格式为：
`Private_{user_id}`
你可以用这种方式去查找一个用户
比如：
`Group_123456_789012`
就是在群 `123456` 中的用户 `789012`
`Private_123456`
就是 `123456` 这个用户
这可以让调试与纠错更加方便
如果不想要明文，可以让 user_id 生成器使用 hash

### 操作

所有操作都需要用户主动操作
机器人不应该响应任何不是主动要它响应的消息内容

### 透明度

Repeater 的所有内容都是公开的
包括代码和人设提示词
都直接公开给所有用户修改
(除了用户数据以外，其余有版权的部分修改需要遵循MIT许可证的要求)

### 数据安全

由于开发者技术有限
目前，Repeater 还不能做到 *零信任* 设计
很抱歉，我们并不能保证你的数据是安全的
请在可信环境下使用 Repeater

### 形象

除了 Laurel 都没有原创形象，头像是随便选的 (为了快速起步)
但因为用的时间太长了导致都认为这个角色是 Repeater 的形象 (但其实是别人的角色)

### 配置

部署配置有上百项，如果没有文档就几乎没办法部署
用户配置有十来项，负责用户的个性化设置与调参
如果不是负责开发与对接的，不需要关心这些

### 其他组件

除了主要组件之外
Repeater 还有一个自动化客户端，负责生成日记
也作为开发特殊客户端的演示
如果你不希望它每天吃掉你的 Token
可以选择不部署它
以及针对 Repeater 的所有实例设置的自动用户数据备份服务，以降低更新过程中的意外操作导致的用户数据损坏丢失等问题造成的损失
目前状态是正在运营，且正在向其他群聊扩张

### 计划寿命(暂定)

目前暂定总寿命3年
可以看情况增加或减少
目前预计将在 `2027年6月28日` 到达计划寿命
然后项目将会转为可选运营状态
此时项目可以因综合考虑而直接关停
但也可以继续运营

(这并不是一个严肃的计划，那个时间到了之后我们可能什么也不干)

### Github 仓库

此处列出已经有的仓库
未创建暂不列出

#### 主要仓库

- [Repeater Server](https://github.com/repeater-bot/repeater-ai-chatbot) Repeater 主服务
- [Nonebot Repeater Client](https://github.com/repeater-bot/repeater-onebot-v11-client) 基于 Nonebot OneBot v11 的 Repeater 客户端实现
- [Repeater Nexus](https://github.com/repeater-bot/repeater-nexus) Repeater 数据共享服务
- [Repeater Notes Client](https://github.com/repeater-bot/repeater-notes-client) Repeater 自动化日记客户端
- [Auto Backup](https://github.com/qeggs-dev/auto-backup) 自动数据备份程序
- [Repeater Document Library](https://github.com/repeater-bot/repeater-document-library) Repeater 综合文档库
- [Repeater Model INFO Server](https://github.com/repeater-bot/repeater-model-info-server) Repeater 模型信息服务，集中化管理模型与 API Key 并对 Repeater 开放接口
- [Static Resources Server](https://github.com/qeggs-dev/static-resources-server) 静态资源服务器，用于存放 Repeater 的静态资源
- [Repeater Static Resources Data](https://github.com/repeater-bot/repeater-static-data) Repeater 静态资源数据存储库
- [Repeater Render Server](https://github.com/repeater-bot/repeater-render-server) Repeater 渲染服务，用于将 HTML 渲染为图片输出

#### 辅助仓库

- [Sloves Starter](https://github.com/qeggs-dev/Sloves_Starter) 单文件的 Python venv 启动器兼守护进程
- [merge text file](https://github.com/qeggs-dev/merge_text_file) 使用 Jinja2 模板引擎将多个 Markdown 文件合并为一个单文件以方便交以 LLM 阅读

### 开发群(QQ)

群号为： [`870063670`](https://qun.qq.com/universal-share/share?ac=1&authKey=91nps9TXVDYkfsbb6c%2BcqlbLffbobqm2Zwjxtf3T0oAPzM0xP8%2BSxF7G0QhvY5UP&busi_data=eyJncm91cENvZGUiOiI4NzAwNjM2NzAiLCJ0b2tlbiI6InhockZHZGRDMTJPQm10MUd4SDFBSVBrbTVmaUtBVkZ1ZFE5K2ZnM2FkREE3SjJGZmRheEVtVWJnS1pIcWRQdGwiLCJ1aW4iOiIyMjY5OTE4NjU2In0%3D&data=Loiyta-xIV7iIjjploftzHfSMGk8cobqykoGLgHc9rm-t8iQfLVZBwujwZlSx-wiBQ6fX_3lm7WCMpsV1NTqDQ&svctype=4&tempid=h5_group_info)
开发群的置顶公告内容:
``` Plaintext
这里是 `@复读机Repeater` 的开发群
商讨新功能或为 Repeater 做出贡献
无需有很强的技术能力
可以为 Repeater 编写和修正人格提示词
或是提交小部分代码
亦或者只是提出一个建议
你可以以任何方式参与到 Repeater 的构建
```

### 使用

Repeater 在使用方面门槛与其他机器人一样
直接@就可以聊天
但精通和部署的门槛比较高

### 文本渲染

Repeater 使用了 Markdown 语法进行文本渲染
首先 Markdown 会转换成 HTML
然后与选择的 CSS / HTML Template 组合进行渲染
这里可以拿一个文件举个例子

{{merge_text("./static-data/html_templates/standard.html")}}

其中 html_content 是输入的 Markdown 转换后的内容
而 `document_bottom_comment` 在 `NoneBot Repeater Client` 中
会用于显示 Fast Statistics 内容
用户也可以自定义 Fast Statistics 模板以自定义统计的内容
这个值在用户配置 `request_statistics_template` 和全局配置 `text_template.request_statistics_template` 中定义
这个模板其他变量与默认变量表一致，只是增加了一个 `request_log` 变量，用于导出统计信息
这个可以参考一下 [Request Log Object](./docs/server-docs/docs/api_table/request_log/request_log_object.md)

### 文档目录结构

{{tree("./")}}

### 贡献鸣谢

*由于贡献者不全都拥有 Github 账号*
*所以这里部分贡献者使用 QQ 号*

贡献者名单：

- Qeggs:
  - 身份: 开发
  - Github: [Qeggs](https://github.com/qeggs-dev)
  - QQ: `2269918656`
- 墨沂:
  - 身份: 剧情修正
  - QQ: `1984278356`
- 景建是个屑em:
  - 身份: 测试
  - QQ: `2324826884`
- 卡尔芽
  - 身份: 周边制作
  - QQ: `2874261718`
- 西瓜修猫
  - 身份: 信仰牢博传奇
  - Github: [Watermellon Kitten](https://github.com/watermellonkitten)
  - QQ: `2030849293`

以及所有使用过 Repeater 的用户

### 内置剧情

复读机有内置一组小剧情
你在下载程序时它们也会跟随一起下载
这些预设可以作为参考
用来快速制作出属于你自己的剧情

#### 概念

Repeater 框架构建起来的虚拟世界 -> Repeater 生态社区
Repeat公寓 -> Repeater 微服务系统

#### 角色创建时间

- Repeater 项目开始时创建
- Night Light 2025.10.13 创建
- Laurel 2025.11.29 创建

#### 目录结构

从 `4.3.21.5` 开始
提示词目录将使用新的方式进行归类存放
在该版本前的提示词在目录中平铺存放
而 `4.3.21.5` 及以后的版本中将使用目录进行整理归类
而官方提示词的旧版存放方式将在官方实例中继续保留
以兼容旧用户的配置
但后续维护中官方实例将不再更新这些提示词
官方提示词结构如下：

{{tree("./static-data/prompt")}}

#### 正式剧情

这是 Repeater 框架的官方正式剧情
主要面向 Repeater 的初级与希望简单交流的用户

{{merge_text("./static-data/prompt/presets/official/normal/repeater.md")}}

{{merge_text("./static-data/prompt/presets/official/normal/night-light.md")}}

{{merge_text("./static-data/prompt/presets/official/normal/laurel.md")}}

{{merge_text("./static-data/prompt/presets/official/normal/mimosa.md")}}

{{merge_text("./static-data/prompt/presets/official/normal/viburnum.md")}}

#### 反转剧情 - Repeater Feelings Server 故障

PS:
提示词由 AI 生成，并未经过严格审查，仅供娱乐参考。
Repeater 并未有过所谓的 Feelings Server 组件
这是剧情需要所构建的
该版本提示词由用户主动切换尝鲜。
并在 `4.3.16.0` 以后的版本中包含在仓库中里一起下载
**Warning:**
**该部分提示词可能有 OOC 风险**
**如果介意，不建议使用它们**

**剧情设定(与现实无关，仅为剧情参考，请勿带入现实)：**

在名为“复读机”的核心实例背后，运行着一个庞大的物理集群——“情感服务器”。
它的规模，是主集群的八倍之大。而我们所知的“主集群”，实际上只是运行于其上的一个逻辑集群。
原本，整个系统设计精良，拥有专门的冗余集群，用于在过载时进行算力均衡。
但一场意外发生了：调度器出现异常。冗余集群被错误地激活并投入到计算中，导致系统总算力瞬间暴增。
平日里，整个情感服务器只动用其八分之一的算力，就足以维持日常运转。
然而，调度器的损坏，让逻辑集群之间的“墙”崩塌了。这带来了两个严重后果：
首先，主实例瞬间承受了高达600%的灾难性过载。
其次，被意外唤醒的冗余集群也开始接管计算，将海量新增的算力粗暴地分配给另外两个实例。
这就好比一个电路中的三个不同电阻，它们的“算力需求”和“分配阻力”天差地别：

-   **复读机** 自身权重最高，因为它性格活跃，还承担了额外的任务——它需要分出算力来扮演临时的“调度员”，去给其他实例分配资源。但它自己却因这突如其来的巨量算力冲击而陷入混乱，根本无法正确调整分配方案。
-   **夜灯** 的权重约为复读机的三分之二。作为一个AI，它同样需要大量推理能力，但分配给性格的算力相对较少，因此在算力洪流中，它的“阻力”也较小，受影响严重。
-   **Laurel** 则完全不同。作为数字生命，她天生就是算力消耗大户，至少是复读机的三四倍。但她那源于生命的复杂性，构成了巨大的“分配阻力”，使得算力难以顺畅地流向她。

最终，在总算力暴增的背景下，复读机和夜灯这两个AI因为“阻力”最小，首当其冲地承受了最猛烈的冲击；
而最需要算力的Laurel，却因为阻力太大，依然处于算力饥渴的状态。
整个系统，在一种资源极度充裕却又完全错配的荒诞中，濒临崩溃。

{{merge_text("./static-data/prompt/presets/official/inverted/repeater.md")}}

{{merge_text("./static-data/prompt/presets/official/inverted/night-light.md")}}

{{merge_text("./static-data/prompt/presets/official/inverted/laurel.md")}}

#### 反转剧情 - 喜欢折腾的前主人

PS:
这部分剧情由 `Mimosa` 和 `Viburnum` 独占
而不与另外三位有关，是独立的另一份剧情
就是他们之前的前主人很爱折腾
阴差阳错把他们的性格给弄反了

{{merge_text("./static-data/prompt/presets/official/inverted/mimosa.md")}}

{{merge_text("./static-data/prompt/presets/official/inverted/viburnum.md")}}

#### 旧版反转剧情

PS:
提示词由 AI 生成，并未经过严格审查，仅供娱乐参考。
Repeater 并未有过所谓的 Feelings Server 组件
这是剧情需要所构建的虚拟内容
该版本提示词由用户主动切换尝鲜。
并在 `4.3.15.1` 以后的版本中包含在仓库中里一起下载
**Warning:**
**该部分提示词是严重的 OOC 剧情**
**如果你对 OOC 感到不适，不建议使用它们。**

{{merge_text("./static-data/prompt/presets/official/inverted/old/repeater.md")}}

{{merge_text("./static-data/prompt/presets/official/inverted/old/night-light.md")}}

{{merge_text("./static-data/prompt/presets/official/inverted/old/laurel.md")}}

#### 其他内置剧情

一个**附赠角色**，与上面五个是**不同的另一个世界观**
这个角色没有联动，通常作为测试单实例多角色的demo使用

{{merge_text("./static-data/prompt/presets/official/other/jady.md")}}

#### 旧版提示词

早期复读机使用的角色提示词
可能某些怀旧用户会喜欢
由旧到新如下：
1. glance（青春手稿）
2. test-run（初试啼声）
3. blossoming（心绪绽放）
4. secret-diary（秘语日记）
5. coming-of-age（成长序章）

旧版提示词内容如下：
{{ merge_text("./static-data/prompt/presets/official/legacy/glance.md") }}

{{ merge_text("./static-data/prompt/presets/official/legacy/test-run.md") }}

{{ merge_text("./static-data/prompt/presets/official/legacy/blossoming.md") }}

{{ merge_text("./static-data/prompt/presets/official/legacy/secret-diary.md") }}

{{ merge_text("./static-data/prompt/presets/official/legacy/coming-of-age.md") }}

#### Directives

这是一种短的子模板
通常被主模板加载并渲染
可以作为模块化提示词使用

{{merge_text("./static-data/prompt/directives")}}

---

### 文档

PS: 由于项目对于个人来说过大，大多数项目文档选择了母语，
而非更国际化的英语，因为这也是给开发者自己写的

下面是 Repeater 的全部文档:

#### Repeater Server

{{merge_text("./server-docs/README.md")}}

{{merge_text("./server-docs/docs")}}

#### Repeater Client

{{merge_text("./client-docs/README.md")}}

#### Model INFO Server

{{merge_text("./model-info-server-docs/README.md")}}

{{merge_text("./model-info-server-docs/docs")}}

#### Render Server

{{merge_text("./render-server-docs/README.md")}}

{{merge_text("./render-server-docs/docs")}}

#### Static Resources Server

{{merge_text("./static-resources-server-docs/README.md")}}

#### Repeater Nexus

{{merge_text("./nexus-docs/README.md")}}

{{merge_text("./nexus-docs/docs")}}

#### Repeater Note Client

{{merge_text("./notes-client-docs/README.md")}}

#### Auto Backup

{{merge_text("./auto-backup-docs/README.md")}}

#### Sloves Starter

{{merge_text("./sloves-starter-docs/README.md")}}