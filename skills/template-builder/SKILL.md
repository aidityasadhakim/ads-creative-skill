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

Look for `brand-dna.md` files inside subdirectories of `brands/` in the current working directory (pattern: `brands/*/brand-dna.md`).

- If none found:
  > ⚠️ Brand DNA belum ada. Jalankan dulu:
  > `/ads-creative:brand-dna [url atau deskripsi brand]`

  Then STOP.

- If multiple brand directories exist, ask the user which one to use.

**2. Read Brand DNA**

Read the chosen `brands/{brand-name}/brand-dna.md` file. Extract and internalize all sections:
- Visual System (fonts, colors)
- Photography Style
- Product Photography Direction
- Packaging details
- Ad Creative Style
- **IMAGE PROMPT MODIFIER** paragraph — this is critical

**3. Read Templates**

Read the templates file at:
`${CLAUDE_PLUGIN_ROOT}/templates/image-templates.md`

This file contains 40 numbered templates. Each template has placeholders in `[SQUARE BRACKETS]`.

---

## Filling Process

For **each template to be filled** (either just the one requested, or all 40), fill all `[PLACEHOLDER]` values using:

| Placeholder type | Source |
|-----------------|--------|
| Brand colors (primary, accent, etc.) | Brand DNA — Visual System |
| Font / typography references | Brand DNA — Visual System |
| Photography style descriptors | Brand DNA — Photography Style |
| Surface, background, props | Brand DNA — Product Photography Direction |
| Packaging details (color, finish, shape) | Brand DNA — Packaging |
| Product name | User's product name input |
| Product description details | User's product name + infer reasonable details |
| Ad copy / headlines / hooks | Create on-brand copy using the 5 Brand Voice Adjectives |
| Mood / atmosphere adjectives | Brand DNA — Photography Style mood keywords |
| Post-it / sticky note colors | Match brand palette |
| CTA / caption text | Match brand's CTA style and tone |

### Rules

- **DO NOT translate** prompts to English — all prompts stay in Bahasa Indonesia
- **DO NOT alter** the technical structure of each prompt (lighting setups, ratios, etc.)
- **DO prepend** the IMAGE PROMPT MODIFIER paragraph to every prompt
- **DO make** every prompt fully self-contained — no placeholders remaining
- **DO write** ad copy (headlines, hooks, captions) that is creative, punchy, and on-brand
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
**Brand DNA:** brands/[brand-name]/brand-dna.md
**Brand Directory:** brands/[brand-name]/

---

## Template 1 — [Template Name]

**Category:** [category]
**Aspect Ratio:** [parsed from template, e.g. 4:5]

### Prompt

[Complete filled prompt, ready to send to image model]

---

## Template 2 — [Template Name]

...
```

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
