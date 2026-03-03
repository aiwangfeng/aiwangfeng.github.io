# LangGraph 简介与应用（面向落地）

> LangGraph 是 LangChain 生态里用于构建 **“有状态（stateful）、可循环（cyclic）、可中断/可恢复（interrupt + checkpoint）”** 的 LLM/Agent 工作流框架。它把 agent 的控制流显式建模为 **图（graph）/状态机（state machine）**，从而更适合生产环境的复杂流程编排。

## 1) LangGraph 解决什么问题？
传统“单次调用 LLM”或“线性链式（chain）”在以下场景会变得脆弱：
- 需要 **循环**：检索→总结→发现缺口→再检索
- 需要 **分支**：根据意图路由到不同子流程/子 agent
- 需要 **长流程**：多步工具调用、错误重试、回退策略
- 需要 **可恢复**：中途失败后从某一步继续，而不是重跑
- 需要 **人类介入**：关键步骤审批、补充信息、纠错

LangGraph 的核心价值：把这些“控制流”从 prompt 里抽出来，变成可测试、可观测、可维护的图结构。

## 2) 核心概念（State / Nodes / Edges）
根据 LangChain 官方文档（Graph API overview）：
- **State（状态）**：共享数据结构，表示当前工作流快照。通常用 `TypedDict` / dataclass / Pydantic 定义 schema。
- **Nodes（节点）**：函数（可以是 LLM 调用，也可以是普通代码/工具调用）。输入 state，输出对 state 的更新。
- **Edges（边）**：决定下一步执行哪个节点。可以是固定跳转，也可以是条件分支。

一句话：**nodes 做事，edges 决定下一步做什么**。

### “super-step / Pregel”执行模型（为什么能并行）
LangGraph 的底层执行借鉴 Pregel 的“super-step”思想：同一 super-step 内可并行执行多个节点；当所有节点都 inactive 且无消息在途时终止。这使得某些多 agent/多工具步骤可以并行化（取决于你的图设计）。

## 3) Persistence / Checkpoint（持久化与可恢复）
LangGraph 的一个生产级特性是 **checkpointing**：
- 编译 graph 时配置 **checkpointer**，会在每个 super-step 保存 state 快照
- 这些快照与 `thread_id` 绑定，可用于：
  - **断点续跑**（失败后恢复）
  - **time-travel debugging**（回看每一步 state）
  - **human-in-the-loop**（中断等待人工输入，再继续）

官方文档：Persistence（checkpointer 会在每个 super-step 保存 checkpoint 到 thread）。

## 4) LangGraph vs LangChain（怎么搭配）
一个实用的理解方式：
- **LangChain**：提供 LLM、prompt、tool、retriever、agent executor 等“组件库”。
- **LangGraph**：提供“编排与运行时”（workflow runtime），把组件组织成可循环/可分支/可持久化的图。

你可以在 LangGraph 的 node 里：
- 直接调用 LLM
- 调用 LangChain 的工具（tools）
- 甚至把一个 LangChain agent 当作一个 node

## 5) 典型应用场景（可直接对号入座）
### 5.1 可靠的工具调用 Agent（带重试/回退）
- Router 节点判断：要不要调用工具？调用哪个工具？
- Tool 节点执行工具
- Verifier 节点校验输出（schema/事实/安全）
- 不通过则回到 Router 或进入“人工介入”节点

### 5.2 多 Agent 协作（Supervisor / Teams）
LangChain 官方博客把 multi-agent 连接方式总结为图结构非常自然：
- **Collaboration（共享草稿）**：多个 agent 共享 scratchpad（信息透明，但可能冗长）
- **Supervisor（监督者路由）**：监督者决定调用哪个子 agent；子 agent 各自独立上下文，最终摘要回写全局 state
- **Hierarchical Teams（分层团队）**：子节点本身也是一个子图（graph），更适合复杂系统

### 5.3 Deep Research / 报告生成
- 规划（plan）→ 检索（search）→ 阅读（read）→ 归纳（synthesize）→ 缺口检测（gap）→ 循环补齐 → 输出
- 中间可插入“引用检查/事实核验”节点

### 5.4 客服/工单自动化（带审批）
- 分类与路由 → 拉取用户/订单信息 → 生成回复草稿 → 风险/合规检查 → 人工确认 → 发送

### 5.5 长任务与异步任务
- 需要跨小时/跨天的任务（例如持续监控、分批处理）
- 通过 checkpoint + thread_id 保持状态

## 6) 一个最小心智模型（伪代码）
下面是“状态机式 agent”的最小结构（示意，不保证可直接运行）：

```python
from typing import TypedDict, List

class State(TypedDict):
    messages: List[str]
    next: str

def router(state: State) -> dict:
    # 用 LLM/规则决定下一步：tool / respond / human
    return {"next": "tool"}

def tool_node(state: State) -> dict:
    # 调用外部工具（搜索/数据库/API）并把结果写回 state
    return {"messages": state["messages"] + ["tool result..."]}

def respond(state: State) -> dict:
    # 生成最终答复
    return {"messages": state["messages"] + ["final answer..."]}

# 图：START -> router -> (tool_node -> router) or respond -> END
```

你真正落地时，关键不是“能跑”，而是：
- state schema 是否清晰
- 每个 node 是否可测试
- 是否有 checkpoint / tracing / eval

## 7) 生产落地最佳实践（结合前面 AI automation）
1. **先把 state 设计好**：把“必须记住的东西”放进 state（而不是全塞进对话历史）
2. **节点职责单一**：一个 node 做一件事（检索/解析/校验/写入）
3. **把不确定性显式化**：低置信度就走 human-in-the-loop 分支
4. **工具调用加护栏**：allowlist + 参数校验 + 最小权限
5. **加 verifier 节点**：schema 校验、敏感信息检查、引用检查
6. **启用 checkpoint**：支持恢复与审计；为长任务/审批流打基础
7. **可观测性**：记录每步输入输出（脱敏）、耗时、错误、重试次数

## 8) 参考链接
- Graph API overview（State / Nodes / Edges、super-step）
  https://docs.langchain.com/oss/python/langgraph/graph-api
- Persistence（checkpoint / thread / StateSnapshot）
  https://docs.langchain.com/oss/python/langgraph/persistence
- LangGraph Multi-Agent Workflows（collaboration / supervisor / hierarchical teams）
  https://blog.langchain.com/langgraph-multi-agent-workflows/

---

## 你如果要“用 LangGraph 做一个可用的自动化 agent”，我需要你回答 3 个问题
1) 你的目标任务是什么（例如：自动生成周报/自动处理工单/自动研究并写报告）？
2) 需要哪些工具（网页搜索、GitHub、数据库、shell、内部 API）？
3) 哪些步骤必须人工确认（高风险点）？
