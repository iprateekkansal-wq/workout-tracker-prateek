# Workout Tracker — Claude Context

Personal workout tracking PWA built for long-term use. Core philosophy: **build it once, don't touch it for years**. Every decision prioritises simplicity and longevity over cleverness.

## Ownership & Deployment

- **Owner**: Prateek (iprateekkansal@gmail.com)
- **Repo**: `iprateekkansal-wq/workout-tracker-prateek`
- **Dev branch**: `claude/workout-tracking-app-u2CAV`
- **Production**: auto-deploys from `main` via Vercel
- **Live URL**: https://workout-tracker-prateek-seven.vercel.app

## Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Framework | Vue 3 (CDN, no build step) | Zero toolchain to maintain; works in any browser forever |
| Charts | Chart.js 4 (CDN) | Single CDN line, no dependencies |
| Backend / Auth | Supabase (free tier) | Managed Postgres + auth, generous free tier |
| Hosting | Vercel | Auto-deploy on push to main |
| Offline storage | localStorage | Synchronous, always available, no network needed |

**There is no build step.** Everything is a single file: `index.html`. No npm, no webpack, no node_modules. To "deploy", push to main and Vercel picks it up automatically.

## Architecture: Single File App

The entire app lives in `index.html`:
- Lines 1–242: `<head>` — meta tags, CDN scripts, all CSS
- Lines 243–755: `<body>` HTML template (Vue template)
- Lines 756–1560+: `<script>` — all JavaScript (exercise library, storage helpers, Supabase client, Vue app)

CDN scripts loaded in order:
1. Vue 3 global prod build
2. Chart.js 4
3. Supabase JS v2 (UMD)

## Data Architecture

### Offline-first with cloud sync

localStorage is the **primary, synchronous** data store. Supabase is an **async, fire-and-forget** background sync. The app works fully offline — Supabase sync failures are silent and non-blocking.

### localStorage keys

| Key | Contents |
|---|---|
| `wtp_sessions` | Array of all session objects |
| `wtp_exercises` | Array of all exercises (seed + custom) |
| `wtp_last_backup` | ISO timestamp of last JSON export |

### Session object shape

```javascript
{
  id: string,           // 'x' + random + timestamp (uuid())
  date: string,         // 'YYYY-MM-DD' local date
  name: string,         // auto-named from muscle groups
  created_at: string,   // ISO timestamp
  completed_at: string|null,
  inProgress: boolean,
  isPartial: boolean,
  retro: boolean,       // true if logged retrospectively
  blocks: Block[]
}
```

### Block object shape (inside session.blocks)

```javascript
// Exercise block
{ type: 'exercise', exercise_key: string, done: boolean, skipped: boolean,
  targetReps: number, sets: [{ weight, reps, prefillWeight, done }] }

// Cardio block
{ type: 'cardio', exercise_key: string, done: boolean,
  actualMinutes: number, speed: number }
```

### Exercise object shape

```javascript
{ id: string, key: string, name: string, group: string,
  bodyweight: boolean, timeBased: boolean, type: string }
```

Seed exercises are hardcoded in `SEED_EXERCISES` array — they are never written to Supabase. Only user-created exercises go to the `custom_exercises` table.

## Supabase Setup

### Project

- **URL**: `https://xguorgktfqmnpfrsdesh.supabase.co`
- **Anon key**: hardcoded in `index.html` (safe — RLS enforces per-user access)
- **NEVER use or expose the service_role key anywhere in the app**

### Database Tables

```sql
-- Sessions: one row per workout
sessions (id text PK, user_id uuid, date text, name text,
          created_at timestamptz, completed_at timestamptz,
          in_progress boolean, is_partial boolean, retro boolean,
          blocks jsonb)

-- Custom exercises added by user (seed exercises NOT stored here)
custom_exercises (id text PK, user_id uuid, key text, name text,
                  "group" text, bodyweight boolean)

-- Keep-alive ping target (public read, no auth needed)
health_check (id int PK default 1, updated_at timestamptz)
```

All tables have RLS enabled. Policy: `auth.uid() = user_id` for sessions and custom_exercises.

### Auth

Magic link (email OTP). No password. User enters email → gets a link → taps it → signed in. Supabase persists the session in localStorage and auto-refreshes tokens, so the user stays signed in indefinitely under normal use.

The magic link redirects to the app URL. Supabase dashboard → Authentication → URL Configuration must have the Vercel URL set as Site URL and in Redirect URLs.

### Sync behaviour

- **On login**: two-way sync — pulls remote data not in localStorage, pushes local data not in Supabase
- **On every write**: `persistSession()`, `createExercise()`, `deleteDetailSession()`, `importData()` all fire-and-forget sync to Supabase
- **If Supabase CDN fails to load**: `sb` is null, `authLoading` is immediately set to false, app works in local-only mode with no auth screen

### Keep-alive

`.github/workflows/keep-alive.yml` — GitHub Actions cron, every Monday 9am UTC. Pings `health_check` table with the anon key to prevent Supabase free tier pausing after 7 days of inactivity. Data is never deleted by pausing — project just needs ~30s to wake on next request.

## Screens & Navigation

Navigation uses a `screen` ref (string) with `screenHistory` stack for back navigation.

| Screen key | Description |
|---|---|
| `home` | Streak, last session, new session button, data export |
| `session` | Active workout — add exercises, log sets, finish |
| `session-detail` | View a completed session, delete it |
| `history` | Scrollable list of all completed sessions |
| `reports` | Stats, heatmap, progression charts, cloud sync status |
| `library` | Full exercise list, filterable by muscle group |
| `exercise-detail` | Progression chart + recent sessions for one exercise |

Auth overlay (`authLoading` / `!currentUser`) sits above all screens as `position:fixed`.

Bottom nav tabs: Home, History, Reports, Library.

## Key Functions

| Function | What it does |
|---|---|
| `startNewSession()` | Creates empty session, navigates to session screen, opens picker |
| `startRetroSession()` | Same but with a past date |
| `addExerciseToSession(key)` | Adds exercise block with 3 prefilled sets |
| `getPrefillWeight(key)` | Looks up most recent logged weight for that exercise |
| `markBlockDone(bi)` | Marks all sets done, advances to next block |
| `persistSession()` | Saves active session to localStorage + syncs to Supabase |
| `finishSession()` | Marks session complete, triggers celebration overlay |
| `createExercise()` | Adds custom exercise to localStorage + Supabase |
| `deleteDetailSession()` | Deletes session from localStorage + Supabase |
| `loadFromSupabase()` | Two-way sync on login |
| `syncSessionToSupabase(session)` | Upsert one session |
| `syncExerciseToSupabase(ex)` | Upsert one custom exercise |
| `sendMagicLink()` | Supabase OTP sign-in |
| `signOut()` | Clears Supabase session |
| `exportData()` | Downloads JSON backup of all sessions + exercises |
| `importData()` | Restores from JSON backup, bulk syncs to Supabase |

## Exercise Library

`SEED_EXERCISES` is a hardcoded array of ~50 exercises covering: Chest, Back, Shoulders, Biceps, Triceps, Legs, Glutes, Core, Cardio. Each has a stable `key` (snake_case, never changes — used as foreign key in session blocks).

On app mount, any seed exercises missing from localStorage are added automatically (forward-compatibility for future seed additions).

User-created exercises get a key like `exercise_name_<timestamp36>` and are stored in both localStorage and Supabase `custom_exercises`.

## Styling

iOS-native dark theme. CSS variables in `:root`:
- `--bg`: #000 (background)
- `--card`: #1c1c1e, `--card2`: #2c2c2e, `--card3`: #3a3a3c
- `--orange`: #ff9500 (primary accent)
- `--blue`: #0a84ff, `--green`: #30d158, `--red`: #ff453a

No external CSS framework. All styles are inline in the `<style>` block in `<head>`.

## Git Workflow

- Develop on `claude/workout-tracking-app-u2CAV`
- Merge to `main` to deploy (Vercel auto-deploys on push to main)
- Tag stable checkpoints: `git tag -a vYYYY-MM-DD -m "description"`

## Checkpoints

| Date | Commit | Description |
|---|---|---|
| 2026-05-21 | `1e5966c` | Supabase auth + sync + keep-alive + CLAUDE.md. Full working state. |

To roll back to a checkpoint: `git checkout 1e5966c` (detached HEAD to inspect) or `git revert` to undo commits while keeping history.

## What NOT to Do

- **Don't add a build step** — no npm, no bundler, no compilation. CDN only.
- **Don't split into multiple files** — single `index.html` is intentional.
- **Don't use the service_role key** — anon key only in the frontend.
- **Don't add features speculatively** — this app is built to be left alone. Only add what's explicitly requested.
- **Don't normalise the blocks** — storing blocks as JSONB inside sessions is intentional. Full normalisation (sets as rows) would be over-engineering for a personal app.
- **Don't break offline-first** — Supabase sync must always be fire-and-forget. Never `await` a Supabase call in a way that blocks the UI.
