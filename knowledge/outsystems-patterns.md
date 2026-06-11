# OutSystems ODC Patterns — app-agnostic

Reusable architecture patterns for OutSystems ODC reactive apps, distilled from real
code review. These are **platform patterns, not Mentor behavior** (for Mentor see
[mentor-behavior.md](mentor-behavior.md)) and **not specific to any app**. Apply them
when building features on any ODC project. Each pattern says when to use it and, just as
importantly, when NOT to.

---

## 1. Selective refresh via boolean toggle (not Destination)

**When**: a child block must react to a parent change (e.g. a tree that should refresh
when the edited item's name/code changed).

**Anti-pattern**: `Destination` to the same screen to force a reload. Cost: re-fetch of
every screen aggregate, loss of UI state (active tab, scroll, expansion), visible latency.

**Pattern**:
1. Local Var `RefreshXToggle` (Boolean).
2. Pass it as an Input to the child block.
3. In the child block, `OnParametersChanged` detects the toggle change and does its work
   (e.g. RefreshData of its own aggregate).
4. In the parent, on the action that should trigger refresh: `RefreshXToggle = not RefreshXToggle`.

**When NOT to apply**: if the screen genuinely needs to reload (e.g. navigation to another
record), `Destination` is correct. This pattern is for "keep screen state but refresh one part".

---

## 2. Trust the binding — don't force RefreshData if the aggregate is already updated via binding

**When**: the user edits screen fields bound to `Aggregate.List.Current.X`, and the save
sends those values to the server.

**Anti-pattern**: calling `RefreshData(Aggregate)` at the end of the save "just in case".
Causes an unnecessary round-trip (re-fetch + re-render).

**Pattern**: trust the client-side state. The fields are bound to `.List.Current.X`; the
user edited them directly; you sent them to the server. After a successful save the
client-side state already reflects the truth — no re-fetch needed.

**When NOT to apply**: if the server **computes** something the client doesn't know (new
record `Id`, a derived code, a changed status), RefreshData is necessary.

---

## 3. Naming `WB_Event_*` for block event handlers

**When**: you create a screen action that handles a block's event.

**Pattern**: name it `WB_Event_<context>_<EventName>` (e.g. `WB_Event_OnTextChanged_<Field>`,
`WB_Event_<List>OutputList`).

**Why**: searching the project for `WB_Event_` then shows all block event wiring. A uniform
convention buys readability + searchability. Migrate ad-hoc handler names to it whenever
you next touch a screen.

---

## 4. Data Actions > N separate aggregates for composite data

**When**: you need several related lists at screen load.

**Anti-pattern**: N separate aggregates on the screen (N parallel round-trips).

**Pattern**: 1 **Data Action** on the screen with multiple Output Parameters. Runs in
parallel at load, 1 server round-trip, centralized server logic, explicit SQL control.

**Operational detail**: Data Actions run in parallel with `AtStart` aggregates, but an
aggregate's `OnAfterFetch` may fire before the Data Action finishes. If `OnAfterFetch`
needs the Data Action outputs, start it with `RefreshData(DataAction)` to force sync.

**Trade-off**: you lose granular per-list refresh (one Data Action, all-or-nothing).
Apply when the lists update together.

---

## 5. Stateless block via Input/event — state lives in the parent

**When**: you build a block that manipulates a list (add/remove/update rows).

**Anti-pattern**: a block with a Local Variable that copies the Input + emits changes →
two copies of the state, complex sync, race conditions.

**Pattern**:
- Block receives the list as an **Input Parameter** (read-only from the block's view).
- Each internal action processes a **local copy** of the Input + emits everything via an
  **event** to the parent.
- Parent receives via a `WB_Event_*` handler and does `LocalList = OutputList`.
- Block has **no own aggregate, no data Local Var, no OnInitialize**.

**Why**: single source of truth — all "truth" lives in the parent; the block is a pure
function over the Input.

**Known cost**: latency between emit → re-assign can cause flicker in fast interactions.
If that's critical, consider another pattern; otherwise the clarity gain wins.

---

## 6. Consolidate multiple Assigns into one AssignNode

**When**: two or more assignments happen at the same logical moment (same trigger, same
branch, same flow step).

**Anti-pattern**: 2–3 separate AssignNodes on the canvas for a series of related assignments.

**Pattern**: a single AssignNode with multiple assignment lines.

**When NOT to apply**: if the assignments depend on each other (Assign B uses Assign A's
value) or happen at different logical moments.

---

## 7. Anti-pattern: flags as screen Input Parameters

**Anti-pattern**: a screen Input Parameter like `ShowSaveSuccess: Boolean` to signal
behavior between invocations of the same screen via `Destination`. Causes URL propagation
(`?ShowSaveSuccess=true`), odd F5 behavior (URL persists, flag fires again), and duplicate
code to "clear" the flag.

**Pattern**:
- Screen Inputs identify **what** to show (Ids, edit-vs-create mode, level type). Not **how**.
- For user feedback after an action, use a `MessageNode` directly in the handler.
- For communication between actions of the same screen, use Local Variables (they live in
  memory, not the URL).

---

## 8. Diff-based reconcile for child collections

**When**: you persist a list of "children" (multi-values, secondary references, any
many-to-one) where the user can add/edit/remove rows in the UI.

**Anti-pattern**: delete-all-then-insert-all. Loses Ids (breaks audit, FKs in other
tables), fires DB triggers for everything, and can cause ghosts under concurrency.

**Pattern**:
1. Fetch DB list filtered by parent Id (1 aggregate).
2. ForEach over DB list → `ListAny` (in-memory) over input list by Id → if absent → individual delete.
3. ForEach over input list → skip blanks → set parent Id + type → individual `CreateOrUpdate`.

**Optional optimizations** (apply only when profiling justifies — they add complexity):
- Special-case empty input + non-empty DB → SQL bulk delete (1 round-trip instead of N).
- Skip upsert when the row is identical to the DB version.
- Bulk insert via SQL for new rows.

**Trade-off**: O(N+M) round-trips per reconcile. Imperceptible for small lists (≤10),
measurable for large ones (50+). Watch for hidden `Max records` caps on the internal
aggregate — rows beyond the cap become undeletable orphans.

**When NOT to apply**: if the collection is append-only (audit log), a direct `CreateRecord` is better.
