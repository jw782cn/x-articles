# [2026-04-09] 扩展 Managed Agents：将大脑与双手解耦

原文链接：Scaling Managed Agents: Decoupling the brain from the hands by Anthropic (Lance Martin, Gabe Cemaj, Michael Cohen)

我们工程博客上一个持续探讨的话题是如何构建高效的 agent，以及如何为长时间运行的工作设计 harness。这些工作的一个共同主线是：harness 编码了关于 Claude 自身无法独立完成什么的假设。然而，这些假设需要被频繁审视，因为随着模型的改进，它们可能已经过时。

举一个例子：在之前的工作中，我们发现 Claude Sonnet 4.5 会在感知到 context 上限接近时过早地结束任务——这种行为有时被称为"context 焦虑"。我们通过在 harness 中添加 context 重置来解决这个问题。但当我们在 Claude Opus 4.5 上使用同样的 harness 时，我们发现这种行为已经消失了。那些重置机制变成了累赘。

我们预期 harness 会持续演进。因此我们构建了 Managed Agents：一个 Claude Platform 上的托管服务，通过一小组接口代你运行长周期 agent，这些接口的设计初衷是要比任何特定实现都更持久——包括我们今天运行的那些实现。

构建 Managed Agents 意味着要解决一个计算领域的老问题：如何为"尚未被构想出的程序"设计系统。几十年前，操作系统通过将硬件虚拟化为抽象层——进程、文件——来解决这个问题，这些抽象层足够通用，可以服务于尚不存在的程序。抽象层比硬件更持久。read() 命令不关心它访问的是 1970 年代的磁盘组还是现代的 SSD。上层的抽象保持稳定，底层的实现可以自由变化。

Managed Agents 遵循同样的模式。我们将 agent 的组件虚拟化：一个 session（记录所有发生事件的追加式日志）、一个 harness（调用 Claude 并将 Claude 的工具调用路由到相应基础设施的循环）、以及一个 sandbox（Claude 可以运行代码和编辑文件的执行环境）。这使得每个组件的实现都可以被替换，而不会干扰其他组件。我们对这些接口的形状有明确的主张，但对接口背后运行什么保持开放。

## 不要养宠物

我们一开始把所有 agent 组件放进一个容器里，这意味着 session、agent harness 和 sandbox 共享一个环境。这种方法有一些好处，包括文件编辑是直接的系统调用，也不需要设计服务边界。

但是，把所有东西耦合到一个容器里，我们遇到了一个老旧的基础设施问题：我们养了一只"宠物"。在"宠物 vs 牲畜"的类比中，宠物是一个有名字的、需要精心照料的个体，你承受不起失去它；而牲畜是可互换的。在我们的场景中，服务器变成了那只宠物：如果容器挂了，session 就丢了。如果容器无响应，我们就得把它"救"回来。

救治容器意味着要调试无响应的卡死 session。我们唯一的观察窗口是 WebSocket 事件流，但它无法告诉我们故障出在哪里，这意味着 harness 中的 bug、事件流中的丢包、或者容器下线，呈现出来的症状完全一样。要搞清楚出了什么问题，工程师必须在容器内开一个 shell，但因为那个容器通常也存放着用户数据，这种方式本质上意味着我们无法进行调试。

第二个问题是，harness 假设 Claude 正在处理的内容都在它所在的容器里。当客户要求我们将 Claude 连接到他们的 VPC 时，他们要么把自己的网络与我们的网络对等互联，要么在自己的环境中运行我们的 harness。一个烘焙在 harness 中的假设，在我们想要将它连接到不同基础设施时就变成了问题。

我们最终得出的解决方案是将我们所说的"大脑"（Claude 及其 harness）与"双手"（执行操作的 sandbox 和工具）以及"session"（session 事件的日志）解耦。每个部分都变成了一个接口，对其他部分做尽可能少的假设，每个部分都可以独立地故障或被替换。

harness 离开容器。 将大脑与双手解耦意味着 harness 不再驻留在容器内部。它像调用其他任何工具一样调用容器：execute(name, input) → string。容器变成了"牲畜"。如果容器挂了，harness 将故障作为工具调用错误捕获，并传回给 Claude。如果 Claude 决定重试，一个新的容器可以用标准配方重新初始化：provision(\{resources\})。我们不再需要把故障的容器救回来。

从 harness 故障中恢复。 harness 也变成了"牲畜"。因为 session 日志位于 harness 之外，harness 中没有任何东西需要在崩溃中幸存。当一个 harness 故障时，新的 harness 可以通过 wake(sessionId) 重启，使用 getSession(id) 获取事件日志，然后从最后一个事件处恢复。在 agent 循环期间，harness 通过 emitEvent(id, event) 写入 session，以保持事件的持久记录。

安全边界。 在耦合设计中，Claude 生成的任何不可信代码都在与凭据相同的容器中运行——所以一次 prompt 注入只需要说服 Claude 读取自己的环境变量。一旦攻击者拿到了那些 token，他们就可以创建新的、不受限制的 session 并将工作委派给它们。缩小权限范围是一个显而易见的缓解措施，但这编码了一个假设——Claude 用有限的 token 能做什么——而 Claude 正在变得越来越聪明。结构性的修复方案是确保 token 永远无法从 Claude 生成代码运行的 sandbox 中被访问到。

我们使用了两种模式来确保这一点。认证信息可以与资源绑定，或者存放在 sandbox 外部的保险库中。对于 Git，我们使用每个仓库的 access token 在 sandbox 初始化时克隆仓库，并将其接入本地的 git remote。Git push 和 pull 在 sandbox 内部即可工作，agent 本身从不接触 token。对于自定义工具，我们支持 MCP 并将 OAuth token 存储在安全的保险库中。Claude 通过一个专用代理调用 MCP 工具；这个代理接收与 session 关联的 token。然后代理从保险库获取相应的凭据，并向外部服务发起调用。harness 永远不会接触到任何凭据。

## Session 不是 Claude 的 context window

长周期任务通常会超出 Claude context window 的长度，而解决这个问题的标准方法都涉及关于保留什么的不可逆决策。我们在之前关于 context engineering 的工作中探索过这些技术。例如，compaction 让 Claude 保存 context window 的摘要，memory 工具让 Claude 将上下文写入文件，从而实现跨 session 的学习。这可以与 context trimming 配合使用，后者选择性地移除 token，比如旧的工具结果或思考块。

但选择性保留或丢弃 context 的不可逆决策可能导致故障。很难知道未来的对话轮次需要哪些 token。如果消息被 compaction 步骤转换了，harness 会从 Claude 的 context window 中移除已压缩的消息，而这些消息只有在被存储的情况下才可恢复。之前的工作探索了通过将 context 存储为 context window 之外的对象来解决这个问题的方法。例如，context 可以是 REPL 中的一个对象，LLM 通过编写代码来对其进行过滤或切片，以编程方式访问它。

在 Managed Agents 中，session 提供了同样的好处，作为一个存在于 Claude context window 之外的 context 对象。但与存储在 sandbox 或 REPL 中不同，context 被持久地存储在 session 日志中。接口 getEvents() 允许大脑通过选择事件流的位置切片来查询 context。这个接口可以灵活使用，允许大脑从上次停止阅读的地方继续，回退到特定时刻之前几个事件以查看前因，或者在特定操作之前重新阅读 context。

获取到的事件还可以在 harness 中进行转换，然后再传递给 Claude 的 context window。这些转换可以是 harness 编码的任何内容，包括用于实现高 prompt cache 命中率的 context 组织方式和 context engineering。我们将可恢复的 context 存储（在 session 中）与任意的 context 管理（在 harness 中）的关注点分离开来，因为我们无法预测未来的模型将需要什么样的 context engineering。这些接口将 context 管理推入 harness，只保证 session 是持久的且可供查询。

## 多个大脑，多双手

多个大脑。 将大脑与双手解耦解决了我们最早期的客户投诉之一。当团队想让 Claude 对他们自己 VPC 中的资源进行操作时，唯一的途径是将他们的网络与我们的网络对等互联，因为持有 harness 的容器假设所有资源都在它旁边。一旦 harness 不再在容器中，这个假设就消失了。同样的变更也带来了性能上的回报。当我们最初将大脑放在容器中时，这意味着多个大脑需要同样多的容器。对于每个大脑，在容器配置完成之前都无法进行推理；每个 session 都要预先支付完整的容器启动成本。每个 session，即使是那些永远不会触及 sandbox 的 session，都必须克隆仓库、启动进程、从我们的服务器获取待处理的事件。

这段死亡时间体现为 time-to-first-token（TTFT），它衡量的是 session 从接受工作到产出第一个响应 token 之间等待了多久。TTFT 是用户感知最强烈的延迟。

将大脑与双手解耦意味着容器仅在需要时才由大脑通过工具调用（execute(name, input) → string）来配置。因此，一个不立即需要容器的 session 就不用等待。推理可以在编排层从 session 日志中拉取待处理事件后立即开始。使用这种架构，我们的 p50 TTFT 下降了约 60%，p95 下降超过 90%。扩展到多个大脑只需要启动多个无状态的 harness，并仅在需要时将它们连接到双手。

多双手。 我们还希望能够将每个大脑连接到多双手。在实践中，这意味着 Claude 必须对多个执行环境进行推理，并决定将工作发送到哪里——这是一个比在单个 shell 中操作更难的认知任务。我们一开始将大脑放在单个容器中，因为早期的模型还不具备这种能力。随着智能的提升，单个容器反而变成了限制：当那个容器故障时，我们就丢失了大脑正在操控的所有双手的状态。

将大脑与双手解耦使得每只手都成为一个工具，execute(name, input) → string：一个名称和输入传入，一个字符串返回。这个接口支持任何自定义工具、任何 MCP 服务器，以及我们自己的工具。harness 不知道 sandbox 是一个容器、一部手机、还是一个宝可梦模拟器。而且因为没有任何一双手与任何一个大脑耦合，大脑之间可以互相传递双手。

## 结论

我们面临的挑战是一个老问题：如何为"尚未被构想出的程序"设计系统。操作系统已经持续了数十年，靠的是将硬件虚拟化为足够通用的抽象层，以服务于尚不存在的程序。通过 Managed Agents，我们的目标是设计一个能够容纳围绕 Claude 的未来 harness、sandbox 或其他组件的系统。

Managed Agents 是同样精神下的 meta-harness，对 Claude 未来将需要的特定 harness 不持特定立场。相反，它是一个具有通用接口的系统，允许许多不同的 harness。例如，Claude Code 是一个我们在各种任务中广泛使用的出色 harness。我们也展示了特定任务的 agent harness 在狭窄领域中表现卓越。Managed Agents 可以容纳其中任何一个，随着时间的推移匹配 Claude 的智能水平。

Meta-harness 设计意味着对 Claude 周围的接口持有明确的主张：我们预期 Claude 将需要操作状态的能力（session）和执行计算的能力（sandbox）。我们也预期 Claude 将需要扩展到多个大脑和多双手的能力。我们设计了这些接口，使其能够在长时间跨度内可靠且安全地运行。但我们对 Claude 将需要的大脑或双手的数量或位置不做任何假设。

核心观点：Managed Agents 借鉴操作系统将硬件虚拟化为持久抽象层的思路，将 agent 的三个核心组件——session（状态日志）、harness（编排循环）和 sandbox（执行环境）——解耦为独立接口。这种"meta-harness"设计不对具体实现做假设，而是提供通用接口，使系统能够随模型能力的演进而自然适应，同时显著提升了性能（TTFT 降低 60%-90%）、可靠性和安全性。