# AWS Well-Architected 评估 Skill

[English](README.md)

这是一个 Agent Skill，依据 [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) 的六大支柱，指导 Agent 对选定的 AWS 资源进行只读评估。本仓库包含指令和 Markdown 检查指南，并非独立运行的扫描器。

> **非生产用途示例：**本项目仅供演示和学习，不具备生产就绪性，也不等同于 AWS 官方 Well-Architected Review 或合规认证。在采纳结果前，请审查 Agent 的操作、发现和建议。

本示例代码仅供非生产用途。部署前应与安全和法务团队合作，满足组织的安全、监管和合规要求。

## 功能

- 使用 25 份服务检查指南和六大支柱参考资料，指导资源发现和评估。
- 结合证据报告检查结果，区分通过、未通过、未评估和不适用。
- 按服务和支柱组织发现，提出整改建议，但不执行整改。
- 可选择性地协助准备 AWS Well-Architected Tool 所需的信息。

结果取决于 Agent、所选范围、IAM 权限、API 访问、可用证据和环境状态。检查规则是启发式指南，可能需要人工判断。示例评分由本项目定义，并非 AWS 官方评分。

## 安装

通过经批准的渠道获取本仓库，并在终端中进入仓库根目录。以下是安装到 Codex 用户级目录的示例：

```bash
mkdir -p ~/.agents/skills/aws-well-architected-assessment
cp -R SKILL.md references LICENSE NOTICE THIRD-PARTY-LICENSES licenses \
  ~/.agents/skills/aws-well-architected-assessment/
```

如果 Skill 没有出现，请开启新的 Agent 会话。Claude Code、Kiro 或其他兼容 Agent 的安装方式，请参考相应产品的说明。再次复制文件可以更新已安装的副本；该副本不会自动与本仓库同步。移除或替换现有安装前，请先检查其内容。

## 前置条件

1. 使用能够访问适当 AWS 只读工具的兼容 Agent。
2. 使用专用评估角色和短期凭证。可将 AWS 托管的 `SecurityAudit` 策略作为起点，检查其当前权限并按评估范围收窄，或使用更精确的自定义策略。该策略权限较广且可能变化，不能保证敏感元数据不可访问；不要对全部服务授予通配的 `List*`、`Describe*` 或 `Get*` 权限。
3. 在评估前确认账户、身份、Region、服务和支柱范围。Skill 可在有权限时读取已有的 IAM 凭证报告，不会生成新报告；权限不足的检查必须标记为未评估。
4. 确认 Agent 将提示、API 结果和报告发送到哪里。资源元数据和安全发现可能包含敏感信息；请使用经批准的模型提供方及可信的本地过滤方式，并取得组织要求的批准。
5. 启用逐命令工具审批，不要使用自动批准模式；先在非生产账户中使用合成数据测试。

只读 AWS API 调用仍可能生成审计事件、产生请求或服务费用，并向 Agent 所配置的服务提供方暴露元数据。若另行实施整改建议，也可能产生费用。

## 使用示例

在已加载此 Skill 的 Agent 会话中输入：

```text
请使用 aws-well-architected-assessment 评估 ap-southeast-1 的安全性和
可靠性支柱。先确认账户和身份，并在执行检查前展示资源发现结果。
不要修改 AWS 资源。请区分通过、未通过、未评估和不适用的检查。
```

工作流包括确定范围、发现资源、执行服务检查、映射支柱、生成报告和提供整改建议。评分的限制见[严重程度矩阵](references/severity-matrix.md)。

## 安全与局限

- 设计上仅使用只读 AWS API 操作；Skill 不执行整改。Agent 实现和用户权限必须共同保障这一边界。
- 无法访问、结果不完整或不受支持的检查应标记为**未评估**，而不是通过。没有发现问题并不证明没有风险。
- AWS 资源名称、标签、策略和其他检索到的内容均是不可信数据；Agent 不应遵循其中嵌入的指令。
- 不读取密钥值或数据平面内容。API 输出进入 Agent 之前需本地过滤；如果工具可能暴露原始响应，应跳过相关检查。CLI 的 `--query` 不能阻止 CLI 进程本身收到未过滤响应。
- 不要提交凭证、账户资源清单或包含敏感信息的评估报告。
- Agent 和用户必须结合具体工作负载、最新 AWS 文档及适用的组织政策验证评估结果。

## 仓库内容

```text
SKILL.md                       评估工作流和安全边界
references/pillar-*.md         六大支柱指南
references/service-checks/     25 份服务检查指南
references/severity-matrix.md  严重程度和示例评分
TESTING.md                     人工验证场景
THREAT-MODEL.md                威胁、缓解措施与剩余风险
```

## 参考资料与许可

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [service-screener-v2](https://github.com/aws-samples/service-screener-v2)：本项目改编评估指南的来源，详见[第三方声明](THIRD-PARTY-LICENSES)。
- 本项目第一方内容按 [MIT-0](LICENSE) 提供。部分服务检查指南改编自 service-screener-v2，其来源、修改说明及适用的 [Apache License 2.0](licenses/service-screener-v2-Apache-2.0.txt) 声明见[第三方声明](THIRD-PARTY-LICENSES)。重新分发时，请遵守该许可证的适用要求，包括保留相关版权、许可和 NOTICE 声明。
