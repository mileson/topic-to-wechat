# Contributing to News to WeChat

Thank you for your interest in contributing!

## How to Contribute

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Development Setup

```bash
# Install dependencies
pip install mistune pygments pyyaml Pillow wechatpy cryptography requests

# Optional: for cover image generation
pip install playwright
playwright install chromium
```

## Adding a New Theme

1. Copy an existing theme directory under `scripts/themes/`
2. Modify `theme.yaml` with your colors and typography settings
3. Test with `python3 scripts/md_to_styled_html.py article.md -t your-theme`

## Adding a New Cover Style

1. Add a new style function in `scripts/generate_cover.py`
2. Register it in `STYLE_PRESETS` dict
3. Test with `python3 scripts/generate_cover.py article.md --style your-style -o cover.jpg`

## Reporting Issues

- Use GitHub Issues for bug reports and feature requests
- Include steps to reproduce for bugs
- Include your Python version and OS

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
