---
name: hermes-memory-strategy
description: "Use the Hermes memory system effectively — manage the 1375-char hard cap, choose memory vs. skills vs. external sources, and avoid the bloat loop that wastes turns on consolidation."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [Memory, Persistence, Hermes, Context-Engineering]
    related_skills: [hermes-agent, hermes-agent-skill-authoring]
---

# Hermes Memory Strategy

`memory` writes a fact to a durable per-profile store (`~/.hermes/memories/<profile>/{user,memory}.md`). It is injected at the start of every turn, so it costs you tokens on every response for the rest of the session and beyond. **Most "should I save this?" questions answer "no."** This skill encodes when to save, when not to, and how to manage the hard 1375-character cap without burning turns on consolidation.

## The three-place rule: pick the cheapest tier that still works

A piece of information does not need to live in `memory` to be available. Use the cheapest tier that still answers the question reliably:

| Tier | Lives in | Survives across | Cost per turn | Use for |
|---|---|---|---|---|
| **Skill** | `~/.hermes/skills/.../SKILL.md` | All sessions forever, plus loaded on demand | Zero until loaded | Procedures, workflows, gotchas, tool-usage patterns |
| **Memory** | `~/.hermes/memories/.../{user,memory}.md` | Injected every turn, all sessions on this profile | **Tokens every turn** | Who the user is, durable prefs, current long-lived state |
| **External source of truth** | GitHub repo, Notion page, project README, API | Forever, versioned with the thing it describes | Tokens only when queried | Project state, repo contents, dataset details, active task lists |

Default reflex: if the user said *"check my GitHub for project details"* or *"see the repo"*, **do not duplicate the answer in memory**. Memory is for things that *can't* be cheaply re-derived.

## Memory hard cap: 1375 characters total

The store is hard-capped. Adding a new entry that would exceed it errors out, and the error forces a consolidate-before-retry cycle — each cycle is at least one tool call. The session can spend 3+ turns just on memory bookkeeping. Avoid that.

**Practical budgets (verified in production):**

- 7–8 entries is the sweet spot before consolidation pain starts
- Each entry should be 100–250 chars; a single entry over 350 chars is a smell
- If you find yourself about to write the 8th entry, **stop and trim first**

## What belongs in `memory` (and what doesn't)

### Save to `memory` — durable, can't be re-derived cheaply

- **Who the user is:** name, location, role, key handles (GitHub, HF, etc.)
- **Long-lived preferences:** communication style, formatting rules, *what they explicitly said "remember this" about*
- **Current state that survives this session:** active projects at the name-level only, key deadlines, ongoing constraints (e.g. "ELLIS PhD applies Oct 2026")
- **Environment facts the agent will need to know on every turn:** base directory the user told you to stay inside, hard sandbox rules

### Do NOT save to `memory`

- **Anything stale in 7 days:** PR numbers, commit SHAs, current task list, "fixed bug X" logs, "submitted PR Y", "Phase 2 done". Use `session_search` to recall session transcripts.
- **Lists of project details** when the user already pointed you at a source of truth (GitHub, Notion, README). Query the source.
- **Tool-usage procedures / workflows / fix recipes.** Those are skills. The litmus test: "would this help me *do* a class of task, or is it just a fact?" Procedure → skill. Fact → memory.
- **Imperative instructions to self** ("always do X", "never do Y") — these re-read as directives in later sessions and can override the user's current request. Phrase as declarative facts ("User prefers X" not "Always do X").
- **Verbose quotes or large blocks of text.** Memory is not a pastebin. Compress aggressively or don't write.
- **Corrections from this very turn.** Memory writes are slow and visible; for an active correction in conversation, just do the right thing now.

### Save to a **skill** instead when…

- The lesson is procedural ("how to set up X" / "how to recover from Y")
- It applies to a class of tasks, not one user fact
- You'd want it loaded on demand, not in every turn

The `hermes-agent` skill is the framework-config reference; `hermes-agent-skill-authoring` covers the SKILL.md format. Both are bundled and protected — you patch the content of skills, not the framework docs.

## Working under the cap — the consolidation playbook

If you must add a fact and you're already at 85%+, use this order:

1. **Compress first.** Shorten existing entries by removing words, ranges, parentheticals. Target the entries that are most "declarative facts" (e.g. "ELLIS PhD targets: Amsterdam, Tübingen, Edinburgh, ETH Zurich, Saarbrücken" instead of "Long-term goals: ELLIS PhD Sep 2027 start (apply Oct 2026) — targets Amsterdam, Tübingen, ...").
2. **Drop transient stuff.** Anything you added in the last few turns that isn't a durable fact (recent task progress, debug output, intermediate conclusions) is a candidate for `remove`.
3. **Merge overlapping entries.** Two identity-like entries → one tighter entry. Use `replace` with `old_text`.
4. **Only then** retry the `add`.

**Anti-pattern: "consolidate by adding a marker entry."** Don't write "see GitHub for project list" as its own memory entry — that's an instruction-to-self, and the user already said it once in chat. The directive travels with their message, not with memory.

## Memory vs. skills: the bundling rule

If you're saving a workflow or fix that you'd want to re-use, the *correct* home is a skill, not memory. Memory is for who-they-are and current-situation state. Procedure is a skill.

Examples from real sessions:

- *"User has a PAT in `~/.netrc`, write it via Python to avoid heredoc mangling, file is chmod 600, 40-char token, 50KB stdout cap, etc."* → **skill** (`github-auth/references/github-api-via-curl.md`).
- *"User's name is Rajat, GitHub handle is Rajat25022005, prefers terse responses."* → **memory**.
- *"User said check GitHub for project state instead of duplicating it in memory."* → **memory** (it's a meta-preference about how to use memory).

## Pitfalls

- **Writing the same fact twice.** Duplicate entries are common when you `add` without first checking existing entries. Read the store (or the error response showing current entries) before adding.
- **Adding "see also" / "pointer" entries.** "For projects, see GitHub" is a directive disguised as a fact. Either do it (and don't write anything) or write the substantive fact directly.
- **Forgetting `remove` exists.** Memory is not append-only. If a fact is no longer current (e.g. the user told you a deadline moved), patch the entry, don't add a new one next to the stale one.
- **Writing procedure as memory.** "When X happens, do Y" belongs in a skill. The litmus test: rephrase the entry as "do Y when X" — if it's a complete instruction, it's a skill.
- **Hitting the cap mid-session.** The cap is *per profile*. If you're in a real pinch, consider whether the entry is so important it belongs in a **skill** (which has no such cap and is only loaded on demand), not memory.
- **Treating `add` success as license to dump more.** Each `add` makes every future turn more expensive. Be stingy.

## Quick decision flow

1. User said "remember this" / explicitly corrected you → memory.
2. Fact is environmental / about-the-user / current-situation → memory, but compress aggressively.
3. Fact is a workflow, fix, gotcha, or procedure → skill.
4. Fact is project state and user pointed you at a source → don't write, query the source.
5. About to write 8th entry → stop, trim, then write.
