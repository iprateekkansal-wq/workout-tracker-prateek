# Release Notes

## [2026-05-25]

### Improvements
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
