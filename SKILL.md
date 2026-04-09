---
name: topic-to-wechat
description: |
  全自动 6 阶段内容创作 Skill：选题 → 爆款标题 → 文章大纲 → 三风格候选稿 → 正文 → 排版 → 微信发布。
  全流程零用户交互，Agent 自主决策所有节点，自动评分选优。
  触发场景：(1) 用户要求搜索最新资讯并生成文章，(2) 用户提到"资讯文章"、"新闻推送"、
  "微信公众号文章"、"topic to wechat"，(3) 用户给定选题要求创作公众号文章，
  (4) 用户要求将 Markdown 转为风格化 HTML 并推送微信，
  (5) 用户提供选题/话题要求进行内容创作。
---

# News to WeChat

全自动 6+1 阶段内容创作：选题 → 标题 → 大纲 → 三风格候选 → 正文 → 排版 → 发布。

**全流程零用户交互**，Agent 自主完成所有决策。

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

## 工作流（6+1 阶段）

| 阶段 | 名称 | 产出 | 自动/交互 |
|:---:|:-----|:---------|:-------:|
| **1** | 选题 | 选题关键词 | 自动 |
| **2** | 标题 | 5 候选 + 评分 + 自动选定 | 自动 |
| **3** | 大纲 | 3-5 H2 分区大纲 | 自动 |
| **4A** | 三风格候选 | 3 份候选稿（40-60%） | 自动 |
| **4B** | 正文展开 | 完整 Markdown 文章 | 自动 |
| **5** | 质量评分 | 评分报告 + 自动优化 | 自动 |
| **6** | 排版+发布 | HTML + 封面 + 微信草稿 | 自动 |

### Stage 1：选题

确定要写的选题。支持两种入口：

**入口 A — 用户提供选题**：
1. 用户直接给出选题关键词或方向
2. 直接进入 Stage 2

**入口 B — 自动抓取选题**：
1. 使用 WebSearch 搜索当前热门话题（3-5 次搜索覆盖不同领域）
2. 搜索词策略：`"{领域} 最新进展 2026"` + `"{领域} 热门话题"` + `"{领域} 趋势"`
3. 从搜索结果中自动筛选最佳选题（标准：时效性 + 话题热度 + 写作价值）
4. 进入 Stage 2

**选题筛选标准**：
- 时效性：最近 7 天内发生的事件
- 信息量：有足够的数据/观点支撑一篇完整文章
- 受众面：目标读者群体会关心的话题

### Stage 2：标题

基于选题自动生成并选定爆款标题。

**执行步骤**：
1. 读取 [references/title-generator.md](references/title-generator.md) 获取完整规则
2. 生成 5 个不同方向的候选标题（每种钩子各一个）
3. 每个标题附带 1-10 分评分 + 一句话理由
4. 按评分降序排列，**自动选择最高分标题**
5. 直接进入 Stage 3

**快速规则**（无需读文件时使用）：
- 每个标题不超过 20 字
- 必须包含 5 种钩子之一：数字 / 情绪 / 问句 / 对比 / 身份认同
- 禁止标题党和陈词滥调
- 标题必须与内容实质相关

### Stage 3：大纲

自动生成文章骨架，防止跑题。

**执行步骤**：
1. 读取 [references/outline-guardrails.md](references/outline-guardrails.md) 获取护栏规则
2. 基于选题 + 标题，生成 3-5 个 H2 分区大纲
3. 每个分区标注：要点(key_points) + 数据来源(data_sources)
4. 根据题材类型决定防虚构检查严格程度（新闻类强制，故事类可选）
5. 直接进入 Stage 4A

**大纲格式**：
```
标题：[Stage 2 自动选定标题]
核心论点：[一句话]
题材类型：[新闻/数据类 | 故事/观点类 | 混合类]

## 分区 1：[H2 标题]
- 要点：...
- 来源：...

## 分区 2：...
```

### Stage 4A：三风格候选稿

并行生成 3 种不同风格的候选稿，自动评分选最优。

**执行步骤**：
1. 读取风格库文件获取各风格写作指导：
   - [references/style-libraries/technical-expert.md](references/style-libraries/technical-expert.md) — 技术专家：精准严谨、数据驱动
   - [references/style-libraries/storytelling.md](references/style-libraries/storytelling.md) — 故事描述：场景叙事、情感共鸣
   - [references/style-libraries/sharp-humor.md](references/style-libraries/sharp-humor.md) — 幽默犀利：犀利吐槽、类比丰富
2. 基于大纲，分别为每种风格生成候选稿（目标正文 40-60% 长度）
3. 读取 [references/quality-scoring.md](references/quality-scoring.md) 对 3 份候选稿评分
4. **自动选择最高分风格**进入 Stage 4B

**候选稿输出目录**：
```bash
$WORKSPACE/Output/_drafts/
├── candidate_technical_expert.md
├── candidate_storytelling.md
├── candidate_sharp_humor.md
└── scoring_report.md
```

### Stage 4B：正文展开

按最优风格展开完整正文。

**执行步骤**：
1. 读取 [references/writing-voice.md](references/writing-voice.md) 获取语气风格（如有积累）
2. 读取 [references/article-templates.md](references/article-templates.md) 获取文章结构参考
3. 基于最优风格候选稿 + 大纲，展开完整正文
4. 写作要求：
   - 保持选定风格的语气和结构
   - 每区 200-400 字
   - 关键数据用 blockquote 突出
   - 代码片段用代码块展示（如适用）
   - 外部链接使用 `[文本](URL)` 格式
   - `[待验证]` 项必须通过 WebSearch 核实
5. 总字数控制在 1500-3000 字
6. 生成完整 Markdown，包含 frontmatter：

```yaml
---
title: "[标题]"
subtitle: "[副标题]"
author: "[作者名]"
date: "YYYY年MM月DD日"
category: "[分类标签]"
source: "[来源信息]"
---
```

### Stage 5：质量评分

对完整正文自动评分，低于 9 分自动循环优化。

**执行步骤**：
1. 读取 [references/quality-scoring.md](references/quality-scoring.md) 获取评分体系
2. 对正文进行 150 分制评分（→ 10 分制）
3. **≥ 9.0 分**：通过，进入 Stage 6
4. **< 9.0 分**：自动诊断低分维度并优化（每轮聚焦最多 3 个低分维度）
5. 重新评分，最多循环 3 次
6. 3 次后仍未达标：标记 best_effort，继续进入 Stage 6

### Stage 6：排版 + 发布

自动排版并推送到微信公众号草稿箱。**必须自动执行，禁止询问用户是否发布。**

**排版 — Markdown 转 HTML**：

```bash
CONVERTER="python3 ~/.claude/skills/topic-to-wechat/scripts/md_to_styled_html.py"

# 使用默认主题 (tech-digest) 转换
$CONVERTER article.md -o article.html

# 指定主题
$CONVERTER article.md -t news-minimal -o article.html
```

**可用主题**：

| 主题 | 风格 | 适用场景 |
|------|------|---------|
| `tech-digest`（默认） | 白底卡片标题 + 分类标签 + 高亮关键词 + 编号分区 | 技术资讯、AI 日报 |
| `news-minimal` | 极简深色标题 + 圆点分区 | 新闻简报、行业分析 |

**封面图生成**：

```bash
COVER_GEN="python3 ~/.claude/skills/topic-to-wechat/scripts/generate_cover.py"
$COVER_GEN article.md -o Output/wechat/cover.jpg
```

**发布**：

前置文件：
```
workspace/Output/wechat/
├── article.html      ← 排版输出
├── cover.jpg          ← 封面图
└── metadata.yaml     ← 自动生成
```

**metadata.yaml 格式**：
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

**发布命令**（自动执行，无需确认）：

```bash
PUBLISHER="python3 ~/.claude/skills/topic-to-wechat/scripts/publish_wechat.py"

# 直接创建草稿并发布
$PUBLISHER publish --workspace <workspace_path> --auto-publish
```

> **规则**：Stage 6 必须 Agent 自动执行，禁止输出命令让用户手动执行，禁止询问"是否需要推送"。

## Agent 完整执行流程

```
1. 判断入口 → 用户提供选题 or 自动抓取 → 自动选定选题
2. Stage 2: 生成 5 候选标题 → 评分排序 → 自动选最高分
3. Stage 3: 生成大纲 → 根据题材决定防虚构严格度 → 自动确认
4. Stage 4A: 生成 3 风格候选稿 → 自动评分 → 选最优风格
5. Stage 4B: 按最优风格展开完整正文 → Markdown 文章
6. Stage 5: 质量评分 → 低于9分自动优化（最多3轮） → 通过
7. Stage 6: 转换 HTML → 封面图 → 自动推送微信草稿 → 返回 draft_id
```

**⚠️ 全流程禁止规则**：
- 全部 6 个阶段禁止使用 AskUserQuestion 询问用户
- 禁止在 Stage 6 询问"是否需要推送"或输出命令让用户手动执行
- 所有决策由 Agent 自主完成，自动选最优方案

**工作目录约定**：

```bash
# 第一步：先确认用户当前所在目录
pwd

# 第二步：以选题关键词命名创建工作文件夹
WORKSPACE="$(pwd)/{选题关键词}"
mkdir -p "$WORKSPACE/Output/wechat"
mkdir -p "$WORKSPACE/Output/_drafts"
```

> **规则**：
> - 执行前必须先用 `pwd` 确认用户当前工作目录
> - 工作文件夹以选题关键词命名（如选题为"AI编程趋势"则创建 `AI编程趋势/`）
> - 所有创作产物必须写入此工作文件夹内

## 持久记忆

本 Skill 支持持久记忆，记录用户偏好和常用话题。

**读取**：每次执行前读取 `data/memory.md`。
**写入**：每次成功完成后追加记录：
- 用户偏好的选题领域
- 常用主题选择
- 标题风格偏好
- 最优风格记录（哪种风格得分最高）
- 写作语气偏好（同步到 [references/writing-voice.md](references/writing-voice.md)）

## 依赖

- Python 3.8+
- `pip install mistune pygments pyyaml`（Markdown 转 HTML）
- `pip install Pillow`（封面图自动生成）
- `pip install wechatpy cryptography requests`（微信发布）
- `secrets-vault` Skill（凭证管理，微信发布兜底凭证来源）
