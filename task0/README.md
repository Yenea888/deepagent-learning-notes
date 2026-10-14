# Task0：准备篇——认识并创建 AgentSeek 模板项目

学习日期：2026-10-14  
课程来源：[准备篇：用 AgentSeek 创建项目](https://datawhalechina.github.io/deepagents-in-action/chapters/pre01-agentseek-create/)

## 这一步要解决什么

前两章的 Notebook 是小实验：逐格运行代码，观察 Agent 的工作机制。

AgentSeek 模板则是一个可以继续开发的完整项目起点。它避免每次从空文件夹手动创建依赖、配置、启动脚本和 Agent 入口。

```text
Notebook：学习一个概念或机制
        ↓
AgentSeek 模板：生成一个可运行、可继续修改的 Agent 项目
```

## 什么是模板

本课程第 1、2 章使用同一个模板：

```text
deepagents/default
```

它会生成一个最小 Deep Agents 项目，核心仍然是：

```python
create_deep_agent(
    model=...,
    tools=[...],
    system_prompt="...",
)
```

但是模板把它放进完整工程结构里，便于配置、启动和后续修改。

## 模板会生成什么

生成项目通常包含：

```text
项目目录/
├── README.md                 # 生成项目的运行说明
├── pyproject.toml            # Python 依赖和项目配置
├── requirements.txt
├── .env.example              # 环境变量示例，不应填入或提交真实 Key
├── .agentseek/
│   └── lifecycle.toml        # AgentSeek 生命周期和服务配置
└── src/<项目名>/
    ├── demo_binding.py       # 创建并暴露 Deep Agent 的入口
    └── settings.py           # 模型等配置读取逻辑
```

默认模板没有前端；它先提供一个可在本地运行的 Deep Agent 工程骨架。

## 创建流程

在准备创建独立项目时，课程给出的主流程是：

```bash
# 1. 安装或更新 AgentSeek 命令行工具
uv tool install --upgrade agentseek

# 2. 用默认 Deep Agents 模板创建新项目
agentseek create deepagents/default --checkout main --no-input
```

创建完成后，进入生成的项目目录，依次查看项目提供的任务、按 README 安装依赖并配置 `.env`，然后检查和启动：

```bash
agentseek task --list

# 完成依赖安装和 .env 配置后再运行
agentseek doctor
agentseek dev
```

这些命令的含义：

| 命令 | 用途 |
|---|---|
| `agentseek create` | 用模板生成新项目 |
| `agentseek task --list` | 查看这个项目定义了哪些安装、启动或检查任务 |
| `agentseek doctor` | 检查环境、依赖、配置是否满足启动条件 |
| `agentseek dev` | 启动项目的本地开发服务 |

## 和第 1、2 章的关系

| 课程内容 | 你要做的事 | 使用的模板 |
|---|---|---|
| 第 1 章 | 理解 Agent 如何调用工具 | `deepagents/default` |
| 第 2 章 | 修改提示词和工具，跑通第一个 Agent | `deepagents/default` |

因此，模板不是另一个 Agent 框架。它只是把 Deep Agents 的示例变成一个可维护项目的工程化起点。

## 本次实际结果

当前状态：**已理解模板的作用，尚未创建独立模板项目。**

已完成的学习环境是课程 Notebook 环境：

```text
deepagents-in-action-main/notebooks/.venv/bin/python
```

第 1 章已在 offline 模式通过。AgentSeek 模板通常需要按生成项目 README 配置模型环境变量，才能进行真实模型调用；它和离线 Notebook 实验是两条不同的学习路径。

## 验证证据：创建项目后应检查什么

真正创建模板项目后，记录以下证据：

```text
1. agentseek create 的成功输出和生成目录名；
2. agentseek task --list 的任务列表；
3. agentseek doctor 的检查结果；
4. 实际修改的 system_prompt 和工具函数；
5. agentseek dev 启动结果；
6. 发出一次 Agent 请求后，工具调用与最终回复的输出。
```

## 下一步

1. 先完成第 2 章 `notebooks/ch02/01-quickstart.ipynb` 的 offline 实验；
2. 确认能解释 `model`、`tools`、`system_prompt`、`invoke()` 分别做什么；
3. 再创建 `deepagents/default` 模板项目；
4. 在生成项目中只改一项提示词和一个简单工具，跑通第一个真实 Agent；
5. 将实际命令与输出补充回本笔记。
