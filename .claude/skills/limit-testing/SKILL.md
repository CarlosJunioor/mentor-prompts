---
name: limit-testing
description: Runs a structured capability-testing session against the OutSystems Mentor AI assistant and updates the generic learning notebook. Use when the user wants to test what Mentor can or cannot do, explore Mentor's limits, validate a hypothesis, or improve the learning points — or whenever the user types /limit-testing.
---

# Limit-testing — test Mentor's capabilities

This is a **lab-notebook session**. Pick one untested thing about Mentor, test it, record
the result honestly, and promote whatever turns out proven. This is how the knowledge base
gets smarter over time. Findings are about **how Mentor behaves** — app-agnostic — so they
benefit every app, not just the one you happen to test on.

## Knowledge base

- `knowledge/learnings-points.md` — the **lab notebook**: test backlog (section A), loose
  observations (section B), skill ideas (section C), session history (section D).
- `knowledge/mentor-behavior.md` — the **source of truth**. Validated items get promoted
  INTO this file (limitations §2, capabilities §3, prompt templates §4).
- `knowledge/apps/<app>/context.md` — pick a real app to source concrete test cases from
  (a pending task makes the best test). The *finding* is generic; the *test case* is app-specific.

The relationship: `/limit-testing` *feeds* `/mentor`. Anything proven here graduates into
the behavior file that `/mentor` relies on.

## State convention (from learnings-points.md section 0)

- ⚪ untested — only a hypothesis
- 🟡 partial — tested once, repeatability not yet confirmed
- 🟢 validated — repeatable, passed in ≥ 2 different scenarios
- 🔴 failed — tested, does not work → becomes a limitation
- 🟣 loose observation — noticed, no hypothesis formed yet

## Workflow

1. **Read** `knowledge/learnings-points.md`. Show the user the backlog (section A),
   highest practical priority first.
2. **Pick one item** to test. Confirm there is a real test case for it — ideally a pending
   task in some app's `context.md` it applies to. One capability per session.
3. **Design the test prompt** — base it on a template from `mentor-behavior.md` §4. Make
   the test isolate exactly the capability in question.
4. **User runs it in Mentor** and reports back. Capture the result verbatim — including
   screenshots / property dumps where the result needs proof (behavior rule §1.7).
5. **Record** the result in `learnings-points.md`: update the item's state emoji and add a
   dated note describing what happened.
6. **Apply the promotion rule** (learnings-points.md section 0):
   - 🟢 validated (≥ 2 scenarios) → promote: new capability → mentor-behavior.md §3, new
     limitation → §2, validated prompt template → §4. Then remove the entry from
     learnings-points.md (or mark "PROMOTED on YYYY-MM-DD").
   - 🟡 partial → keep it in learnings-points.md, note what is still unconfirmed.
   - 🔴 failed → write it into mentor-behavior.md §2 (known limitations).
7. **Update section D** of learnings-points.md with a one-paragraph session summary.

## Skill ideas

When a validated capability deserves its own reusable workflow, record it in
`learnings-points.md` section C. A mature section-C idea can become a brand-new skill in
this folder — use the `/write-a-skill` skill to build it.

## Rules

- Never promote on a single test — the rule is **≥ 2 different scenarios**.
- Keep the behavior file stable; let the churn live in the learnings file.
- Test one capability per session — do not test five things at once.
- Be honest about 🔴 failures — a documented limitation is as valuable as a capability.
