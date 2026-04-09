# 风格配置指南

## 主题架构

每个主题是一个独立目录，位于 `scripts/themes/` 下：

```
scripts/themes/
├── tech-digest/          # 默认主题
│   ├── theme.yaml        # 主题配置
│   └── theme.css         # 自定义 CSS（预留）
└── news-minimal/         # 极简主题
    ├── theme.yaml
    └── theme.css
```

## theme.yaml 完整字段

```yaml
# 颜色配置
colors:
  primary: "#667eea"          # 主色调（标题、链接、分割线）
  secondary: "#764ba2"        # 辅助色（渐变结束色）
  accent: "#f093fb"           # 强调色
  text: "#2d3748"             # 正文颜色
  text_secondary: "#718096"   # 次要文字颜色
  bg_code: "#1a1a2e"          # 代码块背景
  bg_code_inline: "#edf2f7"   # 行内代码背景
  bg_blockquote: "#f7fafc"    # 引用块背景
  link: "#667eea"             # 链接颜色
  bold_highlight: "#fef3c7"   # 加粗文字底色高亮
  headings:                   # H2 标题轮换颜色
    - "#667eea"
    - "#e53e3e"
    - "#38a169"

# 排版配置
typography:
  font_size: 15               # 正文字号 (px)
  line_height: 1.85           # 行高
  letter_spacing: 0.3         # 字间距 (px)

# 代码高亮风格（Pygments style name）
code_style: "monokai"

# H2 分区图标（按顺序轮换）
section_icons:
  - "1️⃣"
  - "2️⃣"
  - "3️⃣"

# 顶部 Header 配置
header:
  # style: "card"（白底卡片，编辑感）或 "gradient"（渐变色满铺）
  style: "card"
  icon: "🚀"                  # Header 图标
  show_date: true             # 是否显示日期
  # card 模式专属
  highlight_color: "#e53e3e"  # 标题 {{关键词}} 高亮色
  category_color: "#667eea"   # 分类标签颜色
  border_color: "#e2e8f0"     # 卡片边框颜色
  default_category: ""        # 默认分类标签（frontmatter 未指定时使用）
  # gradient 模式专属
  gradient_start: "#667eea"   # 渐变起始色
  gradient_end: "#764ba2"     # 渐变结束色

# 底部 Footer 配置
footer:
  enabled: true
  text: ""                    # 自定义文字（空则显示 "— END —"）
```

## 创建自定义主题

1. 复制现有主题：
   ```bash
   cp -r ~/.cursor/skills/news-to-wechat/scripts/themes/tech-digest \
          ~/.cursor/skills/news-to-wechat/scripts/themes/my-theme
   ```

2. 修改 `my-theme/theme.yaml` 中的颜色和配置

3. 使用时指定主题名：
   ```bash
   python3 ~/.cursor/skills/news-to-wechat/scripts/md_to_styled_html.py \
     article.md -t my-theme -o article.html
   ```

## Pygments 代码高亮风格

常用 `code_style` 值：

| 风格名 | 适用场景 |
|--------|---------|
| `monokai` | 深色背景，经典暗色 |
| `github-dark` | GitHub 暗色风格 |
| `one-dark` | Atom One Dark 风格 |
| `dracula` | Dracula 主题 |
| `solarized-dark` | Solarized 暗色 |
| `solarized-light` | Solarized 亮色 |
| `friendly` | 浅色友好风格 |

## 微信兼容性说明

- 所有样式通过 `style` 属性内联，不依赖外部 CSS
- 图片最大宽度 677px
- 外部链接自动转脚注（微信不支持外链跳转）
- 不使用 JavaScript
- 字体使用系统字体栈
