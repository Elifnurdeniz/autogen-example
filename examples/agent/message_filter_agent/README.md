# MessageFilterAgent 智能体消息过滤示例

**难度等级** 🟢 入门

本章节介绍如何使用 AutoGen 的 MessageFilterAgent 对多智能体消息进行过滤和处理，实现信息提取与上下文控制。

**前置知识**
- AssistantAgent
- MessageFilterAgent
- RoundRobinGroupChat

## 运行方式
```bash
uv run -m examples.agent.message_filter_agent.main
```

## 概述
本示例包含两个智能体：`name_agent` 负责提取用户的名字，`age_agent` 负责提取用户的年龄。通过 `MessageFilterAgent` 包裹 `age_agent`，实现只在 `name_agent` 回复后才允许 `age_agent` 处理消息，保证信息流的顺序和上下文依赖。

## 系统架构
系统包含三个主要智能体：

### 核心组件
- **name_agent**：提取用户名字的智能体
- **age_agent**：提取用户年龄的智能体
- **filter_age_agent**：对 `age_agent` 进行消息过滤的智能体
- **RoundRobinGroupChat**：轮询团队协作，保证消息顺序

### 工具组件
- **MessageFilterConfig**：配置消息过滤规则
- **PerSourceFilter**：指定过滤来源和数量

## 工作流程
1. 用户输入：“我叫张伟，今年18岁”
2. `name_agent` 提取名字，输出：“张伟”
3. `filter_age_agent` 只在 `name_agent` 回复后处理消息，调用 `age_agent` 提取年龄
4. `age_agent` 由于只收到名字，无法提取年龄，输出提示信息

## 核心组件详解

### MessageFilterAgent 的作用
`MessageFilterAgent` 用于对包裹的智能体进行消息过滤，只允许特定来源和数量的消息被处理，适合多智能体协作场景下的信息流控制。

### 关键参数说明
- **per_source**：指定只处理来自 `name_agent` 的最后一条消息
- **termination_condition**：团队终止条件，确保流程不会死循环

### 团队协作机制
系统采用 `RoundRobinGroupChat`，智能体轮流处理消息，保证上下文一致性。

## 执行逻辑
```
用户输入 → name_agent 提取名字 → filter_age_agent 过滤并调用 age_agent → age_agent 输出结果
```

- **第一步**：用户输入个人信息
- **第二步**：name_agent 提取名字
- **第三步**：filter_age_agent 只允许 age_agent 处理 name_agent 的回复
- **第四步**：age_agent 由于缺乏年龄信息，输出无法提取年龄的提示

## 关键特性

- **消息过滤**：只处理特定来源的消息，保证上下文依赖
- **多智能体协作**：实现信息流顺序控制
- **可扩展性**：可根据需求扩展更多过滤规则和智能体角色

---

## 示例运行结果

```text
---------- TextMessage (user) ----------
我叫张伟，今年18岁
---------- ModelClientStreamingChunkEvent (name_agent) ----------
张伟
[Prompt tokens: 15, Completion tokens: 2]
---------- ModelClientStreamingChunkEvent (age_agent) ----------
很抱歉，我无法从“张伟”这个名字中提取出年龄信息。名字本身通常不包含年龄数据。  

如果您能提供更多相关信息（例如出生日期、身份证号、注册信息等），我可以帮助您计算或提取年龄。  

请提供更多上下文或数据，我会尽力协助！
[Prompt tokens: 10, Completion tokens: 63]
```

本示例展示了如何通过消息过滤机制实现多智能体协作与信息流控制，为复杂任务编排提供了基础模板。
