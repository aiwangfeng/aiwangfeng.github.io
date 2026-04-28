# AI Automation：内容与最佳实践（速查）

> 目标：把“能落地的 AI 自动化（AI automation）”拆成可复用的工程实践：从任务选择、工作流设计、提示词与工具调用、到安全治理、评估与上线监控。

## 1. 什么是 AI automation（以及它擅长什么）
AI automation 通常指：用 LLM/多模态模型把**非结构化输入**（自然语言、邮件、文档、网页、日志）转成**结构化决策与动作**（填表、生成工单、调用 API、写代码、跑脚本、更新知识库）。

更适合 AI 的自动化场景（相对传统 RPA/规则引擎）：
- 输入“脏、乱、差”：格式不固定、表达多样、信息不完整
- 路径不固定：需要在多个步骤中动态选择工具/策略
- 异常多：需要容错、解释、回退与人工介入

不适合直接全自动的场景：
- 高风险不可逆动作（转账、删库、发生产公告）
- 强合规/强审计但缺少可观测与审批链路

## 2. 任务选择：从“低风险高频”开始
优先级建议：
1) **读 → 归纳 → 结构化**：会议纪要、邮件分类、需求拆解、信息抽取
2) **结构化 → 生成**：周报、PRD、FAQ、工单回复草稿
3) **生成 → 建议动作**（先不执行）：给出可执行计划/命令/变更清单
4) **建议动作 → 自动执行**：最后再做（需要强 guardrails + 审批）

## 3. 工作流设计：把“智能”限制在可控边界内
### 3.1 推荐的 agentic workflow 形态
- **Plan-then-Execute（先计划后执行）**：先产出步骤清单，再逐步执行并记录
- **Tool-first（工具优先）**：能查就查、能算就算，减少纯“编造”
- **分层代理**：
  - Orchestrator（调度/路由）
  - Specialist（检索/写作/代码/数据分析）
  - Critic/Verifier（校验/对齐/安全检查）

### 3.2 关键工程点
- **把输入输出结构化**：JSON schema / 表格 / 固定 Markdown 模板
- **显式状态机**：每一步有状态（pending/running/needs_review/done）
- **幂等性**：同一请求重复执行不会造成重复副作用（尤其是写入/调用外部 API）
- **可回放（replay）**：保存每次运行的输入、工具调用、输出与版本

## 4. Prompt / 指令最佳实践（面向自动化）
- **明确角色与边界**：能做什么、不能做什么、遇到不确定如何处理
- **强制输出格式**：例如“只输出 JSON，字段为 …，不得输出额外文本”
- **把业务规则写成可测试条款**：例如“缺少订单号则必须追问，不得猜测”
- **少用长篇系统提示堆砌**：更推荐“短指令 + 工具 + 校验 + 反馈闭环”
- **对关键步骤加自检**：例如“输出前检查是否包含 PII/密钥/越权信息”

（结构化提示框架如 CO-STAR 常用于把上下文、目标、风格、受众、响应格式固定下来，便于自动化与评估。）

## 5. Guardrails（护栏）：安全与可靠性的核心
参考 Datadog 对 LLM guardrails 的总结：guardrails 通常在 **输入前 / 生成中 / 输出后** 分层工作，用于防御 prompt injection、敏感数据泄露、工具滥用等风险。

### 5.1 输入侧（Input guardrails）
- 注入/越狱检测：识别“忽略以上指令”“泄露系统提示”等
- PII/敏感信息检测：正则 + 分类器
- 域边界：只允许与业务相关的意图进入 agent

### 5.2 工具调用侧（Tool guardrails）
- **最小权限**：token/权限按任务最小化（RBAC）
- allowlist：只允许调用白名单工具/命令
- 参数校验：对 URL、文件路径、SQL、shell 参数做校验/转义
- 高风险动作必须 **human-in-the-loop** 审批

### 5.3 输出侧（Output guardrails）
- schema 校验：JSON/表格格式不对就拒绝或修复
- 敏感信息脱敏：密钥、token、个人信息默认打码
- 事实性校验：要求引用来源/证据；或二次检索交叉验证

## 6. 评估（Evals）：把“好不好”变成可度量
Amazon 的 agent 评估经验强调：agentic 系统不能只看最终答案，还要评估**工具选择、推理链路、多步一致性、记忆检索效率、任务完成率**，并在生产中持续监控“性能衰减/agent decay”。

LangChain/LangSmith 的观点也类似：你无法在上线前穷尽输入空间；需要在生产中通过 tracing + 在线评估来发现真实失败模式。

### 6.1 建议的指标（从易到难）
- 任务完成率 / 成功率（end-to-end）
- 工具调用次数、失败率、重试率（是否 thrashing/循环）
- 输出格式合规率（schema pass rate）
- 安全事件：注入命中率、敏感信息泄露率
- 人工介入率（HITL rate）与介入原因分布
- 成本与延迟：token、工具耗时、总耗时

### 6.2 数据集与回归
- 建立“黄金用例集”（golden set）：真实生产 case + 人工标注
- 每次改 prompt/模型/工具都跑回归
- 线上抽样评估：例如抽样 10–20% trace 做自动/人工复核

## 7. 可观测性（Observability）：上线后才是开始
生产环境建议记录：
- 完整对话与上下文（注意脱敏）
- 每一步工具调用（输入参数、返回值、耗时、错误）
- agent 轨迹（trajectory）：中间决策与分支
- 版本信息：模型版本、提示词版本、工具版本

并建立：
- 告警：异常工具调用、循环、格式失败率上升、敏感信息命中
- 灰度发布：小流量试运行 → 观察 → 扩大

## 8. Human-in-the-loop（人工介入）设计
HITL 不是“失败才找人”，而是：
- 高风险动作前置审批（approve/deny）
- 低置信度/高不确定性时请求确认
- 把人工反馈沉淀为新用例，反哺 eval 集

## 9. 一个可复用的“AI 自动化落地清单”
1) 明确任务边界与不可做事项
2) 结构化输入/输出（schema）
3) 工具白名单 + 最小权限 + 参数校验
4) 高风险动作必须审批
5) 全链路日志与可回放（脱敏）
6) 离线 eval + 线上抽样 eval
7) 监控：成功率、循环、格式、泄露、成本、延迟
8) 灰度发布与回滚策略

---

## 参考来源（节选）
- Datadog：LLM guardrails best practices（输入/提示构造/输出三层护栏；防注入、泄露、工具滥用等）
  https://www.datadoghq.com/blog/llm-guardrails-best-practices/
- AWS / Amazon：Evaluating AI agents（评估工具选择、多步一致性、错误恢复；持续监控 agent decay；HITL）
  https://aws.amazon.com/blogs/machine-learning/evaluating-ai-agents-real-world-lessons-from-building-agentic-systems-at-amazon/
- LangChain Blog：You don’t know what your agent will do until it’s in production（生产可观测性、无限输入空间、在线评估）
  https://blog.langchain.com/you-dont-know-what-your-agent-will-do-until-its-in-production/
- OWASP GenAI Top 10（提示注入、敏感数据泄露等风险分类）
  https://genai.owasp.org/
