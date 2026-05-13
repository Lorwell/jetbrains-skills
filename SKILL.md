---
name: jetbrains
description: 使用 JetBrains IDE MCP 工具进行代码搜索、编译检查、符号分析、重构等全栈协作 — 充分利用 IDE 的语义理解能力（索引、PSI、检查引擎）替代命令行操作
when_to_use: |
  当用户要求"初始化 jetbrains"/"配置 jetbrains 项目"时，执行 init 流程（无需 IDE MCP 连接也可运行）。
  其余场景仅当上下文中至少有一个 JetBrains IDE MCP 服务器（idea/webStorm/androidStudio）处于连接状态时启用。若未连接则忽略本 skill。
  - 需要搜索代码（按文本、正则、符号名、文件名、glob 模式）
  - 需要排查编译问题（检查文件错误/警告、触发构建）
  - 需要获取方法/类/字段的类型签名和文档（Quick Documentation）
  - 需要重构代码（重命名、格式化）
  - 需要在 IDE 中打开文件与用户同步视角
  - 需要浏览项目结构（目录树、模块、依赖）
  - 需要执行 SQL 查询或浏览数据库结构
---

# JetBrains IDE 协作

> **前置条件**：当前上下文中必须至少有一个 JetBrains IDE MCP 服务器（`idea`、`webStorm` 或 `androidStudio`）处于连接状态，方可使用本 skill。若所有 JetBrains MCP 服务器均未连接，则忽略本 skill，改用通用文件工具（Read、Grep、Glob、Edit 等）完成操作。

通过 JetBrains IDE MCP 工具与开发者协作。核心优势是利用 IDE 的语义理解能力（索引、PSI 树、检查引擎）进行代码操作，比纯文本搜索更精准、比命令行更高效。

## IDE 识别

由于用户可能同时打开多个不同 IDE 的项目（如 IDEA、WebStorm、Android Studio），必须先确定当前项目对应的 IDE 及 MCP 服务器名。

### 映射关系

| IDE | MCP 服务器名 | CLAUDE.md 声明值 |
|-----|-------------|-----------------|
| IntelliJ IDEA | `idea` | `idea` |
| WebStorm | `webStorm` | `webStorm` |
| Android Studio | `androidStudio` | `androidStudio` |

> 工具调用时 Claude 会自动添加 `mcp__<serverName>__` 前缀，本 skill 中所有工具名均不包含此前缀。

### 识别流程

每次使用本 skill 时必须先执行以下流程确定 IDE 类型：

**第一步**：读取当前项目的 `CLAUDE.md`，查找 `jetbrains-ide:` 声明：

```markdown
# CLAUDE.md 中的声明示例
jetbrains-ide: idea
```

若找到则直接使用对应的 MCP 服务器名，跳过第二步。

**第二步**：若 `CLAUDE.md` 中未找到声明，调用「向用户提问」类工具，让用户选择当前项目使用的 IDE：

- **问题**：当前项目使用的是哪个 JetBrains IDE？
- **选项**：IntelliJ IDEA（推荐）、WebStorm、Android Studio

用户选择后，将结果写入 `CLAUDE.md`（追加 `jetbrains-ide: <值>`），后续无需再次询问。

> 确定 IDE 后，后续所有 MCP 工具调用均使用对应的服务器名前缀。若对应的 MCP 服务器未连接，则退回到通用文件工具。

## IDE 功能差异

| 功能 | IDEA | WebStorm | Android Studio |
|------|:----:|:--------:|:--------------:|
| 代码搜索与定位 | ✓ | ✓ | ✓ |
| 编译问题排查 | ✓ | ✓ | ✓ |
| 代码编辑与重构 | ✓ | ✓ | ✓ |
| 项目浏览 | ✓ | ✓ | ✓ |
| `search_symbol` 符号搜索 | ✓ | ✓ | ✗ |
| `read_file` 多模式读取 | ✓ | ✓ | ✗ |
| `generate_psi_tree` PSI 分析 | ✓ | ✓ | ✗ |
| Inspection KTS | ✓ | ✗ | ✗ |
| 数据库操作 | ✓（Ultimate） | ✓ | ✗ |

## 核心原则

1. **始终传递 `projectPath`** — 所有工具调用必须包含当前工作目录作为 `projectPath`
2. **优先用 IDE 工具而非命令行** — `search_text` 优于 `grep`，`find_files_by_name_keyword` 优于 `find`，`get_file_problems` 优于 `gradle build`
3. **用 `open_file_in_editor` 同步视角** — 关键位置打开文件让开发者在 IDE 中看到

## 场景零：项目初始化（`init`）

将当前项目配置为 JetBrains IDE 协作模式：检测可用 IDE → 写入 `jetbrains-ide:` 声明 → 追加协作规则（含默认加载说明）→ 配置 `enabledSkills`。后续会话中 skill 自动加载并识别 IDE 类型，无需重复询问。

> 触发方式：用户说"初始化 jetbrains"、"jetbrains init"、"配置 jetbrains 项目"等。详细执行流程见 [INIT 参考](references/INIT.md)。

## 场景一：代码搜索与定位

### 按符号名搜索（推荐首选）

适合找类、方法、字段等编程符号，结果包含精确的坐标位置。

```json
{ "q": "KnowledgeBaseVersionService", "limit": 10, "projectPath": "..." }
```

找不到项目符号时加 `"include_external": true` 搜索 SDK 和库。

> **可用性**：IDEA、WebStorm。Android Studio 不支持，可用 `search_in_files_by_text` 替代。

### 按文本搜索

**IDE 索引搜索**（推荐，比命令行更快，结果用 `||` 高亮）：

```json
{ "searchText": "executeAiTask", "fileMask": "*.kt", "directoryToSearch": "packages/ai", "maxUsageCount": 20, "projectPath": "..." }
```

**MCP 通用搜索**（支持 glob 路径过滤）：

```json
{ "q": "executeAiTask", "paths": ["packages/ai/**/*.kt"], "limit": 20, "projectPath": "..." }
```

### 按正则搜索

**IDE 索引正则**：

```json
{ "regexPattern": "@GetMapping.*knowledge", "fileMask": "*.kt", "caseSensitive": false, "maxUsageCount": 10, "projectPath": "..." }
```

**MCP 通用正则**（支持 glob 路径过滤）：

```json
{ "q": "fun\\s+\\w+\\(.*tenantId.*\\)", "paths": ["packages/knowledge/**/*.kt"], "limit": 10, "projectPath": "..." }
```

### 按文件名搜索

通过索引快速查找，只匹配文件名不匹配路径。

```json
{ "nameKeyword": "KnowledgeBaseVersion", "fileCountLimit": 10, "projectPath": "..." }
```

### 按 glob 模式搜索文件

```json
{ "globPattern": "src/**/*Service*.kt", "subDirectoryRelativePath": "packages/ai", "fileCountLimit": 20, "projectPath": "..." }
```

也可用 `search_file`（支持 `!` 排除）：

```json
{ "q": "**/*ServiceImpl.kt", "paths": ["packages/knowledge/"], "limit": 20, "projectPath": "..." }
```

### 读取文件内容

**`read_file`**（多模式，支持反编译 Jar）：支持 `slice`、`lines`、`line_columns`、`offsets`、`indentation` 模式。

```json
{ "file_path": "packages/ai/src/main/kotlin/.../Service.kt", "mode": "lines", "start_line": 30, "end_line": 80 }
```

**`get_file_text_by_path`**（MCP 插件专用，支持截断控制）：

```json
{ "pathInProject": "packages/ai/src/main/kotlin/.../SomeService.kt", "truncateMode": "MIDDLE", "maxLinesCount": 100, "projectPath": "..." }
```

> **可用性**：`read_file` 仅 IDEA、WebStorm 可用。Android Studio 使用 `get_file_text_by_path`。

## 场景二：编译问题排查

### 检查文件错误

用 IntelliJ 检查引擎分析文件中的错误和警告，无需触发完整构建。

```json
{ "filePath": "packages/ai/src/main/kotlin/.../SomeService.kt", "errorsOnly": true, "projectPath": "..." }
```

返回 severity、description、1-based 行列号。比 `gradle build` 快得多。

### 触发构建

修改代码后验证编译是否通过：

```json
{ "filesToRebuild": ["packages/ai/src/main/kotlin/.../SomeService.kt"], "projectPath": "..." }
```

指定 `filesToRebuild` 只编译相关文件，更快。`rebuild: true` 全量重建。

## 场景三：理解代码

### 获取符号信息

等价于 IDE 的 Quick Documentation（Ctrl+Q），返回签名、类型、文档，可能包含声明代码。

```json
{ "filePath": "...", "line": 42, "column": 15, "projectPath": "..." }
```

适合理解：方法的参数类型和返回值、类的继承关系、字段的类型声明。

### PSI 树分析

需要理解代码结构时生成 PSI 树：

```json
{ "code": "fun foo(a: String): Int = a.length", "language": "Kotlin" }
```

> **可用性**：仅 IDEA、WebStorm。Android Studio 不支持。

## 场景四：代码编辑

### 文本替换

```json
{ "pathInProject": "...", "oldText": "old", "newText": "new", "replaceAll": false, "projectPath": "..." }
```

- `replaceAll: true` 替换所有匹配（默认）
- `replaceAll: false` 只替换第一个
- 支持 `regex: true` 和 `caseSensitive` 控制
- 自动保存

### 重命名重构（语义感知）

自动更新项目中所有引用，不是简单的文本替换。

```json
{ "pathInProject": "...", "symbolName": "oldMethod", "newName": "newMethod", "projectPath": "..." }
```

> 重命名编程符号时**始终优先使用此工具**而非文本替换。

### 创建文件

```json
{ "pathInProject": "...", "text": "package ...", "overwrite": false, "projectPath": "..." }
```

自动创建父目录。`overwrite: false` 时冲突会抛异常。

### 格式化

```json
{ "path": "...", "projectPath": "..." }
```

## 场景五：数据库操作

> **可用性**：IDEA Ultimate、WebStorm。DDL 数据源（无底层数据库连接）不支持 `execute_sql_query`、`preview_table_data` 等需要 DBMS 连接的工具。

### 浏览数据库结构

```json
// 列出所有数据库连接
{ "projectPath": "..." }

// 测试连接是否可用
{ "id": "connection-id", "projectPath": "..." }

// 列出 Schema
{ "connectionId": "...", "selectedOnly": false, "projectPath": "..." }

// 列出支持的对象类型
{ "connectionId": "...", "projectPath": "..." }

// 列出 Schema 中的对象
{ "connectionId": "...", "databaseName": "...", "schemaName": "...", "kind": "table", "projectPath": "..." }

// 获取对象结构（表、视图等）
{ "connectionId": "...", "databaseName": "...", "schemaName": "...", "kind": "table", "objectName": "users", "projectPath": "..." }
```

### 执行 SQL 查询

```json
{
  "connectionId": "...",
  "databaseName": "...",
  "schemaName": "...",
  "queryText": "SELECT * FROM users LIMIT 10",
  "projectPath": "..."
}
```

返回 CSV 格式的查询结果。

### 预览表数据

```json
{
  "connectionId": "...",
  "databaseName": "...",
  "schemaName": "...",
  "tableName": "users",
  "maxRowCount": 100,
  "projectPath": "..."
}
```

### 查询历史与会话管理

```json
// 查看最近执行的查询
{ "connectionId": "...", "projectPath": "..." }

// 取消正在运行的查询
{ "sessionId": 123, "projectPath": "..." }
```

## 场景六：项目浏览

### 目录结构

```json
{ "directoryPath": "packages", "maxDepth": 2, "projectPath": "..." }
```

### 模块、依赖与仓库

```json
// 模块列表
{ "projectPath": "..." }

// 依赖列表
{ "projectPath": "..." }

// VCS 根目录
{ "projectPath": "..." }
```

### 当前打开的文件

```json
{ "projectPath": "..." }
```

了解用户当前正在关注哪些文件。

## Inspection KTS（仅 IDEA）

编写和运行自定义检查脚本：

```json
// 获取 Inspection API 文档
{ "language": "Kotlin", "wrapInTags": true }

// 获取示例模板
{ "language": "Kotlin", "includeAdditionalExamples": true }

// 运行检查脚本
{ "inspectionKtsCode": "// inspection.kts code", "contextPath": "src/main/kotlin/.../Target.kt", "projectPath": "..." }
```

## 工具选择速查

| 需求 | 推荐工具 | 可用性 |
|------|----------|--------|
| 找类/方法/字段 | `search_symbol` | IDEA、WebStorm |
| 搜文本子串 | `search_text` 或 `search_in_files_by_text` | 全部 |
| 搜正则 | `search_regex` 或 `search_in_files_by_regex` | 全部 |
| 找文件（按名称） | `find_files_by_name_keyword`（快，按索引） | 全部 |
| 找文件（按 glob） | `find_files_by_glob` 或 `search_file` | 全部 |
| 读文件 | `read_file`（多模式）或 `get_file_text_by_path` | 全部 |
| 看方法签名 | `get_symbol_info` | 全部 |
| 查编译错误 | `get_file_problems`（快）或 `build_project`（完整） | 全部 |
| 重命名 | `rename_refactoring` | 全部 |
| 替换文本 | `replace_text_in_file` | 全部 |
| 新建文件 | `create_new_file` | 全部 |
| 格式化 | `reformat_file` | 全部 |
| 在 IDE 打开 | `open_file_in_editor` | 全部 |
| 获取打开文件 | `get_all_open_file_paths` | 全部 |
| 执行 SQL | `execute_sql_query` | IDEA Ultimate、WebStorm |
| 浏览表结构 | `get_database_object_description` | IDEA Ultimate、WebStorm |
| 预览表数据 | `preview_table_data` | IDEA Ultimate、WebStorm |
| PSI 树分析 | `generate_psi_tree` | IDEA、WebStorm |
| Inspection KTS | `generate_inspection_kts_api` / `run_inspection_kts` | IDEA |
| 看目录 | `list_directory_tree` | 全部 |
| 项目模块 | `get_project_modules` | 全部 |
| 项目依赖 | `get_project_dependencies` | 全部 |
| VCS 仓库 | `get_repositories` | 全部 |

## 注意事项

- **`projectPath` 必传**：所有工具调用必须包含当前工作目录作为 `projectPath`
- **文件路径格式**：大多数工具接受项目相对路径，部分也支持绝对路径、Jar 条目（`/path/lib.jar!/pkg/Foo.class`）和 URL（`file://`、`jar://`）
- **行号 1-based**：`get_file_problems`、`get_symbol_info` 等工具的行列号均从 1 开始
- **自动保存**：`replace_text_in_file`、`rename_refactoring` 操作后自动保存
- **DDL 数据源**：无底层数据库连接，不支持 `execute_sql_query`、`preview_table_data` 等需要 DBMS 连接的工具

## 参考

- [INIT 参考](references/INIT.md) — 项目初始化（init）的详细执行流程与写入规则
- [编辑参考](references/EDITING.md) — 代码搜索、分析、编辑、数据库、项目信息等工具的详细参数与示例
