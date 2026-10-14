# 第 2 章：快速上手——我的第一个工具 Agent

学习日期：2026-10-14  
补充完成：2026-10-15（offline Notebook）  
课程正文：[第 2 章：快速上手](https://datawhalechina.github.io/deepagents-in-action/chapters/ch02-quickstart/)

## 这一章学习什么

第 1 章证明了 Agent 能调用工具；第 2 章开始学习自己做出一个工具 Agent：

```text
定义工具函数
  ↓
把工具、模型和系统提示词交给 create_deep_agent()
  ↓
调用 agent.invoke() 运行任务
  ↓
检查工具是否真的执行，以及工具结果是否正确
```

本章由两份 Notebook 配套：

```text
notebooks/ch02/01-quickstart.ipynb
notebooks/ch02/02-research-assistant.ipynb
```

学习顺序：先完成 `01-quickstart.ipynb`，再学习研究助手。

## 验证什么

本次先验证一个 Deep Agent 能否正确调用自定义天气工具：

```text
用户问：北京今天天气怎么样？
        ↓
模型请求：get_weather(city="北京")
        ↓
Python 工具返回：It's always sunny in 北京!
        ↓
模型给出最终回复
```

接着验证 Agent 可以连续使用两个工具：先把美元换成人民币，再把结果乘以通胀系数。

## 核心组成

### 1. 工具函数

```python
def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"
```

普通 Python 函数可以成为 Agent 工具。工具定义有三个重要部分：

| 部分 | 本例 | 作用 |
|---|---|---|
| 参数类型标注 | `city: str` | 告诉 Agent 参数名称和应传入的类型 |
| Docstring | `Get weather for a given city.` | 告诉 Agent 这个工具能做什么、何时使用 |
| 默认值 | 例如 `to_currency="CNY"` | 表示这个参数可不传，工具会使用默认值 |

### 2. 创建 Deep Agent

```python
agent = create_deep_agent(
    model=model,
    tools=[get_weather],
    system_prompt="You are a helpful assistant.",
)
```

三个参数的作用：

```text
model：使用哪个聊天模型
tools：模型可以请求调用哪些自定义工具
system_prompt：这个 Agent 的长期工作规则和角色
```

Deep Agent 在自定义工具之外，还自带文件读写、搜索文件、执行和子任务委派等 Harness 能力。

### 3. 运行 Agent

```python
result = agent.invoke(
    {"messages": [{"role": "user", "content": "北京今天天气怎么样？"}]}
)

print(result["messages"][-1].content)
```

`invoke()` 启动的不是“一次模型回答”，而是一个模型—工具循环：模型决定要不要调用工具；工具结果回到消息历史；模型再决定下一步，直到它不再请求工具为止。

## 关键练习：计算器 Agent

这一节定义两个工具：

```python
def calculate(expression: str) -> float:
    """Evaluate a math expression and return the result."""

def convert_currency(
    amount: float,
    from_currency: str,
    to_currency: str = "CNY",
) -> dict:
    """Convert an amount from one currency to another."""
```

目标任务：

```text
100 USD
  ↓ convert_currency
720 CNY
  ↓ calculate：720 × 1.08
777.6
```

这里要观察的是：第二次工具调用必须发生在第一次工具返回之后，而且计算表达式需要用到实际换汇结果。

## 工具报错时的两种情况

| 情况 | 例子 | 结果 |
|---|---|---|
| 参数校验错误 | `amount="a lot"`，但参数要求数字 | 框架生成一条 `status="error"` 的 ToolMessage；真实模型可根据提示修正后重试 |
| 工具内部异常 | 查询没有配置的 `JPY` 汇率 | Python 函数抛出异常，整次 `invoke()` 停止 |

实际项目中，可预期的业务错误应该在工具内部转换为清楚的返回信息，避免直接中断整个 Agent。

## 研究助手是什么

第二份 Notebook `02-research-assistant.ipynb` 会进一步组合：

```text
网络搜索 → 待办规划 → 虚拟文件保存资料 → 必要时委派子 Agent → 综合成报告
```

初学阶段先不配置真实 API。课程 Notebook 的 offline 模式会使用脚本模型和本地搜索样例，能观察 Agent 与工具的流程，但不发送真实模型或搜索请求。

要运行真正的联网研究助手，才需要：

```text
SILICONFLOW_API_KEY：模型调用
TAVILY_API_KEY：联网搜索
search 依赖组：Tavily 搜索库
```

## 实际结果与证据

当前状态：**已完成第 2 章 Notebook 的 offline 学习。**

本次使用的入口：

```text
notebooks/ch02/01-quickstart.ipynb
```

已确认的关键现象：

```text
1. Deep Agent 收到“北京天气”问题后，请求 get_weather(city="北京")；
2. 工具成功返回：It's always sunny in 北京!；
3. 工具请求与工具结果使用相同调用 ID 关联；
4. 100 USD 换算为 720 CNY 后，Agent 再计算 720 × 1.08，结果为 777.6；
5. amount="a lot" 会被类型校验拦截，形成 status="error" 的 ToolMessage；
6. JPY 不在汇率表中时，工具内部抛出 KeyError，整次 invoke() 停止。
```

这证明类型标注不只是给人看的注释：框架会用它校验模型传给工具的参数。也证明多工具任务的关键是前一个工具的真实输出，要被后一个工具调用使用。

学习中保留的观察重点：

```text
模型最终回答只是总结；
ToolMessage 的 status、参数和返回值才是工具正确执行的证据。
```

## 下一步修正

1. 运行 `notebooks/ch02/02-research-assistant.ipynb`，观察本地搜索样例、Todo 和虚拟文件如何协作；
2. 再创建 `deepagents/default` AgentSeek 模板项目；
3. 在生成项目中修改一项 `system_prompt` 和一个简单工具；
4. 配好模型环境变量后，跑通第一个独立的真实 Agent；
5. 将模板项目的创建命令、实际输出与问题补充到 Task0 和本笔记。
