# <App Name> — Mentor working context

> Copy this file to `knowledge/apps/<app-name>/context.md` and fill it in. This is the
> **hub file** for one app: its current state, ownership, and accumulated decisions.
> It holds only app-specific facts — the generic rules for working with Mentor live in
> [`../../mentor-behavior.md`](../../mentor-behavior.md) and the generic ODC patterns in
> [`../../outsystems-patterns.md`](../../outsystems-patterns.md). Don't duplicate those
> here; reference them.
>
> The hub is part of a **per-app file set** (all in the same folder; `/mentor` scaffolds
> them automatically):
> - `context.md` — this hub: project context, ownership, features done/pending, key actions
> - `database.md` — entities, attributes, FKs, static entity values, notable aggregates
> - `blocks.md` — screens & blocks inventory: actions, bindings, events, asymmetries
> - `bugs-backlog.md` — known bugs / suspicions not yet worked on
> - `handoff.md` — where the last session stopped + next step (overwritten each session)
>
> Keep this hub lean: deep data facts go to `database.md`, screen/block detail to
> `blocks.md`. One fact lives in exactly one file.

---

## 1. Project context

- **App**: <name>
- **Platform**: OutSystems ODC (<edition>)
- **Realm UUID**: `<uuid>`
- **Development env UUID**: `<uuid>`
- **Main modules / UI flows**: `<...>`
- **Language**: <UI / internal communication language>

> Note any terminology the team uses precisely (e.g. UI flow vs module distinctions) —
> getting the words right in prompts avoids ambiguity with Mentor.

### Ownership and responsibilities
- **<You>** — owner of: `<area / UI flow / screens>`. Decisions you can make without
  external approval: `<...>`.
- **<Teammate>** — owns `<area>`. Coordinate before any change.
- **Shared blocks** (`<list>`): warn other consumers before touching.

### Exact names worth noting (typos, gotchas)
- `<screen>` → `<DetailName>` (note any historical typos to NOT "fix").

---

## 2. Current app state — features done and pending

### 2.1 Closed features (don't touch)
For each: what, where (action/screen/block), why (which bug/requirement), important notes
(limitations, by-hand vs Mentor, derived pending items).

- _(none yet)_

### 2.2 In-progress / pending — your territory (can proceed without external coordination)
- _(none yet)_

### 2.3 In-progress / pending — shared territory (needs coordination)
- _(none yet)_

### 2.4 Larger pending items
- _(none yet)_

---

## 3. Identifiers and critical references

App-specific facts Mentor needs translated into the prompt: entity attributes (watch FK
names), enums/static entities, screen/action Local Vars and Inputs, Server Action folders
and signatures, structural asymmetries between similar screens.

### Entities
- `<Entity>`: <attributes, FK gotchas>.

### Enums / Static
- `<Entity>.<Value>` = <id>.

### Key actions (structure, signature)
- `<ServerAction>`: <inputs / outputs / locals / responsibility blocks>.

---

## 4. How to use this file for a new task

1. **Scope check** — does the task touch only your area, or shared blocks / other screens?
   If shared, see §1 ownership for who to coordinate with.
2. **Already done?** — read §2.1. Reuse the pattern; don't duplicate or break it.
3. **Crosses a pending item?** — read §2.2–2.4. Decide whether to resolve together or keep scope tight.
4. **Apply the Mentor rules** ([mentor-behavior.md](../../mentor-behavior.md)) — investigation
   first, surgical implementation (one change per prompt), validate visually, publish only at the end.
5. **Document Comments** (mentor-behavior §1.8) before closing — non-obvious IfNodes,
   critical server actions, snapshots.
6. **Document** — add the finished feature to §2.1 (what / where / why / notes).

When you discover something new about Mentor, route it: app-specific facts stay here;
generic Mentor behavior goes to [mentor-behavior.md](../../mentor-behavior.md);
unproven hypotheses go to [learnings-points.md](../../learnings-points.md).

---

Last updated: <date>
