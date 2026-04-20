# [2026-04-08] 发布 Claude Managed Agents

原文链接：Launching Claude Managed Agents by @RLanceMartin30.2万阅读 | 1211点赞 | 2556收藏

摘要 —— Claude Managed Agents 是一个预构建、可配置的 agent 框架，运行在托管基础设施上。你只需将 agent 定义为一个模板——包括工具、技能、文件/代码仓库等。agent 框架和基础设施都由平台为你提供。这个系统的设计目标是跟上 Claude 快速增长的智能水平，并支持长时间跨度的任务。一些有用的链接：

## 为什么要做 Claude Managed Agents

Claude messages API 是通往模型的直接通道：它接收消息并返回内容块。基于 messages API 构建的 agent 使用一个框架（harness）来将 Claude 的工具调用路由到处理器，并管理上下文。这带来了几个挑战：

解决这些挑战很重要，因为我们预期未来的 Claude 将在人类最重大的挑战上运行数天、数周甚至数月。Claude Agent SDK 是第一步，提供了出色的通用 agent 框架。Claude Managed Agents 是这一进程的下一步：一个包含框架和托管基础设施的系统，旨在支持安全、可靠地在我们预期 Claude 工作的时间跨度内执行任务。

## 如何开始

一个简单的入门方式是使用我们的开源 claude-api skill，它在 Claude Code 中开箱即用。获取最新版本的 Claude Code，然后运行以下子命令来开始 Claude Managed Agents 的入门流程。我很看好 skill 作为新功能入门方式的潜力，自己也大量使用了这个 skill。

也可以参阅我们的文档，了解如何通过 SDK 或 CLI 快速开始，以及在 Claude Console 中原型化 agent。

## 使用场景

你可以在 Claude 博客中看到大量有趣的案例。以下是我在这些案例和自己工作中注意到的一些常见模式：

## 核心概念

入门时需要理解三个核心概念：

可以这样理解：agent 是一个配置，environment 是描述你希望 agent 访问的代码执行沙箱的模板，session 是任何一次 agent 执行。一个 agent 可以有多个 session。

## 使用方式

参见文档：

## 它是如何工作的

我和 @mc_anthropic、@gcemaj、@jkeatn 一起写了一篇 Anthropic 工程博客，讲述构建 Claude Managed Agents 的过程：我们在文中分享的一个经验是，构建能够随 Claude 智能水平扩展的 agent 是一个基础设施挑战，而不仅仅是框架设计的问题。

基于这个认识，我们没有设计一个特定的 agent 框架；我们预期 agent 框架会持续演进。相反，我们将"大脑"（Claude 及其框架）与"双手"（执行操作的沙箱和工具）以及"会话"（session 事件日志）解耦。

每一部分都变成了一个对其他部分做很少假设的接口，每个部分都可以独立地失败或被替换。我们分享了这种设计如何赋予系统可靠性、安全性和灵活性，以便在未来添加新的框架、沙箱或承载 session 的基础设施。

## 结语

我对探索不同模式的多 agent 编排或长时间运行任务的项目感到兴奋。我过去写过的一个痛点是让 agent 框架跟上模型能力的发展。Claude Managed Agents 为你处理了 agent 框架和基础设施，让你可以在 agent 这个 Claude API 中的新核心原语之上进行探索。

核心观点：Claude Managed Agents 将 agent 框架和基础设施托管化，解决了框架跟不上模型能力进化、以及长时间任务需要可靠基础设施这两大痛点，让开发者专注于 agent 应用本身而非底层架构。