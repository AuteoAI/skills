---
name: auteo-generate-image
description: Generate or edit images with the user's Auteo (auteo.ai) account — text-to-image and image-to-image with GPT Image, Nano Banana, Seedream, Z-Image and FLUX. Use when the user asks to create, draw, render, design or edit a picture, illustration, product shot, thumbnail, poster or any still image, even if they do not mention Auteo. Spends the user's credits. NOT for video (use auteo-generate-video) or music (use auteo-generate-music).
license: UNLICENSED
allowed-tools: Bash, Read
metadata:
  version: 0.1.0
  requires:
    bins: [auteo]
---

# Generate images with Auteo

Uses the `auteo` CLI (`npm i -g @auteo/cli`, then `auteo auth login`). If a command exits with code 3, follow the auteo-account skill to sign in first. MCP users can do the same with the `auteo_*` tools listed at the end.

## 1. Pick a model

```bash
auteo model list --type text_to_image --json     # or --type image_to_image for edits
auteo model get <model> --json                   # every parameter, options and constraints
```

Always read `model get` before generating: aspect ratios and resolution tiers differ per model, and `resolutionsByAspectRatio` says which tiers each ratio allows.

Rough guide (check the live list, it changes):

| Need                                  | Good first choice                                                |
| ------------------------------------- | ---------------------------------------------------------------- |
| Best overall quality, text in images  | `openai/gpt-image-2-6-sunburst`, `google/nano-banana-pro`        |
| Fast, cheap drafts                    | `google/nano-banana`, `alibaba/z-image`                          |
| Editing a photo with reference images | `google/nano-banana-2`, `bytedance/seedream-5-5`                 |
| Free for Plus / Ultra subscribers     | models with `subscriberUnlimited: true` (e.g. `alibaba/z-image`) |

## 2. Choose parameters

- `--aspect-ratio` is required. Use what the user asked for; otherwise `1:1` for avatars and icons, `16:9` for banners and thumbnails, `9:16` for stories, `3:4` for portraits. Some models also offer `auto`.
- `--resolution` is required when the model has tiers (`1K` / `2K` / `4K`). Higher tiers cost more. Default to `1K` for drafts; only go higher when the user wants print or large display, and say it costs more.
- `--n` (1–4) makes several variations; each one is charged.
- `--image <path|https-url>` (repeatable) adds reference images for editing or style. Local files are uploaded; the number allowed is in `model get` (`inputAssetIds.max`).
- Write the prompt in English unless the user wants text in another language rendered in the image.

## 3. Check the price, then generate

```bash
auteo generate cost google/nano-banana-2 --aspect-ratio 16:9 --resolution 2K --json
auteo generate create google/nano-banana-2 --prompt "a red fox in fresh snow, cinematic light" \
  --aspect-ratio 16:9 --resolution 2K --out ./images --json
```

`--out <dir>` waits for the result and downloads the files; the JSON lists `files` and `outputs[].url`. Without `--out`/`--wait` the command returns a `requestId` immediately; fetch it later with `auteo generate wait <requestId> --out ./images`.

Tell the user the file paths and the credits spent (`cost`).

## When it fails

- Exit 4, not enough credits: stop and follow the auteo-account skill (tell the user the price and balance, link to top up). Do not lower settings on your own.
- Exit 2, invalid parameters: re-read `auteo model get <model>` and fix the ratio / tier combination.
- Exit 5, prompt rejected by moderation: rewrite the prompt without the flagged content; resending the same prompt fails again.
- Exit 6, limit: too many generations in progress or too many submissions; wait and retry.
- Timeout or network error during `create`: do not resubmit blindly. Run `auteo generate list --type image --limit 5 --json`; if the request is not there, retry with `--client-request-id <the id printed before>` so it cannot be charged twice.
- `status: failed` in the result: report `errorMessage`; if `suggestedModel` is set, offer it to the user.

## MCP tools

`auteo_list_models`, `auteo_get_model`, `auteo_estimate_cost`, `auteo_upload_image` (returns an `assetId` for `inputAssetIds`), `auteo_generate_image`, `auteo_get_generation` (call again after `pollAfterMs` until `done`), `auteo_list_generations`.
