---
name: template-builder
description: Fill all 40 ad creative prompt templates with brand DNA context for a specific product. Requires brand-dna to have been run first. Saves ready-to-generate prompts to output/{brand}-{product}-prompts.md.
argument-hint: [product name or description]
disable-model-invocation: true
---

# /ads-creative:template-builder

Fill all 40 prompt templates with brand-specific values, producing ready-to-use prompts for image generation.

## Input

`$ARGUMENTS` — product name or description (e.g. `Protein Bar Chocolate Fudge 60g`).

If `$ARGUMENTS` is empty, ask the user:
> Produk apa yang ingin dibuatkan template iklannya?
> Contoh: `Protein Bar Chocolate Fudge 60g` atau `Whey Protein Vanilla 1kg`

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

For **every one of the 40 templates**, fill all `[PLACEHOLDER]` values using:

| Placeholder type | Source |
|-----------------|--------|
| Brand colors (primary, accent, etc.) | Brand DNA — Visual System |
| Font / typography references | Brand DNA — Visual System |
| Photography style descriptors | Brand DNA — Photography Style |
| Surface, background, props | Brand DNA — Product Photography Direction |
| Packaging details (color, finish, shape) | Brand DNA — Packaging |
| Product name | User's `$ARGUMENTS` input |
| Product description details | User's `$ARGUMENTS` + infer reasonable details |
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

Save to `output/{brand-name}-{product-slug}-prompts.md` in the current working directory.

```markdown
# Filled Prompts: [Brand Name] — [Product Name]

**Brand:** [brand name]
**Product:** [product name from $ARGUMENTS]
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

Repeat for all 40 templates.

---

## Post-Output

After saving:
1. Confirm: `✅ 40 template telah diisi dan disimpan ke output/{brand}-{product}-prompts.md`
2. Remind: `Untuk generate gambar, jalankan: /ads-creative:template-generate [nomor 1-40]`
3. Show a preview of Template 1's filled prompt as a sample.
