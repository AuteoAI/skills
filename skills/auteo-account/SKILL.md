---
name: auteo-account
description: Check the user's Auteo (auteo.ai) account — sign-in state, plan, credit balance and models that are free for them — and handle "not logged in" or "not enough credits". Use when the user asks how many credits they have, what plan they are on, whether they can afford a generation, or before a large batch of image, video or music generations. Also use when another Auteo command fails with exit code 3 (not logged in) or 4 (not enough credits).
license: UNLICENSED
allowed-tools: Bash
metadata:
  version: 0.1.0
  requires:
    bins: [auteo]
---

# Auteo account and credits

Everything here goes through the `auteo` CLI (`npm i -g @auteo/cli`). Add `--json` when you need to read the result; it prints `{ "ok", "data", "error" }`.

## Check sign-in

```bash
auteo auth status --json
```

- `ok: true` → signed in; `data.email` is the account.
- Exit code 3 / `CLI.NOT_LOGGED_IN` → run `auteo auth login`. It opens auteo.ai in the browser; tell the user to click **Allow** there and wait for the command to finish (up to 5 minutes). On a machine without a browser use `auteo auth login --no-browser` and give the user the printed URL.
- A key created by hand in Profile → API keys (`https://auteo.ai/me/settings#api-keys`) can be passed as `AUTEO_TOKEN=atk_...` instead.

## Plan and credits

```bash
auteo account --json
```

Read from `data`:

| Field                       | Meaning                                                                        |
| --------------------------- | ------------------------------------------------------------------------------ |
| `plan`                      | `null` without a subscription, otherwise `tier` (`starter` / `plus` / `ultra`) |
| `credits.remaining`         | credits usable on any model                                                    |
| `unlimitedModels[].modelId` | models this user is not charged for right now                                  |
| `lowCredits`, `topUpUrl`    | balance is below the cheapest video; point the user to `topUpUrl`              |
| `agentLimits`               | API limits: in-progress generations, submissions per minute, daily free cap    |

Tell the user the balance in plain words. Do not show the token.

## Can they afford it?

Price one generation before submitting anything expensive (videos, 4K images, `--n 4`):

```bash
auteo generate cost <model> [same options you would pass to create] --json
```

`data.credits` is the list price, `data.enoughCredits` compares it with the balance, `data.unlimitedForYou` means it is free for this user. For a batch, multiply by the number of generations and compare with `credits.remaining`.

## When credits run out

Exit code 4 (`BILLING.INSUFFICIENT_CREDITS`), or `enoughCredits: false`:

1. Stop. Do not retry, and do not switch to a cheaper model or a lower resolution on your own.
2. Tell the user how many credits are needed and how many they have (the error includes both when available).
3. Give them the top-up link (`topUpUrl`, normally `https://auteo.ai/pricing`) and wait until they say they have topped up.

## Other failures

| Exit code | Meaning                     | What to do                                                            |
| --------- | --------------------------- | --------------------------------------------------------------------- |
| 3         | not logged in / key revoked | `auteo auth login`                                                    |
| 6         | rate or usage limit         | wait until `error.resetAt`, then retry once                           |
| 7         | CLI too old                 | `npm i -g @auteo/cli@latest`, then rerun                              |
| 1         | network or server error     | retry once; for a failed submission check `auteo generate list` first |
