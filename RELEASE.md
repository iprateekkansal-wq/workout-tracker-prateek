# Release Notes

## [2026-06-02]

### Improvements
- **Reopen a completed session to add exercises retrospectively** (commit pending, 2026-06-02)
  - Completed sessions now show a "Reopen Session" button in the detail view — opens the session in the editable session screen so a forgotten exercise can be added after the fact
  - The session keeps its completed status and original completion timestamp the entire time it is being edited; the flags are only re-evaluated on exit, so a session that is reopened and backed out of is never demoted or stripped of its timestamp
  - Reopening is blocked if another session is already active, with an alert explaining why

---

## [2026-05-25]

### Improvements
- **Two UX improvements to logging flow** (commit `f57e6f5`, 2026-05-25)
  - Weight/rep fields now select all text on focus — tap to immediately overwrite, no manual erase needed
  - Done exercise blocks now show × with a confirmation dialog before deleting logged data

- **Focused log-exercise screen** (commit `40d6a6d`, 2026-05-25)
  - Selecting an exercise from the picker navigates directly to a focused logging screen — no extra tap
  - Session list shows compact blocks: done exercises show logged set summary (e.g. 45kg×11 · 40kg×12), undone show "tap to log"
  - Logging screen has two blobs: previous sessions (tappable → progression chart) and set inputs + Done
  - Progression chart opens as a full-screen overlay; back returns to logging screen

- **Show last 3 sessions history in exercise block** (commit `9f33a61`, 2026-05-25)
  - Each exercise block now shows up to 3 recent sessions above the inputs
  - Format: date · set1weight×reps · set2weight×reps · set3weight×reps
  - Tapping any history row navigates to the exercise progression graph

- **Per-set prefill from last session** (commit `b95643d`, 2026-05-25)
  - Each set is now prefilled with the corresponding set from the last session (set 1 → last set 1, set 2 → last set 2, etc.)
  - If last session had fewer than 3 sets, the last set's values are reused for remaining sets
  - Falls back to previous behaviour if no history exists for that exercise

- **Close exercise picker on tap** (commit `0831c73`, 2026-05-25)
  - Tapping an exercise now closes the picker immediately and returns to the session screen
  - Previously required manually closing the picker after adding an exercise

---

## [2026-05-23]

### Bug Fixes
- **Fix loading screen stuck on PWA page reload after OTP login** (commit `2504dee`, 2026-05-23)
  - `getSession()` can be slow on page reload (reads from cookie, may check token expiry)
  - `onAuthStateChange` fires before `getSession()` resolves, setting `currentUser` but not clearing `authLoading`
  - Result: app was ready but hidden behind loading screen indefinitely
  - Now clear `authLoading` in `onAuthStateChange` so UI is never blocked waiting for `getSession()`

- **Fix OTP code input truncating to 6 digits** (commit `27dc597`, 2026-05-23)
  - Supabase sends 8-digit OTP codes but input was limited to `maxlength="6"`
  - Users were unable to enter the full code, causing all sign-in attempts to fail
  - Updated maxlength, placeholder, button validation condition, and instructions to match 8 digits

### Security
- **Move Supabase credentials to GitHub Secrets** (commit `6f3fe1b`, 2026-05-23)
  - Moved hardcoded JWT tokens from `.github/workflows/keep-alive.yml` to GitHub Secrets
  - Requires adding `SUPABASE_URL` and `SUPABASE_ANON_KEY` as GitHub repository secrets
  - Workflow now references credentials via `${{ secrets.SUPABASE_URL }}` and `${{ secrets.SUPABASE_ANON_KEY }}`
  - Impact: More secure credential handling; workflow deployment credentials no longer exposed in version control

---

## Release Process

All releases must follow these steps:

1. **Branch**: Develop on `claude/workout-tracking-app-u2CAV`
2. **Test**: Verify changes locally before requesting merge
3. **Approval**: Request explicit approval before pushing to `main`
4. **Documentation**: Update RELEASE.md with release notes before merge
5. **Version**: Tag stable releases with `git tag -a vYYYY-MM-DD -m "description"`
6. **Deploy**: Push to `main` triggers auto-deploy via Vercel

---

## Archive

| Date | Commit | Version | Description |
|---|---|---|---|
| 2026-05-21 | `1e5966c` | - | Supabase auth + sync + keep-alive + CLAUDE.md. Full working state. |
