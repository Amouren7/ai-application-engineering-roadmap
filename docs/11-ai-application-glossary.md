> 飞书原文：[11｜AI 应用工程词典](https://zcnfzozvzo07.feishu.cn/wiki/ZArqwdVqiiNbPVkdEzgcCUHJnGf)
> GitHub 教学版：面向零基础学习者的详细讲解

# 11｜AI 应用工程词典

> 词典不是让你背定义，而是帮助你在遇到问题时快速定位：这个词在系统哪一层、它解决什么问题、它不解决什么问题。

## LLM / Token / Context

- **LLM**：处理语言的模型。能生成合理文本，不等于事实永远正确。
- **Token**：模型处理文本的计量单位，影响容量、延迟和成本。不是“一个汉字一定等于一个 Token”。
- **Context**：本次调用真正提供给模型的信息。模型没看到的内容不能指望它稳定使用。

## Prompt / Structured Output / Embedding

- **Prompt**：任务、角色、限制和输出要求的组合。不是越长越好，而是要减少歧义。
- **Structured Output**：让结果符合字段和类型，方便程序继续处理。仍然要做业务校验。
- **Embedding**：把文字表示成可比较的向量，常用于语义检索。它不负责判断资料真假。

## Retrieval / RAG / Citation

- **Retrieval**：从外部资料中找相关内容。
- **RAG**：把检索结果放进上下文，再让模型生成答案。
- **Citation**：告诉用户答案依据来自哪里，便于复核。引用错了仍然是错误。

## Tool / Workflow / Agent

- **Tool**：程序提供给模型调用的一项具体动作。
- **Workflow**：步骤和分支预先设计好，适合稳定流程。
- **Agent**：根据状态动态选择下一步的循环系统，必须有工具边界和停止条件。

## Memory / State / Trace

- **State**：当前任务的结构化状态，例如已检索片段和重试次数。
- **Memory**：跨步骤或跨任务保存的信息；保存前必须考虑隐私、期限和删除。
- **Trace**：一次请求经过各步骤的记录，用于定位延迟和错误。

## Evaluation / Guardrail / Fallback

- **Evaluation**：用固定样例和指标判断修改是否真的变好。
- **Guardrail**：可执行的边界，如工具白名单、金额上限和人工确认。
- **Fallback**：主路径失败后的安全替代路径，如拒答、重试、转人工。

遇到一个新名词时，先问三句话：它在哪一层？解决哪个具体问题？如果不用它，最简单的替代方案是什么？

## 小测验

“用户问公司制度，系统需要找到原文并附引用”至少涉及哪些词？

参考：Context、RAG、Retrieval、Citation，可能还涉及 Embedding 和 Evaluation；不一定需要 Agent。

> 下一步：遇到具体故障时，进入 12 按现象排查。
