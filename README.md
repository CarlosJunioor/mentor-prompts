# Mentor System

<img width="500" height="500" alt="mentor-prompts" src="https://github.com/user-attachments/assets/0f95ebef-7588-4bf9-b747-7f7666eacb2b" />

A small, reusable system that helps you work on **any OutSystems ODC app** by using
Claude Code to produce great prompts for **Mentor** (OutSystems' built-in AI assistant),
and to improve that skill over time.

It is built from `.md` files that call each other. Inspired by
[mattpocock/skills](https://github.com/mattpocock/skills) — the skill format and the
`write-a-skill` skill come from there (MIT licensed).

> **Disclaimer:** This project is not affiliated with, endorsed by, or supported by
> OutSystems. "OutSystems" and "Mentor" are trademarks of OutSystems. Everything here
> is community-created learning material.

## The idea

Mentor cannot be trained or fine-tuned — it is a closed tool. All the leverage is in the
**prompt** and the **context** you give it. This system is that context, packaged so it
gets a little smarter every session.

There are **two AIs** involved. Never confuse them:

```
You → Claude Code (reads everything here, knows the rules) → writes a prompt
                                                                    │
                                                                    ▼
                                                  Mentor (sees ONLY that prompt)
```

Mentor never reads these files. Only Claude does. Claude's job is to turn this knowledge
into a clean, surgical prompt that Mentor can act on.

## Generic vs per-app

The knowledge is split so the same system works across every app you touch:

- **Generic knowledge** (how Mentor behaves — applies to every app) lives at the top of
  `knowledge/`.
- **Per-app knowledge** (one app's entities, screens, owners, bugs) lives under
  `knowledge/apps/<app-name>/`. Each app you work on gets its own folder; they all share
  the generic rules.

## Folder layout

```
mentor-system/
├── README.md                          ← you are here
├── knowledge/
│   ├── mentor-behavior.md             ← GENERIC source of truth: how Mentor behaves + prompt templates
│   ├── outsystems-patterns.md         ← GENERIC reusable ODC architecture patterns
│   ├── learnings-points.md            ← GENERIC lab notebook: untested hypotheses about Mentor
│   ├── mentor-interviews.md           ← GENERIC interview log: what Mentor says about itself (raw Q&A)
│   ├── app-context.template.md        ← blank template for a new app
│   └── apps/
│       └── <app-name>/                ← per-app knowledge (one folder per app)
│           ├── context.md             ← hub: state, ownership, features, key actions
│           ├── database.md            ← entities, FKs, static entity values
│           ├── blocks.md              ← screens & blocks inventory
│           ├── bugs-backlog.md        ← known bugs not yet worked on
│           └── handoff.md             ← where the last session stopped
└── .claude/
    └── skills/
        ├── mentor/SKILL.md            ← /mentor  — start a task the right way
        ├── limit-testing/SKILL.md     ← /limit-testing — test Mentor's limits, grow the notebook
        ├── talk-with-mentor/          ← /talk-with-mentor — interview Mentor about itself
        │   ├── SKILL.md
        │   └── QUESTIONS.md           ← question bank by topic, with coverage state
        └── write-a-skill/SKILL.md     ← /write-a-skill  — build new skills (from Matt Pocock)
```

## How the files call each other

```
        /mentor ──reads──► mentor-behavior.md ◄──promotes into── /limit-testing
           │                      ▲     +                              ▲
           │                      │     outsystems-patterns.md         │
           │                      │ promotion rule (≥2 scenarios)      │
           ├──reads──► apps/<app>/context.md  (the app for this task)  │
           │                                                           │
           └──writes observations──► learnings-points.md ◄──reads/updates┘
```

- **`mentor-behavior.md`** — stable, generic. Validated rules, limitations, capabilities,
  and the prompt templates (§4). `/mentor` reads it every task, for every app.
- **`apps/<app>/context.md`** — stable, app-specific. The project map, ownership, and
  accumulated decisions for one app. `/mentor` reads the one matching the current task.
- **`learnings-points.md`** — volatile, generic. Hypotheses and observations about Mentor.
  `/limit-testing` works here. When something is proven (≥ 2 scenarios) it is **promoted**
  into `mentor-behavior.md` and removed from here.
- **`mentor-interviews.md`** — raw, generic. Verbatim Q&A from `/talk-with-mentor`
  sessions. Mentor's self-reports are **claims, not facts** — distilled claims enter
  `learnings-points.md` as hypotheses and only graduate via `/limit-testing`.

## Getting started

**To use it — clone, don't fork:**

```bash
git clone https://github.com/CarlosJunioor/mentor-system
cd mentor-system
claude        # open Claude Code in this folder
/mentor       # go
```

Your per-app context (`knowledge/apps/<your-app>/`) is git-ignored, so your company's
entities, screens, and owners stay on your machine and never get published. To receive
improvements to the generic knowledge later, just `git pull` — your local app folders
never conflict.

**To contribute back — fork.** If you discover something new about Mentor (a limitation,
a capability, a prompt pattern), that generic knowledge is exactly what this repo
collects. See [CONTRIBUTING.md](CONTRIBUTING.md).

## How to use it

1. Open Claude Code **in this folder** (`mentor-system/`). The skills load automatically.
2. **Starting a task on an OutSystems app?** Type `/mentor`. Claude reads the generic
   rules + the app's context, grills you until the change is clear, and produces
   Mentor-ready prompts the right way (investigation first, one change per prompt, safety
   reminders). If there's no folder for your app yet, Claude offers to scaffold one from
   `knowledge/app-context.template.md`.
3. **Want to test what Mentor can do?** Type `/limit-testing`. Claude picks an untested
   capability, helps you design the test, records the result, and promotes what works.
4. **Want to ask Mentor about itself?** Type `/talk-with-mentor`. Claude composes interview
   questions (features, modes, limits), logs Mentor's answers verbatim, and turns every
   claim into a testable hypothesis — nothing Mentor says about itself is trusted until
   `/limit-testing` proves it. The loop: **talk → test → trust**.
5. **Want to add a new skill** as the system grows? Type `/write-a-skill`.

## Adding a new app

1. Copy `knowledge/app-context.template.md` to `knowledge/apps/<app-name>/context.md`.
2. Fill in section 1 (project context + ownership) — enough to start.
3. Run `/mentor` and pick that app. The context grows as you work.

## The loop (this is the whole point)

```
use /mentor on a task → notice something new about Mentor → jot it in learnings-points.md
        ▲                                                              │
        │                                                              ▼
keep mentor-behavior.md sharp ◄── /limit-testing validates & promotes ◄── run /limit-testing later
```

Every session the knowledge base gets a little better. There is no single "god-tier
prompt" to discover — the god-tier prompt **is** this growing system.

## Note

There are no apps under `knowledge/apps/` yet. Add your first one by copying
`knowledge/app-context.template.md` to `knowledge/apps/<app-name>/context.md` (see
**Adding a new app** above). The template shows the structure a per-app context grows into.

Your per-app folders under `knowledge/apps/` are git-ignored — your real app's entities,
owners, and bugs stay on your machine and never get published.

## License

MIT — see [LICENSE](LICENSE). Free for the whole OutSystems community to use, modify, and
share. The skill format and `write-a-skill` skill are adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (also MIT).

## Contributing

Issues and pull requests are welcome — especially **validated Mentor learnings** (new
limitations, capabilities, or prompt patterns). The one hard rule: contributions must be
**app-agnostic** — never commit a specific app's entities, screens, owners, UUIDs, or
environment URLs. See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide, including
how findings get validated and promoted.
