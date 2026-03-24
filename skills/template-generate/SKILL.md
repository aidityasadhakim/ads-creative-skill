---
name: template-generate
description: Generate an image from a filled prompt template using viostudio.id API (nano-banana-2). Takes a template ID (1-40). Checks credits, submits generation, polls for completion, and saves the result log.
argument-hint: [template_id 1-40]
disable-model-invocation: true
---

# /ads-creative:template-generate

Generate an image from a filled prompt template via viostudio.id API.

## Input

`$ARGUMENTS` — template number, 1 through 40.

If `$ARGUMENTS` is empty or not a valid number, ask:
> Template nomor berapa yang ingin di-generate? (1-40)
> Contoh: `/ads-creative:template-generate 5`

---

## Pre-checks

### 1. API Key

Check for `VIOSTUDIO_API_KEY` environment variable.

If not set, ask the user:
> ⚠️ API key viostudio.id belum dikonfigurasi.
>
> Set environment variable sebelum menjalankan Claude Code:
> ```bash
> export VIOSTUDIO_API_KEY=vio_sk_xxxxx
> ```
> Atau berikan API key-nya sekarang dan saya akan gunakan untuk sesi ini.

If user provides a key inline, use it for this session. Then STOP until key is available.

### 2. Filled Prompts File

Look for `*-prompts.md` files in the `output/` directory of the current working directory.

If none found:
> ⚠️ Belum ada filled prompts. Jalankan dulu:
> `/ads-creative:template-builder [nama produk]`

Then STOP.

If multiple files found, ask user which one to use.

### 3. Parse the Requested Template

Read the prompts file and extract the section matching Template `$ARGUMENTS`.

Look for the heading `## Template $ARGUMENTS` (or `## Template $ARGUMENTS —`).

Extract:
- **Full prompt text** — everything under `### Prompt` until the next `---`
- **Aspect ratio** — scan the prompt text for patterns like `Rasio aspek 4:5`, `rasio 4:5`, `aspect ratio 4:5`, or `4:5`. Default to `1:1` if none found.

### 4. Check Credits

```bash
curl -s -X GET "https://api.viostudio.id/v1/account/credits" \
  -H "Authorization: Bearer $VIOSTUDIO_API_KEY"
```

- If the response indicates insufficient credits (balance < 3), warn:
  > ⚠️ Credits tidak cukup untuk generate (butuh minimal 3 credits untuk nano-banana-2).
  > Sisa credits: [amount]
  
  Then STOP.

- If credits are sufficient, continue.

---

## Generation Flow

### Step 1 — Submit Generation

```bash
curl -s -X POST "https://api.viostudio.id/v1/images/generate" \
  -H "Authorization: Bearer $VIOSTUDIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nano-banana-2",
    "prompt": "[FULL PROMPT TEXT]",
    "aspect_ratio": "[ASPECT RATIO]"
  }'
```

Parse the response to get the generation ID (field may be `id`, `generation_id`, or similar).

If the response contains an error, display it and STOP.

### Step 2 — Poll for Completion

Poll `GET /v1/generations/{generation_id}` with these intervals:

| Phase | Interval | Max attempts |
|-------|----------|-------------|
| Phase 1 | every 1s | 10 times (0–10s) |
| Phase 2 | every 5s | 10 times (10–60s) |
| Phase 3 | every 10s | until done or 10min total timeout |

```bash
curl -s -X GET "https://api.viostudio.id/v1/generations/{generation_id}" \
  -H "Authorization: Bearer $VIOSTUDIO_API_KEY"
```

Update the user between phases:
> ⏳ Status: [status] — [elapsed]s

Possible statuses: `queued`, `processing`, `completed`, `failed`

If status is `failed`, report the error and STOP.

If 10 minutes pass without completion, report a timeout and STOP.

### Step 3 — Handle Completion

On `completed` response:
1. Extract the image URL from the response (field may be `asset_url`, `output_url`, `url`, or similar — check all)
2. Extract `credits_used` if present in the response

---

## Save Result Log

Create directories if needed, then save to `output/generated/template-{id}-{timestamp}.md`:

```markdown
# Generated: Template {id}

- **Template:** {template name}
- **Model:** nano-banana-2
- **Aspect Ratio:** {ratio}
- **Status:** completed
- **Credits Used:** {credits or "unknown"}
- **Generated At:** {ISO timestamp}
- **Image URL:** {url}

## Prompt Used

{full prompt text}
```

---

## Final Output to User

```
✅ Template {id} berhasil di-generate!

🖼️  Image URL: {url}
💳  Credits used: {credits}
📄  Log: output/generated/template-{id}-{timestamp}.md
```

---

## API Reference

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/account/credits` | GET | Check remaining credits |
| `/v1/images/generate` | POST | Submit image generation |
| `/v1/generations/{id}` | GET | Poll generation status |
| `/v1/assets` | POST | Upload reference image (if needed) |

**Auth header:** `Authorization: Bearer $VIOSTUDIO_API_KEY`  
**Default model:** `nano-banana-2` — 3 credits/image  
**Base URL:** `https://api.viostudio.id/v1`
