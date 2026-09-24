# Milestone Tracker

A static, single-file web app for sourcing and organizing friends'
annually-celebrated life milestones — birthdays, kids' birthdays, wedding
anniversaries, death/remembrance anniversaries — so you remember and reach
out. No accounts, no backend, no build step.

**Status:** alpha. The app works locally and persists to the browser; the
sample data is still the fictional Rahimi/Chen family, and several flows
(described below) remain concept-stage.

## Run it

```bash
cd ~/workspace/milestone-tracker
python3 -m http.server
```

Then open the printed URL (usually http://localhost:8000/). Or enable
GitHub Pages on the repo (Settings → Pages → deploy from branch) and open
the published URL — the app is one static file and needs no build.

## Data & persistence

- All state (people, pods, relationship edges, milestones, verification
  states, UI preferences) lives in `localStorage` under the key
  `milestoneTracker.v1`.
- **First run** seeds the app from the in-code sample data and saves it
  immediately.
- **Reset sample data** (top of the page) clears local data and restores the
  sample seed on reload. No separate account or server state exists.
- Nothing leaves the browser. There is no sync, no account, and no server
  call of any kind. Sample data uses real-sounding family names — keep that
  in mind before sharing a copy or a public link.

## What's real vs. still alpha

Working and persisted: add people (New Pod / Existing Pod with typed
relationship to the pod's primary member), add milestones with date and type,
Verify / Dispute verification workflow, closeness tiers, private milestones
with lock gating, Viewing-as perspective across all views, Month/Week
calendar with filters, relationship graph, search, and reset.

Concept/still demo: the sourcing link flow (the generated URL doesn't route
anywhere real yet), notes fields, and per-person reminder notifications.

Not built by decision: calendar export/sync ("leave it as a spec for now";
see `docs/DESIGN-DECISIONS.md` §12).

## Design record

Everything decided so far — relationship model, perspective rules,
verification workflow, private milestones, persistence — is captured in
`docs/DESIGN-DECISIONS.md`. Read that before changing behavior.
