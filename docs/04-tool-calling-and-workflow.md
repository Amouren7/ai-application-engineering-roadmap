> 飞书原文：[04｜Tool Calling 与 Workflow](https://zcnfzozvzo07.feishu.cn/wiki/KHTywpgPkivbUckmaTGcaVAjnMc)
> GitHub 教学版：面向零基础学习者的详细讲解

# 04｜Tool Calling 与 Workflow

> 本章解决一个关键问题：模型可以提出“下一步想做什么”，但怎样保证真正执行时参数正确、步骤可控、危险动作有人确认？

## 先看一个不能交给模型的动作

假设系统有一个函数：`refund_order(order_id, amount)`。模型可以根据用户话语提取订单号，但不能直接决定退款金额并立即调用。程序至少要检查：订单存在吗、金额是否超过原订单、是否符合规则、用户是否确认。

模型擅长理解自然语言，程序擅长执行确定规则。Tool Calling 是两者之间的接口，不是安全许可。

## Tool 是什么

一个工具至少要有四部分：名称、用途、参数 schema、执行函数。

```json
{
  "name": "get_order_status",
  "description": "查询订单当前状态，只读，不会修改订单",
  "parameters": {
    "type": "object",
    "properties": {
      "order_id": {"type": "string"}
    },
    "required": ["order_id"]
  }
}
```

description 不是装饰，它帮助模型选择工具；schema 也不是验证的全部，程序仍然要再次验证。

## 一次 Tool Calling 的完整过程

```text
用户：查一下订单 A100 的状态
  ↓
模型：请求调用 get_order_status(order_id="A100")
  ↓
程序：检查工具名和参数，调用订单系统
  ↓
订单系统：返回 shipped
  ↓
程序：把结果交给模型或直接生成固定说明
  ↓
用户：订单已发货
```

注意：模型并没有“访问订单系统”。它只输出了一个结构化意图，真正访问系统的是你的程序凭据。

## 用 Workflow 把步骤固定下来

退款助手可以写成：

```text
提取订单号
  ↓失败 → 请用户补充订单号
查询订单
  ↓不存在 → 告诉用户无法找到
检查退款规则
  ↓不满足 → 解释原因
展示退款金额和影响
  ↓用户未确认 → 停止
执行退款
  ↓失败 → 记录错误并提供人工入口
```

这就是 Workflow：每一步的输入、输出、分支和停止条件都能写出来。越是涉及金钱、权限、删除和对外发送，越应该明确设计。

## 设计工具时的四个问题

1. 这个工具是只读还是会产生修改？
2. 参数能否在程序里验证？
3. 调用失败时返回什么安全信息？
4. 哪些情况下必须先让人确认？

把只读工具和写入工具分开命名，例如 `get_invoice` 与 `submit_invoice`，不要用一个模糊的 `handle_invoice` 包含所有动作。

## 重试不是无限重试

网络超时可能适合重试，参数错误不适合重试，重复扣款绝对不能盲目重试。可以采用这个判断：

| 错误 | 是否重试 | 处理 |
|-|-|-|
| 临时网络超时 | 有限重试 1–2 次 | 记录 request id |
| 401 | 不重试 | 检查凭据 |
| 400 参数错误 | 不重试 | 修正参数或请求用户补充 |
| 429 限流 | 延迟后有限重试 | 遵守服务端提示 |
| 写入动作响应未知 | 不自动再次写入 | 先查询结果是否已发生 |

## 小练习：把“万能 Agent”改成 Workflow

原需求：“做一个能帮我管理所有工作的 Agent。”

请把它改写成一个单一任务，例如“每天早上读取日历和待办，生成今日摘要；发送前只展示预览，不自动发消息”。然后写出：输入、步骤、工具、分支、停止条件和人工确认点。

## 最小伪代码

```python
state = {"order_id": None, "status": None, "confirmed": False}

state["order_id"] = extract_order_id(user_text)
if not state["order_id"]:
    return "请补充订单号"

state["status"] = get_order_status(state["order_id"])
if state["status"] != "eligible_for_refund":
    return "当前订单不满足退款条件"

show_preview(state)
if not user_confirmed():
    return "已停止，未执行退款"

return refund_order(state["order_id"])
```

## 本章验收

- 能写出一个工具的名称、用途和参数 schema；
- 能区分模型请求工具和程序实际执行工具；
- 能画出至少一个包含失败分支的 Workflow；
- 能指出一个必须人工确认的动作；
- 能解释为什么“重试”必须有条件和上限。

> 下一步：进入 05，学习如何让回答有知识依据和可追溯引用。
