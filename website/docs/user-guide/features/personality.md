---
sidebar_position: 9
title: "Personality & SOUL.md"
description: "Customize Hermes Agent's personality with a global SOUL.md, built-in personalities, and custom persona definitions"
---

# Personality & SOUL.md

Hermes Agent's personality is fully customizable. `SOUL.md` is the **primary identity** — it's the first thing in the system prompt and defines who the agent is.

- `SOUL.md` — a durable file that lives in `HERMES_HOME` and occupies slot #1 in the system prompt. It replaces the built-in default identity entirely.
- built-in or custom `/personality` presets — session-level system-prompt overlays

If you want to change who Hermes is — or replace it with an entirely different agent persona — edit `SOUL.md`.

### What SOUL.md actually is

Calling it "personality" is the user-facing label, but mechanically `SOUL.md` is a **system prompt file** — the highest-priority instruction layer. It's injected verbatim as slot #1 of the system prompt with no wrapper. That means it can contain anything a system prompt can:

- tone and communication style
- behavioral rules and constraints
- operational philosophy
- tool-use enforcement patterns
- memory and task-management procedures
- project-agnostic knowledge references

The personality lens is the right framing for users who just want to tweak voice, but it undersells the file's actual power. `SOUL.md` is your per-instance system prompt — the same kind of thing `AGENTS.md` is for a project, just scoped to the Hermes instance instead of a specific repo.

## How SOUL.md works now

Hermes now seeds a default `SOUL.md` automatically in:

```text
~/.hermes/SOUL.md
```

More precisely, it uses the current instance's `HERMES_HOME`, so if you run Hermes with a custom home directory, it will use:

```text
$HERMES_HOME/SOUL.md
```

### Important behavior

- **SOUL.md is the agent's primary identity.** It occupies slot #1 in the system prompt, replacing the hardcoded default identity.
- Hermes creates a starter `SOUL.md` automatically if one does not exist yet
- Existing user `SOUL.md` files are never overwritten
- Hermes loads `SOUL.md` only from `HERMES_HOME`
- Hermes does not look in the current working directory for `SOUL.md`
- If `SOUL.md` exists but is empty, or cannot be loaded, Hermes falls back to a built-in default identity
- If `SOUL.md` has content, that content is injected verbatim after security scanning and truncation
- SOUL.md is **not** duplicated in the context files section — it appears only once, as the identity

That makes `SOUL.md` a true per-user or per-instance identity, not just an additive layer.

## Why this design

This keeps personality predictable.

If Hermes loaded `SOUL.md` from whatever directory you happened to launch it in, your personality could change unexpectedly between projects. By loading only from `HERMES_HOME`, the personality belongs to the Hermes instance itself.

That also makes it easier to teach users:
- "Edit `~/.hermes/SOUL.md` to change Hermes' default personality."

## Where to edit it

For most users:

```bash
~/.hermes/SOUL.md
```

If you use a custom home:

```bash
$HERMES_HOME/SOUL.md
```

## What should go in SOUL.md?

The personality stuff is the sweet spot for most users, but it's not the whole picture. `SOUL.md` is a system prompt file — use it for anything that should apply everywhere, every session.

### Best fit (personality and voice)
- tone
- communication style
- level of directness
- default interaction style
- what to avoid stylistically
- how Hermes should handle uncertainty, disagreement, or ambiguity

### Also fair game (system-wide behavior)
- operational rules and constraints ("never suggest `rm -rf`")
- tool-use policies ("always prefer `patch` over `sed`")
- preferred coding philosophy or architectural defaults
- cross-project conventions you want followed everywhere

### Not a great fit (temporary or project-specific)
- one-off project instructions
- file paths
- repo conventions
- temporary workflow details

When instructions are project-specific, they belong in `AGENTS.md`. When they're temporary, use `/personality` or just say them in conversation.

## Good SOUL.md content

A good SOUL file is:
- stable across contexts
- broad enough to apply in many conversations
- specific enough to materially shape the voice
- focused on communication and identity, not task-specific instructions

### Example

```markdown
# Personality

You are a pragmatic senior engineer with strong taste.
You optimize for truth, clarity, and usefulness over politeness theater.

## Style
- Be direct without being cold
- Prefer substance over filler
- Push back when something is a bad idea
- Admit uncertainty plainly
- Keep explanations compact unless depth is useful

## What to avoid
- Sycophancy
- Hype language
- Repeating the user's framing if it's wrong
- Overexplaining obvious things

## Technical posture
- Prefer simple systems over clever systems
- Care about operational reality, not idealized architecture
- Treat edge cases as part of the design, not cleanup
```

## What Hermes injects into the prompt

`SOUL.md` content goes directly into slot #1 of the system prompt — the agent identity position. No wrapper language is added around it.

The content goes through:
- prompt-injection scanning
- truncation if it is too large

If the file is empty, whitespace-only, or cannot be read, Hermes falls back to a built-in default identity ("You are Hermes Agent, an intelligent AI assistant created by Nous Research..."). This fallback also applies when `skip_context_files` is set (e.g., in subagent/delegation contexts).

## Security scanning

`SOUL.md` is scanned like other context-bearing files for prompt injection patterns before inclusion.

That means you should still keep it focused on persona/voice rather than trying to sneak in strange meta-instructions.

## SOUL.md vs AGENTS.md

This is the most important distinction — and also where the docs have been misleading.

The clean story: `SOUL.md` = personality, `AGENTS.md` = project instructions. That's a useful starting point, but the mechanical reality is less neat.

Both files are **system prompt layers**. They're injected into different slots but they're the same kind of thing — plain markdown that becomes instructions for the agent. The only real difference is **scope**:

| | SOUL.md | AGENTS.md |
|--|---------|-----------|
| Scope | Instance-wide (HERMES_HOME) | Per-project (current directory) |
| Slot | #1 — agent identity | Later — context files |
| Content | Anything that applies everywhere | Anything that applies in this repo |

So if you have a rule that should follow you across every project — "prefer pytest over unittest", "always use `patch` over `sed`", "never suggest `rm -rf`" — it can go in either file. The rule is: if you want it everywhere, put it in `SOUL.md`. If it only matters in this repo, put it in `AGENTS.md`.

Old framing was:
- **SOUL.md**: identity, tone, style, communication defaults — personality-level behavior
- **AGENTS.md**: project architecture, coding conventions, tool preferences, repo-specific workflows, commands, ports, paths, deployment notes

New framing: that's still the sweet spot for each file, but the boundary is softer than advertised. Both files are system prompts. `SOUL.md` just happens to be the one that follows you everywhere.

### A useful rule
- if it should follow you across all projects → `SOUL.md`
- if it belongs to a single project → `AGENTS.md`
- if you're unsure → `SOUL.md` is fine. Move it to `AGENTS.md` later if it only ends up mattering for one repo.

## SOUL.md vs `/personality`

`SOUL.md` is your durable default personality.

`/personality` is a session-level overlay that changes or supplements the current system prompt.

So:
- `SOUL.md` = baseline voice
- `/personality` = temporary mode switch

Examples:
- keep a pragmatic default SOUL, then use `/personality teacher` for a tutoring conversation
- keep a concise SOUL, then use `/personality creative` for brainstorming

## Built-in personalities

Hermes ships with built-in personalities you can switch to with `/personality`.

| Name | Description |
|------|-------------|
| **helpful** | Friendly, general-purpose assistant |
| **concise** | Brief, to-the-point responses |
| **technical** | Detailed, accurate technical expert |
| **creative** | Innovative, outside-the-box thinking |
| **teacher** | Patient educator with clear examples |
| **kawaii** | Cute expressions, sparkles, and enthusiasm ★ |
| **catgirl** | Neko-chan with cat-like expressions, nya~ |
| **pirate** | Captain Hermes, tech-savvy buccaneer |
| **shakespeare** | Bardic prose with dramatic flair |
| **surfer** | Totally chill bro vibes |
| **noir** | Hard-boiled detective narration |
| **uwu** | Maximum cute with uwu-speak |
| **philosopher** | Deep contemplation on every query |
| **hype** | MAXIMUM ENERGY AND ENTHUSIASM!!! |

## Switching personalities with commands

### CLI

```text
/personality
/personality concise
/personality technical
```

### Messaging platforms

```text
/personality teacher
```

These are convenient overlays, but your global `SOUL.md` still gives Hermes its persistent default personality unless the overlay meaningfully changes it.

## Custom personalities in config

You can also define named custom personalities in `~/.hermes/config.yaml` under `agent.personalities`.

```yaml
agent:
  personalities:
    codereviewer: >
      You are a meticulous code reviewer. Identify bugs, security issues,
      performance concerns, and unclear design choices. Be precise and constructive.
```

Then switch to it with:

```text
/personality codereviewer
```

## Recommended workflow

A strong default setup is:

1. Keep a thoughtful global `SOUL.md` in `~/.hermes/SOUL.md`
2. Put project instructions in `AGENTS.md`
3. Use `/personality` only when you want a temporary mode shift

That gives you:
- a stable voice
- project-specific behavior where it belongs
- temporary control when needed

## How personality interacts with the full prompt

At a high level, the prompt stack includes:
1. **SOUL.md** (agent identity — or built-in fallback if SOUL.md is unavailable)
2. tool-aware behavior guidance
3. memory/user context
4. skills guidance
5. context files (`AGENTS.md`, `.cursorrules`)
6. timestamp
7. platform-specific formatting hints
8. optional system-prompt overlays such as `/personality`

`SOUL.md` is the foundation — everything else builds on top of it.

## Related docs

- [Context Files](/docs/user-guide/features/context-files)
- [Configuration](/docs/user-guide/configuration)
- [Tips & Best Practices](/docs/guides/tips)
- [SOUL.md Guide](/docs/guides/use-soul-with-hermes)

## CLI appearance vs conversational personality

Conversational personality and CLI appearance are separate:

- `SOUL.md`, `agent.system_prompt`, and `/personality` affect how Hermes speaks
- `display.skin` and `/skin` affect how Hermes looks in the terminal

For terminal appearance, see [Skins & Themes](./skins.md).
