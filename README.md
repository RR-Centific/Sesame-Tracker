# Project Sesame — Moderator Checklist & Admin Tracking App

## What is this project?

**Project Sesame** (production app: **Sesame Tracker**) is a data-collection moderator app plus an admin dashboard for tracking video collection toward a 100-hour target. Moderators use a runbook-aligned checklist to complete one C0 calibration capture and one T1 natural-tracking capture in each of 1–6 selected rooms. Admins see progress toward that target (demo metrics until SharePoint is wired). On-screen brand in the app is **Sesame**.

The app is a single file: **`index.html`** (GitHub Pages serves it from the repo root). Edit that file only. SharePoint Lists and Power Automate are the intended backend; they are not wired yet, so the UI still runs on local placeholder data.

**Moderator app (in `index.html` today):**
- Sign in with a username (no password). Client demo: `moderator`
- Sidebar of **assigned** sessions only (not a global “today” list); can collapse on desktop
- Session detail with phased checklists (Device Prep / Session Checklist / Post-Session)
- Required Hydra Intake metadata: Feather Task ID, people count, age range(s), and 1–6 actual room names
- One C0 capture per session plus a generated T1 checklist for every selected room
- Capture metadata for C0 duration/review and per-room T1 retained views, attempts, duration, and review decision
- Step ticks saved locally; writes to SessionLog when flows are wired
- Menu: placeholder page links (Checklist, Project Updates, Guidelines, Troubleshooting), sign out, demo Reset app, reminders, Latest Update; username and theme toggle are in the nav

**Admin app (in `index.html` today):**
- Sign in as `admin` (no password)
- Read-only Kilo-style dashboard: hours vs 100h target, sessions, sites, completion rate, breakdowns, moderator table
- Figures are **demo stand-in data** until SessionLog is live

**Scope**: Mobile-first, also usable on tablet and desktop. Vanilla HTML/CSS/JS, deployed on GitHub Pages.

---

## What's in this folder?

| Path | What it is | For whom |
|---|---|---|
| `README.md` | This file — project overview | Anyone (before other docs) |
| `Dev_Notes.md` | Full working context: architecture decisions, to-do list, roadmap | Claude chats picking this up later |
| `Changelog.md` | Version-by-version history of changes and why | Anyone curious about project evolution |
| `Team_Handoff.md` | How to maintain/modify the app without the original owner | Human teammates inheriting this project |
| `Reference Files/` | Orbit architecture, design system, team guidelines, Kilo UI reference | Anyone writing or extending the app |
| `index.html` | Single-file moderator + admin app (GitHub Pages homepage) | Local testing and the live site |
| `Session Checklist.xlsx` | Protocol source used for Session Checklist copy | Content owners / future copy updates |

---

## How the app is used

### For Moderators (current local build)
1. Open `index.html` in a browser
2. Sign in as `moderator` (amber demo banner). All seven placeholder sessions appear, one per day (Aug 23–29); the two sessions before Aug 25 are already completed
3. See **Your Assigned Sessions** in the sidebar
4. Enter the required Intake metadata; the selected room count generates one T1 group per room
5. Work through Device Prep → C0 once → T1 in every room → Post-Session
6. Record capture metrics and package SHA-256, then Complete session (enabled only when steps, required metadata, and notes are complete)
7. When Power Automate is wired, step events, metadata updates, and completion will sync to SharePoint

**Typical flow**: Sign in → Device Prep → Intake metadata → C0 → T1 room(s) → Ingestion and package verification → Complete

### For Project Leadership (Admin view)
1. Open `index.html`, sign in as `admin`
2. Review collection progress (hours vs 100-hour target, sessions, sites, moderator table)
3. No data entry — monitoring only. Numbers are demo until the backend is wired.

---

## How it fits into the bigger picture

**Upstream**: A scheduling system (separate from Sesame) assigns moderators to sites and sessions. Sesame reads the assigned sessions at login.

**Sesame's role**: Ensures the data-collection protocol is followed. Will provide reference clips/GIFs. Logs step completion for audit (SessionLog) once the backend is live.

**Downstream**:
- **QA process** (separate): The data team reviews recorded video against the SessionLog. If video doesn't match checklist, it's flagged. Sesame's audit trail makes QA much easier.
- **Analytics**: Project leadership uses admin dashboards to track progress toward 100-hour target, identify bottlenecks, and make staffing decisions.
- **Reporting**: HR/scheduling can see which moderators were productive, which sites had issues, etc.

**Integration points**:
- SharePoint Lists (backend): Read Sessions + Participants, Write SessionLog
- Power Automate flows: Sync session data to admin dashboards
- Reference media: Links to video clips/GIFs stored in SharePoint Document Library or shared drive

---

## Where to go from here

| Your goal | Read this | Why |
|---|---|---|
| **I'm setting up the project for the first time** | `Dev_Notes.md` → `ARCHITECTURE_REFERENCE.md` | Dev_Notes tells you what's set up and what's not; Architecture explains how to extend it |
| **I need to fix a bug or add a feature** | `Dev_Notes.md` → `ARCHITECTURE_REFERENCE.md` → then the code | Dev_Notes has recent context; Architecture covers patterns you'll need |
| **I'm taking over this project from someone else** | `Team_Handoff.md` | Written for you specifically — how to maintain it |
| **I need the full history of what changed** | `Changelog.md` | Each version documents why changes were made |
| **I'm writing code and need to understand the current state** | `Dev_Notes.md` → **look at `state` object** in `index.html` | Dev_Notes tells you what's shipped, what's in progress, and why certain decisions were made |
| **I want to know about architectural patterns (buttons, auth, sync, etc.)** | `Reference Files/ARCHITECTURE_REFERENCE.md` | Covers design system, components, state management, backend patterns — all inherited from Project Orbit |

---

## Quick start for a new session with Claude

If you're reopening this project in a new Claude chat:

1. **Read this README** (5 min) — you're here
2. **Read `Dev_Notes.md`** (10 min) — tells you what's built, what's pending, and recent decisions
3. **Read `Reference Files/ARCHITECTURE_REFERENCE.md`** (20 min) — if you're writing code, this is essential
4. **Then describe what you need** — Claude will know the context

---

## Version and status

**Current version**: `0.3.091426` — details in `Changelog.md`
**Status**: Client-demo moderator UI with placeholder data; backend not wired. Live demo: https://RR-Centific.github.io/Sesame-Tracker/  
Details in `Dev_Notes.md`.

For deployment instructions, see `Team_Handoff.md` → "Maintaining the application."

---

## Questions?

- **"How does X work?"** → Check `Dev_Notes.md` (design decisions) or `ARCHITECTURE_REFERENCE.md` (patterns)
- **"What changed in the last update?"** → Check `Changelog.md`
- **"How do I deploy this?"** → Check `Team_Handoff.md` → "Maintaining the application"
- **"How do I add a new backend table?"** → Check `Team_Handoff.md` → "Things that must not change" + `ARCHITECTURE_REFERENCE.md` section 8
