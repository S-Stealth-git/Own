# Pradakshina Lap Counter

A phone-friendly web app that counts your rounds automatically. You set a start
point, start the timer, put the phone in your pocket, and every time you walk
away and come back to that same point it counts one more round — out loud, with a
beep and a buzz, so you never have to hold the number in your head.

Built for walking pradakshina at Karya Siddhi Hanuman Temple, Frisco, TX, but the
start point is wherever you stand when you press the button, so it works anywhere.

## What it does

- **Automatic counting** — it counts a **full turn around your start point**, not a
  return to an exact spot. Swing as wide as you like around someone standing
  there; the circle still completes and the round still counts.
- **Speaks the count** ("one", "two", "three"…), beeps twice and vibrates, so you
  get confirmation without taking the phone out.
- **Anti-miscount rules** — GPS noise close to the start point is ignored, wild
  jumps are discarded as bad fixes, low-accuracy readings are thrown out, and a
  minimum time per round applies. Standing still will not add rounds.
- **Timer** — total time, last round, average round. That's your production-hours
  number at the end.
- **Round log** — time of day and duration for each round; Copy gives you CSV.
- **Manual +1 / −1** — for the round you did before you hit Start, or a miss.
- **Survives a reload** — count, timer, log and start point are saved on the phone.
- **Works offline** — installable (Add to Home Screen) and cached, so patchy
  signal at the temple doesn't matter. GPS itself needs no data connection.

## The two counting modes

### Full circle (default — use this)

A round is counted when you have turned a full **350°** around the saved start
point. The app tracks the *direction* from your start point to you, and adds up
how much that direction has swept. One lap of the temple = one full sweep,
whatever shape you walk.

This is deliberately independent of any radius, which is what makes it robust:

- Somebody standing on your exact spot? Walk 2 m or 20 m around them — still 360°.
- A wide day and a tight day count identically.
- GPS being off by 10 m barely moves the angle when you're 30 m out.

Two knobs:

| Setting | Default | What it does |
|---|---|---|
| Count a round after turning | 350° | The 10° of slack means you don't have to walk the last couple of steps back onto the exact spot. |
| Ignore wobble within | 6 m | Standing right on top of the start point, the *direction* from it to you flips randomly with GPS noise. Inside this distance the angle is frozen rather than counted. |

### Return to zone (the simple alternative)

The classic geofence: leave a circle around the start point, go a minimum
distance away, step back into the circle → +1. Easy to picture, but it depends on
GPS actually placing you back inside a small circle, so a detour around a person
can miss a round. Radius goes down to 2 m if you want it, but the app will warn
you when you set it finer than your phone's live GPS accuracy.

**Why not a 0.5 m or 2 m radius?** Phone GPS is accurate to about ±5–15 m
outdoors, and worse beside a building. A 2 m circle is smaller than the error, so
your phone often does not know you're inside it and rounds get skipped. The
full-circle mode exists precisely so you get exact counting without depending on
a precision the hardware doesn't have.

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
| Count a round after turning | 350° | Drop to 330–340° if rounds register later than they feel. |
| Ignore wobble within | 6 m | Raise it if your path passes very close to the start point and the count stalls there; lower it only if your loop is tiny. |
| Minimum time per round | 25 s | Slightly less than your fastest realistic round. |
| Start-zone radius *(zone mode)* | 12 m | Keep it at or above the accuracy shown in the top-right pill. |
| Must go at least this far away *(zone mode)* | 35 m | About half the narrow width of your circuit. |

Watch the top-right pill — green (±≤15 m) is good, red means the sky view is poor
(under a roof, tight against a wall).

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

Fixes with accuracy worse than ±45 m are discarded in both modes — a single bad
fix can otherwise look like a whole lap.

### Full circle mode (winding angle)

1. For each GPS fix, compute the compass bearing from the start point to you.
2. A new bearing is only trusted after you have genuinely moved 3 m from where the
   last one was taken — that filters standing-still jitter.
3. Add the signed change in bearing to a running total. Because the changes are
   signed, random wobble cancels out; only actually going *around* accumulates.
4. A change larger than 100° between two readings is thrown away as a bad fix
   rather than a walk.
5. Within the "ignore wobble" distance of the start point, the angle is frozen —
   the bearing is meaningless up close.
6. Total ≥ 350° (and past the minimum round time) → count a round, then subtract a
   full 360° so the next round is measured from the same registration and the
   count can't drift over an hour.

### Zone mode (geofence)

1. **inside** → you're within the radius.
2. Cross past `radius + 8 m` (hysteresis, so hovering on the edge can't flicker) →
   **outside**, and it tracks the furthest distance you reach.
3. Back within the radius → counts a round only if the furthest distance ≥ the
   "must go this far away" setting and the time since the last round ≥ the
   minimum. Otherwise it resets quietly and tells you why.
