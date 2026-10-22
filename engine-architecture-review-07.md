# Pass 7: Nested Generators, the Id Migration Table, and the Remaining Gaps

Date: 2026-10-05. Follows Pass 6. Where the two disagree, this pass is authoritative.

## 0. Short answer

- **Nested generation works with one code path.** A village of houses of facades of windows passes edits, regeneration, freezing, detaching, overrides and identity checks at every level. The kernel has one recursive `expand` function and no notion of depth.
- **The row generator was not changed.** It became a village because generators were registered for the things it already emits. The house and facade generators do not know what they are nested in.
- **The id migration table is built.** An upgrade that renames keys records an explicit old-id to new-id table in the document. It is deterministic, it is reported before the upgrade is committed, and the cooked world carries it.
- **The work forced seven more general rules** (section 2). Two of them change behaviour you may have assumed, so they need your review: what "detach" means for a generated child, and how coarse "hold until upgrade" is.
- **Tests:** 39 pass (the 28 from Pass 6, unchanged in intent, plus 11 nested). The kernel grew from about 1,670 to about 2,080 lines.
- **Largest remaining architectural risk:** a generator that reads another generator's output. Nothing in the prototype does this yet (section 4).

## 1. What was built

The prototype is still a headless Rust library in `prototypes/identity`, with no Godot in it.

| Level | Generator | Children | Keys |
|---|---|---|---|
| 1 | `row` (unchanged from Pass 6) | houses along a polyline | `house@<point>.lot<n>` |
| 2 | `house` (new) | one roof, four facades | `roof#main`, `facade#north` … |
| 3 | `facade` (new) | windows along the wall, optional door | `window@run.bay<n>`, `door#main` |

A default village has 6 houses and 102 addressable things. A full path looks like `house@a.lot0/facade#north/window@run.bay1`.

### 1.1 Results

| # | Test | What it shows |
|---|---|---|
| n01 | Three levels build deterministically | Second build runs nothing; deleting the cache changes no hash; save, load, save is byte-identical |
| n02 | Edit three levels down | One realize unit runs and no plan. The edit survives the village going to zero houses and back. Moving the house re-plans the village only: zero units run inside the house |
| n03 | Overrides layer from outside in | An override on one house re-plans that house and its four facades and nothing else. The village parameter still drives every house without an override. An override on one facade changes that facade's keys; edits inside it follow the ladder |
| n04 | Geometric edits at depth | Deleting, moving and adding a window inside one facade re-plans that facade only, and all three survive regeneration above them |
| n05 | Freezing at every depth | A window, a facade, a house and the whole village can each be frozen. A frozen house is a pinned member to the village. Tags still reach into frozen content by name |
| n06 | Detaching at every depth | A detached window keeps its id. A detached house keeps its id and all 16 ids below it, with zero units re-run. Reconnecting brings both back under the village with the same ids |
| n07 | Key migration at two levels, and one with no migration | Section 3 |
| n08 | Generator change two levels down | Nothing changes until upgrade, even with the cache deleted |
| n09 | Contracts compose downward | A house generator that passes its own contract check but churns its facades' keys is caught, and so is the village above it |
| n10 | Copy and preview | A copy carries edits at every depth with new ids. A preview reports a key change two levels down without touching anything |
| n11 | Kernel has no depth logic | A source scan fails if the kernel names a level or compares a depth |

### 1.2 What "no special cases" means here, exactly

The kernel has one place that distinguishes a top-level scope from a nested one: only a top-level scope is told about foreign obstacles in the world (rule 6 below). Everything else, including the ladder, freezing, detaching, staging and the contract checks, is the same code at every depth.

## 2. General rules the nested work forced

These continue the numbering from Pass 6, which ended at six.

**7. Ids chain per level.** An id is the hash of the parent's id and the child's own key, not the hash of the owner and the whole path. In Pass 6 the two were the same thing because paths had one segment. With the chained form any generated thing can become an owner without renaming anything below it. This is what lets a detached house keep all its ids.

**8. A record is written against every segment of its path.** Each segment is a name in the namespace of the generator that minted it, at that generator's key epoch. So a record stores one (generator, epoch, fingerprint) step per segment, and the ladder resolves one segment at a time inside the scope that owns it. A record's label is the worst label of its segments. Pass 6 stored one step per record, which cannot express "the village renamed its houses but the facade did not rename its windows".

**9. Geometric edits are not target-free below the top level.** Pass 3 said a pin, an addition or an exclusion needs no target. That is true only at the top. Inside a facade, the record must still find *its facade*. So a geometric record is addressed to a scope, and the scope is resolved by the ladder like any other target. Only its last segment is a hint.

**10. Detach severs exactly one generator relationship.** Detaching a window gives an authored object, as before. Detaching a house gives a document node that is *still a house generator*: it is now its own owner, it no longer follows the village, and its own parameters still regenerate it. Records, frozen children and retired keys below it move with it. To own the windows by hand you detach again, one level down. **This needs your review**: the alternative is "detach flattens the whole subtree to authored objects", which is simpler to explain and destroys more.

**11. A nested generator sees the world only through its brief.** Its inputs are the parameters its parent passes, its own edit records, and its own frozen and retired children. Its position is not an input. This is why moving a house re-plans nothing inside it. The cost: a facade cannot ask about terrain or neighbours itself; the parent must pass what it needs.

**12. A detached child is an obstacle to the scope it left, and to no scope above it.** My first version told the village about a detached window. The village then treated the window as blocking the house it belongs to, and replaced the house. To every ancestor a detached piece is still part of the child it sits in.

**13. Identity contracts must hold for the whole subtree.** A parameter can be harmless to a generator's own keys and still churn the keys of what it feeds. The contract check now walks down through registered generators. A generator checked alone can pass and still be wrong.

Two smaller ones:

- **A move you make yourself is not a changed target.** Moving a house used to flag every edit on it as "same name, different thing". An explicit move now re-bases the fingerprints it invalidates.
- **A save must record how many id migrations it has seen.** Resolving an old id from the start of the table is wrong if a key was renamed away and later renamed back.

### 2.1 One consequence worth a decision: "hold until upgrade" is per owner

An owner records the version of every generator its subtree was built with. If any of them changes, the whole owner is held at its stored output until you upgrade it. So a new facade generator holds every village that contains a facade.

- **Why I chose it:** one stored snapshot and one upgrade action per owner; no partial states inside a village.
- **Cost:** a widely used low-level generator makes most of the world stale at once, and the held snapshot of each owner is stored in the document until upgraded.
- **Alternative:** hold per scope. It is more state and more upgrade actions, and it allows a village whose houses are at mixed versions.

I recommend keeping per-owner and adding "upgrade all" as an editor action. It is your call.

## 3. The id migration table

As you decided: an explicit old-id to new-id table, generator evolution stays allowed.

### 3.1 How it works

1. Upgrading or thawing an owner records the versions it is leaving.
2. When that regeneration is committed, every old path that no longer exists is carried forward one segment at a time, each by the declared migration of the generator that minted that segment.
3. The result is appended to the document as one entry: old id to new id or ids, the old ids with no successor (with the path they had), and plain-language notes.
4. The cook output carries the resolved table. A save resolves an id from the migration count it was written at.

Only declared migrations are used. Fingerprints may re-match an *edit record* and flag it for review; they never produce an id alias. A guess is acceptable where a person reviews it and not acceptable inside a save game.

### 3.2 Results (n07)

| Step | Before committing | After |
|---|---|---|
| Village renames houses, migration declared | Check reports 102 ids renamed, 0 dropped | Every old id resolves to one entity with the same position, visual and collider. All three records at depth 1 and 3 are migrated. Inbox empty |
| Same upgrade on a second engine | | Identical table |
| Facade renames windows, migration declared | 60 renamed | Only window ids change. An id saved before the first upgrade resolves through both entries |
| Facade renames windows, no migration | "no migration", 60 dropped | The edit is re-matched by fingerprint and flagged. The 60 old window ids resolve to nothing, and the table says so. Other ids are untouched |

The staged diff shows the note ("102 identities renamed, 0 without a successor") before commit, so an upgrade that would break ids can be discarded.

### 3.3 Limits

- **Table size is the number of ids below the renamed keys.** Renaming 6 houses wrote 102 entries. A rename near the top of a large owner is a large table.
- **Scatter instances are not covered.** They are keyed by lattice slot, not by path. A new lattice version changes their ids and writes no table.
- **Split and merge are coded but untested.** The table can hold one-to-many, and notes are produced for both, but no test exercises them.
- **The diff of a renaming upgrade shows "N removed, N added".** It does not yet use the table to show "renamed".
- **The runtime side does not exist.** Nothing consumes the table yet.

## 4. Remaining gaps, ranked by architectural risk

"Architectural risk" here means: if this goes wrong, how much of the core model has to change. It is a different question from how much work the item is.

| Rank | Gap | Risk | Why | Cost to probe |
|---|---|---|---|---|
| 1 | A generator reading another generator's output | High | Fences along a road, houses on generated terrain, scatter avoiding houses. This decides evaluation order, cycle handling and how far one edit spreads. The prototype has only owner-to-child flow and document obstacles. Rule 11 makes it sharper: nested scopes cannot query at all | Small, in the headless prototype: two owners, one reading the other |
| 2 | Staging and document cost at scale | Medium-high | Every transaction clones the document and walks every owner. Preview and upgrade checks now do a full build on a copy. Held and frozen snapshots store whole subtrees in the document. All fine at 102 things; unknown at 100,000 | Very small: a timing run at three sizes. No new design needed to measure |
| 3 | Luau at the generator boundary | Medium | "No stable key from a loop index" is currently enforced by the Rust compiler. In a script it must be enforced at run time by opaque key values. Also float determinism and reload. It decides whether the enforcement story survives scripting | Small spike: one generator in Luau through the same context |
| 4 | Upgrade granularity (section 2.1) | Medium | A design choice more than an unknown. Wrong choice costs document size and a noisy editor, not the model | None: a decision |
| 5 | Split and merge migrations | Low-medium | Contained in the ladder and the table; the paths exist. Unknown is the user-facing rule for an edit whose target split in two | Small: two tests |
| 6 | Scatter instance identity across lattice changes, per-instance edits | Low-medium | A second identity scheme beside key paths. Only matters once gameplay refers to individual instances | Small |
| 7 | Runtime use of the id table | Low | Mechanical once a save format exists | Later |
| 8 | Attachment obstacles and ordering, fingerprint tolerances | Low | Tuning, visible only with real content | Later |

Not ranked because it is the next phase, not a gap in this prototype: the editor-side adapter (undo, picking a thing and getting its path, gizmos, showing a staged diff).

### 4.1 Recommendation

- **Probe 1 before the adapter.** It is the only item that could still change the shape of the kernel, and it is cheap to find out.
- **Measure 2 at the same time.** It is an hour and tells us whether "clone and rebuild" is a prototype shortcut or a real plan.
- **Do the Luau spike (3) after that or in parallel with early adapter work.** It does not block the adapter.
- **Decide 4. Leave 5 to 8 until content needs them.**

I would not add tests for 5 to 8 now. They would grow the suite without reducing the risk that matters.

## 5. What this pass does not show

- **One hierarchy.** Three levels, one shape. A generator that nests itself, or two different generators under one parent with records between them, is untested.
- **Small numbers.** 102 things. No timing was taken.
- **Reconnect loses live-ness.** A re-adopted house comes back frozen. Thawing it regenerates it under the village, and the staged diff shows what its time alone had changed. Folding its own parameter changes back in as overrides is not built.
- **Label for a nested geometric edit is "Matched", not "Applies".** It reports that the scope was found. This is cosmetic but visible.
- **Preview is no longer plan-only.** It is a full build on a copy, thrown away.

## 6. Decisions for you

1. **Detach semantics (rule 10).** Keep "detach severs one level, a generated child stays generated", or change to "detach flattens the subtree"? I recommend keeping it.
2. **Upgrade granularity (section 2.1).** Keep per-owner hold with an "upgrade all" action, or go per scope? I recommend per-owner.
3. **Next probes.** Cross-generator reads plus the scale measurement, then the adapter, with the Luau spike alongside? Or go straight to the adapter?

## Appendix: running it

```
cd prototypes/identity
cargo test
```

`tests/scenarios.rs` holds the 21 scenarios and 7 enforcement checks from Pass 6. `tests/nested.rs` holds n01 to n11. The house and facade generators are in `src/gens.rs`; `nested_registry()` is the flat project plus those two.
