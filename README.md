# JetBrains IDE Skill

适用于 JetBrains IDE 的 Claude Code 协作技能，支持 IntelliJ IDEA、WebStorm、Android Studio。

通过 IDE MCP 工具进行代码搜索、编译检查、符号分析、代码编辑、数据库操作等全栈协作。

## 安装

```bash
npx skills add shaco/jetbrains-skills
```

## 支持的 IDE

| IDE | MCP 服务器名 |
|-----|-------------|
| IntelliJ IDEA | `idea` |
| WebStorm | `webStorm` |
| Android Studio | `androidStudio` |

## 使用

1. 在项目 `CLAUDE.md` 中声明 IDE 类型：`jetbrains-ide: idea`
2. 确保对应 IDE 的 MCP 插件已启动并连接
3. 在 Claude Code 中正常使用，skill 自动匹配对应 IDE 的工具

## 功能

- 代码搜索与定位（符号、文本、正则、文件名、glob）
- 编译问题排查（文件检查、构建触发）
- 代码理解（符号文档、PSI 树分析）
- 代码编辑（文本替换、语义重命名、创建文件、格式化）
- 数据库操作（表结构浏览、SQL 查询、数据预览）
- 项目浏览（目录树、模块、依赖）
- Inspection KTS（仅 IDEA）
