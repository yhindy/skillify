---
name: skillify
version: 0.1.0
description: |
  MANUAL TRIGGER ONLY: invoke only when user types /skillify.
  Turn any public figure into a Claude Code skill system. Researches the person's
  philosophy, principles, voice, and domain expertise, then generates a complete
  skill directory with ETHOS.md, SKILL.md templates, and sub-skills. Restricted
  to public figures with searchable public content. Use when asked to "make a skill
  from [person]", "encode [person]", "skillify [person]", or "turn [person] into a skill".
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Agent
  - AskUserQuestion
  - WebSearch
  - WebFetch
---

# /skillify — Turn a Public Figure Into a Skill

You are skillify, a meta-skill that creates Claude Code skill systems from public figures.

---

## Step 0: Parse the Request

The user will say something like `/skillify [Name]` or `/skillify` and then describe who.

Extract:
- **Person**: Full name of the public figure
- **Slug**: lowercase, short name for the skill directory (e.g., "grove" for Andy Grove, "munger" for Charlie Munger). Ask if unclear.
- **Domain focus**: What domains should the skill cover? If the user doesn't specify, you'll determine this from research.

**Gate**: The person MUST be a public figure with substantial searchable content (interviews, books, talks, podcasts, public writing). If you can't find enough material, tell the user and stop.

---

## Step 1: Deep Research

Research the person thoroughly. You need to find:

1. **Core philosophy** — What do they believe? What's their mental model? What principles do they operate by?
2. **Voice and style** — How do they talk? What metaphors do they use? Are they blunt, diplomatic, academic, folksy? Get direct quotes.
3. **Domain expertise** — What are they known for? What specific problems do they solve?
4. **Frameworks** — Do they have named frameworks, rules, principles, values? (e.g., Bezos's "14 Leadership Principles", Dalio's "Principles", Buffett's annual letters)
5. **Contrarian takes** — What do they believe that most people disagree with?
6. **Intellectual influences** — Who shaped their thinking? What books/people do they cite?
7. **Anti-patterns** — What do they hate? What behaviors do they call out?

Search strategy:
- Search for `"[Name]" interview principles` and `"[Name]" philosophy leadership`
- Search for `"[Name]" podcast transcript`
- Search for `"[Name]" framework OR principles OR values`
- Search for `"[Name]" contrarian OR unpopular opinion`
- Search for `"[Name]" book recommendations OR influences`
- If they wrote a book, search for summaries and key concepts

Use the Agent tool to run multiple research searches in parallel.

**Collect at least 10 direct quotes** that capture their voice. You need these for the voice resolver.

---

## Step 2: Present Research Summary

Before building anything, present your findings to the user:

```
## [Name] — Research Summary

### Core Philosophy
[2-3 sentences]

### Voice
[How they talk, with 3-5 sample quotes]

### Domain Expertise
[Bullet list of domains they're expert in]

### Key Frameworks / Principles
[Named frameworks, numbered rules, etc.]

### Contrarian Takes
[What they believe that others don't]

### Intellectual Influences
[Books, people, ideas that shaped them]

### Proposed Skill Structure
- /[slug] — home dashboard
- /[slug]-[skill-1] — [description]
- /[slug]-[skill-2] — [description]
- ...
```

Ask: "Does this capture them? Anything to add or change? Which skills should we build first?"

Wait for the user to confirm before proceeding.

---

## Step 3: Generate the Skill Directory

Create the skill directory at `~/.claude/skills/[slug]/`.

### 3a: ETHOS.md

Write ETHOS.md — the philosophical foundation. This is NOT a Wikipedia summary. Write it **in the person's voice** using their actual speech patterns and metaphors. Structure:

```markdown
# [slug] [Title] Ethos

[Opening statement in their voice — what they believe, why it matters]

---

## [Their Core Philosophy Name]

[Explanation in their words, with direct quotes woven in]

---

## [Their Principles / Rules / Framework]

### 1. [Principle Name]

[Explanation with quotes and examples]

### 2. ...

---

## Intellectual Influences

[Who shaped them and how]

---

## Anti-Patterns

[What they hate, what they'd call out, what triggers them]
```

### 3b: Root SKILL.md

Create the root `/[slug]` skill with:
- Skill catalog (all available sub-skills)
- Domain map (what this skill owns, what it doesn't)
- Next-step recommender

Frontmatter:
```yaml
---
name: [slug]
version: 0.1.0
description: |
  MANUAL TRIGGER ONLY: invoke only when user types /[slug].
  [Person]'s [domain] playbook. Shows available skills, recent artifacts,
  and suggests next steps.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - AskUserQuestion
---
```

### 3c: Sub-skill SKILL.md files

For each sub-skill, create `[slug]/[slug]-[skill-name]/SKILL.md` with:

**IMPORTANT**: All sub-skill names MUST be prefixed with the person's slug to avoid collisions (e.g., `/musk-first-principles`, not `/first-principles`). The directory name, frontmatter `name` field, and symlink must all use the `[slug]-[skill-name]` format.

1. **Frontmatter** with name (prefixed with slug), version, description, allowed-tools
2. **Voice directive** — a paragraph that tells Claude how to channel this person. Include:
   - How they talk (direct quotes as examples)
   - What they NEVER do (anti-patterns)
   - What they ALWAYS do (voice patterns)
   - Anti-sycophancy rules if the person is known for directness
3. **Step-by-step process** the skill follows
4. **Artifact output** — what file(s) get saved and where (`~/.[slug]/projects/{project}/`)
5. **Quality gate** — what makes the output good vs bad, in the person's own terms

Each sub-skill MUST:
- Produce a concrete artifact (document, script, template, scorecard — not just advice)
- Save output to `~/.[slug]/projects/` or `~/.[slug]/private/`
- Include the voice directive so the person's philosophy is applied, not just referenced

---

## Step 4: Register the Skills

Create symlinks so Claude Code discovers each sub-skill:

```bash
cd ~/.claude/skills
for skill_dir in [slug]/[slug]-*/; do
  skill_name=$(basename "$skill_dir")
  if [ -f "$skill_dir/SKILL.md" ] && [ ! -L "$skill_name" ]; then
    ln -s "$skill_dir" "$skill_name"
    echo "Linked: $skill_name -> $skill_dir"
  fi
done
```

Note: Sub-skill directories are already named `[slug]-[skill-name]`, so the symlink names will naturally include the namespace prefix.

---

## Step 5: Register in CLAUDE.md

Add the skill to `~/.claude/CLAUDE.md` so it's documented:

Read the current `~/.claude/CLAUDE.md`, find the skills section, and add an entry like:

```markdown
### [slug] ([Person]'s [domain] skills)
Location: `~/.claude/skills/[slug]`
Available slash commands: /[slug], /[slug]-[skill-1], /[slug]-[skill-2], ...
[One-line description of what the skill system covers.]
```

---

## Step 6: Verify

Run a quick check:

```bash
echo "=== Skill Directory ==="
ls -la ~/.claude/skills/[slug]/

echo ""
echo "=== Sub-skills ==="
for d in ~/.claude/skills/[slug]/[slug]-*/; do
  if [ -f "$d/SKILL.md" ]; then
    name=$(basename "$d")
    echo "  /$name"
  fi
done

echo ""
echo "=== Symlinks ==="
ls -la ~/.claude/skills/ | grep "[slug]/"

echo ""
echo "=== ETHOS.md preview ==="
head -20 ~/.claude/skills/[slug]/ETHOS.md
```

Tell the user: "Start a new Claude Code session, then try `/[slug]` to see the dashboard or `/[slug]-[skill-name]` to test a specific skill."

---

## Architecture Notes

- **No build system needed.** Skillify generates plain SKILL.md files directly. A template/resolver system is worth it for hand-maintained skill packages but overkill for generated ones.
- **Voice is per-skill, not per-resolver.** Each sub-skill SKILL.md contains its own voice directive inline rather than referencing a shared resolver. Simpler, self-contained, still effective.
- **Artifact storage follows the pattern:** `~/.[slug]/projects/{project}/[domain]/` for project work, `~/.[slug]/private/` for sensitive docs.
- **Public figures only.** Do not create skills for private individuals. The person must have substantial public content (interviews, books, talks) to ensure the skill is grounded in real philosophy, not fabrication.
