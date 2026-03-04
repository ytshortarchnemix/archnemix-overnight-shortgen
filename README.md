# archnemix-overnight-shortgen

> **Archnemix ShortGen** — Overnight batch video generation pipeline.
> A proprietary tool under the [Archnemix](https://shortgen-archx.pages.dev) brand.

---

Overnight batch processing pipeline for **Archnemix ShortGen**. Users submit scripts via the ShortGen frontend and receive a unique claim code. Jobs are queued into batches and processed automatically via GitHub Actions using a self-contained Docker image. Finished videos are uploaded to a private HuggingFace dataset and retrieved by the user with their claim code.

> ⚠️ **This repository is public for GitHub Actions infrastructure reasons only.**
> Viewing is permitted. Using, copying, or forking this code is not.
> See [LICENSE](./LICENSE) for full terms.

---

## Architecture

```
User submits via frontend
        ↓
archnemix-overnight-controller (HF Space)
  - Generates claim code: ARCHX-XX-0-X-00-XXXX
  - Finds batch slot in GitHub repo (fills existing, creates new if full)
  - Writes Videos/batch{N}/{CLAIM_CODE}/manifest.json to GitHub via API
        ↓
GitHub push event fires overnight_batch.yml
  - find_ready_batch.py checks: is any batch full OR >1 hour old?
  - If yes: selects that batch, sets BATCH_DIR output
  - If no: workflow exits cleanly (just a check)
        ↓
Docker container (ghcr.io/YTShortMakerArchx/archnemix-overnight:latest)
  process_batch.py runs for each job in the batch:
    1. Clean script (ban words, unicode, aligner-breaking chars)
    2. Kokoro TTS (voice baked into image, 4 voices)
    3. MFA forced alignment → word timestamps
    4. Generate ASS subtitles (styled, burned in)
    5. FFmpeg compose (background video + audio + subtitles)
    6. Upload video.mp4 + status.json to HF dataset repo (HF_TOKEN_WR)
    7. Delete job folder from GitHub repo
        ↓
User polls /overnight/status/{claim_code}
  - Controller checks HF dataset repo for status.json
  - Returns download URL when complete
```

## Batch Rules

| Rule | Value |
|------|-------|
| Max jobs per batch | 5 |
| Timeout (process anyway) | 1 hour |
| Videos kept in HF dataset | 30 hours |
| Background video loop | Yes (if audio > bg duration) |
| Background trim | Random start point (if audio < bg duration) |
| Max script length | 3500 chars |
| Warning note for long videos | >3 minutes |

## Claim Code Format

```
ARCHX-XX-0-X-00-XXXX
│     │  │ │  │  └── 4 random uppercase letters
│     │  │ │  └───── literal "00"
│     │  │ └──────── 1 random uppercase letter
│     │  └────────── literal "0"
│     └───────────── 2 random uppercase letters
└─────────────────── fixed prefix "ARCHX"

Example: ARCHX-KQ-0-T-00-VZMR
```

## Voices Available

| Key | Kokoro Model | Description |
|-----|-------------|-------------|
| `male_high` | am_michael | Michael (US Male) — High Quality |
| `male_medium` | am_adam | Adam (US Male) — Clear |
| `female_high` | af_sarah | Sarah (US Female) — High Quality |
| `female_medium` | af_sky | Sky (US Female) — Natural |

All 4 voice models are baked into the Docker image — zero downloads at runtime.
