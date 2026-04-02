# 第 1 章：先跑起来再说

> **段位门槛：所有人** | ⏱️ 30 分钟 | 原则：所有理论都不如亲眼看一次 Agent 自己干活

## 1.1 安装（5 分钟）

```bash
# 克隆教学版
git clone https://github.com/Windy3f3f3f3f/claude-code-from-scratch.git
cd claude-code-from-scratch
npm install && npm run build
```

### 配置 API Key

支持两种后端，选一种即可：

<!-- tabs:start -->

#### **🔵 Anthropic（推荐）**

```bash
export ANTHROPIC_API_KEY="sk-ant-xxx"
# 可选：使用代理
export ANTHROPIC_BASE_URL="https://your-proxy.com"
```

#### **🟢 OpenAI 兼容**

```bash
export OPENAI_API_KEY="sk-xxx"
export OPENAI_BASE_URL="https://api.openai.com/v1"
```

<!-- tabs:end -->

## 1.2 第一次对话（10 分钟）

```bash
npm start
```

**依次对 Agent 说以下指令，仔细观察它的行为：**

### 实验 1：看 Agent 自主选择工具

```
👤 你：帮我看看当前目录有什么文件
```

🔍 **观察：** Agent **自己**选择了 `list_files` 工具。你没告诉它用哪个——它根据你的自然语言自己判断的。

### 实验 2：看工具执行结果

```
👤 你：读一下 package.json 的内容
```

🔍 **观察：** Agent 选了 `read_file`，返回的文件内容带行号。这些行号不是文件里的，是工具自动加的——方便 Agent 后续做精确编辑。

### 实验 3：看安全确认机制

```
👤 你：在当前目录创建一个 hello.txt，写入 "hello world"
```

⚠️ **观察：** 弹出了确认提示！因为这是一个**新文件**。Agent 不会偷偷创建你不知道的文件。

### 实验 4：看精确编辑

```
👤 你：帮我把 hello.txt 里的 world 改成 agent
```

🔍 **观察：** Agent 用了 `edit_file`，通过精确字符串匹配替换。不是重写整个文件，而是只改变化的部分。

### 实验 5：看命令执行

```
👤 你：运行 cat hello.txt 看看结果
```

🔍 **观察：** Agent 用了 `run_shell` 执行系统命令，并展示结果。

### 实验 6：看危险命令拦截

```
👤 你：帮我删除 hello.txt
```

⚠️ **观察：** 又弹确认了！因为 `rm` 被代码标记为**危险命令**。

## 1.3 感受上下文增长（5 分钟）

持续和 Agent 对话 10+ 轮，让它读几个大文件。然后：

```
👤 你：/cost
```

查看 token 消耗数据。你会发现 **input token 越来越多**——因为每轮都重发全部历史。

继续聊，直到看到类似的提示：

```
ℹ Context window filling up, compacting conversation...
ℹ Conversation compacted.
```

这就是自动压缩触发了。

## 1.4 你刚刚学到了什么？

🎯 **恭喜！** 你用 30 分钟亲身体验了 Agent 产品的三个核心问题：

| 你看到的现象 | 背后的产品问题 |
|---|---|
| Agent 自己选择工具和执行步骤 | → 需要设计**循环边界**（最大步数、预算） |
| 危险操作弹出确认 | → 需要设计**信任机制**（安全分级） |
| Token 持续增长 → 触发压缩 | → 需要设计**成本模型**和**压缩策略** |

**接下来的章节，我们就来逐一深入理解这些问题。**
