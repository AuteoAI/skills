---
name: auteo-generate-speech
description: Turn text into natural speech (text to speech, voiceover, narration) with the user's Auteo (auteo.ai) account using VoxCPM 2 and IndexTTS 2.5 and a set of preset English and Mandarin voices. Use when the user asks to read text aloud, make a voiceover, narration, an audio version of a script, or a spoken line in a given voice or mood, even if they do not mention Auteo. Spends the user's credits, priced per character. NOT for songs or singing (use auteo-generate-music) and NOT for sound inside a video (use auteo-generate-video with audio).
license: UNLICENSED
allowed-tools: Bash, Read
metadata:
  version: 0.1.0
  requires:
    bins: [auteo]
---

# Generate speech with Auteo

Uses the `auteo` CLI (`npm i -g @auteo/cli`, version 0.2.0 or later, then `auteo auth login`). If a command exits with code 3, follow the auteo-account skill first.

## 1. Pick a model and a voice

```bash
auteo model list --type text_to_speech --json
auteo model get openbmb/voxcpm2 --json    # the voiceId param lists every preset voice
```

| Model                    | Notes                                                                                |
| ------------------------ | ------------------------------------------------------------------------------------ |
| `openbmb/voxcpm2`        | default choice: natural voice cloning; `--instruction` is a free-form speaking style |
| `indexteam/indextts-2.5` | expressive speech; `--instruction` describes the emotion to convey                   |

Both models share the same preset voices. In `model get`, the `voiceId` parameter has `voices[]` with each voice's `id`, `name`, `gender`, `language` (`en` or `zh`) and `description`. Pick a voice whose language matches the text; the spoken language follows the text itself.

## 2. Check the price

Speech is priced per character of text and per take: 2 credits per 1000 characters, at least 0.5 credits per take, rounded up to 0.1. Subscriptions do not make it free. Always price the exact text first:

```bash
auteo generate cost openbmb/voxcpm2 --prompt "$(cat script.txt)" --json
```

`--n 2` asks for two takes and costs twice as much. The text is at most 2000 characters per request; split longer scripts into several requests.

## 3. Generate

The text to speak goes in `--prompt`; `--voice` is required.

```bash
auteo generate create openbmb/voxcpm2 --prompt "$(cat script.txt)" --voice nova \
  --instruction "warm, unhurried, like a bedtime story" --out ./speech --json
```

Options:

- `--instruction <text>` (up to 200 characters): on VoxCPM 2 a speaking style ("whispering", "excited sports commentary"); on IndexTTS 2.5 an emotion ("calm and reassuring", "angry").
- `--n <1-4>` for several takes to choose from; each take is charged.
- `--seed <number>` to make a result repeatable; take i uses seed + i.

## 4. Wait and deliver

Speech usually takes seconds to a minute. `--out` waits and downloads the mp3 files (`<requestId>-1.mp3`, …); the JSON `outputs[]` has each take's `audioUrl` and `durationSeconds`. If the wait ends first, run `auteo generate wait <requestId> --type speech --out ./speech`.

Tell the user the file paths and durations. With several takes, let the user pick.

## When it fails

- Exit 4, not enough credits: stop, tell the user the price and balance, give the top-up link (see auteo-account).
- Exit 5: the text or instruction was rejected by content moderation. Rewrite it; resending the same text is rejected again.
- Exit 2: missing `--prompt` or `--voice`, unknown voice, text over 2000 characters or instruction over 200, `--n` outside 1–4. Fix and retry.
- Exit 6: too many generations in progress; wait for running ones.
- Timeout during `create`: run `auteo generate list --type speech --limit 5 --json` before retrying, and retry with the same `--client-request-id`.

## MCP tools

`auteo_list_models` (`type: text_to_speech`), `auteo_get_model` (voices), `auteo_estimate_cost` (`model`, `text`, `n`), `auteo_generate_speech` (`text`, `voiceId`, `instruction`, `seed`, `n`), `auteo_get_generation` (repeat after `pollAfterMs`), `auteo_list_generations` (`type: speech`).
