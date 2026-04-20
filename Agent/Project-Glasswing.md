# [2026-04-07] Project Glasswing：Anthropic 联合12家巨头用AI守护全球关键软件安全

原文链接：Project Glasswing by Anthropic发布日期：2026-04-07

## 引言

今天我们宣布 Project Glasswing，这是一项新倡议，汇聚了 Amazon Web Services、Anthropic、Apple、Broadcom、Cisco、CrowdStrike、Google、JPMorganChase、Linux Foundation、Microsoft、NVIDIA 和 Palo Alto Networks，共同致力于保护全球最关键的软件安全。

我们发起 Project Glasswing，是因为我们在 Anthropic 训练的一个新前沿模型中观察到的能力，我们相信这些能力可能重塑网络安全格局。Claude Mythos Preview 是一个通用的、尚未发布的前沿模型，它揭示了一个严峻的事实：AI 模型的编程能力已经达到了这样的水平——在发现和利用软件漏洞方面，它们可以超越除最顶尖的人类之外的所有人。

Mythos Preview 已经发现了数千个高危漏洞，包括每一个主要操作系统和 Web 浏览器中的漏洞。鉴于 AI 进步的速度，这类能力在不久的将来就会扩散，甚至可能落入那些并不致力于安全部署的行为者手中。其后果——对经济、公共安全和国家安全而言——可能是严重的。Project Glasswing 是一次紧急尝试，旨在将这些能力用于防御目的。

作为 Project Glasswing 的一部分，上述合作伙伴将使用 Mythos Preview 作为其防御性安全工作的一部分；Anthropic 将分享我们的发现，以便整个行业受益。我们还向另外 40 多个构建或维护关键软件基础设施的组织提供了访问权限，使他们能够使用该模型扫描和保护第一方和开源系统。Anthropic 承诺为这些工作提供高达 1 亿美元的 Mythos Preview 使用额度，以及 400 万美元直接捐赠给开源安全组织。

Project Glasswing 是一个起点。没有任何单一组织能独自解决这些网络安全问题：前沿 AI 开发者、其他软件公司、安全研究人员、开源维护者以及世界各国政府都有不可或缺的角色。保卫全球网络基础设施的工作可能需要数年；而前沿 AI 能力可能在未来几个月内就会大幅提升。要让网络防御者占据优势，我们需要立即行动。

## AI 时代的网络安全

我们每天依赖的软件——负责运行银行系统、存储医疗记录、连接物流网络、保持电网运转等等——一直包含 bug。许多是小问题，但有些是严重的安全缺陷，一旦被发现，可能让网络攻击者劫持系统、中断运营或窃取数据。

我们已经看到网络攻击对重要企业网络、医疗系统、能源基础设施、交通枢纽以及全球政府机构信息安全造成的严重后果。在全球舞台上，来自中国、伊朗、朝鲜和俄罗斯等国的国家支持的攻击已经威胁到支撑民用生活和军事准备的基础设施。即使是较小规模的攻击，例如针对个别医院或学校的攻击，仍然可能造成重大经济损失、暴露敏感数据，甚至危及生命。目前全球网络犯罪的年度成本估计约为 5000 亿美元。

许多软件缺陷多年来未被发现，因为找到并利用它们需要只有少数精英安全专家才具备的专业知识。随着最新前沿 AI 模型的出现，发现和利用软件漏洞所需的成本、精力和专业水平都大幅下降。过去一年，AI 模型在阅读和推理代码方面变得越来越有效——特别是，它们展示了发现漏洞并找出利用方法的惊人能力。Claude Mythos Preview 在这些网络安全技能上展示了一次飞跃——它发现的漏洞在某些情况下经受了数十年的人工审查和数百万次自动化安全测试，而它开发的 exploit 越来越复杂。

在首届 DARPA Cyber Grand Challenge 十年之后，前沿 AI 模型现在正在与最优秀的人类竞争，争夺发现和利用漏洞的能力。如果没有必要的安全措施，这些强大的网络能力可能被用来利用世界上最重要软件中的众多现有缺陷。这可能使各种网络攻击变得更加频繁和具有破坏性，并赋予美国及其盟友的对手更大的权力。因此，解决这些问题是民主国家的重要安全优先事项。

尽管 AI 增强的网络攻击风险很严重，但也有理由保持乐观：使 AI 模型在坏人手中变得危险的同样能力，也使它们在发现和修复重要软件缺陷方面具有无价的价值——并且能够生产出安全 bug 少得多的新软件。Project Glasswing 是朝着让防御者在即将到来的 AI 驱动网络安全时代中获得持久优势迈出的重要一步。

## 用 Claude Mythos Preview 识别漏洞和 exploit

在过去几周里，我们使用 Claude Mythos Preview 识别了每一个主要操作系统和每一个主要 Web 浏览器中的数千个 zero-day 漏洞（即软件开发者此前未知的缺陷），其中许多是关键级别的，同时还涵盖了一系列其他重要软件。

在我们的 Frontier Red Team 博客的一篇文章中，我们为已修补的一部分漏洞提供了技术细节，以及在某些情况下 Mythos Preview 找到的利用方法。它能够几乎完全自主地识别这些漏洞中的绝大多数——并开发出许多相关的 exploit——无需任何人工引导。以下是三个例子：

上述漏洞已报告给相关软件的维护者，目前均已修补。对于许多其他漏洞，我们今天提供了详情的加密哈希值（见 Red Team 博客），我们将在修复到位后披露具体信息。

CyberGym 等评估基准也印证了 Mythos Preview 与我们次优模型 Claude Opus 4.6 之间的巨大差距：

评估项目

Mythos Preview

Opus 4.6

网络安全漏洞复现

83.1%

66.6%

## 合作伙伴反馈

Cisco：

AWS：

Microsoft（Igor Tsyganskiy，网络安全和微软研究执行副总裁）：

CrowdStrike：

Linux Foundation：

JPMorganChase（Pat Opet，首席信息安全官）：

Google：

Palo Alto Networks：

## Benchmark 表现

Mythos Preview 强大的网络安全能力源自其出色的 agentic 编码和推理能力。以下是各项编码任务的评测结果：

评估基准

Mythos Preview

Opus 4.6

SWE-bench Verified

77.8%

53.4%

SWE-bench Pro

82.0%

65.4%

SWE-bench Multilingual

59.0%

27.1%

SWE-bench Multimodal

87.3%

77.8%

Terminal-Bench 2.0

93.9%

80.8%

GPQA Diamond

94.6%

91.3%

HLE（无工具）

56.8%

40.0%

HLE（有工具）

64.7%

53.1%

MMMU

86.9%

83.7%

BrowseComp

79.6%

72.7%

注：BrowseComp 中 Mythos Preview 使用的 token 比 Opus 4.6 少 4.9 倍。

我们不打算让 Claude Mythos Preview 普遍可用，但我们的最终目标是让用户能够安全地大规模部署 Mythos 级别的模型——不仅用于网络安全目的，也用于这类高度能力模型将带来的无数其他好处。为此，我们需要在开发网络安全（及其他）安全措施方面取得进展，以检测和阻止模型最危险的输出。我们计划在即将推出的 Claude Opus 模型中启用新的安全措施，使我们能够改进和完善它们。

## Project Glasswing 计划

今天的公告是一项长期努力的开始。要取得成功，需要技术行业及其他领域的广泛参与。

Project Glasswing 合作伙伴将获得 Claude Mythos Preview 的访问权限，用于发现和修复其基础系统中的漏洞或弱点——这些系统代表着全球共享网络攻击面的很大一部分。我们预计这项工作将集中在本地漏洞检测、二进制文件黑盒测试、端点安全和系统渗透测试等任务上。

Anthropic 承诺向 Project Glasswing 及额外参与者提供的 1 亿美元模型使用额度将覆盖此研究预览期间的大量使用。之后，Claude Mythos Preview 将以每百万输入/输出 token 25/125 美元的价格提供给参与者（参与者可以通过 Claude API、Amazon Bedrock、Google Cloud Vertex AI 和 Microsoft Foundry 访问该模型）。

除了模型使用额度的承诺外，我们还通过 Linux Foundation 向 Alpha-Omega 和 OpenSSF 捐赠了 250 万美元，向 Apache Software Foundation 捐赠了 150 万美元，以帮助开源软件的维护者应对这一变化的格局。

我们打算让这项工作扩大范围并持续数月，我们将尽可能多地分享信息，以便其他组织将这些经验应用到自己的安全工作中。在 90 天内，Anthropic 将公开报告我们学到的内容，以及已修复的漏洞和可以披露的改进。我们还将与领先的安全组织合作，为安全实践在 AI 时代的演进制定一套实用建议，可能包括：

Anthropic 还一直在与美国政府官员讨论 Claude Mythos Preview 及其攻防网络安全能力。正如我们上面指出的，保护关键基础设施是民主国家的首要国家安全优先事项——这些网络安全能力的出现是美国及其盟友必须在 AI 技术上保持决定性领先地位的又一原因。政府在帮助维持这一领先地位以及评估和缓解与 AI 模型相关的国家安全风险方面有着不可或缺的角色。

我们希望 Project Glasswing 能够在行业和公共部门播下更大努力的种子。我们邀请其他 AI 行业成员加入我们，帮助制定行业标准。从中期来看，一个独立的第三方机构——能够汇聚私营和公共部门组织——可能是继续开展这些大规模网络安全项目的理想归宿。

核心观点：Anthropic 发布了 Claude Mythos Preview，这是一个在网络安全能力上实现质的飞跃的前沿模型，已发现包括每个主要操作系统和浏览器在内的数千个 zero-day 漏洞。为应对 AI 能力扩散带来的网络安全威胁，Anthropic 联合 AWS、Apple、Google、Microsoft 等12家科技巨头发起 Project Glasswing，承诺投入1亿美元使用额度用于防御性安全工作，旨在让防御者在 AI 驱动的网络安全新时代中占据先机。