# Contributing to Mentor Prompts

Thanks for wanting to make this better. This repo is a **shared brain about how the
OutSystems Mentor AI behaves** — every contribution makes prompts safer and sharper for
everyone in the community.

## The one hard rule: app-agnostic only

Never commit anything from a specific app:

- ❌ entity names, screen names, action names from your real project
- ❌ Realm / environment UUIDs, environment URLs
- ❌ teammate names, ownership details, business logic
- ✅ how Mentor *behaves* — limitations, capabilities, prompt patterns — described
  generically ("Mentor fails to insert a node between two existing nodes inside IfNode
  branches"), with any example identifiers anonymized (`SaveOrder` → `[ActionName]`)

Your real app's facts belong in your local, git-ignored `knowledge/apps/<app>/` folder.
PRs containing app-specific data will be asked to anonymize before merge.

> Already committed real app data (or a secret) by mistake? **Don't open a public issue** —
> report it privately so it can be scrubbed. See [SECURITY.md](SECURITY.md).

### Before you commit — privacy checklist

The app-agnostic rule is only as good as the 30 seconds you spend checking. Before every
commit:

1. **Review the diff:** `git diff --staged` — read it for real names, URLs, or UUIDs.
2. **Grep your diff** for your app/company/teammate name (you know what to look for).
3. **Confirm nothing private is staged:** `git status` should never show anything under
   `knowledge/apps/`. That folder is git-ignored — **never** use `git add -f` to override
   it, and don't let a GUI client's "include ignored files" toggle stage it.
4. **Anonymize** any example identifier to `[brackets]` (`SaveOrder` → `[ActionName]`).

## What to contribute

| Contribution | Where it goes | Bar to merge |
|---|---|---|
| 🟣 New observation / hypothesis about Mentor | `knowledge/learning-points.md` (section B) | Saw it once, described precisely (what you asked, what Mentor did, ODC version/date if known) |
| 🔴 New **limitation** (something Mentor fails at) | `knowledge/mentor-behavior.md` §2 | Validated in **≥ 2 distinct scenarios** (different apps or different actions) |
| 🟢 New **capability** (something Mentor does reliably) | `knowledge/mentor-behavior.md` §3 | Validated in **≥ 2 distinct scenarios** |
| 📋 New / improved **prompt template** | `knowledge/mentor-behavior.md` §4 | Used successfully at least twice; includes the anti-hallucination block where relevant |
| 🏗️ Generic ODC architecture pattern | `knowledge/outsystems-patterns.md` | Reusable on any ODC app |
| ⚙️ Skill improvements (`/mentor`, `/limit-testing`) | `.claude/skills/` | Tested in a real Claude Code session |
| 📝 Docs, typos, clarity | anywhere | Just open the PR |

**Not sure if your finding meets the ≥2-scenario bar?** Contribute it as a 🟣 observation
to `learning-points.md` anyway — that file is the lab notebook, and `/limit-testing`
sessions exist precisely to validate entries and promote them up.

## How findings get validated (the promotion pipeline)

```
you notice something  →  learning-points.md (hypothesis, 1 sighting)
                              │
                              ▼  /limit-testing session, or a 2nd confirmed sighting
                         mentor-behavior.md §2 or §3 (validated rule)
                              │
                              ▼  if it changes how prompts should be written
                         mentor-behavior.md §4 (template updated)
```

When you promote an entry in a PR, **remove it from `learning-points.md`** in the same
PR — a fact lives in exactly one place.

## How to submit

1. **Fork** the repo and branch from `main`.
2. Make your change. For behavior findings, follow the existing entry style in the target
   file (look at neighboring entries and match their format and tone).
3. **Describe the evidence in the PR**: what you prompted (anonymized), what Mentor did,
   how many times, on how many different apps/actions. The evidence is what gets reviewed
   — not just the wording.
4. Open the PR. Small and focused beats big and mixed — one finding per PR is ideal.

## Reporting without a PR

Found a behavior finding but don't want to write it up? Open an **issue** with:
- the prompt you gave Mentor (anonymized),
- what Mentor did vs. what you expected,
- whether you could reproduce it.

That's a valuable contribution too — someone else can validate and promote it.

> **Not a behavior finding — a leak or a secret?** Do **not** open a public issue.
> Report it privately per [SECURITY.md](SECURITY.md) so it can be scrubbed.

## Style notes

- English, matching the tone of the existing files (direct, evidence-first).
- Prompt templates use `[brackets]` for placeholders and include the
  `CRITICAL — Anti-hallucination constraints` block where the template investigates
  or implements.
- Don't renumber existing sections; append.

## License

By contributing you agree your contribution is licensed under the repo's
[MIT license](LICENSE).
