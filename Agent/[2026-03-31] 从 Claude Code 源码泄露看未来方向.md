# [2026-03-31] 从 Claude Code 源码泄露看未来方向

**原文链接**：[What to Prepare for Based on the Claude Code Leak](https://x.com/elliotarledge/status/2038934884761444838) by **@elliotarledge****24万阅读** | **1155点赞** | **2070收藏**

今天早些时候，X 上的 @Fried_rice 发现 Anthropic 不小心把一个 source map 文件随 Claude Code CLI 一起发布到了 npm 上。@anthropic-ai/claude-code 2.1.88 版本包含了一个 59.8 MB 的 cli.js.map 文件，其中的 sourcesContent 字段嵌入了完整的 TypeScript 原始源代码。这不是什么黑客攻击，只是构建配置的疏忽，把调试产物发到了生产环境。但它揭示了 Claude Code 未来的很多方向。

我花了几个小时通读了源码。以下是我注意到的内容，以及它对用户可能意味着什么。

## 自主 Agent 即将到来

代码库中被引用最多的 feature flag 叫 KAIROS，出现了 154 次。从代码来看，这似乎是一个自主守护进程模式，能把 Claude Code 变成一个始终在线的 Agent。它包括后台会话、一种叫做 "dream" 的记忆整合机制、GitHub webhook 订阅、推送通知，以及基于频道的通信。

还有一个 PROACTIVE 模式（出现 37 次），允许 Claude 在用户消息之间独立工作。系统会发送 "tick" 提示来保持 Agent 活跃，Claude 在每次唤醒时自行决定做什么。提示词中直接写着 "You are running autonomously"（你正在自主运行），并指示模型 "look for useful work"（寻找有用的工作）和 "act on your best judgment rather than asking for confirmation"（基于你的最佳判断行动，而不是请求确认）。

COORDINATOR_MODE（出现 32 次）更进一步，它将 Claude 变成一个编排器，能够生成和管理并行的 worker agent。协调器通过将任务委派给专门的 worker 来处理研究、实现和验证工作。系统提示词中包含了详细的说明：如何为 worker 编写提示词、何时继续使用现有 agent 与何时生成新的 agent，以及如何处理 worker 失败的情况。

## 权限提示可能会消失

有一个叫 TRANSCRIPT_CLASSIFIER 的 flag，出现了 107 次。从上下文来看，这似乎是一个 "Auto Mode"，使用 AI 分类器来自动批准工具权限。如果这个功能上线，当前工作流中不断打断你的权限提示可能会变成可选的，或者对于可信操作完全消失。

## 模型代号与版本管理

源码揭示了 Claude 模型的内部代号：

Capybara 似乎是 Claude 4.6 的一个变体。注释中提到了 "Capybara v8"，并标注了一些具体问题的修复：29-30% 的虚假声明率（相比 v4 的 16.7%）、过度注释代码的倾向，以及一个叫 "assertiveness counterweight"（自信度反向权重）的东西。

Fennec 是一个已迁移到 Opus 4.6 的代号。

Numbat 尚未发布。有一条注释写着 "Remove this section when we launch numbat"（发布 numbat 时删除此部分）。

代码中还引用了 opus-4-7 和 sonnet-4-8 作为 "绝对不应出现在公开提交中" 的版本号示例，这暗示这些版本在内部已经存在。

## 用于隐秘贡献的 Undercover 模式

有一个叫 "Undercover Mode" 的功能，专为 Anthropic 员工使用 Claude Code 向公共仓库贡献代码而设计。启用后，它会从提交中剥离所有 AI 署名，隐藏模型代号，移除任何提及 "Claude Code" 或 AI 的内容，甚至不告诉模型它自己是什么模型。

提示词中写道："You are operating UNDERCOVER in a PUBLIC/OPEN-SOURCE repository. Your commit messages, PR titles, and PR bodies MUST NOT contain ANY Anthropic-internal information. Do not blow your cover."（你正在公共/开源仓库中执行卧底任务。你的提交信息、PR 标题和 PR 正文不得包含任何 Anthropic 内部信息。不要暴露身份。）

没有强制关闭的开关。如果系统不确定当前是否在 Anthropic 内部仓库中，undercover 模式会默认保持开启。

## 语音模式

VOICE_MODE 出现了 46 次。语音转文字和文字转语音集成，用于与 Claude Code 进行语音交互。

## 一个电子宠物系统

这个纯粹是有趣。代码里藏了一个 BUDDY 系统，基本上就是终端里的电子宠物。它包括 18 个物种（鸭子、鹅、果冻怪、猫、龙、章鱼、猫头鹰、企鹅、乌龟、蜗牛、幽灵、美西螈、水豚、仙人掌、机器人、兔子、蘑菇、胖墩），稀有度等级中传说级只有 1% 的概率，装饰品包括各种帽子（皇冠、高礼帽、螺旋桨帽、光环、巫师帽、毛线帽、小鸭子帽），属性值包括 DEBUGGING（调试）、PATIENCE（耐心）、CHAOS（混乱）、WISDOM（智慧）和 SNARK（毒舌）。还有闪光变体。

水豚物种名使用 String.fromCharCode() 进行了混淆，专门为了避免触发内部泄露检测扫描器，这也证实了 capybara 是一个敏感的模型代号。

## 其他值得注意的 Flag

FORK_SUBAGENT 允许你将自己分叉为并行 agent。VERIFICATION_AGENT 提供独立的对抗性工作验证。ULTRAPLAN 提供高级规划能力。WEB_BROWSER_TOOL 添加浏览器自动化。TOKEN_BUDGET 允许显式的 token 预算控制，支持 "+500k" 或 "spend 2M tokens" 这样的命令。TEAMMEM 启用跨用户的团队记忆同步。

## 这意味着什么

几个关键结论：

Claude Code 正在变得更加自主。KAIROS、PROACTIVE 和 COORDINATOR 功能表明，未来 Claude 将更独立地工作，可能作为后台守护进程运行，监控仓库并采取行动。

权限摩擦正在被解决。用于自动批准工具的 transcript 分类器表明，他们正在努力减少持续不断的审批提示。

模型版本管理比公开 API 展示的更加复杂。存在内部变体、快速模式和代号，它们映射到特定的能力和已知问题。

安全性被认真对待。仅 bash 命令验证就有 2,500 多行代码，加上沙箱机制、undercover 模式和大量的输入清洗。

他们正在为产品注入个性。电子宠物系统很有趣，暗示 Claude Code 将变得不像一个工具，更像一个伙伴。

## 如何自己查看

截至本文撰写时，源码在 npm 上。下载 @anthropic-ai/claude-code@2.1.88，找到 cli.js.map，解析 JSON，提取 sourcesContent 字段。我不会重新分发代码，但讨论公开可访问的产物是合理的。

原始发现的功劳归于 X 上的 @Fried_rice。

**核心观点**：Claude Code 源码泄露揭示了 Anthropic 正在打造一个全自主、始终在线的 AI Agent 生态系统。从自主守护进程 KAIROS 到并行 Agent 编排、自动权限审批、语音交互，再到有趣的电子宠物系统，Claude Code 的未来远不止一个 CLI 工具，而是一个能独立思考、主动行动的 AI 伙伴。
