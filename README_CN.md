# News to WeChat

> Claude Code / Cursor Skill：联网搜索最新资讯 → 生成主题文章 → 推送微信公众号草稿箱，全流程自动化。

[English](README.md)

## 功能特性

- **资讯搜索 → 文章生成** — 搜索多个来源，自动生成结构化 Markdown 文章
- **主题化 HTML 转换** — 将 Markdown 转为微信兼容的内联 CSS HTML，支持多主题切换
- **封面图自动生成** — 自动生成 900×383 封面图，5 种风格预设可选
- **微信草稿推送** — 图片上传到微信 CDN，通过官方 API 创建草稿
- **2 套内置主题** — `tech-digest`（白底卡片编辑感）和 `news-minimal`（极简深色标题）
- **自定义主题** — 复制主题文件夹，修改 `theme.yaml` 即可

## 快速开始

### 前置条件

- Python 3.8+
- [Claude Code](https://claude.com/claude-code) 或 [Cursor](https://cursor.sh)（支持 Skill）

### 安装依赖

```bash
pip install mistune pygments pyyaml Pillow wechatpy cryptography requests

# 可选：封面图生成需要
pip install playwright && playwright install chromium
```

### 配置微信凭证

```bash
cp data/credentials.example.yaml data/credentials.yaml
# 编辑 data/credentials.yaml，填入你的微信公众号 App ID 和 App Secret
```

### 使用方式（在 Claude Code 中）

```
/news-to-wechat 今天的 AI 新闻
```

或直接对话：

> 搜索最新的 AI 新闻，帮我生成一篇公众号文章。

## 工作流程

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  联网搜索    │────▶│  生成         │────▶│  转换为        │────▶│  推送微信     │
│  (3-5 次)   │     │  Markdown    │     │  风格化 HTML   │     │  草稿箱      │
└─────────────┘     └──────────────┘     └───────────────┘     └──────────────┘
```

### 阶段 1：资讯搜索
使用 `WebSearch` 多角度搜索 5-10 条高质量资讯。

### 阶段 2：文章生成
生成包含 frontmatter（标题、副标题、作者、日期、分类）的结构化 Markdown。

### 阶段 3：HTML 转换
```bash
python3 scripts/md_to_styled_html.py article.md -t tech-digest -o article.html
```

### 阶段 3.5：封面图生成
```bash
python3 scripts/generate_cover.py article.md --style accent-bar -o cover.jpg
```

### 阶段 4：推送到微信
```bash
python3 scripts/publish_wechat.py publish --workspace ./workspace
```

## 项目结构

```
news-to-wechat/
├── SKILL.md                        # Skill 定义与工作流
├── scripts/
│   ├── md_to_styled_html.py        # Markdown → 主题化 HTML 转换器
│   ├── generate_cover.py           # 封面图生成器（HTML 渲染）
│   ├── publish_wechat.py           # 微信发布 CLI
│   ├── publish/
│   │   ├── base.py                 # 基类（凭证加载、工作区 IO）
│   │   └── wechat.py               # 微信 API 实现
│   └── themes/
│       ├── tech-digest/            # 默认主题（卡片 + 编号图标）
│       └── news-minimal/           # 极简主题（深色标题）
├── references/
│   ├── article-templates.md        # 文章结构模板
│   └── style-guide.md              # 主题配置指南
├── examples/
│   └── sample-article.md           # 示例文章
└── data/
    ├── credentials.example.yaml    # 凭证模板
    └── memory.md                   # 持久化偏好记录
```

## 自定义主题

1. 复制现有主题：
   ```bash
   cp -r scripts/themes/tech-digest scripts/themes/my-theme
   ```

2. 编辑 `my-theme/theme.yaml` — 修改颜色、排版、图标

3. 使用新主题：
   ```bash
   python3 scripts/md_to_styled_html.py article.md -t my-theme -o article.html
   ```

详见 [references/style-guide.md](references/style-guide.md)。

## 封面风格预设

| 风格 | 说明 | 适用场景 |
|------|------|---------|
| `gradient` | 渐变背景 + 居中卡片 | 通用资讯 |
| `accent-bar` | 深色背景 + 左侧强调条 | 技术 / AI 类 |
| `split` | 左文右色块双栏 | 对比类文章 |
| `minimal` | 渐变背景 + 大号标题 | 短标题 |
| `geometric` | 浅色背景 + 几何装饰 | 编辑感、资讯类 |

## 依赖

| 包 | 用途 |
|---|------|
| `mistune` | Markdown 解析 |
| `pygments` | 代码语法高亮 |
| `pyyaml` | YAML 配置加载 |
| `Pillow` | 图片处理 |
| `wechatpy` + `cryptography` | 微信 API 认证 |
| `requests` | HTTP 请求 |
| `playwright` | 封面图渲染（可选） |

## 参与贡献

详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 安全

详见 [SECURITY.md](SECURITY.md)。

## 许可证

[MIT](LICENSE) © 2026 Mileson

## 作者

- X: [Mileson07](https://x.com/Mileson07)
- 小红书: [超级峰](https://xhslink.com/m/4LnJ9aB1f97)
- 抖音: [超级峰](https://v.douyin.com/rH645q7trd8/)
