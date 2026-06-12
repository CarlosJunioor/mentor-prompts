# Mentor Behavior — operating manual (app-agnostic)

How the OutSystems **Mentor** AI assistant behaves, and the rules for getting good,
safe results out of it. **Nothing here is specific to any one app** — these rules came
from hours of trial-and-error and hold on any OutSystems ODC project. App-specific
state (entities, screens, owners, bugs) lives in `knowledge/apps/<app>/context.md`,
never here.

> **Two-layer rule (never forget).** Mentor never reads these files — only Claude does.
> ```
> You → Claude (reads this + the app context) → writes a prompt → Mentor (sees ONLY the prompt)
> ```
> Always translate knowledge into things Mentor can see in the project: action names,
> variable names, exact expressions. Never mention `.md` files in a prompt for Mentor.

---

## 1. Golden rules (apply always)

### 1.1 Investigation before implementation
For **any change** to an action/aggregate/widget, first send an **investigation prompt**
that maps: current state of the action (node by node, with connectors), variables
involved (scope, type), widget bindings, event wiring. Without investigation Mentor
guesses and breaks things. With it, the implementation prompt can be precise.

### 1.2 One change per prompt
- 2 branches to change → 2 separate prompts.
- create var + call action + bind widget → 3 prompts.
- Never ask "do X and Y and Z" — Mentor fails on Z and undoes X/Y.

### 1.3 Insert nodes; never reorganize existing connectors
- Mentor inserts new nodes reliably.
- Mentor **fails** when it has to reorganize existing connectors (it breaks paths,
  leaves branches without incoming connectors, then "auto-corrects" into a worse state).
- When a change requires reorganizing, **do the wiring by hand** in ODC Studio, or
  design the prompt so Mentor only inserts and you wire.

### 1.4 Validate visually between steps
Mentor says "0 errors" even when it broke the flow. Observed patterns: reports success
while a screenshot shows loose Assigns; auto-corrects failed sequences and hides the
result; "Done" does not mean correct. **Before publishing**, open the action in Service
Studio, look at the node graph, confirm visually.

### 1.5 Publish only at the end
Before publishing, validate visually. Before that it's **Ctrl+S local** — if something
goes wrong, **Ctrl+Z** rescues.

### 1.6 Ctrl+Z is the parachute
Active until the module is closed. Use it without fear — you lose Mentor's work (which
was wrong anyway) but the previous state is recovered.

### 1.7 Verify Mentor's claims with facts
When Mentor says "it's already how you want" or "verified", ask for a **screenshot or a
verbatim paste of properties**. It has lied directly, and it has also been right when
doubted — either way, confirm with your own eyes, not its words.

### 1.8 Comments as standard practice on new features
On any new feature or significant refactor, add Comment nodes in the non-obvious places:
- Non-trivial IfNodes (complex condition, ambiguous label, logic depending on snapshots/external state).
- Server Action calls with non-obvious side-effects.
- Snapshots and the branching that depends on them.
- Workarounds and non-obvious fixes (otherwise someone "cleans" them in 6 months and breaks everything).
- Disabled nodes kept for an architectural reason (otherwise delete them).
- An architectural overview at the top of large actions.

Comment language: match the codebase (OutSystems codebases are usually English).
Anchor types: 1 node → direct anchor; N related nodes → anchor all; whole-action
architectural comment → no anchor, in free space at the top.

---

## 2. Known limitations (what Mentor fails at)

- **Does not create `RaiseExceptionNode` from scratch** (cannot resolve the exception reference).
- **Fails at event wiring on block instances** (occasional exception: sometimes works with `OnTextChanged`). The exact condition for the exception is not yet pinned down.
- **Does not edit inline Records** (Local Vars / Structures defined inline) with types/lengths — it only sees the textual signature. **Workaround: add attributes to inline Records by hand in ODC Studio first**, then Mentor can freely reference them in filters, bindings, expressions.
- **May touch files you didn't ask about** — always ask for the list of what changed.
- **Reports "X changes made"** — always ask for the explicit list; do not trust the count.
- **Inserting a node between two existing nodes inside IfNode branches breaks connectors** — confirmed repeatedly. Pattern: inserts → breaks connectors (ambiguous paths, branches with no incoming) → tries to auto-correct → fails worse. Ctrl+Z is essential. **This kind of insertion is by hand by default.**
- **Can "flatten" Assign sequences in investigation reports** and omit IfNodes. Always confirm visually on the canvas.
- **Code review hallucinates** — mixes real findings with invented ones in the same confident tone. **Mitigation (validated repeatedly): include the `CRITICAL — Anti-hallucination constraints` block** (see templates) — it drastically reduces hallucinations.
- **Refactor strategies are naïve in ODC** — it identifies structural duplication correctly but proposes consolidations that don't work on the platform. **Use Mentor to detect duplication; decide the refactor by hand.**
- **Invents references in disabled nodes during caller analysis** — the **final output** (e.g. the list of zero-caller candidates) is verifiable and usually correct, but the **detailed evidence trail** ("caller in disabled node", "additional reference in X") can mix real findings with inventions. **Always validate candidates with Find Usages before acting.**
- **Caller analysis treats disabled calls as valid callers** — listing "callers found" is not enough; always ask "how many of those callers are in disabled nodes?".
- **"Step 1 — …" labelling makes it fill in out-of-scope steps** — even when you write "out of scope", numbered steps make Mentor invent extra ones. **Mitigation: use descriptive headers with no numbering** ("Action structure", "Inputs and outputs", "Observations of interest").
- **"Local Variable written but never read afterwards inside this action" is misleading cross-action** — it's true *literally inside the action*, but the var may be read in other actions or via block bindings. Mentor has no cross-action visibility by default. Never treat it as dead-code proof without manual Find Usages cross-action.
- **Does not classify Data Actions automatically** — when listing a screen's aggregates it may confuse a Data Action with a Server Action or Aggregate. To classify correctly, **ask explicitly**: "is X a Server Action / Client Action / screen Aggregate / Data Action / something else?".
- **Detailed mappings age poorly** — when another author refactors, an old mapping can be confidently wrong (inventing nodes/inputs that no longer exist, or never existed). Never treat a Server Action mapping as stable for more than 1–2 weeks in multi-author code. **Adversarial re-validation**: ask "does action Y exist? does Input Z exist?" — Mentor reliably answers "Path not found" / "I do not see X", in contrast to its confident invention.
- **Invents connectivity gaps** — may report a "connectivity gap, not fully mapped" in a small action that has none. For small actions (<20 nodes), ask for the full verbatim mapping; the gap usually disappears.
- **System action dependencies must be added explicitly** — Mentor has no visibility on system actions (e.g. `LogMessage`) unless the dependency is added to the module via `Manage Dependencies`. When it reports "action not available", check by hand whether the System element is referenced. Assume adding dependencies is a by-hand step by default.

---

## 3. Confirmed capabilities (what Mentor does reliably)

- **Comment nodes and anchors** — creates them reliably (verbatim text, dashed anchor to target, no side-effects); also edits existing Comment text; supports multiple anchors from one Comment.
- **Code review intra-action** — works with supervision (catches disabled nodes, repeated sequences, duplicate MessageNodes). Score ~6/10 without anti-hallucination constraints; always validate manually.
- **Code review cross-action, same screen** — works well; exhaustively inventories a screen's actions and spots structurally identical ones. Score ~9/10 with anti-hallucination constraints. Good for mapping large screens.
- **Code review cross-screen, several screens** — works; inventories multiple screens at once and identifies asymmetries between them. Score ~8.5/10 with anti-hallucination constraints + per-screen visibility marker + explicit gap analysis (template 4.4).
- **Code review intra-Server-Action** — validated end-to-end on a critical 28-node action: full mapping of inputs/outputs/locals, callee inventory, responsibility blocks, expression complexity, risk-sensitive areas. Score ~9.5/10; all cleanup candidates confirmed by hand. Consistent epistemic humility ("I do not see X"). See template 4.6.
- **Dead code detection, feature-scoped** — identifies real Server Actions and Aggregates with zero callers, with anti-hallucination constraints. **Does NOT work module-wide** — Mentor scopes to feature folders. Final output is verifiable; the detailed caller analysis can hallucinate (see §2). **Always validate candidates with Find Usages.**
- **UI flow vs module distinction** — when a prompt uses imprecise terminology (e.g. calls a UI flow a "module"), Mentor corrects it and scopes appropriately.
- **Descriptive headers vs "Step N" labelling** — prompts with descriptive headers (no numbering) are respected; numbered steps cause out-of-scope invention (see §2).
- **Identifies Data Actions when asked explicitly** — distinguishes Data Actions / Server Actions / Client Actions / Aggregates when the prompt asks for the category in the form "is X a Server Action / Client Action / Aggregate / Data Action / something else?". Without that explicit ask it tends to omit or confuse the category.
- **Edits Entity schema — adds attributes with type/length/mandatory** — validated: adds attributes with verbatim names, correct types, correct mandatory/default/FK, leaves pre-existing attributes intact, 0 active errors. **Prompt pattern**: (1) explicit list of attributes to add (verbatim name + type + length + mandatory + default + FK); (2) explicit list of pre-existing attributes that must stay intact; (3) anti-hallucination block with "Do NOT modify any existing attribute" + "Do NOT add any attribute besides those listed" + "STOP if any step fails — do not partially apply"; (4) verbatim final report. Inline Records still NOT supported (see §2).
- **Creates Advanced Query (SQL) nodes** — creates the SQL node with text + params + output Structure, and preserves the SQL **verbatim (no smart-quotes)** when given ready-made SQL. Also creates Local Variables, edits block-instance arguments, and inserts nodes inline (when the prompt explicitly authorizes re-pointing the predecessor's connector). **Does NOT write SQL from scratch with aliases of a multi-instanced entity** (generates invalid refs); and when *replacing/altering* a node it breaks surrounding connectors (→ by hand). ODC SQL dialect = **Aurora PostgreSQL** (`string_agg`, `||`, `position`, `cast as varchar`, `{Entity}.[Attr]`, `@param`).
- **Creates a new Entity with FKs**; **adds joins with aliases to an existing aggregate** (may need a second attempt — *adding* works even though *recreating* an aggregate with N aliases fails); **creates an aggregate from scratch** with multiple sources + join + filter-with-path + sort; **adds an Input Parameter** to a server action; **inserts a node inline** when re-routing one connector is explicitly authorized.
- **Does not normalize names** — accepts ugly/typo identifiers without "helpfully" renaming them, so it won't break references for cosmetic reasons.

> **`ConvertList` with a `{...}` mapping is NOT typeable** — neither as text in an Assign nor as a widget argument (`Syntax error: unexpected '{'`). Do the mapping **by hand** in the UI (put the source list in the field, fill the "Mapping from X to Y" sub-fields). Mentor will auto-correct against "STOP" and scramble the node if pushed to type it.

---

## 4. Prompt templates (copy and adapt)

> **General prompt-writing rule.** Phased investigation prompts must NOT number steps
> ("Step 1 — …"). Use descriptive headers ("Action structure", "Inputs and outputs",
> "Observations of interest") so Mentor doesn't fill in out-of-scope steps. Template 4.4
> still uses "Step N" historically — consider rewriting it without numbering next time.

Replace everything in `[brackets]` with exact, verbatim identifiers from the project.

### 4.0 Codebase discovery (first run / new app)
Use this as the **very first prompt** when you start on an app you have no context for yet
— a fresh download with an empty `knowledge/apps/<app>/`. It surveys the app's structure
breadth-first so you can seed the app's `context.md`. It deliberately does NOT map every
node; deep per-action mapping comes later via 4.1 / 4.6, scoped per feature.

> Mentor scopes to feature folders and is unreliable module-wide (§2). So ask only for the
> high-level structure it can actually see, and require EXHAUSTIVE/PARTIAL + a visibility
> statement per section. Expect to run this **once per module / feature area**, not once
> for a whole large app — re-run scoped when a section comes back PARTIAL.

```
Investigation only — do not implement anything. Do not modify any node, property,
or connector. This is a structural survey of the application.

Scope: application [app name]. Survey the overall structure as far as you can see it.

Application structure
- List the modules you can see in this application.
- For each module, list its UI flows.
- For each UI flow, list its screens and blocks.

Logic
- List the Server Action / Logic folders, and the actions inside each.
- List the Client Actions and Data Actions you can see.

Data
- List the Entities. For each: its identifier and key attributes (name + type),
  and flag foreign-key attributes.
- List any Static Entities and their values.

For EACH section above, state your visibility first, then mark the section
EXHAUSTIVE or PARTIAL with an explicit reason.

CRITICAL — Anti-hallucination constraints:
- State your visibility scope before each section.
- If you cannot see a module, folder, screen, block, or entity — say so explicitly.
- Do not invent names. Do not infer from training knowledge or from other apps' names.
- Distinguish "I see X" vs "X probably exists based on a name I saw".
- Do NOT propose changes, refactors, or deletions. Inventory only.

Report findings only, grouped under the headers above. Then stop.
```
**Usage notes:** the output seeds the app's per-app file set — route it the same way the
`/mentor` skill does: modules / UI flows / ownership → `context.md` §1; entities / static
entities / enum values → `database.md`; screens, blocks, and their actions → `blocks.md`.
If a section returns PARTIAL, re-run this scoped to a single module or UI flow. Validate
names against ODC Studio before trusting them — first-run surveys age fast and Mentor
can flatten or omit (§2).

### 4.1 Investigation (intra-action)
```
Investigation only — do not implement anything.

[Location: screen, block, action, aggregate]

Report findings on these points:

1. [specific question 1]
2. [specific question 2]
3. ...

Report findings only. Then stop.
```

### 4.2 Surgical implementation
```
Implementation task — make exactly this change, nothing more.

[exact location]

[verbatim description of the change, with exact expressions]

Constraints (CRITICAL):
- Do NOT [thing A].
- Do NOT [thing B].
- Do NOT reroute any other connector.

If at any point you cannot resolve a reference, STOP and report which
step failed. Do not attempt fallback. Do not partially apply.

After implementing: report active error count (must be 0) and confirm
[verbatim detail]. Then stop.
```

### 4.3 Add a Comment node with anchor
```
Implementation task — make exactly this change, nothing more.

Location: [screen/block] → [action name].

Add one Comment node anchored next to [node name and short identifier —
e.g. the IfNode named "X" whose condition is "..."].

The Comment node text must be exactly:

[verbatim text, multi-line allowed]

Constraints (CRITICAL):
- Do NOT modify the target node's condition, label, or any property.
- Do NOT modify any connector in the action.
- Do NOT touch any other node.
- The Comment node must be a Comment widget (not a MessageNode, not an
  Assign, not anything else). It must NOT be wired into the execution flow.
- The anchor must NOT be an execution connector — it is a visual/
  documentation link only.

If you cannot create a Comment node or anchor it, STOP immediately and
report which step failed. Do not attempt fallback.

After implementing: report active error count (must be 0), confirm the
Comment node was created with the exact text above, confirm the anchor
target, and confirm no other node was touched. Then stop.
```

### 4.4 Code review cross-action or cross-screen
> Note: still uses "Step N" labelling. Consider rewriting with descriptive headers.
```
Investigation only — do not implement anything. Do not modify any node.

Location: [screen(s) / exact scope]

Step 1 — Action inventory
For EACH location separately, list every action with:
- Action name
- Trigger / where it is called from
- One-line description

Mark each location as EXHAUSTIVE or PARTIAL with explicit reason.

Step 2 — Pattern detection
[specific criteria for what to look for — duplication, asymmetry, etc.]

Step 3 — [other review-specific steps]

CRITICAL — Anti-hallucination constraints:
- For each location separately, state visibility before reporting.
- If you cannot see a node, action, or property — say so explicitly.
- Do not invent details. Do not infer beyond what is visible.
- Distinguish "I see X" vs "X probably does Y based on name".
- Do NOT propose refactor strategies. Platform feasibility is out of
  scope. Report observed patterns only.

Report findings only, grouped by the steps above. Then stop.
```

### 4.5 Dead code detection, feature-scoped
```
Investigation only — do not implement anything. Do not modify any node.

Location: feature scope inside module [module name]. Specifically the
folders [list explicit Logic folders] and the UI flow [name] (all screens
and blocks).

For this scope, identify Server Actions and Aggregates with zero callers.

Inventory
List every Server Action defined in the listed Logic folders + every
Aggregate defined inside the UI flow screens/blocks. Mark scope as
EXHAUSTIVE or PARTIAL with explicit reason.

Caller analysis
For each item, identify callers (screens, blocks, other server actions).
Distinguish:
- "Caller found in [location], enabled" — concrete reference, node visually confirmed enabled.
- "Caller found in [location], DISABLED" — concrete reference, node visually confirmed disabled.
- "No callers found" — zero references seen.

Dead code candidates
Items with zero ENABLED callers (disabled-only callers count as practically
dead for review purposes). For each candidate include: confidence level,
public/exposed status, body status (empty / has logic / disabled-only),
naming hints suggesting deprecation (_OLD, _v2, suffix numbers).

CRITICAL — Anti-hallucination constraints:
- State visibility scope before listing.
- If you cannot see a folder, action, or aggregate — say so explicitly.
- Do not infer items based on names of other modules or training knowledge.
- Distinguish "I see X defined" vs "X is referenced but defined elsewhere".
- For "disabled node references" — be EXPLICIT that you visually confirmed
  the disabled marker. Do NOT mark a reference as disabled unless you can see it.
- Do NOT propose deletions or refactors. Report observations only.

Report findings only, grouped by the headers above. Then stop.
```
**Usage notes:** the final candidate list is reliable when validated with Find Usages;
the evidence trail has known hallucinations (§2). Run feature-by-feature, not module-wide.

### 4.6 Code review intra-Server-Action
```
Investigation only — do not implement anything. Do not modify any node,
property, or connector. This is a code review of one Server Action.

Location: Server Action [name], in folder [Logic\Server Actions\...]
(module [module name]).

Context: [short description and why this review — e.g. "this is a critical
Server Action with prior production bugs; map it exhaustively for review,
NOT to propose changes."]

Action structure
List every node in execution order, with:
- Node type (Assign, IfNode, Server Action call, Aggregate, MessageNode,
  Exception handler, End, etc.)
- Node label (verbatim)
- For Assigns: target and value (verbatim if short; summarize if very long)
- For IfNodes: condition (verbatim) and what each branch leads to
- For Server Action calls: action name and key input bindings
- For Exception handlers: which exception type it catches
- For disabled nodes: state EXPLICITLY that you visually confirmed the
  disabled marker (not inferred from name or context)

Mark the inventory as EXHAUSTIVE or PARTIAL with explicit reason.

Inputs and outputs
List Input Parameters, Local Variables, and Output Parameters, with type
and (if visible) where each is read/written inside the action.

Callee inventory
List every Server Action, Aggregate, or external call invoked. One line per call site.

Observations of interest
Report ONLY what you can directly observe — no refactor proposals:
- Cleanup candidates: disabled nodes still present; branches with no incoming
  connectors; Local Variables declared but unused; Assigns writing a target
  never read afterwards; duplicate sequential Assigns.
- Architectural observations: sections with clearly different responsibilities;
  very complex condition expressions; workaround-style logic.
- Risk-sensitive areas: where one node's output feeds another's input in a way
  that would break silently if wrong (no validation between).

CRITICAL — Anti-hallucination constraints:
- State your visibility before reporting.
- If you cannot see something — say so explicitly.
- For ANY claim that a node is disabled, state that you visually confirmed the marker.
- Do not invent details about callees. If you cannot see a called action's body,
  say "I cannot see the body of [X], only its signature."
- Distinguish "I see X" vs "X probably does Y based on name".
- Do NOT propose refactors, consolidations, or platform-specific optimizations.
- Do NOT propose deletions. Cleanup candidates are observations, not recommendations.

Report findings only, grouped under the headers above. Then stop.
```
**Usage notes:** descriptive headers (no "Step 1 —") are the correct method to avoid
invented sections. Validate the final output with a ODC Studio spot-check (disabled
markers, verbatim expressions, Find Usages for outputs). The architectural mapping it
produces doubles as the basis for documentation Comment nodes (§1.8).

---

## 5. When you discover something new about Mentor
- New limitation → add to §2 here (once validated in ≥ 2 scenarios).
- New capability → add to §3 here (once validated in ≥ 2 scenarios).
- New validated prompt pattern → add to §4 here.
- Unproven hypotheses and one-off observations → `knowledge/learning-points.md`
  (the lab notebook), not here. `/limit-testing` validates them and promotes the
  proven ones up into this file.
