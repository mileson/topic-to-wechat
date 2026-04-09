---
name: news-to-wechat
description: |
  联网获取最新资讯，生成专题文章，转换为风格化 HTML（可配置主题），并推送到微信公众号草稿箱。
  完整 pipeline：搜索资讯 → 生成 Markdown 文章 → Markdown 转风格化 HTML → 推送微信草稿。
  触发场景：(1) 用户要求搜索最新资讯并生成文章，(2) 用户提到"资讯文章"、"新闻推送"、
  "微信公众号文章"、"news to wechat"，(3) 用户给定话题要求生成公众号文章，
  (4) 用户要求将 Markdown 转为风格化 HTML 并推送微信。
---

# News to WeChat

联网获取最新资讯 → 生成专题文章 → 转换风格化 HTML → 推送微信公众号草稿。

## 凭证规则

微信公众号 API 凭证通过 Skill 本地配置文件管理，secrets-vault 作为兜底回退。

**凭证文件**：`data/credentials.yaml`（位于本 Skill 目录下）

```yaml
wechat_mp:
  app_id: "你的 AppID"
  app_secret: "你的 AppSecret"
```

**读取优先级**：
1. `data/credentials.yaml` — 优先读取，字段非空即生效
2. `secrets-vault`（`wechat_mp` namespace）— 本地未配置时自动回退

禁止在代码或工作目录中硬编码凭证。

## 工作流（4 阶段）

### Stage 1：资讯获取

使用 WebSearch 搜索用户指定话题的最新资讯。

**执行步骤**：

1. 确认话题关键词（用户直接提供或交互确认）
2. 使用 WebSearch 搜索 3-5 次，覆盖不同角度
3. 对搜索结果去重、按相关性排序
4. 提取 5-10 条高质量资讯条目，每条记录：标题、摘要、来源 URL、日期

**搜索策略**：
- 优先搜索最近 7 天内容
- 搜索词包含年份（当前 2026）确保时效性
- 多角度搜索：`"{话题} 最新进展 2026"` + `"{话题} news latest"` + `"{话题} 趋势"`

### Stage 2：生成 Markdown 文章

将收集到的资讯整理为结构化 Markdown。

**文章结构**（参见 [references/article-templates.md](references/article-templates.md)）：

```yaml
frontmatter:
  title: "文章标题，支持 {{高亮关键词}} 语法"
  subtitle: 副标题（可选）
  author: 作者名
  date: YYYY年MM月DD日
  category: 分类标签（显示在卡片顶部，如 "AI 前沿速递"）
  source: 来源信息（可选，显示在日期旁）

body:
  - H2 分区（每条资讯一个 H2）
  - 每个分区包含：背景介绍 + 核心内容 + 关键引用/代码
  - 外部链接使用 [文本](URL) 格式
```

**标题高亮语法**：用 `{{}}` 包裹需要高亮的关键词，生成 HTML 时自动渲染为主题配置的 `highlight_color`。
示例：`"大模型与 Agent {{最新突破}}"` → "最新突破" 显示为红色。

**写作规范**：
- 每个 H2 对应一条资讯主题
- 开篇 1-2 句话概括要点
- 关键数据或代码用代码块展示
- 链接保留原始 URL，转换时自动变脚注
- 总字数 1500-3000 字

### Stage 3：Markdown → 风格化 HTML

使用内置转换脚本将 Markdown 转为微信兼容的风格化 HTML。

**脚本路径**：

```bash
CONVERTER="python3 ~/.cursor/skills/news-to-wechat/scripts/md_to_styled_html.py"
```

**基础用法**：

```bash
# 使用默认主题 (tech-digest) 转换
$CONVERTER article.md -o article.html

# 指定主题
$CONVERTER article.md -t news-minimal -o article.html

# 转换并预览
$CONVERTER article.md -o article.html -p

# 查看可用主题
$CONVERTER --list-themes dummy.md
```

**可用主题**：

| 主题 | 风格 | 适用场景 |
|------|------|---------|
| `tech-digest`（默认） | 白底卡片标题 + 分类标签 + 高亮关键词 + 编号分区 | 技术资讯、AI 日报 |
| `news-minimal` | 极简深色标题 + 圆点分区 | 新闻简报、行业分析 |

**自定义主题**：复制 `scripts/themes/tech-digest/` 为新目录，修改 `theme.yaml` 即可。
详见 [references/style-guide.md](references/style-guide.md)。

**脚本输出 JSON**：

```json
{
  "title": "文章标题",
  "theme": "tech-digest",
  "html_path": "/path/to/article.html",
  "html_length": 12345
}
```

### Stage 3.5：封面图处理

微信公众号接口要求 `thumb_media_id`（封面图素材 ID），缺少封面图将导致发布失败。
本 Skill 采用**优先级回退**策略自动处理封面：

**决策流程**：

```
用户指定了封面图？
  ├─ YES → 使用用户指定的封面（metadata.yaml 中 medias.cover.path）
  └─ NO  → 自动生成封面图
            ├─ 读取文章 frontmatter（title / subtitle / category）
            ├─ 匹配当前主题配色（继承 theme.yaml 颜色）
            ├─ 自动选择或指定风格预设
            └─ 生成 900×383 (2.35:1) JPEG → 写入 Output/wechat/cover.jpg
```

**封面生成脚本**：

```bash
COVER_GEN="python3 ~/.cursor/skills/news-to-wechat/scripts/generate_cover.py"

# 从 Markdown frontmatter 读取信息，自动选择风格
$COVER_GEN article.md -o Output/wechat/cover.jpg

# 指定主题（继承配色）
$COVER_GEN article.md -t news-minimal -o Output/wechat/cover.jpg

# 指定风格预设
$COVER_GEN article.md --style accent-bar -o Output/wechat/cover.jpg

# 从 metadata.yaml 读取信息
$COVER_GEN --metadata Output/wechat/metadata.yaml -o Output/wechat/cover.jpg

# 查看可用风格
$COVER_GEN --list-styles
```

**可用封面风格**：

| 风格 | 说明 | 适用场景 |
|------|------|---------|
| `auto`（默认） | 根据标题长度和分类自动选择 | 通用 |
| `gradient` | 渐变背景 + 标题居中 | 长标题、通用资讯 |
| `accent-bar` | 深色背景 + 左侧强调条 + 左对齐 | 技术类、AI 类 |
| `split` | 左文右色块双栏 | 对比类、VS 类 |
| `minimal` | 纯色背景 + 大标题 | 短标题 |
| `geometric` | 几何装饰 + 分类标签 | 编辑感、资讯类 |

**脚本输出 JSON**：

```json
{
  "style": "accent-bar",
  "theme": "tech-digest",
  "size": "900x383",
  "output_path": "/path/to/cover.jpg",
  "file_size": 38100
}
```

**生成封面后必须更新 `metadata.yaml`**，添加 `medias.cover.path` 字段：

```yaml
medias:
  cover:
    path: "cover.jpg"
```

### Stage 4：推送微信公众号草稿

使用内置微信发布脚本（独立副本，不依赖 content-publisher）。

**前置准备**：

1. 在工作目录下创建 `Output/wechat/` 结构：
   ```
   workspace/
   ├── Output/wechat/
   │   ├── article.html      ← Stage 3 输出
   │   ├── cover.jpg          ← Stage 3.5 输出（用户指定或自动生成）
   │   └── metadata.yaml     ← 自动生成
   ```

2. `metadata.yaml` 格式：
   ```yaml
   platform:
     id: wechat
     display_name: 微信公众号
   article:
     title: "文章标题"
     digest: "文章摘要（120字节内）"
   author:
     name: "作者名"
   medias:
     cover:
       path: "cover.jpg"
   ```

**发布命令**：

```bash
PUBLISHER="python3 ~/.cursor/skills/news-to-wechat/scripts/publish_wechat.py"

# 创建草稿（默认安全模式）
$PUBLISHER publish --workspace <workspace_path>

# 创建草稿并自动发布（需谨慎）
$PUBLISHER publish --workspace <workspace_path> --auto-publish

# 查询发布状态
$PUBLISHER status --workspace <workspace_path> --publish-id <PUBLISH_ID>
```

默认行为：仅创建草稿，不自动发布。

**发布流程（6 步）**：
1. 认证 — 通过 secrets-vault 获取 `wechat_mp` 凭证
2. 加载元数据 — 读取 `Output/wechat/metadata.yaml`
3. 加载 HTML — 读取 `Output/wechat/article.html`
4. 上传图片 — 扫描 HTML 中的图片，上传到微信 CDN 并替换 URL
5. 创建草稿 — 调用 `draft/add` API
6. 发布（可选）— 调用 `freepublish/submit` API

## Agent 完整执行流程

收到用户请求后，按顺序执行：

```
1. 确认话题 → 用户交互或直接使用用户指定话题
2. WebSearch 搜索资讯 → 收集 5-10 条高质量条目
3. 生成 Markdown 文章 → 保存到临时工作目录
4. 转换 HTML → 调用 md_to_styled_html.py
5. 准备 Output/wechat/ 结构 → 复制 HTML + 生成 metadata.yaml
6. 封面图处理 → 优先级回退策略：
   a. 检查用户是否指定了封面图（metadata.yaml 中 medias.cover.path）
   b. 如有 → 直接使用
   c. 如无 → 调用 generate_cover.py 自动生成，并更新 metadata.yaml
7. 调用 publish_wechat.py → 创建微信草稿
8. 返回结果 → 报告 draft_id 和状态
```

**工作目录约定**：

```bash
# 在 Shell 当前工作目录（CWD）下创建工作区
WORKSPACE="$(pwd)/news-to-wechat-workspace/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$WORKSPACE/Output/wechat"
```

> **规则**：工作目录必须创建在 Agent 执行 Shell 命令时的当前工作目录下，
> 即用户触发 Skill 时所在的文件夹。禁止使用 `$HOME` 或其他硬编码路径。

## 持久记忆

本 Skill 支持持久记忆，记录用户偏好和常用话题。

**读取**：每次执行前读取 `data/memory.md`（位于本 Skill 目录下）。
**写入**：每次成功发布后追加记录，包括：
- 用户偏好的话题领域
- 常用主题选择
- 文章风格偏好

## 依赖

- Python 3.8+
- `pip install mistune pygments pyyaml`（Markdown 转 HTML）
- `pip install Pillow`（封面图自动生成）
- `pip install wechatpy cryptography requests`（微信发布）
- `secrets-vault` Skill（凭证管理，微信发布唯一凭证来源）
