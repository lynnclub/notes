---
title: "概念"
type: "docs"
weight: 1
---

AI发展日新月异，相关概念不时冒出来。

## 基础概念

```text
Architecture Design（架构设计）
↓
Pretraining（预训练）
↓
Fine-tuning（微调）、Distillation（蒸馏）、Quantization（量化）、Alignment（对齐）
↓
Inference（LLM大语言模型推理）
↓
Prompt / CoT / ReAct（思考方式）
↓
Function Call（行动决策输出格式）
↓
Tools（可执行能力）
↓
Skill（能力封装）
↓
Agent（自主决策系统）
↓
Workflow / DAG（工程调度层）
↓
Context / Memory / RAG（记忆）
↓
MCP / OpenAPI / OpenSpec（能力标准化）
```

【2026.03更新】LLM 是大脑，Prompt 决定思考方式，Function Call 选择行动，Tools 执行操作，Skill 封装能力，Agent 形成闭环决策，Memory 和 RAG 提供知识增强，Workflow 负责工程调度，MCP 统一能力标准。

### Pretraining（预训练）

预训练在海量通用语料上训练模型参数，让模型学会语言结构与通用知识。这是 LLM 能力的来源阶段，计算量最大、成本最高。

### Fine-tuning（微调）

在特定数据集上继续训练，使模型适配某个领域或任务。属于“模型再训练”，但规模远小于预训练。

常见类型：
1. SFT（监督微调）：人工标注问答数据继续训练
1. 指令微调（Instruction Tuning）：让模型更好理解指令
1. 领域微调：金融 / 医疗 / 法律等垂直领域

### Low-Rank Adaptation（LoRA 微调）

LoRA 是一种参数高效微调（PEFT）方法。

特点：
1. 不修改原模型权重
1. 只新增低秩矩阵
1. 训练参数量极小
1. 显存占用低

适用于：

1. 私有模型定制
1. 低成本领域适配
1. 多任务切换（可加载不同 LoRA）

### Distillation（蒸馏）

用大模型训练小模型。

目标：
1. 降低模型体积
1. 提高推理速度
1. 降低部署成本

### Quantization（量化）

将模型参数从 FP16/FP32 压缩为 INT8/INT4 等。

目标：
1. 减少显存
1. 提高推理速度
1. 属于推理优化手段，不是训练。

### Alignment（对齐）

通过 SFT监督微调、RLHF人类反馈的强化学习 等方式让模型行为符合人类价值和使用习惯。

### Inference（LLM大语言模型推理）

LLM 是核心智能引擎，本质是基于上下文进行 token 概率预测的生成模型，负责理解、推理和生成文本，但不具备直接执行现实操作的能力。

### Prompt / CoT / ReAct（推理方式）

Prompt 是对模型的输入控制；Chain of Thought（链式思考）通过显式推理步骤提升复杂任务表现；ReAct 则把“思考 + 行动”结合，是 Agent 多步决策的基础模式。

### Function Call / Tools（外部能力调用）

Function Call 是模型输出结构化调用意图（通常为 JSON）的机制；Tools 是实际可执行的外部能力（API、数据库、代码执行等）。模型负责决策，系统负责执行。

### Skills（能力封装）

Skill 是对 Tool 与 Prompt 的组合封装，相当于高阶能力模块，使 Agent 调用更稳定、更可复用。

## Agent（智能体）

Agent 是具备目标、规划、决策、执行和反馈循环的系统。其核心是 LLM 推理 + 工具调用 + 状态管理，实现多步任务处理。多 Agent 协作需要角色分工，Planner、Executor、Reviewer。

### Workflow / DAG（工程编排）

Workflow 是固定流程调度；DAG 是有向无环图结构，支持复杂依赖与并行执行。它们属于工程控制层，而非模型智能层。

###  Context / Memory（记忆系统）

Context 是当前对话上下文（短期记忆）；Memory 是外部持久存储（长期记忆），用于跨会话状态保存与个性化。

### RAG / Embedding / 向量数据库（知识增强）

Embedding 将文本转为向量；向量数据库存储并检索相似语义；RAG（检索增强生成）在生成前先检索相关知识，减少幻觉并接入私有数据。

### MCP / OpenAPI / OpenSpec（能力标准化）

MCP 是模型上下文协议，用于统一能力接入；OpenAPI 是接口描述规范；OpenSpec 用于描述 AI 能力与工具结构，实现跨系统复用。
