# Build log — 2026-09-24: mockup → persistent repo

## Base source & currency

- Source: `~/workspace/ts-spaces/milestone-tracker-mockup/index.html`
  (artifact; read-only, not modified).
- Size 141,089 bytes / 1,662 lines, timestamped 2026-09-24 00:02 — the
  current mockup, not a stale build. Marker counts confirmed all recent
  features present (primary-member pods, typed edges, pending/disputed
  workflow, Viewing-as scoping, Month/Week calendar, private-milestone lock
  gating, `validate()` regression checks).
- No external `http(s)` dependencies found — nothing to strip.

## What was added (persistence layer)

- Whole working state serialized to `localStorage` (`milestoneTracker.v1`):
  people, pods (incl. `primaryMemberId`), relationship edges, milestones
  (incl. `addedBy`/`verification`), UI prefs (Viewing-as, calendar mode).
- All former `const` data declarations became `let` bindings seeded from
  storage, or from the in-code sample seed on first run; seed is saved
  immediately after first boot.
- "Reset sample data" control in the header clears storage and reseeds.
- Previously inert flows now write through storage: add-person (New Pod and
  Existing Pod incl. typed relationship-to-primary edges, duplicate-name
  linking guard), add-milestone (with date + type fields and a real Save),
  Verify/Dispute, closeness-tier changes, viewing-as and calendar-mode
  restoration.
- Seed milestone tuples were converted to stable milestone records on first
  load (`addedBy` defaults to the person, all `verified`) without changing
  their content.
- The mockup "Preview only — entries aren't stored" notice was removed;
  notes fields still say draft/not saved (they are not persisted).

## Nothing reconstructed

No feature was rebuilt from a stale source — the base already contained all
of them. The work was conversion: persistence, real writes, reset control,
docs.

## Verification

- `node --check` on the extracted inline script: pass.
- Harness run in Node (DOM stubbed): `validate()` reports 0 violations on
  the transformed seed; perspective regression assertions all pass
  (daniel→arman "Uncle", "Uncle's family" badge, you→arman "Father",
  you→nora "Niece", "Family friends"/"Cousins"/"Immediate family"/"Work
  friends" badges); storage round-trips (12 events, 11 people,
  `primaryMemberId` preserved).
- Served via `python3 -m http.server`: HTTP 200, complete document ending in
  `</html>`, new controls present in the served bytes.
- No headless browser available on this machine, so rendered interaction
  (clicking through the new save flows) was not tested — needs a real
  browser pass.

## Known alpha limitations

- New-person lineage can be incomplete (only relationship-to-primary is
  captured); `validate()` logs console errors for the resulting
  inconsistencies without blocking the save.
- The engine's colloquial folding has gaps for novel graph shapes
  (e.g. a sibling of a cousin folds to "Sibling" rather than "Cousin").
- The `sourcing` invite form remains a demo (generated link is illustrative).
- Calendar export/sync stays a spec, not built.
