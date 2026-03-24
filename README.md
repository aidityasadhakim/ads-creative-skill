# ads-creative — Claude Code Plugin

Automated ads creative generation using the [viostudio.id](https://viostudio.id) API.

Research a brand, fill 40 prompt templates with brand context, then generate images via `nano-banana-2`.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| Brand DNA | `/ads-creative:brand-dna [url\|description]` | Research a brand and create a Brand DNA document |
| Template Builder | `/ads-creative:template-builder [product]` | Fill all 40 prompt templates with brand context |
| Template Generate | `/ads-creative:template-generate [1-40]` | Generate an image from a filled template |

## Workflow

```
1. /ads-creative:brand-dna https://www.yourbrand.com
        ↓ creates brands/yourbrand.md

2. /ads-creative:template-builder "Product Name 100g"
        ↓ creates output/yourbrand-product-prompts.md

3. /ads-creative:template-generate 1
        ↓ generates image, saves log to output/generated/
```

## Setup

### 1. Install the plugin

```bash
# Test locally
claude --plugin-dir ./ads-creative-skill

# Install for all your projects
claude plugin install ads-creative
```

### 2. Set your viostudio.id API key

```bash
export VIOSTUDIO_API_KEY=vio_sk_xxxxx
```

Get your API key at [viostudio.id](https://viostudio.id).

## Requirements

- Claude Code 1.0.33+
- `VIOSTUDIO_API_KEY` environment variable (the plugin will prompt you if missing)
- `curl` available in your shell

## Project Output Structure

Files are created in your **current working directory** (your project), not inside the plugin:

```
your-project/
├── brands/
│   └── yourbrand.md          # Brand DNA document
└── output/
    ├── yourbrand-product-prompts.md   # 40 filled prompts
    └── generated/
        └── template-1-*.md   # Generation logs with image URLs
```

## Notes

- All 40 prompt templates are in **Bahasa Indonesia** — optimized for `nano-banana-2`, do not translate
- Default model: `nano-banana-2` (3 credits/image)
- Brand DNA must be created before running template-builder
- Filled prompts must exist before running template-generate

## API

- **Base URL:** `https://api.viostudio.id/v1`
- **Auth:** `Authorization: Bearer {VIOSTUDIO_API_KEY}`
- **Default model:** `nano-banana-2`

## License

MIT
