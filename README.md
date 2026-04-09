# News to WeChat

> A Claude Code / Cursor Skill that searches the web for the latest news, generates themed articles, and publishes them to WeChat Official Account drafts — fully automated, zero user interaction.

[简体中文](README_CN.md)

## Features

- **Fully Automated Pipeline** — 6+1 stages from topic to WeChat draft, zero user confirmation needed
- **Three-Style Candidate Drafts** — Generates 3 drafts (Technical Expert / Storytelling / Sharp Humor), auto-scores and picks the best
- **Quality Scoring & Auto-Optimization** — 150-point scoring system, auto-optimizes if below 9/10 (up to 3 rounds)
- **Themed HTML Conversion** — Converts Markdown to WeChat-compatible inline-CSS HTML with configurable themes
- **Cover Image Generation** — Auto-generates 900×383 cover images with 5 style presets
- **WeChat Draft Publishing** — Uploads images to WeChat CDN and creates drafts via official API
- **2 Built-in Themes** — `tech-digest` and `news-minimal` (Anthropic brand colors)
- **Custom Themes** — Copy a theme folder, edit `theme.yaml`, done

## Quick Start

### Prerequisites

- Python 3.8+
- [Claude Code](https://claude.com/claude-code) or [Cursor](https://cursor.sh) with Skill support

### Install Dependencies

```bash
pip install mistune pygments pyyaml Pillow wechatpy cryptography requests
```

### Configure WeChat Credentials

```bash
cp data/credentials.example.yaml data/credentials.yaml
# Edit data/credentials.yaml with your WeChat App ID and App Secret
```

### Usage (in Claude Code)

```
/topic-to-wechat AI news today
```

Or give a direct prompt:

> Search for the latest AI news and create a WeChat article about it.

## How It Works

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  Topic    │──▶│  Titles  │──▶│  Outline  │──▶│  3-Style     │──▶│  Article  │──▶│  Score   │──▶│  Publish │
│  Search   │   │  (auto)  │   │  (auto)   │   │  Candidates  │   │  Draft    │   │  (auto)  │   │  (auto)  │
└──────────┘   └──────────┘   └──────────┘   └──────────────┘   └──────────┘   └──────────┘   └──────────┘
```

### Stage 1: Topic Selection
User provides a topic or auto-fetches trending topics via WebSearch.

### Stage 2: Title Generation
Generates 5 candidate titles with hooks (number / emotion / question / contrast / identity), scores each, auto-selects the highest.

### Stage 3: Outline Generation
Generates a 3-5 section outline with key points and data sources. Fact-checking strictness varies by topic type.

### Stage 4A: Three-Style Candidates
Parallel generates 3 candidate drafts (40-60% length):

| Style | Description |
|-------|-------------|
| Technical Expert | Precise, data-driven |
| Storytelling | Scene-based narrative |
| Sharp Humor | Witty commentary |

Auto-scores all 3 and selects the best style.

### Stage 4B: Full Article
Expands the winning draft into a complete 1500-3000 word article.

### Stage 5: Quality Scoring
150-point system → 10-point scale. Auto-optimizes if below 9/10 (max 3 rounds).

### Stage 6: Publish
Converts to styled HTML → generates cover → pushes to WeChat draft. Fully automatic.

## Project Structure

```
topic-to-wechat/
├── SKILL.md                        # Skill definition & workflow
├── scripts/
│   ├── md_to_styled_html.py        # Markdown → themed HTML converter
│   ├── generate_cover.py           # Cover image generator
│   ├── publish_wechat.py           # WeChat publisher CLI
│   ├── publish/
│   │   ├── base.py                 # Base class (credential loading)
│   │   └── wechat.py               # WeChat API implementation
│   └── themes/
│       ├── tech-digest/            # Default theme (Anthropic brand colors)
│       └── news-minimal/           # Minimal theme (Anthropic brand colors)
├── references/
│   ├── article-templates.md        # Article structure templates
│   ├── style-guide.md              # Theme configuration guide
│   ├── title-generator.md          # Title generation rules
│   ├── outline-guardrails.md       # Outline guardrails
│   ├── quality-scoring.md          # Quality scoring model
│   ├── writing-voice.md            # Writing voice preferences
│   └── style-libraries/            # 3 writing style definitions
│       ├── technical-expert.md
│       ├── storytelling.md
│       └── sharp-humor.md
├── examples/
│   └── sample-article.md           # Example article
└── data/
    ├── credentials.example.yaml    # Credential template
    └── memory.md                   # Persistent preferences
```

## Custom Themes

1. Copy an existing theme:
   ```bash
   cp -r scripts/themes/tech-digest scripts/themes/my-theme
   ```

2. Edit `my-theme/theme.yaml` — change colors, typography, icons

3. Use it:
   ```bash
   python3 scripts/md_to_styled_html.py article.md -t my-theme -o article.html
   ```

See [references/style-guide.md](references/style-guide.md) for full configuration options.

## Cover Style Presets

| Style | Description | Best For |
|-------|-------------|----------|
| `gradient` | Gradient bg + centered card | General news |
| `accent-bar` | Dark bg + left accent bar | Tech / AI articles |
| `split` | Text left + color block right | Comparison articles |
| `minimal` | Gradient bg + large title | Short titles |
| `geometric` | Light bg + geometric decorations | Editorial feel |

## Dependencies

| Package | Purpose |
|---------|---------|
| `mistune` | Markdown parsing |
| `pygments` | Code syntax highlighting |
| `pyyaml` | YAML config loading |
| `Pillow` | Image processing |
| `wechatpy` + `cryptography` | WeChat API authentication |
| `requests` | HTTP requests |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Security

See [SECURITY.md](SECURITY.md) for reporting vulnerabilities.

## License

[MIT](LICENSE) © 2026 Mileson

## Author

- X: [Mileson07](https://x.com/Mileson07)
- Xiaohongshu: [超级峰](https://xhslink.com/m/4LnJ9aB1f97)
- Douyin: [超级峰](https://v.douyin.com/rH645q7trd8/)
