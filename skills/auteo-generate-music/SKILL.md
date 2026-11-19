---
name: auteo-generate-music
description: Generate songs and instrumental music with the user's Auteo (auteo.ai) account using Suno V6 (V6, V6 Mini, V6 Wild) and YuE2 — from a short description, or from your own lyrics, style and title. Use when the user asks for a song, a jingle, background music, a beat or a soundtrack, even if they do not mention Auteo. Spends the user's credits. NOT for sound effects inside a video (use auteo-generate-video with audio).
license: UNLICENSED
allowed-tools: Bash, Read
metadata:
  version: 0.1.0
  requires:
    bins: [auteo]
---

# Generate music with Auteo

Uses the `auteo` CLI (`npm i -g @auteo/cli`, then `auteo auth login`). If a command exits with code 3, follow the auteo-account skill first.

## 1. Pick a model

```bash
auteo model list --type text_to_music --json
auteo model get suno/v6 --json        # length limits for prompt, lyrics, style, title
```

| Model          | Notes                                                                                    |
| -------------- | ---------------------------------------------------------------------------------------- |
| `suno/v6`      | default choice: natural vocals, two songs per request, title and cover included          |
| `suno/v6-mini` | same price and limits as V6, lighter model                                               |
| `suno/v6-wild` | more experimental results                                                                |
| `m-a-p/yue2`   | one song per request, no cover; length follows the lyrics (lyrics up to 1500 characters) |

Each request is charged once, whatever you ask for. Check it with `auteo generate cost <model> --json`.

## 2. Two ways to ask

**Description mode** — only `--prompt`. The song, lyrics included, is written for you. Best when the user describes a mood or topic.

```bash
auteo generate create suno/v6 --prompt "an upbeat synth-pop song about shipping code on a Friday" --out ./music --json
```

**Custom mode** — any of `--lyrics`, `--style`, `--title`. Lyrics are sung as written. Best when the user supplies lyrics or wants control.

```bash
auteo generate create suno/v6 --lyrics "$(cat lyrics.txt)" --style "indie folk, acoustic guitar, warm" \
  --title "Harbor Lights" --gender female --out ./music --json
```

Options:

- `--instrumental` for no vocals. An instrumental can be just a `--style` ("lo-fi, chill, rain, piano").
- `--gender male|female` for the vocal; leave it out to let the model choose.
- Respect the limits from `model get` (for Suno V6: description 500 characters, lyrics 5000, style 1000, title 80).

## 3. Wait and deliver

Songs usually take one to a few minutes. `--out` waits and downloads the mp3 files (`<requestId>-1.mp3`, `-2.mp3`); the JSON `outputs[]` has each song's `title`, `durationSeconds` and `coverImageUrl`. If the wait ends first, run `auteo generate wait <requestId> --type music --out ./music`.

Tell the user the file paths and song titles. Most models return two takes; let the user pick.

## When it fails

- Exit 4, not enough credits: stop, tell the user the price and balance, give the top-up link (see auteo-account).
- Exit 2: nothing to generate (no prompt or lyrics, or an instrumental without style), text over the model's limit, or an invalid `--gender`. Fix and retry.
- Exit 6: too many generations in progress; wait for running ones.
- Timeout during `create`: run `auteo generate list --type music --limit 5 --json` before retrying, and retry with the same `--client-request-id`.

## MCP tools

`auteo_list_models` (`type: text_to_music`), `auteo_get_model`, `auteo_estimate_cost`, `auteo_generate_music` (`prompt`, `lyrics`, `style`, `title`, `instrumental`, `gender`), `auteo_get_generation` (repeat after `pollAfterMs`), `auteo_list_generations` (`type: music`).
