---
name: template-generate
description: Generate an image from a filled prompt template using viostudio.id API (nano-banana-2). Takes a template ID (1-40). Uploads brand reference images, submits generation, polls for completion, and saves the result log.
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

Read the prompts file and extract:
- **Brand Directory** — from the header line `**Brand Directory:** brands/{brand-name}/`
- **Full prompt text** — section matching `## Template $ARGUMENTS` (or `## Template $ARGUMENTS —`), everything under `### Prompt` until the next `---`
- **Aspect ratio** — scan the prompt text for patterns like `Rasio aspek 4:5`, `rasio 4:5`, `aspect ratio 4:5`, or `4:5`. Default to `1:1` if none found.

### 4. Reference Images

Scan the Brand Directory parsed above for image files with extensions: `.jpg`, `.jpeg`, `.png` (case-insensitive). Exclude `assets.json` and `brand-dna.md` from the scan.

If the Brand Directory is missing or no images are found:
> ⚠️ Tidak ada foto referensi ditemukan di `brands/{brand-name}/`.
>
> Tambahkan foto produk, logo, atau packaging ke folder tersebut sebelum generate.
> Format: JPEG atau PNG, maks. 20 MB per file.
>
> Contoh:
> ```
> brands/{brand-name}/
> ├── brand-dna.md
> ├── logo.png
> ├── product-front.jpg
> └── packaging.jpg
> ```

Then STOP.

**Asset cache check:**

Check if `brands/{brand-name}/assets.json` exists and read it if so. The file has this structure:

```json
{
  "uploaded_at": "2026-03-24T15:00:00Z",
  "assets": [
    {"filename": "logo.png", "asset_id": 123, "url": "https://s3.viostudio.id/..."},
    {"filename": "product-front.jpg", "asset_id": 124, "url": "https://s3.viostudio.id/..."}
  ]
}
```

Apply the following logic:

1. **Cache is stale (older than 25 days):** Ignore the cache entirely — re-upload all images. The viostudio.id API expires assets after 30 days; the 25-day threshold gives a 5-day safety buffer.

2. **Cache exists and is fresh:**
   - Build a map of `filename → asset_id` from the cache
   - For each image file found in the Brand Directory:
     - If filename exists in the cache → use the cached `asset_id`, skip upload
     - If filename is **not** in the cache (new file added) → upload it, collect the new `asset_id`
   - Ignore cache entries whose filename no longer exists in the directory (file was deleted)
   - If any new files were uploaded, update `assets.json` with the new entries (keep existing ones, append new ones, update `uploaded_at` to now)

3. **Cache does not exist:** Upload all images, create `assets.json`.

**To force a full re-upload** (e.g. user replaced a file with the same name), the user can delete `assets.json` and re-run.

After resolving asset IDs, report to user:
> 📸 [N] foto referensi siap (X dari cache, Y di-upload baru):
> - logo.png → asset ID 123 (cached)
> - product-front.jpg → asset ID 124 (cached)
> - new-photo.jpg → asset ID 125 (uploaded)

### 5. Check Credits

```bash
curl -s -X GET "https://api.viostudio.id/v1/account/credits" \
  -H "Authorization: Bearer $VIOSTUDIO_API_KEY"
```

- If the response indicates insufficient credits (`total_credits` < 3):
  > ⚠️ Credits tidak cukup untuk generate (butuh minimal 3 credits untuk nano-banana-2).
  > Sisa credits: [total_credits]

  Then STOP.

- If credits are sufficient, continue.

---

## Generation Flow

### Step 1 — Upload New Reference Images

Only upload image files that were **not** resolved from the cache in Pre-check 4. Skip this step entirely if all images were served from cache.

For each image that needs uploading:

```bash
curl -s -X POST "https://api.viostudio.id/v1/assets" \
  -H "Authorization: Bearer $VIOSTUDIO_API_KEY" \
  -F "file=@brands/{brand-name}/{filename}"
```

**Response** `201 Created`:
```json
{
  "asset_id": 123,
  "content_type": "image/jpeg",
  "url": "https://s3.viostudio.id/assets/...",
  "created_at": "..."
}
```

- Collect the new `asset_id` values and merge with cached IDs into a single array
- If any upload fails (non-201 response), show the error and ask the user whether to continue with successfully resolved images or STOP
- After uploads, write/update `brands/{brand-name}/assets.json` with all current entries and set `uploaded_at` to the current ISO timestamp

### Step 2 — Submit Generation

```bash
curl -s -X POST "https://api.viostudio.id/v1/images/generate" \
  -H "Authorization: Bearer $VIOSTUDIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nano-banana-2",
    "prompt": "[FULL PROMPT TEXT]",
    "aspect_ratio": "[ASPECT RATIO]",
    "reference_asset_ids": [123, 124, 125]
  }'
```

**Response** `202 Accepted`:
```json
{
  "generation_ids": [1001],
  "status": "queued",
  "credits_charged": 3,
  "estimated_seconds": 30
}
```

- Parse `generation_ids` (array) — use the first ID for polling
- Store `credits_charged` from this response
- If the response contains an error, display it and STOP

### Step 3 — Poll for Completion

Poll `GET /v1/generations/{generation_id}` using `generation_ids[0]`:

| Phase | Interval | Max attempts |
|-------|----------|-------------|
| Phase 1 | every 1s | 10 times (0–10s) |
| Phase 2 | every 5s | 10 times (10–60s) |
| Phase 3 | every 10s | until done or 10min total timeout |

```bash
curl -s -X GET "https://api.viostudio.id/v1/generations/{generation_id}" \
  -H "Authorization: Bearer $VIOSTUDIO_API_KEY"
```

**Response**:
```json
{
  "id": 1001,
  "status": "completed",
  "asset_url": "https://s3.viostudio.id/generations/...",
  "credits_charged": 3,
  "error_message": null
}
```

Update the user between phases:
> ⏳ Status: [status] — [elapsed]s

Possible statuses: `queued`, `processing`, `completed`, `failed`

If status is `failed`, report `error_message` and STOP.

If 10 minutes pass without completion, report a timeout and STOP.

### Step 4 — Handle Completion

On `completed`:
1. Extract `asset_url` from the poll response
2. Use `credits_charged` from the initial `202` response (Step 2)
3. Determine the file extension from the `asset_url` (e.g. `.jpg`, `.png`). Default to `.jpg` if unclear.
4. Download the image to `output/generated/template-{id}-{timestamp}{ext}`:

```bash
curl -s -L "{asset_url}" -o "output/generated/template-{id}-{timestamp}{ext}"
```

If the download fails (non-zero exit or empty file), warn the user but do not STOP — the image is still accessible via the URL.

---

## Save Result Log

Create directories if needed, then save to `output/generated/template-{id}-{timestamp}.md`:

```markdown
# Generated: Template {id}

- **Template:** {template name}
- **Model:** nano-banana-2
- **Aspect Ratio:** {ratio}
- **Status:** completed
- **Credits Used:** {credits_charged}
- **Generated At:** {ISO timestamp}
- **Image URL:** {asset_url}
- **Local File:** output/generated/template-{id}-{timestamp}{ext}
- **Reference Images:** {comma-separated filenames} (asset IDs: {comma-separated IDs})

## Prompt Used

{full prompt text}
```

---

## Final Output to User

```
✅ Template {id} berhasil di-generate!

🖼️  Image URL: {asset_url}
💾  Saved to: output/generated/template-{id}-{timestamp}{ext}
💳  Credits used: {credits_charged}
📸  Reference images: {N} foto (asset IDs: {ids})
📄  Log: output/generated/template-{id}-{timestamp}.md
```

---

## API Reference

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/account/credits` | GET | Check remaining credits |
| `/v1/assets` | POST | Upload reference image (multipart/form-data) |
| `/v1/images/generate` | POST | Submit image generation |
| `/v1/generations/{id}` | GET | Poll generation status |

**Auth header:** `Authorization: Bearer $VIOSTUDIO_API_KEY`
**Default model:** `nano-banana-2` — 3 credits/image
**Base URL:** `https://api.viostudio.id/v1`
**Max reference images:** 10 (nano-banana-2)
**Supported upload formats:** JPEG, PNG (max 20 MB each)
