# JetBrains IDE 编辑参考（查询 + 修改）

## 一、代码导航与搜索

### search_text — MCP 通用文本搜索

使用 IntelliJ 搜索引擎进行全文子串搜索，结果中匹配项两侧用 `||` 高亮。

```json
{
  "q": "TenantContext.getTenantId",
  "paths": ["packages/**"],
  "limit": 20,
  "projectPath": "..."
}
```

| 参数 | 说明 |
|------|------|
| `q` | 搜索文本 |
| `paths` | glob 过滤，如 `["src/**", "!**/test/**"]` |
| `limit` | 最大结果数 |

### search_regex — MCP 通用正则搜索

```json
{
  "q": "fun\\s+\\w+\\(.*tenantId.*\\)",
  "paths": ["packages/knowledge/**/*.kt"],
  "limit": 10,
  "projectPath": "..."
}
```

### search_symbol — 符号搜索

按标识符片段搜索类、方法、字段等符号。包含坐标信息（1-based 行/列）。

```json
{ "q": "KnowledgeBaseVersionService", "limit": 10, "projectPath": "..." }
```

找不到项目符号时，加 `include_external: true` 搜索 SDK 和库符号。

> **可用性**：IDEA、WebStorm。Android Studio 不支持。

### search_in_files_by_text — IntelliJ 全文搜索

通过 IDE 索引搜索，比命令行工具更快。结果中匹配项用 `||` 高亮。

```json
{
  "searchText": "executeAiTask",
  "fileMask": "*.kt",
  "directoryToSearch": "packages/ai",
  "maxUsageCount": 20,
  "projectPath": "..."
}
```

| 参数 | 说明 |
|------|------|
| `searchText` | 搜索文本 |
| `fileMask` | 文件过滤，如 `*.kt` |
| `directoryToSearch` | 搜索目录（相对路径） |
| `caseSensitive` | 是否区分大小写（默认 true） |
| `maxUsageCount` | 最大结果数 |

### search_in_files_by_regex — IntelliJ 正则搜索

```json
{
  "regexPattern": "@GetMapping.*knowledge",
  "fileMask": "*.kt",
  "caseSensitive": false,
  "maxUsageCount": 10,
  "projectPath": "..."
}
```

### read_file — 读取文件（多模式）

支持多种模式：`slice`、`lines`、`line_columns`、`offsets`、`indentation`。
可读取 Jar 内类文件并反编译。

```json
// 行范围读取
{ "file_path": "packages/ai/src/main/kotlin/.../SomeService.kt", "mode": "lines", "start_line": 30, "end_line": 50 }

// 缩进块读取（读取方法体）
{ "file_path": "...", "mode": "indentation", "start_line": 42, "max_levels": 1, "include_siblings": false }

// 反编译 Jar 中的类
{ "file_path": "/path/lib.jar!/pkg/Foo.class", "mode": "slice" }
```

> **可用性**：IDEA、WebStorm。Android Studio 使用 `get_file_text_by_path`。

### get_file_text_by_path — MCP 专用文件读取

JetBrains MCP 插件提供的文件读取，支持截断控制。

```json
{
  "pathInProject": "packages/ai/src/main/kotlin/.../SomeService.kt",
  "truncateMode": "MIDDLE",
  "maxLinesCount": 100,
  "projectPath": "..."
}
```

| 参数 | 说明 |
|------|------|
| `pathInProject` | 文件相对路径 |
| `truncateMode` | 截断模式：`START`、`MIDDLE`、`END`、`NONE` |
| `maxLinesCount` | 最大行数 |

### find_files_by_name_keyword — 按文件名搜索

通过索引快速查找，只匹配名称不匹配路径。

```json
{ "nameKeyword": "KnowledgeBaseVersion", "fileCountLimit": 10, "projectPath": "..." }
```

### find_files_by_glob — 按 glob 模式搜索文件

```json
{ "globPattern": "src/**/*Service*.kt", "subDirectoryRelativePath": "packages/ai", "fileCountLimit": 20, "projectPath": "..." }
```

| 参数 | 说明 |
|------|------|
| `globPattern` | glob 模式（相对项目根） |
| `subDirectoryRelativePath` | 搜索子目录 |
| `fileCountLimit` | 最大文件数 |
| `addExcluded` | 是否包含被排除的文件 |

### search_file — glob 路径搜索

```json
{ "q": "**/*ServiceImpl.kt", "paths": ["packages/knowledge/"], "limit": 20, "projectPath": "..." }
```

支持 `!` 排除模式。

### list_directory_tree — 目录树

伪图形格式，类似 `tree` 命令。

```json
{ "directoryPath": "packages/ai/src/main/kotlin", "maxDepth": 3, "projectPath": "..." }
```

## 二、代码分析与检查

### get_file_problems — 文件问题检查

使用 IntelliJ 检查引擎分析文件中的错误和警告。

```json
{
  "filePath": "packages/ai/src/main/kotlin/.../SomeService.kt",
  "errorsOnly": false,
  "projectPath": "..."
}
```

返回问题列表，包含 severity（error/warning）、description、1-based 行列号。

> 只能分析项目目录内的文件。

### get_symbol_info — 符号信息

等价于 IDE 的 Quick Documentation（Ctrl+Q）。返回符号的名称、签名、类型、文档。
如果位置引用了某个符号，会返回该符号的声明代码片段。

```json
{
  "filePath": "packages/ai/src/main/kotlin/.../SomeService.kt",
  "line": 42,
  "column": 15,
  "projectPath": "..."
}
```

### build_project — 项目构建

触发项目构建，等待完成并返回编译错误。

```json
{
  "projectPath": "...",
  "filesToRebuild": ["packages/ai/src/main/kotlin/.../SomeService.kt"],
  "rebuild": false
}
```

- `filesToRebuild`：指定文件时只编译这些文件
- `rebuild: true`：全量重建（仅在未指定 filesToRebuild 时有效）

### generate_psi_tree — PSI 树分析

为代码片段生成 PSI 树，帮助理解代码结构。

```json
{ "code": "fun foo(a: String): Int = a.length", "language": "Kotlin" }
```

> **可用性**：IDEA、WebStorm。Android Studio 不支持。

## 三、代码编辑

### replace_text_in_file — 文本替换

```json
{
  "pathInProject": "packages/ai/src/main/kotlin/.../SomeService.kt",
  "oldText": "old code",
  "newText": "new code",
  "replaceAll": true,
  "projectPath": "..."
}
```

- `replaceAll: true` 替换所有匹配（默认）
- `replaceAll: false` 只替换第一个
- 支持 `regex: true` 和 `caseSensitive` 控制
- 自动保存

### create_new_file — 创建文件

```json
{
  "pathInProject": "packages/ai/src/main/kotlin/.../NewService.kt",
  "text": "package cc.lingya.xiaolingtong.ai\n\nclass NewService",
  "overwrite": false,
  "projectPath": "..."
}
```

自动创建父目录。`overwrite: false` 时冲突会抛异常。

### rename_refactoring — 重命名重构

上下文感知型重命名，自动更新项目中所有引用。

```json
{
  "pathInProject": "packages/ai/src/main/kotlin/.../SomeService.kt",
  "symbolName": "getUserData",
  "newName": "fetchUserData",
  "projectPath": "..."
}
```

> 始终优先使用此工具而非文本替换来重命名编程符号。

### reformat_file — 代码格式化

```json
{ "path": "packages/ai/src/main/kotlin/.../SomeService.kt", "projectPath": "..." }
```

### open_file_in_editor — 在 IDE 中打开文件

```json
{ "filePath": "packages/ai/src/main/kotlin/.../SomeService.kt", "projectPath": "..." }
```

### get_all_open_file_paths — 获取打开的文件

```json
{ "projectPath": "..." }
```

返回当前活动编辑器和其他打开编辑器的文件路径。

## 四、数据库操作

> **可用性**：IDEA Ultimate、WebStorm。

### list_database_connections — 列出数据库连接

获取项目中配置的所有数据库连接。

```json
{ "projectPath": "..." }
```

### test_database_connection — 测试连接

检查指定数据库连接是否有效且可达。

```json
{ "id": "connection-id", "projectPath": "..." }
```

### list_database_schemas — 列出 Schema

```json
{ "connectionId": "...", "selectedOnly": false, "projectPath": "..." }
```

| 参数 | 说明 |
|------|------|
| `connectionId` | 数据库连接 ID |
| `selectedOnly` | `true` 只列出已加载元数据的 schema，`false` 列出所有 |

### list_schema_object_kinds — 列出支持的对象类型

```json
{ "connectionId": "...", "projectPath": "..." }
```

返回支持的 schema 对象类型（如 table、view、routine）。

### list_schema_objects — 列出 Schema 对象

```json
{
  "connectionId": "...",
  "databaseName": "mydb",
  "schemaName": "public",
  "kind": "table",
  "projectPath": "..."
}
```

| 参数 | 说明 |
|------|------|
| `connectionId` | 数据库连接 ID |
| `databaseName` | 数据库名（DBMS 无数据库概念时可留空） |
| `schemaName` | Schema 名 |
| `kind` | 可选过滤：table、view、routine 等。null 返回所有对象 |

### get_database_object_description — 获取对象结构

获取表、视图等对象的列、类型、键、索引等结构信息。

```json
{
  "connectionId": "...",
  "databaseName": "mydb",
  "schemaName": "public",
  "kind": "table",
  "objectName": "users",
  "projectPath": "..."
}
```

返回层级文本表示，包含 columns、types、keys、indexes。

### execute_sql_query — 执行 SQL 查询

```json
{
  "connectionId": "...",
  "databaseName": "mydb",
  "schemaName": "public",
  "queryText": "SELECT * FROM users LIMIT 10",
  "projectPath": "..."
}
```

返回 CSV 格式的查询结果和执行状态（成功/失败及错误详情）。

> DDL 数据源（无底层数据库连接）不支持此工具。

### preview_table_data — 预览表数据

```json
{
  "connectionId": "...",
  "databaseName": "mydb",
  "schemaName": "public",
  "tableName": "users",
  "maxRowCount": 100,
  "projectPath": "..."
}
```

| 参数 | 说明 |
|------|------|
| `maxRowCount` | 最大返回行数，默认 100，必须 > 0 |

> DDL 数据源不支持此工具。

### list_recent_sql_queries — 查询历史

```json
{ "connectionId": "...", "projectPath": "..." }
```

获取指定连接最近执行的查询（含当前正在运行的）。

### cancel_sql_query — 取消查询

```json
{ "sessionId": 123, "projectPath": "..." }
```

通过 `sessionId` 取消正在运行的查询。

## 五、项目信息

### get_project_modules — 模块列表

```json
{ "projectPath": "..." }
```

### get_project_dependencies — 依赖列表

```json
{ "projectPath": "..." }
```

### get_repositories — VCS 根目录

```json
{ "projectPath": "..." }
```

多仓库项目中识别所有仓库。

## 六、Inspection KTS

> **可用性**：仅 IDEA。

### generate_inspection_kts_api — 获取 Inspection API

```json
{ "language": "Kotlin", "wrapInTags": true, "projectPath": "..." }
```

### generate_inspection_kts_examples — 获取示例模板

```json
{ "language": "Kotlin", "includeAdditionalExamples": true, "projectPath": "..." }
```

### run_inspection_kts — 运行检查脚本

```json
{
  "inspectionKtsCode": "// inspection.kts code",
  "contextPath": "src/main/kotlin/.../Target.kt",
  "projectPath": "..."
}
```
