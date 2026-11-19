# Auteo skills

Agent skills that teach Claude Code, Codex, Cursor and other coding agents to generate images, videos and music on [auteo.ai](https://auteo.ai) with your own account and credits.

## Quick start

```bash
npm i -g @auteo/cli && auteo auth login   # install the CLI and sign in once
npx skills add AuteoAI/skills             # install the skills into your agent
```

Then ask your agent things like "make a 16:9 thumbnail of a fox in the snow", "animate this image into a 5 second clip" or "write me a lo-fi track for coding".

## Install

The skills call the `auteo` command, so install and sign in first:

```bash
npm i -g @auteo/cli
auteo auth login            # opens auteo.ai in your browser; click Allow
auteo account               # shows your plan and credits
```

On a machine without a browser use `auteo auth login --no-browser`, or create a key in **Profile → API keys** (`https://auteo.ai/me/settings#api-keys`) and set `AUTEO_TOKEN=atk_...`.

Install all skills (works with Claude Code, Codex, Cursor and other agents supported by the [skills CLI](https://github.com/vercel-labs/skills)):

```bash
npx skills add AuteoAI/skills
```

Install only one:

```bash
npx skills add AuteoAI/skills --skill auteo-generate-video
```

List what is available without installing: `npx skills add AuteoAI/skills --list`.

## Skills

| Skill                  | What it does                                                                    |
| ---------------------- | ------------------------------------------------------------------------------- |
| `auteo-generate-image` | text-to-image and image editing with GPT Image, Nano Banana, Seedream, ...      |
| `auteo-generate-video` | text, first-frame and reference video with Seedance, MiniMax H3, ...            |
| `auteo-generate-music` | songs and instrumentals with Suno V6 and YuE2, from a description or lyrics     |
| `auteo-account`        | sign-in, plan, credit balance, price checks and what to do when credits run out |

Every generation spends credits from your auteo.ai account. The skills check prices before expensive jobs and stop to ask you to top up instead of retrying when credits run out.

## Prefer MCP?

The same capabilities are available as MCP tools (`auteo_generate_image`, `auteo_generate_video`, `auteo_generate_music`, ...):

- local: `claude mcp add auteo -- npx -y @auteo/cli@latest mcp`
- remote: `https://auteo.ai/api/mcp` with `Authorization: Bearer atk_...`

See the [@auteo/cli README](https://www.npmjs.com/package/@auteo/cli) for Cursor and Codex configs.

## Repository layout

```
skills/
  auteo-account/SKILL.md
  auteo-generate-image/SKILL.md
  auteo-generate-music/SKILL.md
  auteo-generate-video/SKILL.md
```

Each `SKILL.md` has `name` and `description` frontmatter; the description tells the agent when to use it.
