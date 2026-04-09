# News to WeChat

> Claude Code / Cursor Skill：联网搜索最新资讯 → 生成主题文章 → 推送微信公众号草稿箱，全流程自动化，零用户交互。

[English](README.md)

## 功能特性

- **全自动流水线** — 6+1 阶段从选题到微信草稿，全程无需用户确认
- **三风格候选稿** — 并行生成 3 份候选稿（技术专家 / 故事描述 / 幽默犀利），自动评分选最优
- **质量评分与自动优化** — 150 分制评分体系，低于 9/10 自动优化（最多 3 轮）
- **主题化 HTML 转换** — 将 Markdown 转为微信兼容的内联 CSS HTML，支持多主题切换
- **封面图自动生成** — 自动生成 900×383 封面图，5 种风格预设可选
- **微信草稿推送** — 图片上传到微信 CDN，通过官方 API 创建草稿
- **2 套内置主题** — `tech-digest` 和 `news-minimal`（Anthropic 品牌配色）
- **自定义主题** — 复制主题文件夹，修改 `theme.yaml` 即可

## 快速开始

### 前置条件

- Python 3.8+
- [Claude Code](https://claude.com/claude-code) 或 [Cursor](https://cursor.sh)（支持 Skill）

### 安装依赖

```bash
pip install mistune pygments pyyaml Pillow wechatpy cryptography requests
```

### 配置微信凭证

```bash
cp data/credentials.example.yaml data/credentials.yaml
# 编辑 data/credentials.yaml，填入你的微信公众号 App ID 和 App Secret
```

### 使用方式（在 Claude Code 中）

```
/topic-to-wechat 今天的 AI 新闻
```

或直接对话：

> 搜索最新的 AI 新闻，帮我生成一篇公众号文章。

## 工作流程

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  选题     │──▶│  标题    │──▶│  大纲    │──▶│  三风格       │──▶│  正文     │──▶│  评分    │──▶│  发布    │
│  (自动)   │   │  (自动)  │   │  (自动)  │   │  候选稿(自动) │   │  展开     │   │  (自动)  │   │  (自动)  │
└──────────┘   └──────────┘   └──────────┘   └──────────────┘   └──────────┘   └──────────┘   └──────────┘
```

### 阶段 1：选题
用户提供选题或自动抓取热门话题。

### 阶段 2：标题生成
生成 5 个候选标题（数字 / 情绪 / 问句 / 对比 / 身份认同钩子），自动评分选最高分。

### 阶段 3：大纲生成
生成 3-5 个分区大纲，防虚构检查根据题材类型条件触发。

### 阶段 4A：三风格候选稿
并行生成 3 份候选稿（目标正文 40-60% 长度）：

| 风格 | 说明 |
|------|------|
| 技术专家 | 精准严谨、数据驱动 |
| 故事描述 | 场景叙事、情感共鸣 |
| 幽默犀利 | 犀利吐槽、类比丰富 |

自动评分选最优风格。

### 阶段 4B：正文展开
按最优风格展开为完整 1500-3000 字文章。

### 阶段 5：质量评分
150 分制 → 10 分制，低于 9 分自动循环优化（最多 3 轮）。

### 阶段 6：排版与发布
转换为风格化 HTML → 生成封面图 → 自动推送微信草稿箱。

## 项目结构

```
topic-to-wechat/
├── SKILL.md                        # Skill 定义与工作流
├── scripts/
│   ├── md_to_styled_html.py        # Markdown → 主题化 HTML 转换器
│   ├── generate_cover.py           # 封面图生成器
│   ├── publish_wechat.py           # 微信发布 CLI
│   ├── publish/
│   │   ├── base.py                 # 基类（凭证加载）
│   │   └── wechat.py               # 微信 API 实现
│   └── themes/
│       ├── tech-digest/            # 默认主题（Anthropic 品牌配色）
│       └── news-minimal/           # 极简主题（Anthropic 品牌配色）
├── references/
│   ├── article-templates.md        # 文章结构模板
│   ├── style-guide.md              # 主题配置指南
│   ├── title-generator.md          # 标题生成规则
│   ├── outline-guardrails.md       # 大纲护栏规则
│   ├── quality-scoring.md          # 质量评分模型
│   ├── writing-voice.md            # 写作语气偏好
│   └── style-libraries/            # 3 种写作风格定义
│       ├── technical-expert.md
│       ├── storytelling.md
│       └── sharp-humor.md
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
