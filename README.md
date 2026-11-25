# X player card skill

This repository contains an agent skill for building a small web app that
runs inside an X (Twitter) post via a [player card](https://developer.x.com/en/docs/x-for-websites/cards/overview/player-card).

The skill, `x-player-card-app`, walks through the `twitter:card` / `twitter:player`
meta tags, the headers that let X embed the page in an iframe, how to check
the live URL with `curl`, and what to do if the post only shows a normal
preview.

## Install

Copy the skill folder into your agent's skills directory so the agent can
load `SKILL.md`. For example:

```text
.cursor/skills/x-player-card-app/SKILL.md
~/.claude/skills/x-player-card-app/SKILL.md
```

The source file in this repo lives at
`skills/x-player-card-app/SKILL.md`. After copying, restart or refresh the
agent if it only discovers skills at startup.

## Caveat

X may cache cards and may need to validate or approve player cards (or
whitelist the domain) before an interactive player appears. If a post of the
page URL shows only a regular link preview, that is a common outcome even
when the tags and headers are correct.
