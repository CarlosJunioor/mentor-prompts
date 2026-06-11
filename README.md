# Mentor System

A small, reusable system that helps you work on **any OutSystems ODC app** by using
Claude Code to produce great prompts for **Mentor** (OutSystems' built-in AI assistant),
and to improve that skill over time.

It is built from `.md` files that call each other. Inspired by
[mattpocock/skills](https://github.com/mattpocock/skills) — the skill format and the
`write-a-skill` skill come from there (MIT licensed).

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
│   ├── app-context.template.md        ← blank template for a new app
│   └── apps/
│       └── <app-name>/                ← per-app knowledge (one folder per app)
│           └── context.md             ← that app's state, rules, identifiers
└── .claude/
    └── skills/
        ├── mentor/SKILL.md            ← /mentor  — start a task the right way
        ├── limit-testing/SKILL.md     ← /limit-testing — test Mentor's limits, grow the notebook
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

## How to use it

1. Open Claude Code **in this folder** (`mentor-system/`). The skills load automatically.
2. **Starting a task on an OutSystems app?** Type `/mentor`. Claude reads the generic
   rules + the app's context, grills you until the change is clear, and produces
   Mentor-ready prompts the right way (investigation first, one change per prompt, safety
   reminders). If there's no folder for your app yet, Claude offers to scaffold one from
   `knowledge/app-context.template.md`.
3. **Want to test what Mentor can do?** Type `/limit-testing`. Claude picks an untested
   capability, helps you design the test, records the result, and promotes what works.
4. **Want to add a new skill** as the system grows? Type `/write-a-skill`.

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

Issues and pull requests are welcome. Keep contributions **app-agnostic** — never commit a
specific app's entities, screens, owners, UUIDs, or environment URLs. Per-app specifics
belong only in your local, git-ignored `knowledge/apps/<app-name>/` folder.
