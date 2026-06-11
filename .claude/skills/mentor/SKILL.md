---
name: mentor
description: Helps plan and execute a task on any OutSystems ODC app by producing safe, surgical prompts for the Mentor AI assistant. Use when starting any OutSystems task — a new feature, bug fix, refactor, investigation, or code review — or whenever the user types /mentor.
---

# Mentor — task helper for OutSystems ODC apps

This skill makes Claude help the user do a task the **right way** on an OutSystems ODC
app. Claude does not talk to Mentor — Claude produces **Mentor-ready prompts** that the
user pastes into Mentor inside ODC Studio. It works for **any app**: the generic rules
for working with Mentor are app-agnostic; the app-specific facts come from a per-app
context file.

## The two-layer rule (critical — never forget)

```
User → Claude (this skill, reads the knowledge) → writes a prompt → Mentor (sees only that prompt)
```

Mentor **never reads the knowledge files** — only Claude does. Always translate knowledge
into things Mentor can see in the project: action names, variable names, exact
expressions. Never mention the `.md` files in a prompt for Mentor.

## Knowledge base — read before doing anything

**Generic (every app):**
- `knowledge/mentor-behavior.md` — **source of truth** for how Mentor behaves: golden
  rules (§1), known limitations (§2), confirmed capabilities (§3), prompt templates (§4).
  Read this fully at the start of every task.
- `knowledge/outsystems-patterns.md` — reusable ODC architecture patterns. Skim for
  anything relevant to the change.
- `knowledge/learnings-points.md` — unproven hypotheses about Mentor. Skim; do not treat
  as reliable.

**App-specific (pick the app for this task):**
- `knowledge/apps/<app>/context.md` — the **app's** source of truth: project state,
  ownership, identifiers, features done, pending items.
- Plus any other notes in `knowledge/apps/<app>/`.

**Selecting the app (ALWAYS ask first — onboarding):** On every `/mentor` invocation,
before reading any app context or doing anything else, ask the user one question:
**"What's the name of the app you're working on?"** Never auto-pick or assume — not even
when `knowledge/apps/` holds exactly one app, and not when it holds none. Wait for the
answer, then:
- **A folder `knowledge/apps/<app>/context.md` exists** (match the name
  case-insensitively) → load it and proceed.
- **No folder for that app** (a new app, or a fresh download with nothing configured) →
  this is a **first run**. (1) Scaffold `knowledge/apps/<app-name>/context.md` from
  `knowledge/app-context.template.md`. (2) **Immediately produce the codebase-discovery
  investigation prompt (mentor-behavior.md §4.0)** — this is the Step 1 goal: a prompt the
  user pastes into Mentor to survey the whole codebase. (3) When the user pastes Mentor's
  response back, write the discovered structure into the app's `context.md` (§1 modules /
  UI flows / ownership, §3 entities / enums / key actions). Only after the context is
  seeded do you move on to the user's actual task.
If the name is ambiguous or you're unsure, list whatever folders exist under
`knowledge/apps/` and let the user pick.

## Workflow

1. **Onboard — ask the app name first.** The very first thing on every `/mentor`: ask the
   user **"What's the name of the app you're working on?"** (see "Selecting the app"
   above). Then load that app's `context.md` plus `mentor-behavior.md`. **If the app has
   no context yet (first run / fresh download), your Step 1 goal is discovery:** produce
   the codebase-survey investigation prompt (mentor-behavior.md §4.0) for the user to run
   in Mentor, then build `context.md` from the response — before identifying the task.
2. **Identify the task** — new feature, bug fix, refactor, investigation, or code review.
3. **Scope check** — use the app's ownership section. If the task touches shared blocks or
   another owner's screen, flag who to coordinate with before any change.
4. **Overlap check** — read the app's done-features and pending sections. Reuse existing
   patterns; don't duplicate or break them. Decide whether to resolve overlapping pendings
   together or keep scope tight.
5. **Grill before prompting** — interview the user one question at a time until the change
   is fully unambiguous. Recommend an answer for each question. If a question can be
   answered from the knowledge base, answer it yourself.
6. **Apply the golden rules** (mentor-behavior.md §1) when designing prompts:
   - Investigation prompt first, before any implementation (§1.1).
   - One change per prompt — never bundle (§1.2).
   - Insert nodes only; never ask Mentor to reorganize existing connectors (§1.3).
   - Include the `CRITICAL — Anti-hallucination constraints` block.
7. **Produce the prompts** using the templates in mentor-behavior.md §4 (4.1 investigation,
   4.2 implementation, 4.3 comments, 4.4 code review, 4.5 dead code, 4.6 server-action
   review). One prompt per change, in order. Fill templates with exact, verbatim identifiers.
8. **Remind the user of the safety loop** — validate visually between steps (§1.4),
   Ctrl+S not publish (§1.5), Ctrl+Z is the parachute (§1.6), publish only at the end.

## Closing a task

- Document the finished feature into the app's `knowledge/apps/<app>/context.md`
  (what, where, why, important notes).
- If Mentor did something new or surprising, add a 🟣 observation to
  `knowledge/learnings-points.md` section B. Do not stop the task to test it.
- If observations piled up, suggest running `/limit-testing` later to validate them.

## Rules

- Mentor's "0 errors" / "done" is never trusted — always verify visually (§1.4, §1.7).
- Never bundle multiple changes into one prompt.
- Keep prompts surgical: exact location, exact change, explicit `Do NOT` constraints.
