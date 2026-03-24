# ads-creative — Claude Code Plugin

Automated ads creative generation using the [viostudio.id](https://viostudio.id) API.

Research a brand, fill prompt templates with brand context, then generate images via `nano-banana-2`.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| Brand DNA | `/ads-creative:brand-dna [url\|description]` | Research a brand and create a Brand DNA document |
| Template Builder | `/ads-creative:template-builder [product] [id\|"all"]` | Fill one template or all 40 with brand context |
| Template Generate | `/ads-creative:template-generate [1-40]` | Upload reference images and generate from a filled template |

## Workflow

```
1. /ads-creative:brand-dna https://www.yourbrand.com
        ↓ creates brands/yourbrand/brand-dna.md
        ↓ instructs you to add reference images + set API key

2. Add product/logo/packaging photos to brands/yourbrand/
   export VIOSTUDIO_API_KEY=vio_sk_xxxxx

3. /ads-creative:template-builder "Product Name 100g" 5
        ↓ fills template 5, saves/appends to output/yourbrand-product-prompts.md

4. /ads-creative:template-generate 5
        ↓ uploads reference images, generates image, saves log to output/generated/
```

## Installation

### From GitHub (recommended)

```
/plugin marketplace add aidityasadhakim/ads-creative-skill
/plugin install ads-creative@ads-creative
/reload-plugins
```

### Local testing

```bash
git clone https://github.com/aidityasadhakim/ads-creative-skill
claude --plugin-dir ./ads-creative-skill
```

### Update

```
/plugin marketplace update ads-creative
/reload-plugins
```

## Setup

### API key

```bash
export VIOSTUDIO_API_KEY=vio_sk_xxxxx
```

Add to `.bashrc` / `.zshrc` to persist across sessions. Get your key at [viostudio.id](https://viostudio.id) → Settings → API Keys.

### Reference images

After running `brand-dna`, place product photos, logo, and packaging shots in the brand directory:

```
brands/yourbrand/
├── brand-dna.md        ← created automatically
├── logo.png
├── product-front.jpg
└── packaging.jpg
```

Supported formats: **JPEG, PNG** (max 20 MB per file). Images are **mandatory** — `template-generate` will not proceed without them.

## Requirements

- Claude Code 1.0.33+
- `VIOSTUDIO_API_KEY` environment variable
- `curl` available in your shell

## Template Builder modes

```bash
# Fill one template (fast — recommended)
/ads-creative:template-builder "Protein Bar 60g" 5

# Fill a few specific templates
/ads-creative:template-builder "Protein Bar 60g" 1
/ads-creative:template-builder "Protein Bar 60g" 12

# Fill all 40 at once
/ads-creative:template-builder "Protein Bar 60g" all
```

Single-template mode appends to the prompts file (or replaces an existing entry) without touching other templates.

## Project output structure

Files are created in your **current working directory**, not inside the plugin:

```
your-project/
├── brands/
│   └── yourbrand/
│       ├── brand-dna.md          # Brand DNA document
│       ├── logo.png              # Your reference images
│       └── product-front.jpg
└── output/
    ├── yourbrand-product-prompts.md   # Filled prompt templates
    └── generated/
        └── template-5-*.md            # Generation logs with image URLs
```

## Notes

- All 40 prompt templates are in **Bahasa Indonesia** — optimized for `nano-banana-2`, do not translate
- Default model: `nano-banana-2` (3 credits/image, up to 10 reference images)
- Reference images are uploaded to viostudio.id at generation time via `POST /v1/assets`
- `brand-dna` must run before `template-builder`
- `template-builder` must run for the requested template before `template-generate`

## API

- **Base URL:** `https://api.viostudio.id/v1`
- **Auth:** `Authorization: Bearer {VIOSTUDIO_API_KEY}`
- **Default model:** `nano-banana-2`

## License

MIT
