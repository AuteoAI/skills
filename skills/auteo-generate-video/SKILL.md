---
name: auteo-generate-video
description: Generate AI videos with the user's Auteo (auteo.ai) account — text-to-video, image-to-video (first / last frame) and reference-to-video with Seedance 2.5, Seedance 2, MiniMax H3, FastWan and LTX-Video. Use when the user asks to make, animate or render a video clip, an ad, a shot, or to bring an image to life, even if they do not mention Auteo. Spends the user's credits (videos are the most expensive). NOT for still images (use auteo-generate-image) or music (use auteo-generate-music).
license: UNLICENSED
allowed-tools: Bash, Read
metadata:
  version: 0.1.0
  requires:
    bins: [auteo]
---

# Generate videos with Auteo

Uses the `auteo` CLI (`npm i -g @auteo/cli`, then `auteo auth login`). If a command exits with code 3, follow the auteo-account skill first.

## 1. Pick a model and read its parameters

```bash
auteo model list --type text_to_video --json          # also image_to_video, reference_to_video
auteo model get bytedance/seedance-2 --json
```

| Model                       | Inputs                               | Notes                            |
| --------------------------- | ------------------------------------ | -------------------------------- |
| `bytedance/seedance-2-5`    | text, reference images / videos      | top quality, sound on by default |
| `bytedance/seedance-2`      | text, reference images / videos      | 480P–4K, sound optional          |
| `bytedance/seedance-2-fast` | text, reference images / videos      | cheaper and faster, up to 720P   |
| `minimax/minimax-h3`        | text, first / last frame, references | 768P or 2K, native audio         |
| `fastvideo/fastwan`         | text only                            | very cheap 16:9 drafts           |
| `lightricks/ltx-video`      | text or a first frame                | cheap 480P drafts                |

Always confirm with `model get`: it lists the duration range, resolution tiers, aspect ratios and which asset options exist.

## 2. Choose parameters

- `--duration <seconds>` and `--resolution <tier>` are required. Price = per-second rate × duration, so keep drafts short (5 s) and low (`480P` / `720P`) unless the user asks for more.
- `--aspect-ratio`: required for text-only and reference input, not allowed with `--first-frame` / `--last-frame` (the frame decides the shape).
- Frames: `--first-frame <path|url>` and optionally `--last-frame` to animate an image.
- References: `--image <path|url>` (repeatable) are reference images; mention them in the prompt as "image 1", "image 2". `--reference-video <local.mp4>` where supported. Do not mix frames and references.
- `--audio` / `--no-audio` only for models that list `generateAudio`.
- Describe motion and camera in the prompt ("slow dolly-in", "the fox starts running"), not just the scene.

## 3. Check the price, then generate

```bash
auteo generate cost bytedance/seedance-2 --duration 5 --resolution 720P --json
auteo generate create bytedance/seedance-2 --prompt "the fox starts running through the snow, tracking shot" \
  --first-frame ./images/fox.png --duration 5 --resolution 720P --out ./videos --json
```

Videos take minutes. `--out` waits (default up to 600 s, change with `--wait-timeout`) and downloads the mp4. If it is still processing when the wait ends, the command exits 1 with `done: false`; run `auteo generate wait <requestId> --out ./videos` again instead of submitting a new one.

Report the file path, duration and credits spent. Ask before generating several videos in a row; check the balance with `auteo account --json` first.

## When it fails

- Exit 4, not enough credits: stop, tell the user the price and balance, give the top-up link (see auteo-account). Do not shorten the clip or lower the tier on your own.
- Exit 2: re-read `model get`; common causes are a missing `--resolution`, a duration outside the range, or `--aspect-ratio` passed together with frames.
- Exit 5: the prompt was rejected by moderation; rewrite it.
- Exit 6: too many generations in progress; wait for running ones (`auteo generate list --type video`).
- Timeout during `create`: check `auteo generate list --type video --limit 5 --json` before retrying, and retry with the same `--client-request-id`.

## MCP tools

`auteo_list_models`, `auteo_get_model`, `auteo_estimate_cost`, `auteo_upload_image` (frames and reference images → `assetId`), `auteo_generate_video`, `auteo_get_generation` (repeat after `pollAfterMs`), `auteo_list_generations`. The remote MCP server cannot read local files: pass public https URLs to `auteo_upload_image`.
