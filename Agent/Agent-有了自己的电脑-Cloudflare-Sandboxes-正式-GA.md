# [2026-04-13] Agent 有了自己的电脑：Cloudflare Sandboxes 正式 GA

原文链接：Agents have their own computers with Sandboxes GA by Cloudflare推文：@whoiskatrin | 4.5万阅读 | 685点赞

当我们去年6月推出 Cloudflare Sandboxes 时，前提很简单：AI agent 需要开发和运行代码，而它们需要在安全的地方做这件事。

如果 agent 像开发者一样工作，这意味着克隆仓库、用多种语言构建代码、运行开发服务器等等。要有效地做这些事，它们通常需要一台完整的计算机（如果不需要，也可以选择更轻量的方案）。

许多开发者正在用 VM 或现有容器方案拼凑解决方案，但有很多难题要解决：

我们花时间解决了这些问题，这样你就不用自己操心了。自初始发布以来，我们让 Sandboxes 成为更好的大规模运行 agent 的平台。我们与 Figma 等初始合作伙伴合作，他们在容器中运行 agent（Figma Make）：

我们希望将 Sandboxes 带给更多优秀的组织，所以今天我们很高兴宣布：Sandboxes 和 Cloudflare Containers 都已正式 GA。

让我们来看看 Sandboxes 近期的一些变化：

## Sandboxes 101

在深入了解近期变化之前，让我们先快速看看基础知识。

Cloudflare Sandbox 是由 Cloudflare Containers 驱动的持久、隔离环境。你通过名称请求一个 sandbox。如果它正在运行，你就获取它。如果没有运行，它会启动。空闲时自动休眠，收到请求时唤醒。你可以使用 exec、gitClone、writeFile 等方法轻松地以编程方式与 sandbox 交互。

```
import { getSandbox } from "@cloudflare/sandbox";
export { Sandbox } from "@cloudflare/sandbox";

export default {
  async fetch(request: Request, env: Env) {
    // 按名称请求 sandbox，按需启动
    const sandbox = getSandbox(env.Sandbox, "agent-session-47");

    // 克隆一个仓库
    await sandbox.gitCheckout("https://github.com/org/repo", {
      targetDir: "/workspace",
      depth: 1,
    });

    // 运行测试套件，实时流式返回输出
    return sandbox.exec("npm", ["test"], { stream: true });
  },
};

```

只要你提供相同的 ID，后续请求就可以从世界任何地方访问这个 sandbox。

## 安全凭证注入

Agent 工作负载中最难的问题之一是认证。你经常需要 agent 访问私有服务，但又不能完全信任它们持有原始凭证。

Sandboxes 通过可编程出口代理（egress proxy）在网络层注入凭证来解决这个问题。这意味着 sandbox 中的 agent 永远无法访问凭证，而你可以完全自定义认证逻辑：

```
class OpenCodeInABox extends Sandbox {
  static outboundByHost = {
    "my-internal-vcs.dev": (request, env, ctx) => {
      const headersWithAuth = new Headers(request.headers);
      headersWithAuth.set("x-auth-token", env.SECRET);
      return fetch(request, { headers: headersWithAuth });
    }
  }
}

```

关于其工作原理的深入探讨——包括身份感知凭证注入、动态修改规则，以及与 Workers bindings 的集成——请阅读我们最近关于 Sandbox 认证的博客文章。

## 真正的终端，而非模拟

早期 agent 系统通常将 shell 访问建模为请求-响应循环：运行命令、等待输出、把内容塞回 prompt、重复。这能用，但这不是开发者实际使用终端的方式。

人类会运行某个命令、观察流式输出、中断它、稍后重连、继续。Agent 也能从这种反馈循环中获益。

2月，我们发布了 PTY 支持。这是 Sandbox 中的伪终端会话，通过 WebSocket 代理，兼容 xterm.js。

只需调用 sandbox.terminal 来提供后端：

```
// Worker: 将 WebSocket 连接升级为实时终端会话
export default {
  async fetch(request: Request, env: Env) {
    const url = new URL(request.url);
    if (url.pathname === "/terminal") {
      const sandbox = getSandbox(env.Sandbox, "my-session");
      return sandbox.terminal(request, { cols: 80, rows: 24 });
    }
    return new Response("Not found", { status: 404 });
  },
};

```

然后使用 xterm addon 从客户端调用：

```
// 浏览器：将 xterm.js 连接到 sandbox shell
import { Terminal } from "xterm";
import { SandboxAddon } from "@cloudflare/sandbox/xterm";

const term = new Terminal();
const addon = new SandboxAddon({
  getWebSocketUrl: ({ origin }) => `${origin}/terminal`,
});

term.loadAddon(addon);
term.open(document.getElementById("terminal-container")!);
addon.connect({ sandboxId: "my-session" });

```

这让 agent 和开发者都能使用完整的 PTY 来实时调试这些会话。

每个终端会话都有自己隔离的 shell、自己的工作目录、自己的环境。想开多少个就开多少个，就像在你自己的机器上一样。输出在服务端缓冲，所以重连时会回放你错过的内容。

## 会记忆的代码解释器

对于数据分析、脚本编写和探索性工作流，我们还提供了一个更高层的抽象：持久化代码执行上下文。

关键词是"持久化"。许多代码解释器实现在每个代码片段都是隔离运行的，因此调用之间状态会消失。你不能在一步中设置变量然后在下一步中读取它。

Sandboxes 允许你创建持久化状态的"上下文"。变量和导入跨调用持久化，就像在 Jupyter notebook 中一样：

```
// 创建 Python 上下文。状态在其生命周期内持久化
const ctx = await sandbox.createCodeContext({ language: "python" });

// 第一次执行：加载数据
await sandbox.runCode(`
import pandas as pd
df = pd.read_csv('/workspace/sales.csv')
df['margin'] = (df['revenue'] - df['cost']) / df['revenue']
`, { context: ctx });

// 第二次执行：df 仍然在
const result = await sandbox.runCode(`
df.groupby('region')['margin'].mean().sort_values(ascending=False)
`, { context: ctx, onStdout: (line) => console.log(line.text) });

// result 包含 matplotlib 图表、结构化 JSON 输出和 Pandas HTML 表格

```

## 启动服务器，获取 URL，直接发布

当 agent 能够构建东西并立即展示给用户时，它们会更加有用。Sandboxes 支持后台进程、就绪检查和预览 URL。这让 agent 可以启动开发服务器并在不离开对话的情况下分享实时链接。

```
// 将开发服务器作为后台进程启动
const server = await sandbox.startProcess("npm run dev", {
  cwd: "/workspace",
});

// 等到服务器真正就绪——不要只是 sleep 然后祈祷
await server.waitForLog(/Local:.*localhost:(\d+)/);

// 用公共 URL 暴露运行中的服务
const { url } = await sandbox.exposePort(3000);

// url 是 agent 可以分享给用户的实时公共 URL
console.log(`Preview: ${url}`);

```

通过 waitForPort() 和 waitForLog()，agent 可以根据运行程序的真实信号来编排工作，而不是靠猜测。这比常见的替代方案好得多——那通常是某种版本的 sleep(2000) 加上祈祷。

## 监听文件系统并立即响应

现代开发循环是事件驱动的。保存文件，重新运行构建。编辑配置，重启服务器。修改测试，重跑套件。

我们在3月发布了 sandbox.watch()。它返回一个 SSE 流，底层基于原生 inotify——Linux 用于文件系统事件的内核机制。

```
import { parseSSEStream, type FileWatchSSEEvent } from '@cloudflare/sandbox';

const stream = await sandbox.watch('/workspace/src', {
  recursive: true,
  include: ['*.ts', '*.tsx']
});

for await (const event of parseSSEStream<FileWatchSSEEvent>(stream)) {
  if (event.type === 'modify' && event.path.endsWith('.ts')) {
    await sandbox.exec('npx tsc --noEmit', { cwd: '/workspace' });
  }
}

```

这是那种悄悄改变 agent 能力的基础原语之一。一个能实时观察文件系统的 agent 可以参与到与人类开发者相同的反馈循环中。

## 用快照快速唤醒

想象一个（人类）开发者在笔记本电脑上工作。他们 git clone 一个 repo，运行 npm install，写代码，推一个 PR，然后在等待代码审查时合上笔记本。当需要恢复工作时，只需重新打开笔记本，从上次离开的地方继续。

如果 agent 想在一个简单的容器平台上复制这个工作流，你会遇到问题。如何快速从上次离开的地方恢复？你可以保持 sandbox 运行，但那样就要为空闲计算付费。你可以从容器镜像重新开始，但那样就要等待漫长的 git clone 和 npm install。

我们的答案是快照（snapshots），将在接下来几周推出。

快照保存容器的完整磁盘状态、OS 配置、已安装依赖、修改的文件、数据文件等等。然后让你之后快速恢复。

你可以配置 Sandbox 在休眠时自动创建快照。

```
class AgentDevEnvironment extends Sandbox {
  sleepAfter = "5m";
  persistAcrossSessions = {type: "disk"}; // 也可以指定个别目录
}

```

你也可以编程式地创建快照并手动恢复。这对于工作检查点或分叉会话很有用。例如，如果你想并行运行四个 agent 实例，可以轻松从同一状态启动四个 sandbox。

```
class AgentDevEnvironment extends Sandbox {}

async forkDevEnvironment(baseId, numberOfForks) {
  const baseInstance = await getSandbox(baseId);
  const snapshotId = await baseInstance.snapshot();

  const forks = Array.from({ length: numberOfForks }, async (_, i) => {
    const newInstance = await getSandbox(`${baseId}-fork-${i}`);
    return newInstance.start({ snapshot: snapshotId });
  });
  await Promise.all(forks);
}

```

快照存储在你账户中的 R2 上，提供持久性和位置无关性。R2 的分层缓存系统允许在 Region: Earth 的任何地方快速恢复。

在未来的版本中，内存中的活跃状态也将被捕获，允许运行中的进程从上次中断的地方精确恢复。终端和编辑器将以关闭时的精确状态重新打开。

如果你有兴趣在快照正式上线之前恢复会话状态，可以使用现有的备份和恢复方法。这些方法也通过 R2 持久化和恢复目录，但性能不如真正的 VM 级快照。不过，与简单地重新创建会话状态相比，仍然可以带来显著的速度提升。

启动 sandbox、克隆 'axios' 并运行 npm install 需要 30 秒。从备份恢复只需 2 秒。

敬请关注快照的正式发布。

## 更高限额和 Active CPU 定价

自初始发布以来，我们一直在稳步增加容量。标准定价方案的用户现在可以运行 15,000 个并发 lite 实例、6,000 个 basic 实例，以及超过 1,000 个更大的实例。想运行更多？联系我们！

我们还改变了定价模式，使大规模运行更具成本效益。Sandboxes 现在只对实际使用的 CPU 周期收费。这意味着你不会在 agent 等待 LLM 响应时为空闲 CPU 付费。

## 这就是一台计算机的样子

9个月前，我们发布了一个能运行命令和访问文件系统的 sandbox。那足以验证概念。

我们现在拥有的东西本质上不同了。今天的 Sandbox 是一个完整的开发环境：一个可以连接浏览器的终端、一个具有持久状态的代码解释器、带实时预览 URL 的后台进程、一个实时发射变更事件的文件系统、用于安全凭证注入的出口代理，以及一个让热启动几乎瞬时完成的快照机制。

当你在此基础上构建时，一个令人满意的模式浮现出来：agent 做真正的工程工作。克隆 repo、安装依赖、跑测试、读失败日志、改代码、再跑测试。这种紧密的反馈循环让人类工程师高效——现在 agent 也能享有同样的循环了。

我们目前在 SDK 的 0.8.9 版本。你可以今天就开始使用：

```
npm i @cloudflare/sandbox@latest

```

核心观点：Cloudflare Sandboxes 从一个简单的命令执行环境演进为完整的 agent 开发环境——终端、持久状态、实时文件监听、安全凭证注入、快照恢复，让 AI agent 真正拥有了属于自己的计算机，可以像人类工程师一样完成完整的开发工作流。