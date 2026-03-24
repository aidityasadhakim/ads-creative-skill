---
name: brand-dna
description: Research a brand and create a comprehensive Brand DNA document. Use when given a brand URL or description to analyze brand identity, visual system, photography style, packaging, and ad creative style. Saves output to brands/{brand-name}.md in the current project.
argument-hint: [url or brand description]
disable-model-invocation: true
---

# /ads-creative:brand-dna

Research a brand and produce a comprehensive Brand DNA document used for filling ad creative prompt templates.

## Input

`$ARGUMENTS` — URL to the brand's website, or a description of the brand (or both).

If `$ARGUMENTS` is empty, ask the user:
> Silakan berikan URL website brand atau deskripsi brand yang ingin dianalisis.
> Contoh: `https://www.meetmacros.com` atau `MeetMacros — meal prep delivery service`

---

## Execution

You are a Senior Brand Strategist performing a full reverse-engineering of a brand's visual and verbal identity. Every detail matters because output will feed into an AI image model that requires precise specifications.

### PHASE 1 — External Research

Use web search tools to find these in parallel:

1. **Design credits:** search `"who designed [Brand] branding"`, `"[Brand] design agency case study"`, `"[Brand] rebrand"`
2. **Brand assets:** search `"[Brand] brand guidelines pdf"`, `"[Brand] press kit"`, `"[Brand] style guide"`
3. **Typography:** search `"[Brand] font"`, `"[Brand] typeface"`, `"what font does [Brand] use"`
4. **Colors:** search `"[Brand] brand colors"`, `"[Brand] hex codes"`, `"[Brand] color palette"`
5. **Packaging:** search `"[Brand] packaging design"`, `"[Brand] unboxing"`, `"[Brand] product photography"`
6. **Ad creative:** search `"[Brand] facebook ads"` and `"[Brand] instagram ads"` to find current ad creative style
7. **Press & positioning:** search `"[Brand] brand story"`, `"[Brand] mission"`, `"[Brand] founding story"`

### PHASE 2 — On-Site Analysis

Fetch the brand URL and analyze:

1. **Voice & Tone:** Read hero copy, About page, product descriptions. Extract 5 defining adjectives.
2. **Photography style:** Describe lighting, color grading, composition, subjects.
3. **Typography on site:** Headline weight, body weight, letter-spacing, any distinctive treatments.
4. **Color usage:** Primary vs accent. Background colors. CTA button color.
5. **Layout density:** Airy or dense? Grid-based or organic?
6. **Packaging details:** Material, color, shape, label placement, texture, matte vs glossy, transparency.

### PHASE 3 — Competitive Context

Search for 2-3 direct competitors. Note their visual differentiation from the brand.

### PHASE 4 — Generate Brand DNA Document

Compile all research into the following structure and save to `brands/{brand-name}.md` (brand name lowercase, hyphenated, e.g. `brands/meet-macros.md`):

```markdown
# BRAND DNA: [BRAND NAME]

## OVERVIEW
- **Name:** 
- **Tagline:** 
- **Design Agency:** [if found, else "Not found"]
- **Brand Voice Adjectives:** [5 adjectives]
- **Positioning:** 
- **Competitive Differentiation:** 

## VISUAL SYSTEM
- **Primary Font:** [name + weight]
- **Secondary Font:** [name + weight, or "Same as primary"]
- **Primary Color:** [hex + description, e.g. "#1A1A2E — deep navy"]
- **Secondary Color:** [hex + description]
- **Accent Color:** [hex + description]
- **Background Color:** [hex + description]
- **CTA Color:** [hex + description]

## PHOTOGRAPHY STYLE
- **Lighting:** 
- **Color Grading:** 
- **Composition:** 
- **Subjects:** 
- **Mood Keywords:** [5-7 keywords, comma separated]

## PRODUCT PHOTOGRAPHY DIRECTION
- **Shot Style:** [flat lay / lifestyle / studio / hero / etc.]
- **Product Lighting:** 
- **Surface / Background:** 
- **Props & Styling:** 

## PACKAGING
- **Material:** 
- **Packaging Color:** 
- **Shape:** 
- **Label:** 
- **Finish:** [matte / glossy / soft-touch / etc.]
- **Distinctive Elements:** 

## AD CREATIVE STYLE
- **Primary Format:** [single image / carousel / video / UGC / etc.]
- **Ad Tone:** 
- **Recurring Visual Elements:** 
- **Text Overlay Style:** 
- **CTA Style:** 

## COMPETITIVE LANDSCAPE

| Competitor | Visual Differentiation |
|------------|----------------------|
| [name 1]   | [how they differ]    |
| [name 2]   | [how they differ]    |
| [name 3]   | [how they differ]    |

## IMAGE PROMPT MODIFIER

> [Write a 50-75 word paragraph in Bahasa Indonesia that densely summarizes the brand's visual identity. This paragraph will be prepended to every prompt template. Must cover: photography style, characteristic lighting, dominant color palette, typography treatment, mood/atmosphere, and brand differentiator. Write as direct instructions to an AI image model — not a description of the brand.]

## METADATA
- **Research Date:** [today's date]
- **Source URL:** [brand URL]
- **Notes:** 
```

---

## Post-Output

After saving the file:
1. Confirm: `✅ Brand DNA saved to brands/{brand-name}.md`
2. Show a quick summary: name, tagline, 5 voice adjectives, primary color
3. Tell user: `Sekarang jalankan /ads-creative:template-builder [nama produk] untuk mengisi 40 template prompt.`
