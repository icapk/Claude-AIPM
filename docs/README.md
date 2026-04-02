# AI 产品经理的 Agent 实战课

> **以 Claude Code 为教材，从「会用 AI」进阶到「会造 AI Agent 产品」**

## 这是什么？

这是一份专为 **AI 产品经理** 设计的 Agent 学习教程。

我们以开源项目 [claude-code-from-scratch](https://github.com/Windy3f3f3f3f/claude-code-from-scratch)（~1300 行 TypeScript）为教材，配合 Claude Code 官方源码（~51 万行）作为进阶参照，帮你建立关于 AI Agent 产品的完整认知。

## 为什么产品经理需要学这个？

大多数 AI PM 的知识停留在「调 API → 得结果」。但 Agent 产品的核心逻辑远不止于此。

**不理解引擎，就做不好产品决策：**

| 你以为的 | 真实情况 |
|---|---|
| 调一次 API 得一个结果 | Agent 自己循环调用 3-20 次 |
| 一条消息花 X 个 token | 每轮重发全部历史，累积增长 |
| 加个「安全审核」就行了 | Agent 产品的安全是防 AI 做错事，不是防黑客 |
| 用户量决定成本 | 对话长度才决定成本（非线性） |

## 学习路径

```
Day 1 (2h) → 段位认知 + 跑通 Agent
Day 2 (2h) → Agent 循环 + 工具系统
Day 3 (2h) → 提示词工程 + 成本模型
Day 4 (2h) → 安全设计 + 生产差距
Day 5 (2h) → 设计 Checklist + 实战练习
```

**5 天 × 2 小时 = 10 小时，从任何段位升级到 L4。**

## 参考资源

| 资源 | 链接 |
|---|---|
| 教学版源码 | [claude-code-from-scratch](https://github.com/Windy3f3f3f3f/claude-code-from-scratch) |
| 教学版在线教程 | [在线阅读](https://windy3f3f3f3f.github.io/claude-code-from-scratch/) |
| 官方源码深度解析 | [how-claude-code-works](https://github.com/Windy3f3f3f3f/how-claude-code-works) (33万字) |
