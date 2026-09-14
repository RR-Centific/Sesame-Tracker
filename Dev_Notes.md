# Dev_Notes — Project Sesame Working Context

This document is written for a Claude chat picking this project back up. See `README.md` for the big picture.

**Architecture note**: Project Sesame has:
- **Frontend**: Kilo-style checklist UI (sidebar + main panel, mobile-first, Kilo design system)
- **Backend**: SharePoint Lists + Power Automate flows (not local-only, not Excel)
- **Scope**: Moderator app + optional admin dashboards. No mid-session approval workflow.
- **Goal**: Track every step of data-collection for 200+ sessions in 3 weeks, heading toward 100-hour video target.

---

## File & folder inventory

```
Project Sesame (local folder may still be named Project Wave Checklist App Test)/
├── README.md                          [Front door — orientation, where to go]
├── Dev_Notes.md                       [This file — working context]
├── Changelog.md                       [Version history and why]
├── Team_Handoff.md                    [For humans maintaining the project]
│
├── index.html                         [Single-file app — local testing and GitHub Pages homepage]
├── Session Checklist.xlsx             [Protocol source for Session Checklist copy]
│
└── Reference Files/
    ├── Team Shared Guidelines/
    │   ├── Project_Documentation_Instructions.md    [The four-doc standard we follow]
    │   ├── PROJECT_INSTRUCTIONS.md                  [Original project brief]
    │   ├── ARCHITECTURE_REFERENCE.md                [Patterns from Project Orbit]
    │   ├── CENTIFIC_DESIGN_SYSTEM_01.md             [Centific visual brand]
    │   └── App_Build_Workflow_and_Replication_Guide.md  [General build/deploy/Claude-workflow guide — backend section assumes Excel, see note below]
    │
    └── Project Kilo Files/
        ├── centific-kilo-design-system.md
        ├── index.html                              [Reference implementation]
        └── Kilo Task Tracker.html                  [Another reference app]
```

### What each reference file is for

- **Kilo Task Tracker.html** — UI reference. Copy HTML structure, CSS, and Kilo design system. Modify the task/step data structure for Sesame's sessions/steps.
- **centific-kilo-design-system.md** — Visual tokens: colors (#EF43B3 pink accent, dark mode), spacing, typography. Use exactly.
- **ARCHITECTURE_REFERENCE.md** — Project Orbit's backend patterns. Sesame uses similar concepts (cloud sync, status logging, soft deletes) but with SharePoint Lists instead of Excel. Reference sections 7-9 for state management, Power Automate + backend patterns, and refresh architecture.
- **PROJECT_INSTRUCTIONS.md** — Original brief. Updated to clarify Sesame has backend (SharePoint Lists + Power Automate), unlike local-only Kilo.
- **App_Build_Workflow_and_Replication_Guide.md** — General Centific playbook for these single-file apps: tech stack, deploy, Claude workflow, pre-ship QA, replication steps. **Known divergence**: its section 4-5 (the database and connectors) is written entirely around Excel tables on SharePoint (Excel Online actions, all-Text columns, 256-row pagination cap). Sesame intentionally uses SharePoint Lists instead — see "SharePoint Lists + Power Automate backend" under Key decisions below for why. When following this guide's replication steps for Sesame, substitute: "Get items" (SharePoint) for "List rows" (Excel), List column filters for Excel column filters, and native SharePoint column types where the guide says Text-only. Everything else in the guide (single-file HTML, version stamping, pre-ship QA checklist, dormant-until-wired URLs, deployment) applies to Sesame as written.

---

## Current state

### Version
**0.3.091426** (`APP_VERSION` in `index.html`). See `Changelog.md`.

### Shipped (local-only skeleton)
- [x] Frontend skeleton in `index.html` (Kilo tokens, login, app shell, menu, theme toggle, mobile layout)
- [x] Sidebar: **Your Assigned Sessions**, date/time, type chip, participant short name, phase dots; completed sessions show a full-width pink **completed** pill. Desktop collapse-to-rail control (remembered).
- [x] Session detail: three phases — **Device Prep**, **Session Checklist**, **Post-Session**. Notes + Complete session always visible on Post-Session; Notes are required (helper: “Please describe how the session went and note anything out of the ordinary”); button disabled until `sessionComplete()` and notes are non-empty.
- [x] Runbook-aligned Session Checklist from `Hydra-mmWave-Centific-Collection-Runbook-8_27.pdf`: required Hydra Intake metadata; C0 once per session; 1–6 generated T1 room groups; sequential locking; capture metadata and final package SHA-256; desktop two-pane view and mobile drill-in
- [x] Login against placeholder Moderators. Client demo: `moderator` / `admin` (amber **Demo Version Logins** banner). No username prefill. Legacy: `riley.robertson`, `david.kang`, `wave.admin`
- [x] Moderators only load/see sessions assigned to them (`sessionAssignedTo`). `moderator` sees every placeholder session (Redmond + Las Vegas), one per day Aug 23–29; the two sessions scheduled before Aug 25 are pre-completed, the rest are incomplete
- [x] localStorage per username (`sesame_session_v30_<username>`; bumped for the runbook/data-model change)
- [x] Cloud sync **stubs**: `MODERATORS_READ_URL`, `SESSIONS_READ_URL`, `SESSIONS_WRITE_URL`, `SESSIONLOG_WRITE_URL` are `''`; reads fall back to placeholders; writes log to console
- [x] Menu: placeholder page links (Checklist / Project Updates / Guidelines / Troubleshooting) with the current page marked by `.current` + `aria-current="page"` (hardcoded to Checklist until the other pages exist; chevron hidden on the current item), then Sign out, **Reset app (demo only)** with an amber outline (keeps the user signed in and restores placeholder checklists), then placeholder reminders and a **Latest Update** card (currently New SSDs, Aug 24, 2026). Username is in the top header (left of theme toggle, user icon to the right of the name).
- [x] Single-file layout: `index.html` only (GitHub Pages homepage; `demo.html` removed)
- [ ] Theme toggle is in the nav (works); not a separate “settings” panel
- [ ] Reference media: steps may have `ref_media_id`; UI shows “not yet configured”
- [ ] Menu reminders are still placeholder copy

### In Progress
- Production repo is live on GitHub Pages. Iterate UI against that URL. Backend not started.

### Not yet started
- Real reminders & troubleshooting
- Reference media URLs
- Live admin metrics from SharePoint (UI exists; data is demo)
- Power Automate flows (URLs still empty)
- SharePoint Lists
- Clearing leftover demo logins / placeholder roster before real users

### How to run locally
Open `index.html` in a browser, or the live site at https://RR-Centific.github.io/Sesame-Tracker/. Amber banner: `moderator` (checklist) and `admin` (dashboard). If the list looks stale, sign out or use a private window — storage key is currently `sesame_session_v30_`. Edit `index.html` only.

---

## To-do list

### Before first release (v1.0) — 3 week deadline

#### Week 1: Backend + Frontend skeleton

**Backend (SharePoint + Power Automate):**
- [ ] Create SharePoint Lists:
  - [ ] **Moderators** (id, username, email, site_assigned, active)
  - [ ] **Sites** (id, name, location, contact_info)
  - [ ] **Participants** (id, participant_id_code, demographics?, consent_status)
  - [ ] **Sessions** (id, moderator_id, participant_id, site_id, session_type [single/paired], scheduled_time, start_time, end_time, status [in_progress/completed/abandoned], notes; useable_minutes optional — not collected in the current UI)
  - [ ] **SessionLog** (id, session_id, event [started/step1_done/.../completed], timestamp, moderator_id) — append-only
  - [ ] **ReferenceMedia** (id, session_type, step_number, file_url, description)
- [ ] Create Power Automate flows:
  - [ ] Read flows for each table (return all rows as JSON)
  - [ ] Write flow for SessionLog (append only, no updates)
  - [ ] Write flow for Sessions (update status, useable_minutes)
  - [ ] Get flow URLs, wire into `index.html` constants

**Frontend skeleton:**
- [x] Create `app.html` from Kilo patterns:
  - [x] HTML structure, CSS, Kilo design system
  - [x] Login screen + username validation against Moderators list (placeholder list until the read URL is wired)
  - [x] Sessions loaded from placeholder data (or backend when wired), filtered to the signed-in moderator
  - [x] Cloud sync stubs: read Sessions at login, write SessionLog on step completion (no-ops while URLs are empty)
  - [ ] Verify: JS parses, CSS braces balance — do this before calling a build “done”
  - [x] Version stamp `0.1.082026`

#### Week 2: Content + admin basics

**Workflow definition:**
- [x] Device Prep checklist — landed in `index.html`
- [x] Session Checklist (greet → NDA/consent → explain/rehearse → Hydra M1 / A1 / A2 / T1); greet/explain wording still differs by session type
- [x] Post-Session checklist (Hydra ingestion while on X5 Wi-Fi, finish Feather metadata and mark task Completed, TAR upload, pack/return)
- [ ] Identify reference media needs (clips/GIFs moderators will need)
- [ ] Write key reminders for menu panel
- [ ] Write troubleshooting tips

**Frontend:**
- [ ] Integrate reference media URLs (clips/GIFs embeddable in steps)
- [ ] Add session type logic (different step sequences for single vs. paired)
- [x] Build menu panel (reminders, troubleshooting table, settings)
- [x] Add session completion form (notes required; Complete session gated on all steps + notes)
- [ ] Test on iPhone + Android
- [x] Version bump to `0.3.091426` for the runbook workflow and metadata data-model change

**Admin dashboard (basic):**
- [x] Create admin login view (`admin` / `wave.admin`, `role: admin`)
- [x] Build progress dashboard: hours vs 100h, sessions, sites/moderators (demo stats in `placeholderAdminStats()`)
- [ ] Add session list view with filter/sort
- [ ] Replace demo stats with live SessionLog / Sessions queries

#### Week 3: Polish + final testing + deployment

- [ ] Full end-to-end testing: login → see sessions → complete session → verify in admin dashboard
- [ ] Fix bugs from testing
- [ ] Populate all reference media
- [ ] Brief moderator team on workflow
- [x] Deploy to GitHub Pages (`https://RR-Centific.github.io/Sesame-Tracker/`; source branch `main`)
- [ ] Version bump to 1.0.MMDDYY

### Post-release (if time/scope allows)
- [ ] More detailed analytics in admin dashboard (completion by site, by moderator, etc.)
- [ ] Export session data to CSV
- [ ] Session timer
- [ ] Offline mode (if needed)

---

## Roadmap of deferred work

### Critical questions to answer before Week 1 starts

1. **Workflow definition** (affects step checklist):
   - What are the exact steps for a single-participant session? (e.g., 1. Greet participant 2. Place camera 3. Record 5-7 min video 4. Verify audio/video quality 5. Review with participant 6. End session)
   - What are the steps for paired-participant sessions? (any differences?)
   - Are there warnings/special instructions? (e.g., "Do not reposition camera between sessions")
   - Are there conditional steps? (e.g., "If video quality < acceptable, re-record")

2. **Reference media** (for embedded clips/GIFs):
   - What reference clips/GIFs will moderators need? (e.g., "Here's correct camera placement", "Here's what acceptable video quality looks like")
   - Where will these be stored? (SharePoint Document Library, OneDrive, shared drive?)
   - Who will upload/maintain them? (video team, PM, or Sesame admin?)
   - Will media be per-step or per-session-type?

3. **Key reminders & troubleshooting**:
   - What are the 5-10 most important things moderators should never forget?
   - What are the 5-10 most common issues they'll encounter? (and how to fix?)

4. **Participant tracking**:
   - Will participant_id be auto-generated or entered by moderator?
   - Do we need to validate that same participant doesn't do too many sessions? (or no constraints for now?)

5. **Moderator workflow clarification**:
   - Should Sessions be pre-populated in the app, or created by moderator at start of day?
   - Will Sesame integrate with a scheduling system, or is scheduling out-of-band?

---

## Key decisions & reasoning

### Single-file HTML + vanilla JS (Kilo pattern)
**Decision**: Build as one `.html` file with inline CSS and JavaScript. No framework, no build pipeline.

**Why**: 
- Deployment is trivial (edit `index.html`, push to GitHub Pages)
- No dependencies, no build step
- Fast loading on mobile devices in field
- Proven by Kilo for moderator apps

**Trade-off**: Larger single file (~8-12K lines expected). Mitigated by aggressive section comments and clear naming.

---

### SharePoint Lists + Power Automate backend (hybrid Orbit + Kilo)
**Decision**: Data lives in SharePoint Lists (not Excel, not localStorage). Power Automate flows handle read/write.

**Why**:
- SharePoint Lists have NO 256-row pagination ceiling (critical for 200+ sessions)
- Proper column types (Date, Number, Lookup) — easier than Excel all-Text approach
- Power Automate integrates seamlessly with Microsoft 365
- Admin can view/query data natively in SharePoint
- Scales better than Excel as data grows

**Trade-off**: Slightly more complex than local-only. Session logging happens in real-time (important for audit trail).

**Key pattern**: SessionLog is append-only (immutable). Sessions table is updated for completion status only. No soft-delete needed (no concurrent write collisions on same session).

---

### Session + Step structure (Kilo pattern, adapted for Sessions)
**Decision**: Each session has multiple required steps. Steps are ticked as completed. Progress tracked in real-time.

**Why**:
- Proven UX from Kilo (familiar to moderators)
- Sidebar overview of all sessions + progress bar
- Main panel shows current session's steps
- Easy to reference media (clips/GIFs) alongside steps

**Schema (as implemented in placeholder sessions):**
```js
{
  session_id, moderator_id, site_id,
  session_type: "single|paired",
  participants: ["Full Name", ...],  // UI shows first name + last initial via sessionLabel()
  scheduled_date: "YYYY-MM-DD",
  scheduled_time: "9:00 AM",
  scheduled_at: "YYYY-MM-DDTHH:MM:SS", // sort key
  status: "in_progress|completed",
  useable_minutes, notes,  // UI no longer collects useable_minutes; field remains on the session object for a future backend
  phases: [{
    key, title,
    steps: [
      { kind: "info", title, t },
      { kind: "child-title", key, t, done? }, // top-level; toggles on the main list
      {
        kind: "parent", key, t,
        children: [
          { kind: "child-title", key, t, done? },
          { kind: "child-detailed", key, t, description, done? }
        ]
      }
    ]
  }]
}
```
Flat phases (Device Prep and Post-Session) still use simple step objects. SessionLog events (when wired): `step_completed`, `step_reopened`, `metadata_updated`, and `session_completed`; nested child events also include `parent_key`, `parent`, and `step_key`. The completion payload includes session-level and scenario-level metadata.

---

### Three-level task behavior (Session Checklist redesign)
**Decision**: The Session Checklist supports three task types. These names are for developers only and never appear in the UI:
- **Parent** (`kind: "parent"`): opens its children and cannot be toggled directly; complete only when every child is complete.
- **Child title-only** (`kind: "child-title"`): manually toggled, title only.
- **Child detailed** (`kind: "child-detailed"`): manually toggled, title plus smaller descriptive copy. Completion uses the tertiary color on both title and description; no strikethrough.
- **Section** (`kind: "section"`): visual header only (`before_recording`, `after_recording`). Optional `note` is helper copy under the header, not a step and not in progress totals.

**Responsive behavior**:
- Desktop: parent list on the left; selected child list in a pane to the right.
- Mobile: parent rows have right arrows; tapping drills into a full child list that slides in from the right, with an in-content back button.

**Phase header area**: The phase name is not repeated above the list — the segmented control is the only label. `.seg-control` carries `margin-bottom:43px`, which reproduces the old spacing (20px control margin + 11px label + 12px label margin); change it if the control or list padding changes. The selected segment uses `--seg-active`, a per-theme value deliberately darker than `--card-bg` so the selection reads at a glance.

**Current Session Checklist structure**: Required Intake metadata appears above the task browser: Feather Task ID, one/two people, age range(s), room count, and one actual room name per selected room. Changing room count dynamically rebuilds 1–6 T1 groups while preserving progress for unchanged room keys. Scenario order is C0 once, then `scenario_t1_room_1` through `scenario_t1_room_N`. Each group unlocks only after Intake metadata and all prior tasks/groups are complete. C0 requires accepted duration and review decision. Each T1 room requires retained-view count (maximum 15), total attempts (not less than retained views), duration (maximum 30 minutes), and an Accepted review decision. Post-Session requires the package SHA-256. Section headers are informational and do not count as steps. Do not change keys casually—they will be written to SessionLog.

---

### Cloud-first sync (Orbit pattern)
**Decision**: On login, fetch all assigned sessions from SharePoint. On every step completion, append to SessionLog. Session completion updates Sessions table.

**Why**:
- Audit trail: SessionLog shows exactly what happened, when
- Admin dashboards can query progress in real-time
- Supports QA process (video team can cross-reference SessionLog)
- Scales to 200+ sessions

**Implication**: Moderators need internet connectivity (or the app will cache and sync when connection returns — offline retry is not implemented).

---

### Moderators only see assigned sessions
**Decision**: Filter at login (and again in `allSessions()`) so `moderator_id` on the session must match the signed-in profile. Do not show a global session list.

**Why**: Field moderators should not see another site’s roster. Admins (`role: admin`) skip the session list and open the dashboard instead.

---

### Demo admin dashboard (Kilo KPI pattern)
**Decision**: Admin view is a read-only dashboard in the same HTML file. KPIs follow Kilo (large numbers, site/type/day bars, moderator table). No Chart.js — CSS bars only, to stay zero-dependency.

**Why**: Leadership needs a progress surface now for design review. Live SessionLog queries come later; `placeholderAdminStats()` is explicitly labeled demo data.

---

### Project name is Project Sesame
**Decision**: The product is **Project Sesame**. Production GitHub repo and Pages site are `Sesame-Tracker`. Nav, login brand, page title, and the menu version stamp say **Sesame** (currently `Sesame v0.3.091426`). The local workspace folder and the Aug 2026 demo repo (`Wave-Checklist-Beta`) still use the old Wave name. The legacy demo login `wave.admin` remains; session storage moved to `sesame_session_v30_` for the new runbook schema.

**Why**: Name locked 14 Sep 2026. `mmWave` / Project Wave were temporary names during skeleton and client-demo work.

---

### Placeholder data for UI development
**Decision**: Until flow URLs are non-empty, use `PLACEHOLDER_MODERATORS`, `PLACEHOLDER_CONTACTS`, and `placeholderSessions()`.

**Current stand-ins:**
| Username | Site | Sessions |
|---|---|---|
| `moderator` | Demo (all sites) | All seven placeholder sessions, one per day Aug 23–29; the two before Aug 25 are pre-completed |
| `admin` | All | Demo dashboard (not a session list) |
| `riley.robertson` | Redmond | 4 sessions, Aug 23, 24, 26, 28 2026 (those before Aug 25 pre-completed) |
| `david.kang` | Las Vegas | 3 sessions, Aug 25, 27, 29 2026 (all incomplete) |
| `wave.admin` | All | Demo dashboard (same as `admin`) |

Participants are the ten names from Riley’s `fake_contacts2.csv`, shown as first name + last initial (e.g. `Sarah M.`, paired `Michael C. / Lisa R.`).

---

### No mid-session approval (simpler than Orbit)
**Decision**: Moderators mark session complete, data goes to backend immediately. No "pending review" status.

**Why**:
- Simpler workflow (1 less state machine)
- QA happens separately (out of app)
- Keeps moderator app focused on data collection

---

### Reference media embedded in steps
**Decision**: Each step can reference a clip/GIF (stored in SharePoint, linked by URL).

**Why**:
- Moderators see "correct form" as they work
- Reduces misinterpretation of instructions
- Easy to update/fix without rebuilding app

**Implication**: ReferenceMedia table needed. URLs must be accessible from collection sites (consider permissions).

---

## Working conventions

### How to collaborate with me (Riley)
- **Keep the four docs current with the code**, per `Reference Files/Team Shared Guidelines/Project_Documentation_Instructions.md`. `Dev_Notes.md` updates with every change; `Changelog.md` for notable versions; `README.md` only when the big picture or folder layout changes; `Team_Handoff.md` when operating procedure or unfinished work changes. Do not skip docs until the end of a session.
- **Ask before bumping MAJOR.MINOR** (date segment in `APP_VERSION` can follow the build date). See Changelog versioning notes.
- **Before proposing a big architectural change**, check `Dev_Notes.md` and `ARCHITECTURE_REFERENCE.md` first. Many patterns are locked in for good reason.
- **Ask clarifying questions early**. Ambiguity about workflow/inventory/admin-freshness can compound into rework.
- **Iterative builds over big-bang**. Ship small features, verify, iterate.
- **Show verification results**. Every build should end with: JS parses ✓, CSS balanced ✓, feature checks pass ✓, version considered ✓.

### Code structure expectations
- **High comment density**. Document WHY, not just WHAT. Especially browser quirks, mobile-specific decisions, trade-offs.
- **Section comments for long files**. Use visual banners to mark major sections (cloud sync, login, inventory, etc.).
- **Diagnostic console logs**. `console.log('[App] ...')` at decision points so bug reports can be self-diagnosed.

### Manual edits
If you make a direct edit to `index.html` (not through Claude), **mention it next time** — Claude has no way to know about it otherwise. This prevents Claude from undoing your changes or conflicting with them.

---

## Session-to-session handoff checklist

When closing a session, if you've made progress:

- [ ] Update `Dev_Notes.md` → "Current state" section (what shipped, what's in progress)
- [ ] Update `Dev_Notes.md` → "To-do list" (check off completed items)
- [ ] Create or update `Changelog.md` with the version number and changes
- [ ] If you changed the HTML app significantly, add a note to `Dev_Notes.md` → "Recent decisions" (context for next session)

This way, the next session can pick up without re-explaining everything.

---

## References

- **For architectural patterns**: `Reference Files/ARCHITECTURE_REFERENCE.md` (sections 1–4 are essential before writing code)
- **For the original spec**: `Reference Files/PROJECT_INSTRUCTIONS.md`
- **For deployment**: See `Team_Handoff.md`
- **For version history**: `Changelog.md`
