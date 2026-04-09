# News to WeChat

> A Claude Code / Cursor Skill that searches the web for the latest news, generates themed articles, and publishes them to WeChat Official Account drafts — fully automated.

[简体中文](README_CN.md)

## Features

- **Web Search → Article** — Searches multiple sources for a given topic and produces a structured Markdown article
- **Themed HTML Conversion** — Converts Markdown to WeChat-compatible inline-CSS HTML with configurable themes
- **Cover Image Generation** — Auto-generates 900×383 cover images with 5 style presets (gradient, accent-bar, split, minimal, geometric)
- **WeChat Draft Publishing** — Uploads images to WeChat CDN and creates drafts via official API
- **2 Built-in Themes** — `tech-digest` (editorial card style) and `news-minimal` (clean dark headings)
- **Custom Themes** — Copy a theme folder, edit `theme.yaml`, done

## Quick Start

### Prerequisites

- Python 3.8+
- [Claude Code](https://claude.com/claude-code) or [Cursor](https://cursor.sh) with Skill support

### Install Dependencies

```bash
pip install mistune pygments pyyaml Pillow wechatpy cryptography requests

# Optional: for cover image generation
pip install playwright && playwright install chromium
```

### Configure WeChat Credentials

```bash
cp data/credentials.example.yaml data/credentials.yaml
# Edit data/credentials.yaml with your WeChat App ID and App Secret
```

### Usage (in Claude Code)

```
/news-to-wechat AI news today
```

Or give a direct prompt:

> Search for the latest AI news and create a WeChat article about it.

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  Web Search  │────▶│  Generate     │────▶│  Convert to   │────▶│  Publish to  │
│  (3-5 queries)│    │  Markdown     │     │  Styled HTML  │     │  WeChat Draft│
└─────────────┘     └──────────────┘     └───────────────┘     └──────────────┘
```

### Stage 1: News Search
Uses `WebSearch` to find 5-10 high-quality articles across multiple angles.

### Stage 2: Article Generation
Produces structured Markdown with frontmatter (title, subtitle, author, date, category).

### Stage 3: HTML Conversion
```bash
python3 scripts/md_to_styled_html.py article.md -t tech-digest -o article.html
```

### Stage 3.5: Cover Image
```bash
python3 scripts/generate_cover.py article.md --style accent-bar -o cover.jpg
```

### Stage 4: Publish to WeChat
```bash
python3 scripts/publish_wechat.py publish --workspace ./workspace
```

## Project Structure

```
news-to-wechat/
├── SKILL.md                        # Skill definition & workflow
├── scripts/
│   ├── md_to_styled_html.py        # Markdown → themed HTML converter
│   ├── generate_cover.py           # Cover image generator (HTML rendering)
│   ├── publish_wechat.py           # WeChat publisher CLI
│   ├── publish/
│   │   ├── base.py                 # Base class (credential loading, workspace I/O)
│   │   └── wechat.py               # WeChat API implementation
│   └── themes/
│       ├── tech-digest/            # Default theme (card + icons)
│       └── news-minimal/           # Minimal theme (clean headings)
├── references/
│   ├── article-templates.md        # Article structure templates
│   └── style-guide.md              # Theme configuration guide
├── examples/
│   └── sample-article.md           # Example article
└── data/
    ├── credentials.example.yaml    # Credential template
    └── memory.md                   # Persistent preferences
```

## Creating Custom Themes

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
| `playwright` | Cover image rendering (optional) |

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
