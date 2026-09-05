> 飞书原文：[06｜Agent Engineering](https://zcnfzozvzo07.feishu.cn/wiki/XLXVwMlDgiPiMik7KCmcVf6gnde)
> GitHub 教学版：面向零基础学习者的详细讲解

# 06｜Agent Engineering

> Agent 不是“Prompt 写得更长”，而是一个有状态、会选择下一步、可以调用工具并在条件满足时停止的系统。

## 先理解 Agent Loop

最小 Agent Loop 可以写成：

```text
读取目标和当前状态
        ↓
决定下一步：直接回答 / 调工具 / 请求补充
        ↓
执行动作并记录结果
        ↓
检查是否完成、失败或达到上限
        ├─ 完成 → 输出结果
        ├─ 失败 → 重试、降级或停止
        └─ 未完成 → 回到“决定下一步”
```

类比一个办事助理：他先看任务清单，再决定打电话、查资料还是问你；每做完一步都要更新清单，不能无限绕圈。

## State 为什么重要

没有状态，系统只能靠一长串对话猜自己做过什么；有状态，程序可以明确保存：

```json
{
  "goal": "生成带引用的学习总结",
  "messages": [],
  "retrieved_chunks": [],
  "tool_results": [],
  "attempt": 2,
  "needs_human_confirmation": false,
  "status": "running"
}
```

状态字段要有清晰含义。不要把所有东西都塞进一个 `context` 字符串，否则后面无法判断哪个字段过期、哪个结果可信。

## Stop Condition：什么时候必须停

至少设置这些停止条件：

- 已经得到满足验收标准的结果；
- 连续失败达到上限；
- 循环次数达到上限；
- 工具权限不足或返回未知状态；
- 需要用户确认；
- 发现目标超出系统能力。

“让它自己决定什么时候结束”不是停止条件。你可以让模型提出完成判断，但程序必须有硬上限。

## Memory 的两种含义

短期记忆是当前任务中的状态，例如刚刚检索到的三个片段；长期记忆是跨任务保存的用户偏好、历史记录或总结。

初学者常把所有对话永久保存，这是隐私和成本风险。先问三个问题：保存什么、保存多久、用户能不能查看和删除。没有必要保存的内容就不要保存。

## Retry、Fallback 和人工接管

一个可靠 Agent 不只会“再试一次”：

| 情况 | 合理处理 |
|-|-|
| 工具临时超时 | 有限重试，保留 request id |
| 参数格式错误 | 让模型修正一次，仍失败就停止 |
| 检索没有依据 | 明确拒答或请用户补资料 |
| 写入动作结果未知 | 查询状态，不重复写入 |
| 高风险动作 | 转人工确认 |

降级不是失败。不能完成时给出清楚的下一步，通常比编造一个完整答案更专业。

## 什么时候不应该用 Agent

如果任务有固定的 5 个步骤，每个分支都能提前画出来，就先用 Workflow。Agent 的价值在于路径确实需要根据中间结果动态选择，而不是为了显得先进。

判断问题：

1. 下一步是否依赖刚刚得到的结果？
2. 可选动作是否超过一个？
3. 选择错误的代价是否可控？
4. 是否能设置明确的上限和人工接管？

如果答案大多是否定的，普通调用或 Workflow 更合适。

## 最小伪代码

```python
for step in range(MAX_STEPS):
    decision = model_decide(state)

    if decision.type == "answer":
        return validate_answer(decision.content)

    if decision.type == "tool_call":
        args = validate_args(decision.name, decision.args)
        result = run_tool(decision.name, args)
        state["tool_results"].append(result)
        continue

    if decision.type == "human_confirmation":
        return pause_for_user(state)

    return "无法识别下一步，已安全停止"

return "达到步骤上限，已停止"
```

这段代码的重点不是语法，而是：每轮都有可识别的决定、工具有验证、循环有上限、未知情况会停止。

## 本章动手项目

把 05 的知识库问答升级为“学习研究助手”：它可以检索资料、比较两篇内容、生成带引用的结论；如果引用不足，必须拒答；最多调用工具 5 次；不允许自动发送或删除任何外部数据。

记录三次运行：一次成功、一次检索不足、一次达到上限。说明每次系统为什么停。

## 本章验收

- 能画出 Agent Loop 和状态变化；
- 能写出至少五个停止条件；
- 能解释短期记忆和长期记忆的区别；
- 能处理超时、参数错误和未知写入结果；
- 能说明一个任务为什么不需要 Agent。

> 下一步：进入 07，学习 Skill、MCP 和外部系统连接的边界。
