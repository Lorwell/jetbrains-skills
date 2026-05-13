# JetBrains IDE 项目初始化参考（init）

将当前项目配置为 JetBrains IDE 协作模式：写入 IDE 声明 → 写入优先使用规则，引导智能体在每次会话中主动调用
`jetbrains-skills` skill 并识别 IDE 类型，无需重复询问。

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

在 `CLAUDE.md` 中追加优先使用 JetBrains IDE MCP 的协作规则，并声明 `jetbrains-skills` skill 应默认加载。*
*写入时必须严格按照以下规范内容，不得偏离：**

- 若 `CLAUDE.md` 中已有 JetBrains IDE MCP 相关规则但内容与规范不一致（如措辞不同、缺少关键语句、skill 名称不匹配等），*
  *必须替换为规范内容**，确保与初始化规范完全一致
- 若完全没有相关规则则追加

规范内容：

```markdown
## JetBrains IDE MCP 协作

当进行代码搜索、符号查找、编译检查、重构等操作时，优先使用 JetBrains IDE MCP 工具，因为 IDE 具有完整的项目索引和语义理解能力（PSI
树、检查引擎），比纯文本工具更精准高效。仅在 JetBrains MCP 服务器未连接或对应功能不支持时，退回到通用文件工具。

请立即使用 `jetbrains-skills` skill，根据该 skill 中的描述来完成以下操作：

- **代码搜索与定位**：按符号名搜索（`search_symbol`，推荐首选）、按文本搜索（`search_text` / `search_in_files_by_text`，支持
  glob 路径过滤和文件掩码）、按正则搜索（`search_regex` / `search_in_files_by_regex`，支持大小写控制）、按文件名搜索（
  `find_files_by_name_keyword`，通过 IDE 索引快速查找，只匹配文件名不匹配路径）、按 glob 模式搜索文件（`find_files_by_glob` /
  `search_file`，支持 `!` 排除模式）、读取文件（`read_file` 多模式读取——行范围、缩进块、反编译 Jar 类；或
  `get_file_text_by_path` 支持截断控制）、浏览目录结构（`list_directory_tree`，伪图形格式，类似 `tree` 命令）
- **编译问题排查**：使用 `get_file_problems` 分析文件中的错误和警告（利用 IntelliJ 检查引擎，无需触发完整构建，返回
  severity、description、1-based 行列号，比命令行构建快得多），或使用 `build_project` 触发增量/全量编译验证
- **理解代码**：使用 `get_symbol_info` 获取符号的签名、类型、文档等详细信息（等价 IDE 的 Quick Documentation），或使用
  `generate_psi_tree` 生成 PSI 树分析代码语法结构
- **代码编辑与重构**：使用 `replace_text_in_file` 进行文本替换（支持正则、大小写控制、全量/首次替换，自动保存）、使用
  `rename_refactoring` 进行语义感知重命名（自动更新项目中所有引用，非简单文本替换，重命名编程符号时始终优先使用）、使用
  `create_new_file` 创建文件（自动创建父目录）、使用 `reformat_file` 格式化代码、使用 `open_file_in_editor` 在 IDE
  中打开文件同步视角、使用 `get_all_open_file_paths` 获取用户当前打开的文件列表
- **数据库操作**：使用 `list_database_connections` 列出数据库连接、使用 `list_database_schemas` 列出 Schema、使用
  `list_schema_objects` 列出 Schema 中的对象、使用 `get_database_object_description` 获取表/视图等对象的结构（列、类型、键、索引）、使用
  `execute_sql_query` 执行 SQL 查询（返回 CSV 结果）、使用 `preview_table_data` 预览表数据
- **项目浏览**：使用 `get_project_modules` 获取模块列表、`get_project_dependencies` 获取依赖列表、`get_repositories` 获取
  VCS 仓库信息

所有 IDE MCP 工具调用必须传递 `projectPath` 参数（当前工作目录）。

当用户指定了其他项目路径时，同样遵循 `jetbrains-skills` skill 中的 IDE 识别流程：先检查该项目是否已有 `jetbrains-ide:` 声明且对应 IDE MCP 服务器处于连接状态，若已启用则对该项目同样优先使用 IDE MCP 工具进行操作。
```

## 写入注意事项

- 若 `CLAUDE.md` 中已有 `jetbrains-ide:` 声明，检查当前连接状态是否匹配，不匹配则更新为当前连接的 IDE
- 若已有"JetBrains IDE MCP 协作"相关规则但内容与规范不一致，必须替换为规范内容，确保完全一致
- 若完全没有相关规则则追加

## 映射关系

| IDE            | MCP 服务器名        | CLAUDE.md 声明值   |
|----------------|-----------------|-----------------|
| IntelliJ IDEA  | `idea`          | `idea`          |
| WebStorm       | `webStorm`      | `webStorm`      |
| Android Studio | `androidStudio` | `androidStudio` |
