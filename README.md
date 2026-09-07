English | [繁體中文](README.zh-TW.md)

# docs — Doc Mine & Distill

Turn a project you are studying (a repo, a CLI, a library) into Markdown knowledge where **every sentence carries its evidence** — a `path:line`, an official URL, or a command you actually ran — and anything that cannot be traced is marked `[UNVERIFIED]` instead of guessed.

Ships two skills:

| Skill | What it produces | When to use it |
|---|---|---|
| **`doc-mine`** | A categorized set of files under `docs/` (how it works / when to use it / reference / design trade-offs) plus `docs/00-answers.md` answering what *you* asked | You are studying a project and want its knowledge laid out |
| **`doc-distill`** | **One** self-contained file — no "see also", every term defined inline — to paste, hand over, or feed to another agent | You want a single file to give someone |

Both are **user-invoked only** (`disable-model-invocation: true`): Claude never fires them on its own, and they cost **zero context** until you type the command.

## Prerequisites

None — **zero dependencies**, cross-platform. Everything runs inside Claude Code.

## Installation

```
/plugin marketplace add harry18456/cc
/plugin install docs@harry18456
```

Restart Claude Code to take effect.

### Use locally right away (no push, no marketplace)

Junction the repo into the skills directory and the next session auto-loads it as `docs@skills-dir`:

```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\docs" -Target "D:\side_project\cc-docs"
```

> ⚠️ Before switching to a proper marketplace install later, remove this junction first (`Remove-Item $env:USERPROFILE\.claude\skills\docs`) to avoid loading the same-named skills twice.

For a single session instead: `claude --plugin-dir D:\side_project\cc-docs`.

## How to run it

Open Claude Code in the project you want documented, then type the command:

```
/docs:doc-mine
/docs:doc-distill
```

Extra instructions go right after the command — `/docs:doc-mine 技術和應用分開寫`. To skip the interview, say "直接開始" in that same message.

> Installed through the junction below instead of the marketplace, the names are `/docs@skills-dir:doc-mine` and `/docs@skills-dir:doc-distill`.

## How `doc-mine` works

```
0. glance ────── README + top-level layout, so the interview can name real modules
1. interview ─── ≤2 questions per round, until you say 開始 / start
2. read ──────── small project: main thread reads it; big: 2–4 subagents split by code region,
                 hard cap 4, each told to finish on its own (no nesting)
3. write ─────── docs/<category>.md, every sentence tagged
4. answers ───── docs/00-answers.md: what you asked, and what you assumed → verified right / wrong
5. verify ────── re-open every cited path:line in 00-answers.md, sample 5 per other file;
                 list every [UNVERIFIED]
```

The interview asks five things: what you already know, what you use it for, what you most want to understand, **what you assume but never checked**, and what to skip. The fourth is the point — each assumption comes back as a sourced *right* or *wrong*.

## Evidence tags

| Tag | Source | Example |
|---|---|---|
| `[src: <path>:<line>]` | A file in the project (source, README, config) | `[src: src/main.rs:42]` |
| `[doc: <URL>]` | Official site / docs | `[doc: https://example.com/docs/cli]` |
| `[run: <command>]` | Actually executed; command and output pasted into the doc | `[run: rtk --version]` |
| `[UNVERIFIED]` | None of the above; what is missing goes in parentheses | `[UNVERIFIED](no write path found)` |

Naming, convention, and "it's usually like this" are leads for *finding* evidence, never evidence themselves.

## How `doc-distill` works

Takes the nearest input — notes already in the conversation → an existing `docs/` → the source (≤2 subagents) — and writes one file (default `docs/distill.md`). Before finishing it searches the text for "see", "refer to", "see docs/" and replaces each with the content itself, then re-opens every `[src:]`.

## ⚠️ Notes

- **Subagent cap**: `doc-mine` spawns at most 4, `doc-distill` at most 2, and each is told to finish on its own rather than delegate. Claude Code itself allows subagents to nest 3 levels deep by default; set `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` to `1` in settings if you want a hard stop (it is global — it affects every skill).
- **`[UNVERIFIED]` is a feature**: the skill leaves a visible gap rather than filling it with a plausible guess. The closing report lists every one.
- **Docs location**: follows the repo's existing convention if there is one; otherwise `docs/`.

## This plugin's components

Currently uses **skills** only (`skills/doc-mine/`, `skills/doc-distill/`, auto-discovered by Claude Code — no plugin.json declaration needed). `commands/`, `agents/`, `hooks/`, `output-styles/`, `bin/` are reserved skeletons; future extensions just drop files in.

## Related files

- **Skill bodies**: [`skills/doc-mine/SKILL.md`](skills/doc-mine/SKILL.md), [`skills/doc-distill/SKILL.md`](skills/doc-distill/SKILL.md)
