---
name: template-builder
description: Fill ad creative prompt templates with brand DNA context for a specific product. Supports single template (fast) or all 40 at once. Saves ready-to-generate prompts to output/{brand}-{product}-prompts.md.
argument-hint: [product name] [template_id or "all"]
disable-model-invocation: true
---

# /ads-creative:template-builder

Fill prompt templates with brand-specific values, producing ready-to-use prompts for image generation.

## Input

`$ARGUMENTS` — product name followed by an optional template ID or `all`.

**Parsing rules** (apply in order):
1. If the last token is a number between 1 and 40 → treat it as template ID, rest is product name
2. If the last token is `all` (case-insensitive) → fill all 40, rest is product name
3. If there is no numeric/`all` token at the end → treat entire string as product name, then ask user (see below)

**Examples:**
```
/ads-creative:template-builder Protein Bar Chocolate 5    → fill template 5 only
/ads-creative:template-builder Protein Bar Chocolate all  → fill all 40
/ads-creative:template-builder Protein Bar Chocolate      → ask user
```

If product name is empty after parsing, ask:
> Produk apa yang ingin dibuatkan template iklannya?
> Contoh: `/ads-creative:template-builder Protein Bar Chocolate 5`

If no template ID or `all` was provided, ask:
> Template mana yang ingin diisi?
> - Ketik nomor (1-40) untuk satu template, contoh: `5`
> - Ketik `all` untuk mengisi semua 40 template sekaligus

---

## Pre-checks

**1. Check for Brand DNA**

Search for brand DNA files in both supported formats — check these two patterns in order:

1. **Subdirectory format (new):** `brands/*/brand-dna.md` — e.g. `brands/hotto-mame/brand-dna.md`
2. **Flat file format (legacy):** `brands/*.md` — e.g. `brands/hotto-mame.md` — any `.md` file directly inside `brands/`, excluding `assets.json`

Collect all matches from both patterns into a single candidate list.

- If no candidates found:
  > ⚠️ Brand DNA belum ada. Jalankan dulu:
  > `/ads-creative:brand-dna [url atau deskripsi brand]`

  Then STOP.

- If exactly one candidate found, use it automatically.
- If multiple candidates found, ask the user which one to use.

**2. Read Brand DNA**

Read the chosen brand DNA file. Extract and internalize all sections:
- Visual System (fonts, colors)
- Photography Style
- Product Photography Direction
- Packaging details
- Ad Creative Style
- **IMAGE PROMPT MODIFIER** paragraph — this is critical

**Determine Brand Directory for images:**
- If the file is `brands/{brand-name}/brand-dna.md` → Brand Directory is `brands/{brand-name}/`
- If the file is `brands/{brand-name}.md` (flat) → Brand Directory is `brands/{brand-name}/` (derive the name from the filename, strip `.md`)

**3. Read Reference Images**

Scan the Brand Directory for image files (`.jpg`, `.jpeg`, `.png`, case-insensitive). If found, **read every image file** so you have direct visual context of the actual product.

Use what you see in the images to:
- Identify exact product colors, packaging shape, label details, finish (matte/glossy), and texture
- Note any text, logo, or graphic elements visible on the product
- Observe physical proportions and distinctive visual features of the packaging

This eliminates hallucination — every prompt must describe what is **actually visible** in the photos, not assumed from text alone. The filled prompts must be specific enough that someone who has never seen the product could reconstruct it exactly from the prompt.

If no images are found:
> ⚠️ Tidak ada foto referensi di `brands/{brand-name}/`. Tambahkan foto produk sebelum mengisi template agar prompt tidak mengarang detail yang salah.

Then STOP.

**5. Read Templates**

Read the templates file at:
`${CLAUDE_PLUGIN_ROOT}/templates/image-templates.md`

This file contains 40 numbered templates. Each template has placeholders in `[SQUARE BRACKETS]`.

---

## Filling Process

For **each template to be filled** (either just the one requested, or all 40), fill all `[PLACEHOLDER]` values using:

| Placeholder type | Source |
|-----------------|--------|
| Brand colors (primary, accent, etc.) | Reference images + Brand DNA — Visual System |
| Font / typography references | Brand DNA — Visual System |
| Photography style descriptors | Brand DNA — Photography Style |
| Surface, background, props | Brand DNA — Product Photography Direction |
| Packaging details (color, finish, shape) | Reference images (primary source) + Brand DNA — Packaging |
| Product name | User's product name input |
| Product description details | Reference images (what is visually observable) |
| Ad copy / headlines / hooks | Create on-brand copy using the 5 Brand Voice Adjectives |
| Mood / atmosphere adjectives | Brand DNA — Photography Style mood keywords |
| Post-it / sticky note colors | Match brand palette from reference images |
| CTA / caption text | Match brand's CTA style and tone |

### Rules

- **HARD RULE — Language:** Every piece of copywriting (headlines, hooks, captions, CTA text, overlays) **must be in Bahasa Indonesia**. No exceptions. The literal string `Copywriting harus dengan bahasa indonesia.` is hard-injected at the end of every prompt in the output file — see Output Format below.
- **HARD RULE — Visual accuracy:** Every visual detail (packaging color, shape, label, finish, logo position) must be grounded in what is **actually visible in the reference images**. Do not invent or assume any detail not visible in the photos.
- **DO NOT translate** the prompt structure to English — all prompts stay in Bahasa Indonesia
- **DO NOT alter** the technical structure of each prompt (lighting setups, ratios, etc.)
- **DO prepend** the IMAGE PROMPT MODIFIER paragraph to every prompt
- **DO make** every prompt fully self-contained — no placeholders remaining
- **DO write** ad copy that is creative, punchy, and on-brand
- Keep copy short and natural — these are ad captions, not essays

---

## Output Format

### Output file path

Always save to `output/{brand-name}-{product-slug}-prompts.md` in the current working directory.

### Single template mode

If the output file already exists:
- If `## Template {id}` already exists in the file → **replace** that section with the newly filled version
- If it doesn't exist yet → **append** the new template section to the end of the file

If the output file does not exist → create it with the full header + the single template section.

### All mode

Always write the complete file from scratch (overwrite if exists).

### File structure

```markdown
# Filled Prompts: [Brand Name] — [Product Name]

**Brand:** [brand name]
**Product:** [product name from input]
**Date:** [today's date]
**Brand DNA:** [path to the brand DNA file that was read]
**Brand Directory:** brands/[brand-name]/

---

## Template 1 — [Template Name]

**Category:** [category]
**Aspect Ratio:** [parsed from template, e.g. 4:5]

### Prompt

[Complete filled prompt, ready to send to image model] Copywriting harus dengan bahasa indonesia.

---

## Template 2 — [Template Name]

...
```

> **CRITICAL:** The literal text `Copywriting harus dengan bahasa indonesia.` must be appended — as-is, verbatim — at the end of every single prompt block when writing to the file. This is a hard injection into the output, not a guideline. Do not paraphrase, translate, or omit it.

---

## Post-Output

After saving:

**Single template mode:**
1. Confirm: `✅ Template {id} telah diisi dan disimpan ke output/{brand}-{product}-prompts.md`
2. Show the filled prompt as a preview
3. Remind: `Untuk generate gambar, jalankan: /ads-creative:template-generate {id}`

**All mode:**
1. Confirm: `✅ 40 template telah diisi dan disimpan ke output/{brand}-{product}-prompts.md`
2. Show a preview of Template 1's filled prompt as a sample
3. Remind: `Untuk generate gambar, jalankan: /ads-creative:template-generate [nomor 1-40]`
