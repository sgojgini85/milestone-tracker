# Milestone Tracker — design decisions

Living record of every design decision behind this app, captured 2026-09-24
when the mockup graduated to a runnable repo. New decisions get appended with
a date; nothing here is rewritten after the fact.

## 1. Core scope

A specialized calendar app for sourcing and organizing friends'
annually-celebrated life milestones: individual birthdays, kids' birthdays,
wedding anniversaries, death/remembrance anniversaries, and personal
milestones. The whole point is the congratulation — Sepand wants to remember
and reach out, not to run a CRM.

- **Sourcing must be near-frictionless.** A friend receives one link and can
  submit themselves, their partner, and their children — no account, no app
  install, no login.
- **No confetti.** Celebratory chrome is out.
- **Remembrances are quiet.** "Thinking of you" language, never celebratory
  wording or emoji.
- **Notification priority** combines closeness of the relationship, the
  milestone's significance, and the anniversary's tenure (a 25th anniversary
  outranks a 3rd; kids' birthdays carry extra weight).
- **Notification timing is undecided:** advance notice, day-of, or both are
  all still open. Deliberately not decided yet.

## 2. Relationship model: typed edges are the source of truth

The relationship graph is real — not just spokes back to Sepand. Secondary
connections between other people (cousin ↔ spouse/kids, sibling ↔ sibling)
are first-class, because they drive household groupings and inherited
closeness.

**Stored base edges (only these, never hand-entered derivations):**
`parent` (directional; child is derived), `spouse`, `sibling`, `friend`,
`colleague`. Each base edge is stored exactly once.

**Derived, never stored:** child, grandparent/grandchild, aunt/uncle,
niece/nephew, cousin, in-law terms, household labels ("Immediate family",
"Cousins", "Family friends", "Work friends"), and every
perspective-specific label in Graph, People, search, and person detail.

**Colloquial composition, not strict genealogy.** Relationship search uses
everyday terms: a cousin's child is a niece/nephew, a sibling's child is a
niece/nephew, an aunt/uncle's child is a cousin. Searching "niece" finds
Daniel's daughter Nora, with detail copy like "Niece — via Daniel, your
cousin".

**`validate()` constraint set** (runs on load; sample data must report zero
violations):
1. Parent edges form a DAG — no cycles, no one their own ancestor.
2. No contradictory stored edges between a pair; no duplicates.
3. Shared parents imply siblings — a conflicting stored base edge is invalid.
4. Cousins must share at least one grandparent and no parent.
5. Uncle/aunt must derive through a sibling (or sibling's spouse) of a
   parent/grandparent.
6. Every pod must resolve a derivable household label and have exactly one
   primary member (see §5).
7. Perspective regression: Daniel must derive Arman as "Uncle" and the
   Parents pod as "Uncle's family".

Known alpha limitation: adding a person captures only their relationship to
the pod's primary member, so lineage can be incomplete (e.g. "sibling of
Daniel" with no recorded shared parent). `validate()` will log console
errors for the resulting inconsistencies without blocking the save.

## 3. Sample lineage

- Arman Rahimi and Kian Rahimi are **siblings** (stored edge).
- Kian is Daniel Rahimi's **father** (stored edge).
- Daniel is therefore Arman's nephew and Sepand's cousin **by derivation** —
  no direct Arman↔Daniel edge is stored.
- Daniel's pod has four members: Daniel, Sofia, Nora, **Kian**.

## 4. Global "Viewing as" + visibility rule

A global perspective switcher in the header applies to This Week, Calendar
(month/week), People, person detail, and Graph. Sourcing/invite and the
add-person/add-milestone sheets remain Sepand's outbound tools.

**Visibility rule:** "You" sees everything. Viewing as X shows X's pod, X's
direct connections, and Sepand's family circle (Sepand + the Immediate
family / Cousins pods). Family ties are reciprocal; friend/work pods stay
scoped to direct edges. Example: viewing as Marcus Chen shows the Chen pod
plus Sepand's family circle, while Elise Park is omitted.

## 5. Pods

A pod is a household/family unit. **Every pod has exactly one primary
member** (`primaryMemberId`, enforced by `validate()`); everyone else's
relationship is captured relative to the primary.

- **Pod-first add-person:** the entry choice is "New Pod" vs "Existing Pod"
  (not "create/select person"). Existing Pod shows the pods visible under the
  current Viewing-as, then a form: free-form name + "Relationship to {primary
  member}" (a typed base-edge dropdown, never free text — the edge is stored
  between the new person and the primary). The pod's member list stays visible
  below as context, with each member's primary-relative relationship.
- **Duplicate avoidance:** typing a name that matches an existing visible
  graph member shows an inline hint to link the existing node instead of
  creating a duplicate.
- **New Pod:** the created person becomes the pod's primary member.
- **Pod labels are perspective-relative and derived**, never stored. Viewing
  as Daniel, the Parents pod reads "Uncle's family"; viewing as You it reads
  "Immediate family".

## 6. Perspective-relative labels

Relationship labels are always relative to the current Viewing-as person,
everywhere — People rows, pod badges, person detail, graph edges, search.

**Viewing as Daniel Rahimi:** Parents badge "Uncle's family"; Arman "Uncle ·
married to Laleh"; Laleh "Aunt · married to Arman"; Daniel's own row "You ·
household anchor"; Daniel's pod badge "Immediate family"; searching "uncle"
finds Arman.

**Viewing as You (Sepand):** Parents "Immediate family"; Arman "Dad";
Daniel "Cousin · household anchor"; Daniel's pod "Cousins"; searching
"niece" finds Nora; searching "cousin" finds Daniel.

Owner-relative stored copy (e.g. "Dad", "Mom") is kept per person for the
owner's view; every other perspective derives fresh.

## 7. Add-milestone + verification workflow

- The person field is a **searchable dropdown of existing graph members
  only** — no free text. The option list respects the current Viewing-as
  visibility scope and recomputes when the perspective changes (a selection
  that becomes invisible is cleared). "Add person first" links to the
  add-person flow.
- Every milestone records `addedBy` and a `verification` status:
  `verified` | `pending` | `disputed`.
- A milestone added **for someone else** starts as **pending**; self-entered
  milestones (e.g. via the sourcing link) start as **verified**.
- In the subject's view (Viewing as them), pending milestones are
  **highlighted** ("Added by X — is this right?") with **Verify** (one tap)
  and **Dispute** (optional correction note) actions.
- Disputed items show "You've flagged this for review" to the subject and
  surface the dispute note to the owner (Viewing as You) for correction.
- Verification UI never leaks a private milestone to someone outside its
  visibility (see §8).

## 8. Private milestones

- Milestones can be marked private (e.g. Arman's yearly all-day "Day of
  Cancer Remission", quiet non-celebratory wording).
- Visible only to Sepand and members of the milestone owner's pod; hidden
  from other perspectives even through reciprocal family visibility.
- Visible private milestones carry a subtle lock badge.
- **Private milestones are never exported or synced.**

## 9. Calendar

- Month/Week toggle; week view exists because crowded days overflow month
  view.
- Household and type filters apply to both views.

## 10. Graph page

- Central node is the current viewed-as person; pods arranged around it.
- Primary edges carry perspective-relative relationship labels; secondary
  inter-pod relationships (e.g. Parents ↔ Daniel's family, derived from the
  Arman↔Kian sibling edge) render as unlabeled dashed lines.
- Opening a pod node shows tappable members; respects global Viewing-as;
  mobile-first and legible.

## 11. Sourcing link flow

Invite view generates a per-person link. The recipient fills in their own
milestones (self-entered → verified). Milestones Sepand adds for someone else
enter as pending and surface in that person's view for Verify/Dispute (§7).

## 12. Calendar export/sync — spec only, not built

Sepand: "Leave it as a spec for now." Full spec in
`../goals/milestone-tracker-app/calendar-sync-plan.md` (kept with the goal,
not the repo, until it's built). Shape: app is source of truth, external
calendars are read-only mirrors; dedicated "Milestones" calendar; one yearly
recurring all-day event per milestone; private revocable webcal feed first,
Google Calendar OAuth/API later; private milestones never exported; Viewing-as
is preview-only, never an export scope.

## 13. Persistence model (repo era)

- Whole working state (people, pods, relationship edges, milestones with
  `addedBy`/`verification`, UI prefs) serializes to `localStorage` under
  `milestoneTracker.v1`.
- First run seeds from the in-code sample data and saves immediately; every
  mutation writes through.
- "Reset sample data" (header) clears storage and reseeds on reload.
- Fully static: single `index.html`, no build step, no network dependencies.

## 14. Constraints

- Static files only, no build step, no server.
- No US-hosted runtime dependencies (no CDNs, no Google/Amazon services) —
  the app works offline from any plain static server.
- Sample data uses real-sounding family names: keep the repo **private**
  unless Sepand explicitly approves otherwise.
- Public artifact links are never created without explicit approval.
