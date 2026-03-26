# skillify

Turn any public figure into a Claude Code skill system.

Give it a name, it researches their philosophy, voice, principles, and domain expertise, then generates a complete skill directory you can use immediately.

## What it does

1. **Researches** the person — interviews, podcasts, books, frameworks, direct quotes
2. **Presents findings** for your review before building anything
3. **Generates** a complete skill directory:
   - `ETHOS.md` — philosophical foundation written in their voice
   - Root `SKILL.md` — dashboard with skill catalog and next-step recommender
   - Sub-skill `SKILL.md` files — one per domain, each producing concrete artifacts
4. **Registers** everything so Claude Code discovers the new skills immediately

## Install

```bash
# Clone into your Claude Code skills directory
git clone https://github.com/yousef/skillify.git ~/.claude/skills/skillify
```

## Usage

```
/skillify Andy Grove
/skillify Charlie Munger
/skillify Tobi Lutke
```

Or just `/skillify` and describe who you want to encode.

## What gets generated

For `/skillify Andy Grove`, you'd get:

```
~/.claude/skills/grove/
  ETHOS.md                    # Grove's management philosophy in his voice
  SKILL.md                    # /grove dashboard
  one-on-one/SKILL.md         # /one-on-one — structured 1:1 format
  org-design/SKILL.md         # /org-design — functional vs mission-oriented
  crisis-management/SKILL.md  # /crisis-management — strategic inflection points
  ...
```

Each sub-skill produces **concrete artifacts** (documents, scripts, scorecards, templates) — not just advice.

## Constraints

- **Public figures only.** The person must have substantial searchable content (interviews, books, talks, podcasts). Skillify will not fabricate philosophy.
- **One skill system per person.** Each person gets their own slug and directory.
- **No build system.** Generated skills are plain SKILL.md files — no templates, no resolvers, no compilation step.

## Inspired by

Built after seeing [gstack](https://github.com/garry/gstack) encode Garry Tan's builder philosophy as Claude Code skills. Skillify generalizes the pattern: if someone's thinking is public and structured enough, it can become a skill system.

## License

MIT
