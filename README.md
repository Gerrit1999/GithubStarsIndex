# GitHub Stars Index

> 自动抓取 GitHub Stars，生成 AI 摘要，便于检索。

## 目录

- [功能特性](#功能特性)
- [快速开始](#快速开始)
- [配置项详解](#配置项详解-环境变量--env)
- [Obsidian 同步（可选）](#obsidian-同步可选)
- [本地运行](#本地运行)

---

## 功能特性

- 🤖 自动抓取 GitHub 账号 Star 的全部仓库
- 📝 为每个仓库读取 README，调用 AI 生成内容摘要和技术标签
- ⚡️ **高效率**：支持**并发调用** AI 接口，大幅提升处理大量新项目时的速度
- 🗃️ **数据驱动**：所有信息存储为 `data/stars.json`，支持二次开发
- 🎨 **模版驱动**：使用 Jinja2 模版生成 Markdown 和 HTML 静态页面
- ⏭️ 增量更新，已处理项目状态保存在 JSON 中，避免重复消耗 API
- ⏰ GitHub Actions **定时自动运行**，cron 表达式自由配置
- 🔄 可选：自动将生成的 `stars_zh.md` & `stars_en.md` **推送到 Obsidian Vault 仓库**
- 🌐 可选：自动同步到 **GitHub Pages** 分支，支持多语言 (ZH/EN) 切换与前端交互搜索
- 💻 支持任意 **OpenAI 格式兼容接口**（OpenAI / Azure / 本地 Ollama 等）

---

## 流程概览

```mermaid
graph TD
    Start([开始]) --> Trigger{触发方式}
    Trigger -- "Actions (定时/手动)" --> Sync[运行 sync_stars.py]
    Trigger -- "Local (本地运行)" --> Sync
    
    Sync --> FetchGH[抓取 GitHub Stars]
    FetchGH --> Filter{增量检查}
    Filter -- "已存在 (跳过)" --> Render
    Filter -- "新项目 (需处理)" --> FetchRD[获取 README]
    
    FetchRD --> AI[AI 智能摘要/标签]
    AI --> Store[(data/stars.json)]
    Store --> Render
    
    Render[[Jinja2 模板渲染]] --> Output
    
    subgraph Output [成果产出]
        MD[Markdown 归档]
        HTML[HTML 静态搜索页]
    end
    
    Output --> Dispatch{同步分发}
    Dispatch -- "VAULT_SYNC" --> Obs[推送至 Obsidian Vault]
    Dispatch -- "PAGES_SYNC" --> Pages[部署 GitHub Pages]
    
    Obs --> End([完成])
    Pages --> End
```

---


## 快速开始

### 第一步：Fork 本仓库

点击右上角 **Fork**，将本仓库复制到你自己的账号下。

### 第二步：配置环境 (二选一)

本项目通过环境变量驱动，**配置优先级：GitHub Secrets > .env 文件**。

#### 方案 A：使用 GitHub 环境变量 (推荐，适合持续运行)

进入仓库的 **Settings → Secrets and variables → Actions** 进行配置：

**🔐 必填 Secrets**
- `AI_API_KEY`: 你的 AI 接口 API Key。

**🔐 可选 Secrets**
在 **Secrets** 标签页中配置，未设置时使用内置默认值：
- `AI_BASE_URL`: AI 接口地址 (默认使用 OpenAI 官方地址)。
- `AI_MODEL`: 模型名称 (默认 `gpt-4o-mini`)。

> 如果此前在 Variables 中配置了 `AI_BASE_URL` 和 `AI_MODEL`，请将其值复制到同名 Secrets；工作流改为仅从 Secrets 读取这两项。确认 Secrets 已设置后，可删除原来的 Variables。

**📋 必填 Variables**
- `GH_USERNAME`: 要抓取 Stars 的 GitHub 用户名。

**📋 可选 Variables**
以下参数有内置默认值，通常无需配置：
- `OUTPUT_FILENAME`: 生成文件的基准名 (默认 `stars`)。
- `VAULT_SYNC_PATH`: Vault 里的存放目录 (默认 `GitHub-Stars/`)。
- `PAGES_SYNC_ENABLED`: 是否同步到 Pages (默认 `true`)。

> [!TIP]
> **关于 GitHub API 限制**：
> - **线上运行 (Actions)**：工作流会自动注入 `GITHUB_TOKEN`，额度高达 1,000次/小时，抓取全量 Stars 无压力。
> - **本地运行**：若不配置 `GH_TOKEN`，API 限制为 60次/小时。若 Stars 较多，建议在 `.env` 中填入 `GH_TOKEN` 以提升额度至 5,000次/小时。

#### 方案 B：使用 .env 文件 (适合本地开发)

1. 在仓库根目录，复制 `.env.example` 并重命名为 `.env`。
2. 在 `.env` 中填入必填项。

---

### 第三步：自定义定时频率

编辑 `.github/workflows/sync.yml`，修改 `cron` 表达式：

```yaml
schedule:
  - cron: "0 2 * * 1"  # 示例：每周一凌晨 2 点运行
```

### 第四步：手动触发首次运行

进入 **Actions → 🌟 GitHub Stars Index同步 → Run workflow**，点击运行。

---

## 配置项详解

| 变量名               | 类型 | 说明                       | 默认值                      |
| -------------------- | ---- | -------------------------- | --------------------------- |
| `GH_USERNAME`        | 必填 | 要同步的 GitHub 用户名     | -                           |
| `AI_API_KEY`         | 必填 | AI 接口 Key                | -                           |
| `AI_BASE_URL`        | 可选 | OpenAI 兼容接口地址        | `https://api.openai.com/v1` |
| `AI_MODEL`           | 可选 | 使用的 AI 模型             | `gpt-4o-mini`               |
| `OUTPUT_FILENAME`    | 可选 | 生成 MD/HTML 的文件名基准  | `stars`                     |
| `VAULT_SYNC_ENABLED` | 可选 | 是否开启 Obsidian 同步     | `false`                     |
| `VAULT_REPO`         | 选填 | Vault 仓库 (`owner/repo`)  | -                           |
| `VAULT_SYNC_PATH`    | 可选 | Vault 同步的目录路径       | `GitHub-Stars/`             |
| `PAGES_SYNC_ENABLED` | 可选 | 是否开启 GitHub Pages 部署 | `true`                      |
| `MAX_CONCURRENCY`    | 可选 | AI 并发处理数 (建议 1-10)  | `1`                         |
| `GH_TOKEN`           | 选填 | 本地运行时提升 API 额度    | -                           |

---

## Obsidian 同步（可选）

1. **设置 Token**: 创建一个具有 **Contents: Write** 权限的 [Fine-grained PAT](https://github.com/settings/personal-access-tokens) 存入 Secret `VAULT_PAT`。
2. **配置仓库**: 设置 `VAULT_SYNC_ENABLED=true` 和 `VAULT_REPO=用户名/仓库名`。
3. **设置路径**: 设置 `VAULT_SYNC_PATH`，文件将自动同步到该目录下（自动补全 `_zh.md` / `_en.md`）。

---

## GitHub Pages 部署（可选）

本项目自动生成支持多语言、支持实时搜索的静态网页：

1. 确保 `PAGES_SYNC_ENABLED=true`。
2. 运行一次 Action 后，进入 **Settings -> Pages**。
3. **Branch** 选择 `gh-pages`，目录选择 `/(root)`，保存。

---

## 本地运行

```bash
# 克隆仓库并安装依赖
git clone https://github.com/iblogc/GithubStarsIndex.git
cd GithubStarsIndex

# 安装依赖
pip install -r requirements.txt

# 使用 .env 进行配置
cp .env.example .env
# 编辑 .env 填入 AI_API_KEY 和 GH_USERNAME

# [常规运行] 获取原信息、调用 AI 总结并渲染页面
python scripts/sync_stars.py

# [仅渲染模式] 跳过抓取和 AI 总结，仅依据本地 stars.json 极速重新渲染 HTML/MD
# 适用于前端样式调试、模版修改、多语言翻译调整等场景
python scripts/sync_stars.py --render-only
```

---

## 文件说明

| 文件                         | 说明                                 |
| :--------------------------- | :----------------------------------- |
| `data/stars.json`            | **核心数据集**（抓取的全量项目数据） |
| `templates/`                 | Jinja2 生成模版（Markdown/HTML）     |
| `dist/`                      | 自动生成的本地成品（HTML / MD）      |
| `scripts/sync_stars.py`      | 核心同步与生成脚本                   |
| `.github/workflows/sync.yml` | GitHub Actions 定时工作流            |
| `.env.example`               | 配置示例文件                         |
