> 来源：[飞书原文](https://zcnfzozvzo07.feishu.cn/wiki/H103wWPNmimD1zkphxrc4iQNnmd)
> 同步日期：2026-09-05

# 02｜AI 工程最低编程基础

> 💡 **提示**
> 目标不是成为 Python 专家，而是遇到“请求失败、数据不对、程序卡住、结果变空、换机器不能运行”时，你能自己定位问题。

# 本章解决什么问题

AI Coding Agent 可以快速写出代码，但它不能替你承担理解、验证和维护责任。本章建立 AI 应用工程的地基：Python 最小语法、CLI、HTTP、JSON、环境变量、异常、日志、数据库基本概念和 Git。

# 最小必要知识

| 主题 | 你必须会什么 | 日常类比 |
| --- | --- | --- |
| Python | 变量、函数、列表、字典、模块、文件读写、虚拟环境 | 像把手工步骤写成可重复的工作说明书 |
| CLI | 进入目录、创建环境、运行模块、查看帮助、读取退出码 | 像用文字指挥一个没有图形界面的同事 |
| HTTP / REST | 理解 URL、方法、状态码、请求头、请求体、超时 | 像寄件地址、寄件动作、回执和包裹内容 |
| JSON | 读写对象、数组、字符串、数字、布尔值和空值 | 像机器和人约定好的表格格式 |
| 环境变量 | 把密钥、地址和环境差异放在代码外，不提交到 Git | 像把钥匙放进抽屉，不把钥匙刻在说明书上 |
| 异常与日志 | 区分用户错误、网络错误、服务错误和程序错误；记录可检索上下文 | 像医院病历：只说“坏了”无法帮助下一位医生 |
| 数据库 | 理解表、记录、主键、查询、事务和持久化 | 像有规则的档案柜，不是随手堆文件夹 |
| Git | 提交、查看 diff、分支、回退到已知版本、写清提交信息 | 像给项目保留可比较的存档点 |

# 极简案例：API 健康检查 CLI

需求：输入一个 API 地址，输出请求是否成功、耗时、状态码和错误原因。这个小工具会同时练习 CLI、HTTP、JSON、异常和日志。

```python
import json
import logging
import os
import sys
import time
from urllib.request import Request, urlopen

logging.basicConfig(level=os.getenv("LOG_LEVEL", "INFO"))

def check(url):
    started = time.perf_counter()
    try:
        request = Request(url, headers={"Accept": "application/json"})
        with urlopen(request, timeout=10) as response:
            body = response.read().decode("utf-8")
        elapsed_ms = round((time.perf_counter() - started) * 1000)
        return {"ok": True, "status": response.status, "elapsed_ms": elapsed_ms,
                "json": json.loads(body)}
    except Exception as exc:
        logging.exception("health check failed")
        return {"ok": False, "error_type": type(exc).__name__,
                "message": str(exc)}

if __name__ == "__main__":
    target = sys.argv[1] if len(sys.argv) > 1 else os.getenv("CHECK_URL")
    if not target:
        raise SystemExit("usage: python healthcheck.py URL")
    print(json.dumps(check(target), ensure_ascii=False, indent=2))
```

# 跟着做

1. 创建项目目录和虚拟环境，确认 python --version、python -m venv .venv 和激活命令能工作。
2. 运行本地或可信测试地址，观察成功、超时、无效 JSON 和不存在地址四种结果。
3. 用 Git 提交第一个可运行版本，再修改一处输出字段并查看 diff。
4. 把 URL 从命令行参数改成“参数优先、环境变量兜底”，不要把密钥写进代码。
5. 为结果加 request_id、开始时间、结束时间和错误类型；不要在日志里输出密钥或完整隐私数据。

# AI Coding 学习方法

先让 AI 只解释运行环境和文件结构，再让它实现一个最小函数。每次请求都指定“不引入框架、不改无关文件、给出测试命令”。AI 生成后，你必须自己回答：哪个函数负责网络？异常在哪里被捕获？返回 JSON 的 schema 是什么？退出码何时非零？

# 自己修改

- 增加 timeout 参数，并说明参数校验失败时的退出码。
- 把网络调用与结果格式化拆成两个函数，写一个不访问网络的单元测试。
- 增加脱敏规则，确保日志中不会出现 Authorization、API key 或用户原文。
- 用一次 Git 分支完成修改，合并前阅读完整 diff。

# Debug 挑战

| 现象 | 先查什么 |
| --- | --- |
| 程序提示找不到命令 | 当前目录、虚拟环境、解释器路径和文件名 |
| 返回 401 / 403 | 身份、权限、请求头、环境变量是否加载；不要只重试 |
| 返回 200 但解析失败 | 响应 Content-Type、原文形状和实际 JSON 层级 |
| 偶尔超时 | 网络、服务端延迟、客户端 timeout、重试次数和重复副作用 |
| 换机器不能运行 | Python 版本、依赖、环境变量、路径和 README 步骤 |

# 小项目：可复现的 API 调试包

交付一个包含 CLI、README、环境变量示例、脱敏日志、至少 5 个测试场景和 Git 历史的项目。README 必须写清：如何安装、如何运行、如何模拟失败、如何判断成功、哪些信息不能提交。

# 验收标准

- 能不用教程创建虚拟环境、运行模块、读取环境变量并提交 Git。
- 能解释 HTTP 状态码、请求头、请求体、JSON 解析和 timeout 的关系。
- 能根据日志和响应原文定位一次失败，而不是让 AI 猜。
- 能写一个不依赖网络的测试，并说明它验证了哪条行为。
- 能说出数据库何时比文件更合适，何时不值得引入。

# 自测与常见误区

- 为什么 API key 不能写进代码或提交 Git？因为代码会被复制、日志会被收集、历史提交难以彻底清除。
- 为什么“状态码 200”不等于业务成功？还要检查响应形状、字段、语义和下游校验。
- 为什么重试不是万能修复？重复写操作可能造成重复订单、重复消息或成本增加。
- 常见误区：把所有异常吞掉；日志只写“失败”；用浏览器能打开证明 API 正常；依赖无限增加；不保留可回退的 Git 提交。

# 下一步

进入 03，用 Python 调用模型 API，做出第一个可演示的文档分析应用。
