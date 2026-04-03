# 第 3 章：工具系统——你定义 Agent 的能力边界

> **段位目标：L3+** | ⏱️ 40 分钟 | 文件：`tools.ts`

## 3.1 核心认知：能力 = 工具集

LLM 本身**什么都做不了**。它不能读文件、不能运行命令、不能搜索代码——这些能力全部来自你给它的**工具**。

教学版给了 Agent 6 个工具：

| 工具名 | 能力 | 产品含义 |
|---|---|---|
| `read_file` | 读文件 | Agent 能"看到"代码 |
| `write_file` | 写文件 | Agent 能"创建"新内容 |
| `edit_file` | 编辑文件 | Agent 能"修改"已有代码 |
| `list_files` | 搜索文件 | Agent 能"浏览"项目结构 |
| `grep_search` | 搜索内容 | Agent 能"查找"关键信息 |
| `run_shell` | 执行命令 | Agent 能"操作"系统 |

> 🎯 **关键认知：** 你不给 `run_shell`，Agent 就永远不会执行命令。你不给 `write_file`，Agent 就只能看不能改。**工具集 = 产品能力边界。**

## 3.2 工具是 JSON Schema 描述

模型不看你的 TypeScript 代码。它只看你给它的 JSON 描述：

```json
{
  "name": "edit_file",
  "description": "Edit a file by replacing an exact string match with new content. 
                  The old_string must match exactly (including whitespace and indentation).",
  "input_schema": {
    "type": "object",
    "properties": {
      "file_path": { "type": "string", "description": "The path to the file to edit" },
      "old_string": { "type": "string", "description": "The exact string to find and replace" },
      "new_string": { "type": "string", "description": "The string to replace it with" }
    },
    "required": ["file_path", "old_string", "new_string"]
  }
}
```

模型看到这个描述后：
1. 理解这个工具叫 `edit_file`，用于编辑文件
2. 知道需要提供 `file_path`、`old_string`、`new_string` 三个参数
3. **自己决定**什么时候用它、参数填什么

## 3.3 工具描述 > 工具数量

| 产品决策 | 为什么重要 |
|---|---|
| 给 Agent 哪些工具？ | 决定 Agent 的能力边界 |
| 工具描述怎么写？ | **直接影响模型选对工具的概率** |
| 工具太多怎么办？ | 模型会选错——需要分组|
| 工具结果太大怎么办？ | 占满上下文——需要截断 |

### 官方 Claude Code 的做法

官方有 42+ 工具，但不是全部塞给模型。它用了一个叫 `ToolSearchTool`（工具搜索工具）的机制——模型先告诉系统「我需要什么类型的工具」，系统再动态加载。

**产品启示：当你的 Agent 超过 15 个工具时，一定要考虑工具发现/分组机制。**

## 3.4 工具结果截断

工具返回的结果可能很大（比如读一个 10 万行的文件）。教学版做了简单截断：

```typescript
const MAX_RESULT_CHARS = 50000;

function truncateResult(result: string): string {
  if (result.length <= MAX_RESULT_CHARS) return result;
  // 保留头尾各一半
  const keepEach = Math.floor((MAX_RESULT_CHARS - 60) / 2);
  return result.slice(0, keepEach) +
    "\n\n[... truncated " + (result.length - keepEach * 2) + " chars ...]\n\n" +
    result.slice(-keepEach);
}
```

> 🎯 **PM 认知：** 工具结果如果不截断，会迅速填满上下文窗口。但截断也意味着丢信息。**截断阈值是产品决策**——太小模型看不够，太大浪费 token。

## 3.5 🧪 动手实验：改变工具描述

打开 `src/tools.ts`，找到 `grep_search` 的 description，修改为：

```
"Search for a pattern in files. 
 IMPORTANT: This is the preferred tool for finding code. 
 Always use this instead of reading files one by one."
```

重新构建并运行：

```bash
npm run build && npm start
```

然后对 Agent 说：`帮我找找这个项目里哪里用到了 readFileSync`

**对比修改前后：** Agent 是否更倾向于用 `grep_search` 而不是逐个 `read_file`？

> 🎯 **L4 能力：** 你刚刚通过改一行描述，改变了 Agent 的行为策略。这就是引擎 PM 的核心能力——不需要改代码，只改配置就能验证产品假设。
