# Mentor — Learning Points & Test Backlog (lab notebook)

A **lab notebook** for observations, hypotheses, and capabilities to test about the
Mentor AI — **before** they're validated. App-agnostic: findings here are about how
Mentor behaves, not about any one app. (App-specific observations belong in that app's
`knowledge/apps/<app>/` notes.)

The validated source of truth is [`mentor-behavior.md`](mentor-behavior.md). This file is
where churn lives until something is proven and **promoted** there.

---

## 0. How to use this file

| | `mentor-behavior.md` | `learnings-points.md` |
|---|---|---|
| What | Validated rules + capabilities + templates | Hypotheses + test backlog |
| Stability | Stable, "source of truth" | Volatile, scratchpad |
| When to read | Start of every task | When you want to test something new about Mentor |

### Promotion rule (validated capability → main behavior file)
When an item here is tested with a **clear, repeatable result (passed in ≥ 2 different
scenarios)**, promote it:
- New capability → `mentor-behavior.md` §3.
- New limitation → `mentor-behavior.md` §2.
- Validated prompt template → `mentor-behavior.md` §4.

Then delete the entry here (or mark "PROMOTED on YYYY-MM-DD" for history).

### State convention
- 🟢 **Validated** — tested, repeatable, ready to promote.
- 🟡 **Partial** — tested once; repeatability or edge cases unconfirmed.
- 🔴 **Failed** — tested, doesn't work. Becomes a limitation.
- ⚪ **Untested** — hypothesis only.
- 🟣 **To investigate** — loose observation, no hypothesis yet.

---

## A. Prioritized backlog — capabilities to test

Ordered by practical usefulness. When you pick one, set its state and record the result.
Generic ideas to seed any app's testing:

### A.1 ⚪ Cross-screen pattern replication
**Hypothesis**: Mentor can replicate a sequence of actions from one screen to another,
using the first as an example (adapting target paths/variables).
**How to test**: pick a real pending duplication; prompt Mentor to replicate, then confirm
each handler is wired correctly. **Priority**: HIGH if you have repetitive cross-screen work.

### A.2 ⚪ Bulk Comment adding in a single action
**Hypothesis**: Mentor can create N Comment nodes in one implementation task, anchored to
N different nodes. **How to test**: ask for 3 Comments in one pass; check whether all 3 are
created or any fails. Fallback is template 4.3 (one per prompt).

### A.3 ⚪ Rename action/variable without breaking references
**Hypothesis**: Mentor renames identifiers and updates all references.
**How to test**: a surgical rename of one label/identifier; confirm no connector breaks.

### A.4 ⚪ Working with Aggregates (add/remove sources, attributes, joins, filters)
**Hypothesis**: partially known — adding joins/aliases works, recreating-from-scratch with
many aliases fails (see mentor-behavior §3). Test removing filters and changing join types.

### A.5 ⚪ Mentor edits CSS / Style Sheet of a block
**Hypothesis**: unknown. Test on the next CSS tweak via a directed prompt.

### A.6 ⚪ Mentor analyzes aggregate performance
**How to test**: "Analyze [aggregate] for potential performance issues — unindexed joins,
missing where filters, large fetched volumes."

### A.7 ⚪ Event wiring on block instances — what is the exception condition?
**Hypothesis**: mentor-behavior §2 says it fails at event wiring on block instances except
sometimes `OnTextChanged`. Document 3–4 real attempts to find the pattern. **Priority**:
HIGH — pinning the condition removes a big limitation.

---

## B. Loose observations (not validated, no test planned yet)

- 🟣 **Anti-hallucination block effectiveness varies by output type.** It reliably shifts
  Mentor to "I do not see X" for macro visibility (item/scope), but can fail for micro
  state (a specific node's disabled status) unless the constraint is specific ("state that
  you visually confirmed the disabled marker"). Open question: does it work as well for
  *implementation* tasks as for investigation?
- 🟣 **"Stopped here as requested"** — Mentor follows "Then stop" well. Open: is there a
  stricter "stop on first error, do not auto-correct" phrasing that prevents the
  connector-mangling auto-correction (mentor-behavior §1.3/§2)?
- 🟣 **Mentor categorizes patterns when asked** (IDENTICAL / SIMILAR / CONCEPTUAL). Could
  be reused for structured analysis ("classify each IfNode as: guard / state branch /
  error handling / unreachable / legacy").
- 🟣 **Final output reliable, evidence trail not.** In structured analysis (caller
  analysis, code review) the final list/conclusion can be correct while the detailed
  evidence trail mixes real findings with invented ones in the same confident tone. Never
  trust detailed annotations ("in disabled node only", "additional reference in X")
  without manual validation. (Partially promoted to mentor-behavior §2.)

---

## C. Skill ideas (validated capabilities worth their own template/workflow)

When a capability in section A is validated, consider whether it deserves its own reusable
skill (template + workflow). Use `/write-a-skill` to build it.

- **Documenting an action with Comments** — investigate → identify non-obvious nodes →
  generate template 4.3 prompts → run in sequence.
- **Cross-screen pattern replication** — investigate source pattern → map deltas → surgical
  replication prompt → validate each handler. (Gate on A.1.)
- **Architectural audit of a screen** — action inventory → duplication scan → dead-code
  scan → mixed-responsibility scan → manual refactor decision.
- **Feature dead-code sweep** — identify Logic folders + UI flow scope → template 4.5 →
  validate candidates with Find Usages (mandatory) → delete by hand.
- **Server Action deep audit** — template 4.6 → spot-check disabled markers + dead assigns
  + unwritten outputs → register cleanup candidates → architectural Comment.

---

## D. Session history

Short summaries; detail lives in the relevant app context and the chats.

- _(none yet in this generic notebook)_

---

## E. Operational notes

### When NOT to test new capabilities
- When tired, rushed, or mid-way through a critical feature.
- Before a demo or a near deadline.
- When Mentor is in an unstable state (already broke something, not yet recovered with Ctrl+Z).

### When a testing session makes sense
- Start of a new feature where an untested capability would help a lot.
- Slack between tasks.
- When you suspect a limitation but haven't confirmed it.

### How to jot a quick observation during a real task
1. Don't stop the task to test now.
2. Drop a line in section B with 🟣 and minimal context.
3. Back to the task.
4. When the task closes, decide whether to promote it, turn it into a hypothesis (A), or drop it.
