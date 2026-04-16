# fifth-quadrant-skill

[![repo](https://img.shields.io/badge/repo-fifth--quadrant--skill-blue)](#)
[![language](https://img.shields.io/badge/language-Markdown-informational)](#)
[![status](https://img.shields.io/badge/status-active-success)](#)

这个仓库用于维护第五象限相关的 Agent Skill，当前聚焦飞书（Feishu/Lark）场景：

- 自动批改每日作业并同步结果到多维表格、群消息
- 统计飞书文档评论与评分信息

## 适用人群

- 想把飞书文档/群聊流程自动化的人
- 想把“作业提交 → 批改 → 记录 → 公布/提醒”跑通的人

## 目录结构

```text
.
├── fq-feishu-auto-grade-homework/
│   └── SKILL.md
└── fq-feishu-doc-comment-stats/
    └── SKILL.md
```

## Skills 一览

### 1) fq@auto-grade-homework

路径：`fq-feishu-auto-grade-homework/SKILL.md`

用途：
- 拉取“上行”每日作业（Wiki/Docx）
- 给出评分（优秀/合格/不合格）和评语
- 写入或更新飞书多维表格
- 将优秀作业推送到指定群聊
- 可选提醒未提交成员

适用触发语：
- 执行今日作业批改
- 作业批改
- 批改作业

### 2) fq-feishu-doc-comment-stats

路径：`fq-feishu-doc-comment-stats/SKILL.md`

用途：
- 统计指定 Feishu 文档评论总量
- 提取评论中的评分关键词（如“优秀”“合格”）
- 输出评论时间与引用内容，支持分页处理

## 快速开始

### 前置要求

- 已有 Feishu/Lark 应用（拿到 `APP_ID`、`APP_SECRET`）
- 能访问目标文档/群聊/多维表格（权限与应用范围一致）

### 环境变量

按需准备 `.env`（不要提交到仓库）：

```bash
APP_ID=cli_xxx
APP_SECRET=xxx
FILE_TOKEN=doccnxxxx
FILE_TYPE=docx
TZ=Asia/Shanghai
```

### 使用方式

1. 选择你要用的 Skill，进入对应目录阅读 `SKILL.md`
2. 按 `SKILL.md` 的流程执行（其中包含需要调用的接口与步骤）
3. 若 Lark-MCP 授权过期，按 Skill 文档中的登录命令重新授权

## 安全与协作约定

- 不要在仓库中提交密钥、Token、`.env`。
- 涉及群发消息、自动评分前，建议先在测试范围验证。
- 若新增 Skill，建议使用统一命名：`<namespace>@<skill-name>/SKILL.md`。

## 贡献指南（Contributing）

欢迎 PR。建议遵循这些约定：

- **新增 Skill**：新建目录 `<namespace>-<skill-name>/`，并提供 `SKILL.md`
- **SKILL.md 必备信息**：
  - Skill 标识与一句话简介
  - 前置条件（权限、token、群/文档/表格信息）
  - 可复制执行的步骤（按序、可回滚/可重试）
  - 安全注意事项（不要泄露密钥/Token）
- **敏感信息**：不要提交 `.env`、token、webhook、chat_id 等（除非明确可公开）

## 发布与变更记录

如果需要对外发布，建议采用：

- **版本规范**：SemVer（`MAJOR.MINOR.PATCH`）
- **变更记录**：维护 `CHANGELOG.md`（可选），按版本记录新增/修复/破坏性变更

## 后续可扩展方向

- 新增统一的评分配置（例如权重、阈值、模板）以便复用
- 增加统计结果导出（CSV/Excel）与可视化报表
- 接入定时调度，实现每日自动执行与失败重试

## License

未指定。若需要开源发布，建议补充 `LICENSE`（如 MIT/Apache-2.0）。  