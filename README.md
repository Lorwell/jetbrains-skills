# JetBrains IDE Skill

适用于 JetBrains IDE 的 Claude Code 协作技能，支持 IntelliJ IDEA、WebStorm、Android Studio。

通过 IDE MCP 工具进行代码搜索、编译检查、符号分析、代码编辑、数据库操作等全栈协作。

## 安装

```bash
npx skills add shaco/jetbrains-skills
```

## 子智能体（Sub-agent）支持

Claude Code 的子智能体**不会自动继承**主智能体的 skill。如需在子智能体中也使用 JetBrains IDE MCP 工具，将 `agents/` 目录下的 agent 配置文件复制到对应位置：

```bash
# 用户级（所有项目生效）
cp agents/*.md ~/.claude/agents/

# 或项目级（仅当前项目生效）
cp agents/*.md .claude/agents/
```

安装后，Agent 工具即可使用以下自定义 agent 类型：

| Agent 类型 | 对应内置类型 | 用途 |
|-----------|-------------|------|
| `jetbrains-code-search` | Explore | 代码搜索与定位，使用 IDE 语义索引 |
| `jetbrains-general` | general-purpose | 复杂开发任务（分析、重构、编辑等） |

在 prompt 中指定 `subagent_type`:

```
subagent_type: "jetbrains-code-search"
subagent_type: "jetbrains-general"
```

> 也可在生成子智能体时直接在 prompt 中要求使用 JetBrains IDE MCP 工具，无需安装 agent 配置。详见 [SKILL.md](SKILL.md#与子智能体sub-agent协作)。

## 支持的 IDE

| IDE | MCP 服务器名 |
|-----|-------------|
| IntelliJ IDEA | `idea` |
| WebStorm | `webStorm` |
| Android Studio | `androidStudio` |

## 使用

**首次使用推荐先初始化项目**，自动检测 IDE 并写入配置：

```
/jetbrains:init
```

或直接说"初始化 jetbrains"、"配置 jetbrains 项目"。init 会自动完成：

1. 检测当前连接的 JetBrains IDE MCP 服务器
2. 在 `CLAUDE.md` 中写入 `jetbrains-ide: <值>` 声明
3. 写入「优先使用 JetBrains IDE MCP 工具」的协作规则

手动配置：

1. 在 JetBrains IDE 中自行配置 MCP 工具连接，参考官方文档：[JetBrains MCP Server](https://www.jetbrains.com/zh-cn/help/idea/mcp-server.html)
2. 在项目 `CLAUDE.md` 中声明 IDE 类型：`jetbrains-ide: idea`
3. 确保对应 IDE 的 MCP 插件已启动并连接
4. 在 Claude Code 中正常使用，skill 自动匹配对应 IDE 的工具

## 功能

- 代码搜索与定位（符号、文本、正则、文件名、glob）
- 编译问题排查（文件检查、构建触发）
- 代码理解（符号文档、PSI 树分析）
- 代码编辑（文本替换、语义重命名、创建文件、格式化）
- 数据库操作（表结构浏览、SQL 查询、数据预览）
- 项目浏览（目录树、模块、依赖）
- Inspection KTS（仅 IDEA）
