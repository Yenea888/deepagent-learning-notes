# Task1：我的第一个工具 Agent

学习日期：2026-10-14  
课程：`deepagents-in-action` 第 1 章；第 2 章准备开始

## 任务目标

跑通一个最小工具 Agent，确认它会经历以下过程：用户提出请求 → 模型请求调用工具 → Python 工具实际执行 → 工具结果返回给模型。

## 验证什么

验证离线模式下，LangChain 创建的 Agent 能否正确调用教学工具 `echo`，并获得预期返回值：

```text
echo: hello deepagents
```

同时对比普通 Agent 和 Deep Agent：两者都能调用 `echo`，但 Deep Agent 默认额外提供文件、搜索、执行和任务委派等工具。

## 改了什么

本次没有改课程源码，只完成环境配置并逐格运行课程 Notebook：

```text
notebooks/ch01/01-agent-harness.ipynb
```

为在 PyCharm 中运行 Notebook，使用课程虚拟环境：

```text
notebooks/.venv/bin/python
```

## 关键代码

### 1. 准备离线脚本模型

```python
ECHO_TEXT = "hello deepagents"

offline_responses = [
    AIMessage(content="", tool_calls=[{
        "name": "echo", "args": {"text": ECHO_TEXT}, "id": "ch01-echo",
    }]),
    AIMessage(content="工具调用流程结束；实际返回值见 ToolMessage。"),
]

model = create_model(ScriptedChatModel(responses=offline_responses))
```

`ScriptedChatModel` 是按预设台词回应的模拟模型：它不联网、不使用 API Key，但会让 Agent 框架真实执行 Python 工具。

### 2. 定义工具

```python
@tool
def echo(text: str) -> str:
    """原样返回传入的文本，用于演示 Agent 的工具调用。"""
    return f"echo: {text}"
```

### 3. 创建并运行普通 Agent

```python
langchain_agent = create_agent(
    model=model,
    tools=[echo],
    system_prompt="你是一个简洁的助手。需要工具时优先调用工具，再用一句话总结结果。",
)

question = f"请调用 echo 工具，把文本 '{ECHO_TEXT}' 原样返回，然后告诉我工具返回了什么。"
lc_result = langchain_agent.invoke(
    {"messages": [{"role": "user", "content": question}]}
)
```

## 实际结果

自动运行检查通过：

```text
ch01-agent-harness: passed (offline)
```

在 PyCharm 的第一个 Notebook 代码格中，课程环境也正常启动：

```text
运行模式：offline（脚本模型）
Python：3.12.13 平台：Darwin arm64
deepagents==0.7.22
langchain==1.4.3
langchain-core==1.6.6
langgraph==1.2.13
langchain-openai==1.6.7
```

预期的 Agent 消息链为：

```text
HumanMessage：请调用 echo 工具
AIMessage：请求 echo(text="hello deepagents")
ToolMessage：echo: hello deepagents
AIMessage：工具调用流程结束
```

## 证据与解释

- `Tool Calls: echo` 只表示模型提出了工具调用申请。
- `ToolMessage` 中的 `echo: hello deepagents` 才是 Python 函数实际运行后的证据。
- 工具调用 ID `ch01-echo` 将模型的调用申请和对应工具结果关联起来。
- `create_agent()` 组装普通 Agent；其底层运行流程可看到 `__start__ → model → tools → __end__`。
- `create_deep_agent()` 除 `echo` 外，还默认具备 `read_file`、`write_file`、`grep`、`task` 等工具。这表示它“可用的工具更多”，不表示这些工具在本次任务中都被调用。
- 因此，模型最后说“完成”不能作为成功证据；应检查工具名称、参数、成功状态和 `ToolMessage` 返回值。

## 第 2 章：接下来学习什么

第 2 章会学习自己设计工具并交给 Deep Agent。先完成：

```text
notebooks/ch02/01-quickstart.ipynb
```

重点是：

1. 定义天气、计算和货币换算等自定义工具；
2. 理解工具的三个要素：参数类型标注、docstring、默认值；
3. 观察 Agent 连续调用多个工具；
4. 区分“参数校验错误”和“工具函数内部异常”。

## 下一步修正

1. 在 PyCharm 中从头逐格运行 `notebooks/ch01/01-agent-harness.ipynb`，保存包含 `ToolMessage` 的实际输出。
2. 打开并运行 `notebooks/ch02/01-quickstart.ipynb`，继续使用 offline 模式。
3. 完成第 2 章后，再决定是否创建 AgentSeek 的 `deepagents/default` 模板项目；模板用于做独立应用，不是当前 Notebook 学习的前置条件。
