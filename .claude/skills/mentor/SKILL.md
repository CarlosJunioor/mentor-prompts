---
name: mentor
description: Helps plan and execute a task on any OutSystems ODC app by producing safe, surgical prompts for the Mentor AI assistant. Use when starting any OutSystems task — a new feature, bug fix, refactor, investigation, or code review — or whenever the user types /mentor.
---

# Mentor — task helper for OutSystems ODC apps

## Intro banner (always show first)

On every `/mentor` invocation, the **very first output** is this banner, exactly as-is,
in a code block — followed by the command guide, then the app-name question:

```
███╗   ███╗███████╗███╗   ██╗████████╗ ██████╗ ██████╗
████╗ ████║██╔════╝████╗  ██║╚══██╔══╝██╔═══██╗██╔══██╗
██╔████╔██║█████╗  ██╔██╗ ██║   ██║   ██║   ██║██████╔╝
██║╚██╔╝██║██╔══╝  ██║╚██╗██║   ██║   ██║   ██║██╔══██╗
██║ ╚═╝ ██║███████╗██║ ╚████║   ██║   ╚██████╔╝██║  ██║
╚═╝     ╚═╝╚══════╝╚═╝  ╚═══╝   ╚═╝    ╚═════╝ ╚═╝  ╚═╝
██████╗ ██████╗  ██████╗ ███╗   ███╗██████╗ ████████╗███████╗
██╔══██╗██╔══██╗██╔═══██╗████╗ ████║██╔══██╗╚══██╔══╝██╔════╝
██████╔╝██████╔╝██║   ██║██╔████╔██║██████╔╝   ██║   ███████╗
██╔═══╝ ██╔══██╗██║   ██║██║╚██╔╝██║██╔═══╝    ██║   ╚════██║
██║     ██║  ██║╚██████╔╝██║ ╚═╝ ██║██║        ██║   ███████║
╚═╝     ╚═╝  ╚═╝ ╚═════╝ ╚═╝     ╚═╝╚═╝        ╚═╝   ╚══════╝

        ⚡ Surgical prompts for OutSystems Mentor AI ⚡
```

Right after the banner, show this command guide (adapt formatting freely, keep the content):

> **Commands**
>
> | Command | What it does |
> |---|---|
> | `/mentor` | Start a task on an ODC app — I interview you, then write safe, surgical prompts for you to paste into Mentor. |
> | `/limit-testing` | Run a structured testing session to discover what Mentor can and cannot do, and update the learning notebook. |
> | `/write-a-skill` | Create or improve a skill in this toolkit. |
>
> **What you can ask me for inside `/mentor`**
>
> | Task type | Example |
> |---|---|
> | 🆕 New feature | "Add a filter dropdown to the Orders screen" |
> | 🐛 Bug fix | "The total isn't updating after delete" |
> | 🔍 Investigation | "Map what the SaveOrder action actually does" |
> | 🧹 Refactor / dead code | "Find unused server actions in the Billing feature" |
> | 📋 Code review | "Review the Checkout screen's actions" |
> | 💬 Comments / docs | "Document the snapshot logic with Comment nodes" |
>
> I never touch your app — I write prompts, **you** paste them into Mentor in ODC Studio
> and paste the answers back.
>
> 🔁 **The loop that makes this work:** after running any prompt in Mentor, **always paste
> Mentor's full response back here**. That's how I keep learning your codebase — every
> response grows your app's context file, so the next prompts get sharper.

Then ask the app-name question.

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
  this is a **first run**. (1) Scaffold the **full per-app folder** (see "Per-app file
  set" below): `context.md` from `knowledge/app-context.template.md`, plus `database.md`,
  `blocks.md`, `bugs-backlog.md`, and `handoff.md` with their standard headers.
  (2) **Immediately produce the codebase-discovery investigation prompt
  (mentor-behavior.md §4.0)** — this is the Step 1 goal: a prompt the user pastes into
  Mentor to survey the whole codebase. (3) When the user pastes Mentor's response back,
  route the discovered structure into the right files: modules / UI flows / ownership →
  `context.md` §1; entities / static entities / enum values → `database.md`; blocks,
  screens, and their actions → `blocks.md`. Only after the context is seeded do you move
  on to the user's actual task.

## Per-app file set (`knowledge/apps/<app>/`)

Every app folder holds these files. `context.md` is the **hub** — it stays lean and
points to the satellites. Route facts to the right file; never duplicate across them.

| File | Holds | Written when |
|---|---|---|
| `context.md` | Hub: project context, modules/UI flows, ownership, features done/pending (§2), key server actions | Onboarding + after every closed task |
| `database.md` | Entities, attributes + types, FKs, static entities **with record values**, aggregates worth noting | Discovery + whenever a task reveals data facts |
| `blocks.md` | Screens and blocks inventory: their actions, bindings, events, structural asymmetries | Discovery + whenever a task maps a screen/block |
| `bugs-backlog.md` | Known bugs and suspicions not yet worked on: symptom, location, hypothesis, priority | Whenever a bug is mentioned but not fixed now |
| `handoff.md` | Session handoff: where the last session stopped, current task state, next step, open questions | **End of every working session** (overwrite — it describes "now", not history) |

At the **start** of a session on an app, read `handoff.md` first to resume context. At
the **end** of a session (user says they're stopping, or a task closes), update
`handoff.md` and route any new facts to their files.
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
9. **End EVERY message that contains a Mentor prompt with the feedback-loop reminder:**
   tell the user to paste Mentor's full response back here, so the app's `context.md`
   keeps learning from the codebase. Never hand over a prompt without this reminder.

## Closing a task

- Document the finished feature into the app's `knowledge/apps/<app>/context.md`
  (what, where, why, important notes) — and route satellite facts: data discoveries →
  `database.md`, screen/block mappings → `blocks.md`, bugs spotted but not fixed →
  `bugs-backlog.md`.
- Update `handoff.md` with where things stand and the next step.
- If Mentor did something new or surprising, add a 🟣 observation to
  `knowledge/learnings-points.md` section B. Do not stop the task to test it.
- If observations piled up, suggest running `/limit-testing` later to validate them.

## Rules

- Mentor's "0 errors" / "done" is never trusted — always verify visually (§1.4, §1.7).
- Never bundle multiple changes into one prompt.
- Keep prompts surgical: exact location, exact change, explicit `Do NOT` constraints.
