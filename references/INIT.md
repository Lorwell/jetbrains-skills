# JetBrains IDE 项目初始化参考（init）

将当前项目配置为 JetBrains IDE 协作模式：写入 IDE 声明 → 写入优先使用规则，引导智能体在每次会话中主动调用 `jetbrains` skill 并识别 IDE 类型，无需重复询问。

## 触发方式

用户说"初始化 jetbrains"、"jetbrains init"、"配置 jetbrains 项目"等即可触发。

## 执行流程

### 第一步 — 检测可用 IDE

检查当前上下文中哪些 JetBrains IDE MCP 服务器（`idea`、`webStorm`、`androidStudio`）处于连接状态。

- 若只有一个 IDE 连接 → 直接使用
- 若有多个 IDE 连接 → 询问用户选择当前项目使用的 IDE
- 若没有 IDE 连接 → 提示用户先启动 JetBrains IDE 并连接 MCP

### 第二步 — 写入 IDE 声明

在 `CLAUDE.md` 中写入（若已有则更新）：

```markdown
jetbrains-ide: <idea|webStorm|androidStudio>
```

### 第三步 — 写入协作规则

在 `CLAUDE.md` 中追加优先使用 JetBrains IDE MCP 的协作规则，并声明 `jetbrains` skill 应默认加载（若已有则跳过）：

```markdown
## JetBrains IDE MCP 协作

当进行代码搜索、符号查找、编译检查、重构等操作时，优先使用 JetBrains IDE MCP 工具，因为 IDE 具有完整的项目索引和语义理解能力（PSI 树、检查引擎），比纯文本工具更精准高效。仅在 JetBrains MCP 服务器未连接或对应功能不支持时，退回到通用文件工具。

在每次会话中主动调用 `jetbrains` skill，优先使用 IDE MCP 工具进行代码操作。
```

## 写入注意事项

- 若 `CLAUDE.md` 中已有 `jetbrains-ide:` 声明，检查当前连接状态是否匹配，不匹配则更新为当前连接的 IDE
- 若已有"JetBrains IDE MCP 协作"相关规则则不重复写入，避免内容冗余
- 使用 Edit 工具精确追加，保持现有内容不变

## 映射关系

| IDE | MCP 服务器名 | CLAUDE.md 声明值 |
|-----|-------------|-----------------|
| IntelliJ IDEA | `idea` | `idea` |
| WebStorm | `webStorm` | `webStorm` |
| Android Studio | `androidStudio` | `androidStudio` |
