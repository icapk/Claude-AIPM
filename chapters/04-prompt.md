# 第 4 章：提示词工程——产品人格与动态上下文

> **段位目标：L3+** | ⏱️ 30 分钟 | 文件：`prompt.ts` + `system-prompt.md`

## 4.1 提示词不是一段静态文字

打开 `src/prompt.ts`，你会发现 System Prompt 是**动态拼装**的：

```typescript
return template
  .replace("{{cwd}}", process.cwd())      // 当前工作目录
  .replace("{{date}}", date)               // 今天日期
  .replace("{{platform}}", platform)       // 操作系统
  .replace("{{shell}}", shell)             // Shell 类型
  .replace("{{git_context}}", gitContext)   // Git 分支/状态
  .replace("{{claude_md}}", claudeMd);     // 用户自定义指令
```

## 4.2 三层架构

| 层级 | 内容 | 产品含义 | 谁控制 |
|---|---|---|---|
| **① 静态基座** | 角色定义、行为规则、工具使用指南 | 产品人格与基线 | 产品经理 |
| **② 环境注入** | 日期、OS、Shell、Git 状态 | Agent 感知当前环境 | 系统自动 |
| **③ 用户自定义** | CLAUDE.md 文件内容 | 用户教 Agent 规则 | 用户 |

## 4.3 CLAUDE.md：让用户教 Agent

这是一个非常聪明的产品设计：

用户在项目根目录放一个 `CLAUDE.md` 文件，Agent 启动时**自动读取**并遵守里面的指令。

教学版的加载逻辑：

```typescript
// 从当前目录往上遍历，找到所有 CLAUDE.md
export function loadClaudeMd(): string {
  let dir = process.cwd();
  while (true) {
    const file = join(dir, "CLAUDE.md");
    if (existsSync(file)) {
      parts.unshift(readFileSync(file, "utf-8"));
    }
    const parent = resolve(dir, "..");
    if (parent === dir) break;  // 到根目录了
    dir = parent;
  }
  return parts.join("\n\n---\n\n");
}
```

**产品设计参考：** 你的 AI 产品也可以给用户类似的机制——让用户定义规则，而不是你预设所有行为。

> 例如：客服 Agent 的 `RULES.md`、写作 Agent 的 `STYLE.md`、代码审查 Agent 的 `REVIEW_STANDARDS.md`

## 4.4 Git 上下文注入

Agent 启动时自动读取 Git 信息：

```typescript
export function getGitContext(): string {
  const branch = execSync("git rev-parse --abbrev-ref HEAD");
  const log = execSync("git log --oneline -5");
  const status = execSync("git status --short");
  return `Git branch: ${branch}\nRecent commits:\n${log}\nStatus:\n${status}`;
}
```

**产品含义：** Agent 看到 Git 状态后，能做出更好的判断——比如知道当前在 feature 分支，就不会直接往 main 提交。

## 4.5 🧪 动手实验：用 CLAUDE.md 改变行为

在教学版项目目录创建 `CLAUDE.md`：

```markdown
# 项目规则
- 所有代码必须使用中文注释
- 每次修改文件前，先告诉我你要改什么、为什么改，等我确认后再操作
- 如果不确定，先问我，不要猜测
- 回复时使用简洁的技术语言，不要太啰嗦
```

重启 Agent (`npm start`)，然后让它帮你修改一个文件——观察：
- 它是否遵守了"先解释再修改"的规则？
- 代码注释是否变成了中文？
- 回复风格是否更简洁了？

> 🎯 **PM 认知：** CLAUDE.md 的力量在于——你把「适配用户需求」的成本从自己转移到了用户。但前提是你要设计好：规则格式是什么？生效范围多大？优先级如何？冲突怎么解决？

## 4.6 产品设计建议

设计 AI 产品的提示词时，问自己：

| 问题 | 设计决策 |
|---|---|
| 哪些行为是产品基线？ | → 硬编码到 System Prompt |
| 哪些需要感知环境？ | → 动态注入（日期、上下文、用户状态） |
| 哪些让用户自定义？ | → 提供 CLAUDE.md 式的机制 |
| 提示词变化会影响什么？ | → 可能影响 API 缓存（成本）|
