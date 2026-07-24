# Lift Logger — project context for Claude Code

This repo is a personal gym logging app for Tom, plus (optionally) his
training data files. It is deployed as a static site via GitHub Pages and
used as a phone home-screen web app in the gym.

## Files
- `index.html` — the whole app. Single file: markup, styles, logic. No build step.
- `manifest.json` — PWA manifest (name, colours, icons). Makes it installable.
- `sw.js` — minimal service worker for offline. Bump the `CACHE` constant
  whenever `index.html` changes, or phones will serve a stale cached copy.
- `README.md` — setup and deploy steps.

## The programme lives in one place
Inside `index.html` there is a `PROGRAMME` object holding Sessions A, B and C
(exercises, sets, reps, starting weights, coaching cues). When the training
plan changes, edit that object — nothing else. Keep the shape identical:
each exercise is `{ n, sets, reps, w, step, bw?, secs?, mins?, cue?, flag? }`.

- `w` is the starting working weight shown by default (user adjusts per set).
- `step` is the +/- increment for that lift (2.5 barbell upper, 5 lower, 2 for DB/machine stacks).
- `bw:true` shows "BW" instead of 0 kg.
- `secs`/`mins:true` change the reps unit label (plank in seconds, finisher in minutes).
- `flag:true` renders the cue in warning colour (use for shoulder/back cautions).

## After editing index.html
Bump `CACHE` in `sw.js` (e.g. `liftlog-v1` → `liftlog-v2`) so the update
reaches the phone. Then commit and push; GitHub Pages redeploys automatically.

## Permanent exercise swaps
- Session B: Bulgarian Split Squat → **Goblet Squat** (24/07). Bulgarians
  triggered lower back twinges consistent with the L4/L5 history; Goblet
  Squat is bilateral/upright with no unilateral balance demand.
- Session C: Seated Leg Curl → **Lying Leg Curl** (24/07), user preference.
  The 28 kg baseline is new and not comparable to any prior seated number.

## Week numbering
"Week" is the programme week (block progression), not the calendar week.
It only advances once Sessions A, B and C have all been banked — training
actually done, not time elapsed — so `index.html` auto-suggests it instead
of defaulting to a stale manual counter:
- A session counts as complete once it's been **pushed to GitHub** (not
  when its sets are ticked — ticking can happen without a push, and the
  push is the real "banked" signal). Tracked in `localStorage` under
  `liftlog:pushed` as a set of `YYYY-MM-DD_<session letter>` keys, so
  re-pushing a corrected log for the same day/session doesn't double-count.
- Suggested week = `Math.ceil((completedSessions + 1) / 3)`, minimum 1 —
  the `+1` is what makes the count roll over onto the next week the
  instant the 3rd session of a week is pushed, rather than staying on the
  old week until a 4th session is logged.
- `seedPushedHistory()` backfills Week 1's three real sessions, which
  `liftlog:pushed` can't see on its own: Session A (20/07, done before the
  app existed — no log at all), Session B (22/07) and Session C (24/07,
  both pushed before this tracking existed). It runs on every load via
  `markPushed` (add-to-a-Set, so already-present entries are a no-op) —
  intentionally not a guarded one-shot, so it self-heals a device that
  already ran an earlier, incomplete version of this seed.
- This suggestion pre-fills the stepper on load but stays manually
  editable via the existing +/− buttons, since a missed/skipped session
  can make the auto-count drift from reality.

## Athlete Profile
- Name: Tom
- Training experience:  intermediate 
- Goal: Strength / Fat loss / Improve pull ups
- Schedule: 3 days a week, sessions not more that 50-60 mins
- Equipment access: Commercial gym
- Injuries or limitations: Bad left shoulder, minor Lower back issues
- Height/Weight: 172cm/87kg

## Standing athlete constraints (do not program around silently — flag them)
- Left shoulder: impingement + occasional instability on rotation/cross-body.
  Physio appointment booked. Pronated/neutral grip on vertical pulls, no
  kipping, no weighted pull-ups until 8 clean reps. OHP overhead only if
  pain-free, else neutral-grip DB.
- Herniated L4/L5 (2023, recovered). Deadlift volume stays low, hinge loading
  conservative.
- Sessions 50–60 min. Prefers simplicity, likes heart-rate work.
- Goal: strength, fat loss, and getting pull-up numbers up (currently ~2–3).
  
## Sync (phase 2 — enabled)
The app commits each session to this repo via the GitHub Contents API.
- Settings (gear icon) hold owner / repo / branch / token in `localStorage`
  under key `liftlog:gh`. The token is a **fine-grained PAT scoped to this repo
  only, Contents: read & write.** It never enters the repo — do not commit it,
  do not print it, do not add it to any file here.
- "Push log" writes `logs/YYYY-MM-DD_session{A|B|C}_wk{N}.txt`. One file per
  session per day. If the same file exists it's updated (fetches the blob sha
  first), so re-pushing a corrected session overwrites cleanly.
- Set state still lives in `localStorage` too; the push is a manual button, not
  automatic on every tap (avoids hammering the API).

To read a training week back, concatenate the relevant files in `logs/`.
The "Copy log" button remains as an offline fallback.

## Security note
Client-held token = acceptable only because scope is one low-value repo. If this
ever needs hardening, move the token behind a Cloudflare Worker / serverless
proxy (phase 2 option 2) and have the app call that instead of GitHub directly.
