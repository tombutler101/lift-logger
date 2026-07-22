# Lift Logger

A tap-to-log gym app for a 3-day full-body rotation (Sessions A / B / C).
Runs as a phone home-screen web app. No accounts, works offline, saves on device.

---

## One-time setup

You do this once, from a laptop or desktop. About 20–30 minutes the first time.

### 1. Install Claude Code (first-timers)

The native installer needs no Node.js. You need a paid Claude plan (Pro or Max).

**Mac / Linux** — open Terminal and run:
```
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows** — install through WSL (a Linux environment inside Windows), then run
the same command. If you've never set up WSL, ask Claude Code's install docs or
follow: https://code.claude.com/docs/en/setup

Then start it and sign in when the browser opens:
```
claude
```

Verify it worked:
```
claude --version
```

### 2. Put these files somewhere and open Claude Code there

Drop this whole folder on your machine, then in a terminal:
```
cd path/to/lift-logger
claude
```
Claude Code will read `CLAUDE.md` and understand the project.

### 3. Create a GitHub repo and push (let Claude Code do it)

If you don't have Git or a GitHub account yet, tell Claude Code and it will walk
you through it. Otherwise, just ask it:

> "Create a new public GitHub repo called lift-logger, commit these files, and push."

### 4. Turn on GitHub Pages

On github.com, in the repo: **Settings → Pages → Source: Deploy from a branch →
main / (root) → Save.** After a minute you get a URL like:
```
https://YOURNAME.github.io/lift-logger/
```

### 5. Add it to your phone home screen

Open that URL on your phone.
- **iPhone (Safari):** Share → Add to Home Screen.
- **Android (Chrome):** menu → Add to Home screen / Install app.

It now opens full-screen like an app, works offline, and remembers your sets.

---

## Turn on GitHub sync (one-time)

The app can commit each session straight into this repo. It needs a token.

### Create a fine-grained token
On github.com: **Settings → Developer settings → Personal access tokens →
Fine-grained tokens → Generate new token.**
- **Repository access:** Only select repositories → `lift-logger`.
- **Permissions → Repository → Contents: Read and write.** (Nothing else.)
- **Expiration:** your call — 90 days is sensible; you'll re-paste when it lapses.

Copy the token (starts `github_pat_...`). You only see it once.

### Enter it in the app
Open the logger, tap the **⚙ gear**, fill in owner (your GitHub username),
repo (`lift-logger`), branch (`main`), and paste the token. Save.

That token is stored in your phone's browser only — it is never committed to the
repo. It's scoped to this one repo, so a leak can't touch anything else. Don't
paste it into a shared or synced note.

## Daily use

1. Open the app, pick the session (A / B / C) and set the week number.
2. Log each set as you finish it — adjust weight/reps, tap RPE, it ticks done.
3. Tap **Push log**. It commits `logs/YYYY-MM-DD_sessionX_wkN.txt` to the repo.
   (**Copy** still works as an offline fallback — paste into Claude directly.)

Then in Claude Code, or by pasting the file, ask Claude to update working
weights and build the next session.

**Log RPE per set, at the time.** Recalled RPE is fiction, and it's the whole
input for how the next week is loaded.

---

## Changing the programme

The plan lives in the `PROGRAMME` object near the top of the `<script>` in
`index.html`. Ask Claude Code to edit it — e.g. "bump Back Squat to 60 kg in
Session A" or "swap RDL for Leg Extension in Session B." After any edit to
`index.html`, bump the `CACHE` value in `sw.js` so the change reaches your phone,
then commit and push. Pages redeploys on its own.

---

## Notes

- Once you push, the session lives in `logs/` in the repo — that's your durable
  copy, synced everywhere. Local browser storage is just the working draft.
- Push needs signal. Mid-set in a dead-zone gym, log offline and push when you're
  back on wifi; the data's held safely on the phone until then.
- If a push fails, the status line under the buttons tells you why (bad token,
  404 scope, offline). **Copy** is always there as a fallback.
- Fonts load from Google once, then cache. Offline before first load = system
  fonts (still fully usable).
