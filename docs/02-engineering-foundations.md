> 飞书原文：[02｜AI 工程最低编程基础](https://zcnfzozvzo07.feishu.cn/wiki/H103wWPNmimD1zkphxrc4iQNnmd)
> GitHub 教学版：面向零基础学习者的详细讲解

# 02｜AI 工程最低编程基础

## 本章开始前：准备与产出

| 项目 | 说明 |
|-|-|
| Python | 需要；建议 Python 3.11 或更新版本 |
| API Key | 不需要；先用本地小程序练基础 |
| 安装软件 | 需要 Python、编辑器和 Git；Windows 使用 PowerShell |
| 能否只用网页工具 | 不建议；本章要练习真实命令行和文件操作 |
| 本章产出 | 一个能运行、能故意制造错误、能提交 Git 的文本统计小程序 |

本章不是让你学完整套 Python，而是补齐后面调用 API 必须用到的最小能力：文件、函数、字典、JSON、异常、日志和环境变量。每完成一个命令，都先确认终端显示了什么。

> 这一章不是让你成为计算机专家，而是让你具备“能运行、能修改、能定位问题”的最低能力。

## 你真正需要的基础是什么

AI 应用通常不是从零训练模型，而是把几个普通的软件模块接起来：读取输入、调用接口、解析 JSON、保存结果、处理异常、记录日志。只要其中一环不懂，出错时就只能反复重试。

## 1. 认识命令行

命令行就是用文字告诉电脑做什么。你不需要一开始记住所有命令，只先会这几类：

```text
pwd / Get-Location       我现在在哪个目录
ls / Get-ChildItem       这里有哪些文件
cd <目录>                进入目录
python --version         查看 Python 是否可用
git status               查看项目有没有未保存的修改
```

如果命令失败，先看三件事：当前目录对不对、命令是否拼错、报错是“找不到程序”还是“程序运行失败”。

## 2. Python 只学一条可用主线

先理解变量、列表、字典、函数和循环。AI 应用里最常见的数据形态是字典和列表：

```python
document = {
    "title": "学习笔记",
    "text": "这里是一段文章内容",
}

def make_request(doc):
    return {
        "instruction": "请总结下面的文章",
        "content": doc["text"],
    }

request = make_request(document)
print(request["content"])
```

类比：变量是贴了名字的盒子，列表是按顺序排好的盒子，字典是贴着不同标签的抽屉，函数是可以重复使用的小机器。

先不要追求写得很“高级”。能读懂数据从哪来、经过哪个函数、变成什么样，就已经在建立工程能力。

## 3. 虚拟环境和依赖

不同项目可能需要不同版本的库。虚拟环境像给每个项目准备一个独立工具箱，避免项目之间互相污染。

```text
python -m venv .venv
.
venv\Scripts\Activate.ps1       Windows PowerShell
python -m pip install openai
python -m pip freeze > requirements.txt
```

如果激活命令因系统策略失败，不要把问题误认为 Python 代码错误；先确认 Python 安装和 PowerShell 权限，再选择临时执行方式。

## 4. HTTP 和 JSON

HTTP 是程序之间发消息的规则。一次 API 调用通常包含：地址、方法、请求头、请求体、状态码和响应体。

JSON 是一种结构化文本：

```json
{
  "question": "什么是 RAG？",
  "language": "zh-CN",
  "need_citation": true
}
```

把它想成一张机器能读的表单。字段名写错、逗号漏掉、布尔值写成字符串，都可能导致调用失败或结果被误解。

## 5. 环境变量和密钥

API Key 是钥匙，不要写进代码、截图、公开 README 或聊天记录。程序通过环境变量读取：

```python
import os

api_key = os.environ["OPENAI_API_KEY"]
```

如果程序提示缺少变量，先检查变量是否存在，再检查当前终端和 IDE 是否使用了同一个环境。不要为了“先跑起来”把真实密钥直接粘进代码。

## 6. 异常、日志和状态码

把错误分成三类会更容易排查：

- **输入错误**：文件为空、字段缺失、格式不对；
- **服务错误**：401 未授权、429 限流、500 上游故障；
- **程序错误**：变量名写错、JSON 解析失败、路径不存在。

最小的异常处理应该告诉用户发生了什么，也给日志留下上下文：

```python
import logging

logging.basicConfig(level=logging.INFO)

try:
    result = call_model(request)
except TimeoutError:
    logging.exception("模型调用超时")
    print("服务响应较慢，请稍后重试")
```

不要用一个宽泛的 `except Exception: pass` 把所有错误吞掉。那会让程序表面不崩，实际却没有留下任何线索。

## 7. Git 是你的后悔药

Git 不只是上传 GitHub，它首先是一个“可回到过去的修改记录”。每天至少形成一次清晰提交：

```text
git init
git add .
git commit -m "feat: add document input"
git status
git log --oneline
```

提交信息要说明“改了什么”，不要只写“update”。在让 AI 改代码后，先看 diff，再提交；这样你能知道哪些改动是自己接受的。

## 小项目：做一个文本统计 CLI

目标：运行 `python count_words.py notes.txt`，输出字符数、行数和前 5 个高频词。

验收要求：

1. 文件不存在时给出清楚提示；
2. 空文件不会崩溃；
3. 结果可以重复运行；
4. 用 Git 提交至少两次，第二次专门修复一个你故意制造的错误。

## 常见卡点排查顺序

当代码不能运行时，不要马上问 AI“为什么”。先记录：执行的完整命令、当前目录、输入样例、完整报错、你刚改了什么。然后按“输入 → 代码 → 外部服务 → 输出”的顺序逐段排除。

## 本章验收

- 能进入项目目录并运行 Python 文件；
- 能读写一个字典和 JSON；
- 能解释环境变量为什么比硬编码安全；
- 能区分 401、429、500 和本地代码错误；
- 能阅读自己的 Git diff，并恢复到上一次提交。

## 进一步小白练习

## 手把手完成一个最小 Python 项目

下面的任务只做“读取文本并统计信息”，不要加入模型。这样你能先练习软件工程，而不是把所有问题混在一起。

### 第一步：准备目录

```text
mkdir ai-learning
cd ai-learning
python -m venv .venv
```

在 Windows PowerShell 中激活环境后，创建 `count_words.py` 和 `notes.txt`。如果你不懂目录，也可以把它理解成：先给这个项目准备一个独立抽屉，所有文件都放进这个抽屉。

### 第二步：先写最小功能

```python
from pathlib import Path
import sys

path = Path(sys.argv[1])
if not path.exists():
    print(f"找不到文件：{path}")
    raise SystemExit(1)

text = path.read_text(encoding="utf-8")
print("字符数：", len(text))
print("行数：", len(text.splitlines()))
```

先运行成功，再增加词频。每增加一项能力就运行一次，这叫小步迭代。一次写完所有功能，出错时就很难知道是哪一行造成的。

### 第三步：故意制造失败

把文件名改成不存在的名字，输入空文件，再用不同编码保存一份文件。分别记录错误现象。你会发现“异常处理”不是抽象理论，而是提前替用户解释电脑为什么做不到。

### 第四步：提交 Git

```text
git init
git add count_words.py notes.txt
git commit -m "feat: add text statistics cli"
```

然后添加空文件校验，再提交第二次。用 `git diff HEAD~1` 看第二次到底改了什么。

## 学习代码时的四个问题

看到一行陌生代码，不要马上要求 AI 改写。先问：它接收什么？返回什么？什么时候执行？失败会怎样？这四个问题比背语法更能帮助你读懂 AI 应用。

## 本节验收

请把项目交给一个没有看过代码的人运行。如果他只按照 README 就能完成安装、运行一个正常样例和一个失败样例，你的工程基础才算开始成形。

> 下一步：进入 03，把这些基础连接成第一个 LLM 应用。
