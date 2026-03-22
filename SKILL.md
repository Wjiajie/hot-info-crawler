---
name: hot-info-crawler
description: 专门用于从多个信息平台（X.com, HuggingFace Papers, YouTube, Reddit, 即刻等）抓取用户自定义主题的最新、最热门信息的技能。当用户明确要求"抓取当前热点信息"、"获取最新趋势"或"更新信息库"时，启动该技能。支持的主题和信息源均可在用户配置文件中自定义。
---

# Hot Info Crawler 技能指南

此技能使用浏览器自动化工具执行多平台热点检索任务，支持三级工具回退策略。检索结果**增量写入**本地 Markdown 文件，防止中断丢失。

> 首次使用？请先阅读 `references/install.md` 完成依赖安装与配置。

## 任务触发

### 抓取热点信息

当接收到"抓取热点信息"的指令时：

1. **初始化**：读取配置文件
   - 读取 `~/.hot-info-crawler/config.json` 获取 `outputDir`
   - 读取 `~/.hot-info-crawler/user_config.md` 获取主题、信息源开关、Feed 源、关注账号
   - 如任一配置缺失，执行 onboarding 流程（详见 `references/install.md`）
   - 在 `outputDir` 下创建/打开日期文件 `hot-info-{YYYY-MM-DD}.md`
   - 如文件已存在，扫描已完成的板块标记，从断点继续
2. **执行**：依次检索各板块，**每完成一个板块立即追加写入文件**
3. **完成**：写入完成标记，告知用户文件路径

### 重置配置

当接收到"重置热点抓取配置"、"重新配置热点信息"或类似重置指令时：

1. **确认**：告知用户即将清除所有已有配置，询问是否继续
2. **清除**：删除 `~/.hot-info-crawler/user_config.md` 和 `~/.hot-info-crawler/config.json`
3. **重新初始化**：执行完整的 onboarding 流程（详见 `references/install.md`）
   - 步骤 1：配置输出路径 → 写入 `config.json`
   - 步骤 2：从模板创建 `user_config.md` → 引导用户自定义主题、信息源、账号
4. **确认**：展示最终配置摘要

## 参考文档

详细的信息源配置和执行流程已拆分到 `references/` 目录：

| 文档 | 内容 |
|------|------|
| `references/install.md` | 依赖安装、工具配置与用户配置初始化（首次使用必读） |
| `references/user_config_template.md` | 用户配置文件模板与字段说明 |
| `references/output_config.md` | 输出路径配置、增量写入与断点续跑 |
| `references/follow_builders_feed.md` | Follow-Builders Feed 数据源配置与输出格式 |
| `references/platforms.md` | 平台 URL 模板与使用说明（X / HuggingFace / YouTube / Reddit / 即刻） |
| `references/themes.md` | 平台分配规则（类型与平台的通用映射逻辑） |
| `references/search_workflow.md` | 完整的检索执行流程（初始化 + Feed + 主题检索 + 账号追踪） |

## 用户配置

所有用户特定配置存储在 `~/.hot-info-crawler/user_config.md`，包括：

- **信息源开关**：启用/禁用 HuggingFace、X.com、YouTube、Reddit、即刻等平台
- **Feed 数据源**：可选的结构化数据源（如 Follow Builders）
- **主题列表**：自定义检索主题、关键词和平台分配
- **关注的 X 账号**：额外追踪的 X 账号列表

首次运行时，skill 会自动从模板创建配置文件并引导用户自定义。详见 `references/user_config_template.md`。

## 快速概览

### 三级工具回退策略

按以下优先级选择浏览器工具（详见 `references/search_workflow.md`）：

| 优先级 | 工具 | 适用条件 | 特点 |
|--------|------|----------|------|
| 🥇 优先 | `browser_mcp` | 已安装 Chrome 扩展且 MCP 已连接 | 复用已登录会话，逐步精细操作 |
| 🥈 备选 | `browser_subagent` | browser_mcp 不可用时 | 内置零配置，任务驱动，独立会话 |
| 🥉 兜底 | `read_url_content` | 以上均不可用时 | 最轻量，纯 HTTP 抓取，无 JS 渲染 |

### 检索策略

平台分配由用户配置中主题的 `类型` 字段决定（详见 `references/themes.md`）：

- **技术类**主题 → 默认 HuggingFace Papers + X.com
- **软技能类**主题 → 默认 YouTube + Reddit + X.com
- 用户可在配置中自由覆盖每个主题的优先平台

### 执行要点（增量写入）

每个步骤完成后**立即追加写入**日期文件，并附加板块完成标记：

0. **【初始化】** 读取配置 → 创建/打开日期文件 → 写入文件头 `<!-- section:header_done -->`
1. **【Feed】** 拉取配置的 Feed 数据源 → 生成中文总结 → 写入文件 `<!-- section:feed_done -->`
2. **【主题检索】** 按配置逐个主题检索 → 每个主题完成后写入文件 `<!-- section:theme_{ID}_done -->`
3. **【账号追踪】** 追踪配置的关注账号 → 写入文件 `<!-- section:accounts_done -->`
4. **【完成】** 写入完成标记 `<!-- section:all_done -->`
5. **【同步】** 调用 Obsidian CLI 触发 Remotely Save 同步 → 将文件推送到云端

### 注意事项

- 优先使用 `browser_mcp`；不可用时自动回退到 `browser_subagent`；均不可用时回退到 `read_url_content`
- 使用 `browser_subagent` 或 `read_url_content` 时，需登录的平台（如即刻）可能无法获取完整内容
- 中文关键词信息不足时，自动切换英文关键词重试
- **断点续跑**：如果当天文件已存在且包含部分标记，跳过已完成板块从断点继续（详见 `references/output_config.md`）
- **所有用户特定配置**均从 `~/.hot-info-crawler/user_config.md` 读取
- **Obsidian 同步**：抓取完成后通过 `obsidian://` 原生 URI 协议唤起 Obsidian，配合 Remotely Save 自动同步完成云端推送
