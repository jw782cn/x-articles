# [2026-03-19] 装好了 Paperclip，然后呢

**原文链接**：[Installed Paperclip, Now What !?](https://x.com/NickSpisak_/status/2034635430700679445) by **@NickSpisak_****26万阅读** | **823点赞** | **2988收藏**

我那篇关于 Paperclip 的文章获得了 270 万次浏览。

评论区基本上都是"酷炫演示"、"跑不起来"，以及各种"怎么处理某某问题"的提问……这篇后续文章就是为了给你一些启发，通过使用一些非常强大的 skills 来打开你的思路。现在一切都是 skills……

如果你还不确定 Paperclip 是什么、为什么这个概念如此有趣，可以先看看我之前那篇原文👇

今天我们要做的是叠加 skills……你需要准备：Paperclip（公司本体）、gstack（工程团队）、以及 autoresearch（研发实验室）。全部免费，全部开源，总共不到 10 分钟就能搭好。

先简单说一下……

好了，我们开始吧……记住，你唯一的限制就是你自己的想象力……这只是众多玩法中的一种……

## 三个工具（以及各自的作用）

在我们开始安装之前，先来看看全局。

Paperclip 就是你的公司。它是一个仪表盘，你可以在上面定义公司使命、雇佣 AI agent、设定预算、追踪每个人在做什么。把它想象成办公大楼。

gstack 是你的工程团队。它由 Garry Tan（Y Combinator CEO）打造，为你的 agent 提供 15 个专业角色——CEO、CTO、设计师、QA 工程师、发布经理。每个角色都清楚自己该做什么。把它想象成员工们。

autoresearch 是你的研发实验室。它由 Andrej Karpathy（前 Tesla AI 负责人）打造，让你的 agent 能在一夜之间自主运行实验。给它一个研究问题，然后去睡觉，醒来就有 100 个实验已经跑完了。把它想象成实验室。

三者协同：Paperclip 运营公司，gstack 构建产品，autoresearch 做研究。一个人，零员工。

## 第一步：安装 Paperclip（2 分钟）

打开终端运行：

这会安装所有依赖，创建数据库，然后在 http://localhost:3100 打开你的仪表盘。

仪表盘加载完成后：

搞定。你现在有了一家拥有组织架构、预算和工单系统的 AI 公司。

## 第二步：安装 gstack——你的工程团队（2 分钟）

这一步让你的 agent 拥有真正构建东西的能力。运行：

git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstackcd ~/.claude/skills/gstack./setup

现在你的 agent 拥有了 15 个专业 skill。以下是最重要的几个：

产品规划：

→ /office-hours - 把你粗糙的想法变成一个真正的方案，通过提出聪明的问题并给出三种实现路径

→ /plan-ceo-review - 像 CEO 一样挑战你的范围。从你的功能清单中找到隐藏的十分产品

→ /plan-eng-review - 锁定架构，包含架构图、边界情况和故障模式

开发构建：

→ /review - 代码审查，能捕获你测试漏掉的 bug。显而易见的问题直接自动修复。

→ /qa - 打开真实浏览器，点击测试你的应用，找到 bug，修复它们，然后写好测试防止复发

发布上线：

→ /ship - 同步代码、运行测试、检查覆盖率、推送到 GitHub、创建 Pull Request。一条命令搞定。

→ /document-release - 更新所有文档（README、架构文档、贡献指南），使其与你实际发布的内容保持一致

安全防护：

→ /careful - 在执行任何破坏性操作之前发出警告（删除文件、清空数据库、强制推送代码）

→ /freeze - 锁定除当前工作目录以外的所有文件。在调试期间防止误操作。

真正强大的玩法：同时运行 10-15 个这样的任务。一个 agent 在用 /office-hours 规划新功能，另一个在用 /qa 测试预发布环境，第三个在用 /ship 部署已完成的 PR。所有这些同时进行。

## 第三步：安装 autoresearch——你的研发实验室（2 分钟）

这个工具用于你的 AI 公司需要实验和学习的场景。运行：

启动 Claude Code，让它基于 https://github.com/karpathy/autoresearch.git 创建一个名为 "autoresearch" 的 skill。Skill builder 会为你生成一个 Karpathy 的 ML 训练实验版本。

autoresearch 最初是为 ML 训练实验设计的，但这种模式适用于你的 AI 公司需要的任何迭代研究——测试 prompt、优化工作流、对比不同方案。

它的工作原理：

→ 你给它一个研究问题或实验目标

→ 它修改代码，运行一个 5 分钟的训练实验，检查结果

→ 如果结果有提升，就保留这个改动。如果没有，就扔掉。

→ 然后再来一次。再来一次。全自动。

→ 每小时 12 个实验。一晚上大约 100 个实验。

你去睡觉，醒来就看到已完成的实验，附带清晰的结果报告，告诉你什么有效、什么无效。

## 三个工具如何协同工作

精彩的部分来了。每个工具负责 AI 公司的不同层面：

→ Paperclip 分配任务——"Agent 3，研究我们推荐引擎的最佳方案"→ autoresearch 执行研究——一夜跑完 100 个实验，找到最优方案→ gstack 负责构建——/office-hours 规划功能，/review 检查代码，/qa 做测试，/ship 部署上线→ Paperclip 追踪结果——花了多少预算、完成了哪些任务、做了哪些决策，全部记录在案

工作流程是这样的：

你用手机查看仪表盘，审核已发布的内容，批准下一个 sprint。这就是你作为董事会成员的工作。

## 5 个立即上手的 Prompt

一切安装完成后，试试这些：

Prompt 1 - 规划你的产品："使用 /office-hours 重新构思这个想法：[你的产品创意]。我要三种实现方案和工作量估算。"

Prompt 2 - CEO 级别审核："使用 /plan-ceo-review 挑战这个方案的范围。找到最简版本，用最少的投入实现 80% 的价值。"

Prompt 3 - 开发并发布："使用 /review 检查这段代码的 bug，然后用 /qa 在真实浏览器中测试用户流程，再用 /ship 部署。"

Prompt 4 - 跑一夜的研究："查看 program.md 并启动一个新实验。自主运行，直到完成 50 次迭代。记录所有结果。"

Prompt 5 - 完整 sprint："这是我们的 sprint 目标：[目标]。用 /office-hours 规划，用 /plan-eng-review 锁定架构，构建它，用 /qa 测试，用 /ship 发布。完成后我来审核 PR。"

## 进阶技巧

技巧 1：把 Paperclip 的 heartbeat 设置为与你的工作节奏一致。如果你每天早上 9 点审核工作，就把 agent 设置为 8 点前完成任务。

技巧 2：只要 agent 在处理生产环境代码，就使用 gstack 的 /careful 模式。它会阻止破坏性命令，除非你明确批准。

技巧 3：让 autoresearch 在夜间跑实验。每个实验 5 分钟 × 每小时 12 个 × 8 小时 = 你睡觉时跑完 96 个实验。

技巧 4：使用 Paperclip 的多公司功能来区分不同项目。你的 SaaS 产品和咨询业务应该是不同的公司，有各自独立的预算。

技巧 5：从一个 agent 和一个目标开始。不要第一天就雇 10 个 agent。先雇一个 CEO，让它建议第一个招聘人选，根据公司的实际需要来扩展组织。

三个免费工具。一家 AI 公司。零员工。

Paperclip 运营公司。gstack 构建产品。autoresearch 做研发。你坐在董事会里做决策。

所有仓库已上线。全部开源。整个搭建不到 10 分钟。

你现在唯一的限制，就是你自己的想象力。

PS：如果你喜欢这个入门套装，后面还有更精彩的。

**核心观点**：用 Paperclip + gstack + autoresearch 三个开源工具搭建AI公司：Paperclip管运营、gstack做开发、autoresearch跑实验。一个人、零员工、10分钟搭建完成。
