# Pradakshina Lap Counter

A phone-friendly web app that counts your rounds automatically. You set a start
point, start the timer, put the phone in your pocket, and every time you walk
away and come back to that same point it counts one more round — out loud, with a
beep and a buzz, so you never have to hold the number in your head.

Built for walking pradakshina at Karya Siddhi Hanuman Temple, Frisco, TX, but the
start point is wherever you stand when you press the button, so it works anywhere.

## What it does

- **Automatic counting** — GPS geofence around your start point. Leave the circle,
  go at least a minimum distance away, come back in → +1 round.
- **Speaks the count** ("one", "two", "three"…), beeps twice and vibrates, so you
  get confirmation without taking the phone out.
- **Anti-miscount rules** — a round only counts if you actually went far enough
  away *and* enough time passed, and fixes with bad GPS accuracy are thrown out.
  Standing at the start point while GPS wobbles will not add rounds.
- **Timer** — total time, last round, average round. That's your production-hours
  number at the end.
- **Round log** — time of day and duration for each round; Copy gives you CSV.
- **Manual +1 / −1** — for the round you did before you hit Start, or a miss.
- **Survives a reload** — count, timer, log and start point are saved on the phone.
- **Works offline** — installable (Add to Home Screen) and cached, so patchy
  signal at the temple doesn't matter. GPS itself needs no data connection.

## Deploy it (pick one — both are free)

### Option A — GitHub Pages (recommended, ~3 minutes)

1. Get these files onto the branch you want to publish (this repo's default
   branch is easiest).
2. On GitHub: **Settings → Pages**.
3. Under *Build and deployment*, Source = **Deploy from a branch**;
   Branch = your branch, folder = **/ (root)**. Save.
4. Wait ~1 minute. Your URL appears at the top of that page, in the shape
   `https://<your-username>.github.io/<repo-name>/`.
5. Open that URL on your phone. HTTPS is automatic, which is what GPS requires.

### Option B — Netlify Drop (no account needed to try)

1. Download this folder to your computer.
2. Go to <https://app.netlify.com/drop> and drag the folder onto the page.
3. You get an HTTPS URL instantly. Open it on your phone.

Any static host works the same way (Vercel, Cloudflare Pages, Firebase Hosting) —
there is no server, no build step, and no database. It's plain HTML in one file.

### Install it on your phone (do this once)

- **iPhone (Safari):** open the URL → Share → **Add to Home Screen**.
- **Android (Chrome):** open the URL → ⋮ menu → **Add to Home screen / Install app**.

It then opens full-screen like a normal app and works with no signal.

## How to use it at the temple

1. Stand at the exact spot you treat as the start/finish.
2. Tap **Set start point** and stand still ~5 seconds while it averages the GPS
   fixes (it beeps and buzzes when locked). Allow location when asked — choose
   *While using the app* / *Precise*.
3. Tap **Start**. Phone goes in your pocket, screen **on** (see the note below).
4. Walk. Each time you complete a round you'll hear the new number.
5. When you're done, tap **Pause**. Read the count and the time; tap **Log → Copy**
   if you want the record.

Next time you come back, the start point is still saved — tap **Start** and tap
**Reset** to zero the count for the new day.

## Important: keep the screen on

Phone browsers stop giving GPS updates when the screen is off or the browser is in
the background. So the app asks your phone to **keep the screen awake** while
counting (the "Keep screen awake" switch, on by default). Put the phone in your
pocket with the screen on and it keeps counting.

If your phone still dims and sleeps:

- iPhone: Settings → Display & Brightness → Auto-Lock → **Never** while you walk.
- Android: Settings → Display → Screen timeout → longest setting.
- Turn the brightness right down to save battery. An hour of this uses roughly
  15–25% of a typical battery.

Don't switch to another app or lock the phone mid-round — that's the one thing
that makes it miss a count. (The manual **+1** button is there for that case.)

## Tuning (Settings)

| Setting | Default | What to change it to |
|---|---|---|
| Start-zone radius | 25 m | Bigger (30–40 m) if rounds get missed; smaller (15 m) if your GPS reads ±5 m and the path is tight. Keep it above your typical GPS accuracy shown in the top-right pill. |
| Must go at least this far away | 35 m | Set it to about half the width of your circuit. If your loop is small, lower it; if you get phantom counts while standing still, raise it. |
| Minimum time per round | 25 s | Slightly less than your fastest realistic round. |

Rule of thumb: **radius ≈ your GPS accuracy + 10 m**, and **away distance ≈ 40–60%
of the loop's smallest width**. Watch the top-right pill — green (±≤15 m) is good,
red means the sky view is poor (indoors, under a roof, next to a tall wall).

## Test it without walking

Open the URL with `?demo=1` on the end — for example
`https://you.github.io/repo/?demo=1` — and it simulates walking a 60 m circle so
you can watch the counter, hear the voice, and check your settings from your couch.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire app — UI, GPS logic, lap engine. No dependencies. |
| `manifest.webmanifest` | Makes it installable as a home-screen app. |
| `sw.js` | Service worker; caches the app for offline use. |
| `icon-*.png` | Home-screen icons. |

## How the counting logic works

The app converts each GPS fix into a straight-line distance from your saved start
point, then runs a small state machine:

1. **inside** → you're within the radius.
2. You cross past `radius + 8 m` (the extra 8 m is hysteresis, so hovering on the
   edge can't flicker the count) → state becomes **outside**, and it starts
   tracking the furthest distance you reach.
3. Back within the radius → it counts a round **only if** the furthest distance
   reached ≥ the "must go this far away" setting **and** the time since the last
   round ≥ the minimum. Otherwise it silently resets to inside and says why.

Fixes with accuracy worse than ±45 m are ignored entirely, since a single bad fix
can otherwise look like a whole lap.
