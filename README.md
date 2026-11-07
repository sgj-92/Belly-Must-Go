# Belly Must Go

Weigh-ins, steps and standings for the padel group. Everyone logs their own weight, steps and
training; the app keeps a quiet, well-made record of who is doing what.

**The look:** quiet luxury, like a members' club or a fine watch brand. Ivory paper and ink, one
racing-green accent and a champagne-gold hairline; a single typeface (Jost) throughout, with
hierarchy from size and weight; engraved initials instead of avatars. It follows the phone's light / dark setting.

Add `?demo` to the URL (e.g. `https://bmg.sgj.luxe/?demo`) to preview with sample players
without touching the live data.

### Today

- the date, a greeting and a line from Chanel, Franklin, the Stoics and others
- the group at a glance: weight lost together, steps today, how many are at goal
- **Your season**: % of starting weight lost, position, a milestone line
  (2.5% · 5% · 7.5% · 10% · 12.5% · 15% · 20%), score, this week, steps today, and the gap to the
  person above you on body weight lost
- **The table**: the top five, plus you if you're further down
- **Week in review**: best week and most gained for the last full week
- today's steps, anyone **awaiting a weigh-in** (7+ days), and the **journal** of recent entries

### Standings

| Board | What it ranks |
|---|---|
| **Standings** | Score, with ▲▼ movement versus 7 days ago |
| **Body %** | % of your own start weight lost, so different sizes compete fairly |
| **Weight lost** | Raw kg / lb |
| **This week** | This week's change, plus each past week's best and most gained |
| **Steps** | Steps today, this week, or daily average against the shared goal |
| **Training** | Sessions logged this week and this season (gym, run, padel… typed freely) |
| **Streaks** | Days in a row hitting the step goal, and weeks in a row on the scales |
| **Targets** | Progress towards each person's own target weight |

### Score

| | |
|---|---|
| Every 1% of body weight lost (or gained back) | ±10 |
| Each day the step goal is met | +1 |
| Each week (Mon–Sun) with at least one weigh-in | +3 |
| Best week (largest % drop) | +5 |

To change the values, edit `POINTS` near the top of the script in `index.html`. Milestones are
in `MILESTONES` and the daily lines in `QUOTES`.

### Privacy

Each player picks whether to share their exact weight. When it's off, others see
only changes (kg and %). Anyone determined could still work out the number from
those two, and the app says so when you set it.
An optional 4-digit PIN stops people logging as each other by accident. It's an
honour-system lock, not real security.

---

## Running it

It's one static `index.html` with no build step.

**Demo mode** works straight away. With no Firebase config (or with `?demo`) the app loads sample players
and saves to the browser only. Open `index.html` via any static server
(`python3 -m http.server`) or the hosted URL.

**Going live (shared data for the whole squad)** takes about 10 minutes:

1. Create a project at <https://console.firebase.google.com> (Analytics not needed).
2. **Build → Firestore Database → Create database** (production mode, `europe-west2` London).
3. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable.**
4. **Firestore → Rules:** paste the contents of `firestore.rules` and publish. The project is shared with
   the North vs South Cup, so that file covers both apps: always publish the whole file.
5. **Project settings → Your apps → Web (`</>`)**: register an app and copy the `firebaseConfig` object.
6. Paste it into `FIREBASE_CONFIG` at the top of the script in `index.html`, then commit.
7. Host it on **Vercel** (same as the other apps): import the repo with no framework preset and no build;
   `vercel.json` serves the root folder. Production is `https://bmg.sgj.luxe`.
8. Share the link in the group chat. On iPhone: Share → **Add to Home Screen**.

Once live, the app creates the group with a 10,000-step goal and a season starting today.
Anyone can change either under **Profile → Season & step goal**.

### Admin tools

Open `https://bmg.sgj.luxe/?admin` once on your phone to unlock **Profile → Manage the group** on that
device (`?admin=off`, or the row at the bottom of that section, hides it again). From there:

- **Season start & step goal**
- **Add a player** for someone who hasn't joined, with an optional back-dated first weigh-in
- **Update step totals**: enter a player's total steps since the season start, up to a date (also
  available to each player from **Log steps → Enter a running total instead**). Whatever their
  logged days don't account for is spread evenly over the unlogged days in that period, so weekly
  totals, averages and today's race keep working. Each full goal's worth of spread steps counts as
  one goal day. Totals must only go up over time; spread days show faded on the steps chart.
- **Catch up entries**: a day-by-day grid of steps and weight for any player, prefilled with what's
  already logged. Blank boxes are skipped, clearing a box deletes that entry, and nothing saves if
  any box looks wrong. Weight uses the device's units (`st lb` as two numbers, e.g. `14 2.5`).
- Tap any player to **edit their details**, including removing a forgotten PIN

It's an honour-system unlock, like the PINs: anyone who knows the link could use it.

### Seasons

Only entries on or after the season start date count, and each person's baseline is
their first weigh-in on or after it. To start a new season, move the start date
forward. Old entries are kept.

## Data model (Firestore)

```
groups/belly-must-go               { name, stepGoal, startDate }
  players/{id}                     { name, emoji, targetKg, showWeight, pinHash, created, ts }
  weighins/{playerId}_{YYYY-MM-DD} { pid, date, kg, ts }      // one per player per day
  sessions/{playerId}_{date}_{rnd} { pid, date, text, ts }    // training, any number per day
  steps/{playerId}_{YYYY-MM-DD}    { pid, date, steps, ts }   // latest number wins
  stepTotals/{playerId}_{date}     { pid, date, total, ts }   // running total since season start
```

Weights are always stored in kg; kg / st & lb / lb is a per-device display setting.
