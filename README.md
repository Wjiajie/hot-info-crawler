# 🔥 Hot Info Crawler

> AI Agent Skill — 多平台热点信息自动抓取、中文总结、增量写入 Obsidian 知识库

## 功能简介

一句话触发，AI 助手自动从多个信息平台抓取你关注领域的最新热点，翻译总结成中文，写入本地 Markdown 文件，并同步到 Obsidian 知识库。

### 支持平台

| 平台 | 内容类型 |
|------|----------|
| X.com | 实时热点讨论、行业大 V 观点 |
| HuggingFace Papers | AI/ML 学术论文日榜 |
| YouTube | 深度视频内容（教程、访谈、讲座） |
| Reddit | 深度讨论、社区经验分享 |
| 即刻 (Jike) | 中文社区热点讨论 |

### 核心特性

- ✅ **增量写入**：每完成一个板块立即保存，不怕中途崩溃
- ✅ **断点续跑**：中断后重新运行，自动跳过已完成板块
- ✅ **全中文总结**：所有内容自动翻译为专业中文摘要
- ✅ **三级工具回退**：browser_mcp → browser_subagent → read_url_content
- ✅ **Obsidian 同步**：抓取完成后自动触发云端同步
- ✅ **完全可配置**：主题、平台、关注账号均可自定义

## 工作流程

![workflow](docs/hot-info-crawler-workflow.png)

1. **初始化** — 读取配置，创建日期文件，支持断点续跑
2. **工具选择** — 三级回退策略，确保总能获取数据
3. **Feed 数据源** — 拉取 Follow Builders 等结构化 Feed
4. **主题检索** — 逐主题在对应平台搜索，提取 10+ 条，中文总结
5. **账号追踪** — 关注的 X 大 V 最新动态
6. **完成标记** — 写入 all_done 标记
7. **Obsidian 同步** — 自动推送到云端

## 快速开始

### 1. 安装

将 `SKILL.md` 和 `references/` 目录复制到你的 AI Agent 的 skills 目录下：

```
~/.gemini/skills/hot-info-crawler/
├── SKILL.md
└── references/
    ├── install.md
    ├── user_config_template.md
    ├── output_config.md
    ├── follow_builders_feed.md
    ├── platforms.md
    ├── themes.md
    └── search_workflow.md
```

### 2. 首次运行

对 AI 助手说：**"抓取热点信息"**

首次运行会引导你完成配置：
- 选择输出目录
- 自定义检索主题和关键词
- 选择信息源平台
- 配置关注的 X 账号

### 3. 日常使用

每次只需说 "抓取热点信息"，AI 会自动完成所有工作。

## 配置说明

所有用户配置存储在 `~/.hot-info-crawler/user_config.md`，包括：

- **信息源开关**：启用/禁用各平台
- **Feed 数据源**：结构化数据源（如 Follow Builders）
- **主题列表**：自定义检索主题、关键词和平台
- **关注的 X 账号**：额外追踪的账号

详见 [用户配置模板](references/user_config_template.md)。

## 输出示例

每次运行生成一个日期文件：

```
hot-info-{YYYY-MM-DD}.md
```

包含各板块的中文总结，Markdown 表格格式，可直接在 Obsidian 中阅读。

## 文档索引

| 文档 | 内容 |
|------|------|
| [SKILL.md](SKILL.md) | 技能主文档 |
| [install.md](references/install.md) | 安装与配置指南 |
| [user_config_template.md](references/user_config_template.md) | 用户配置模板 |
| [output_config.md](references/output_config.md) | 输出与增量写入 |
| [follow_builders_feed.md](references/follow_builders_feed.md) | Feed 数据源 |
| [platforms.md](references/platforms.md) | 平台配置 |
| [themes.md](references/themes.md) | 平台分配规则 |
| [search_workflow.md](references/search_workflow.md) | 完整执行流程 |

## License

MIT
