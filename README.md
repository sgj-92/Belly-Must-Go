# Belly Must Go

The arena for the padel group. Everyone logs their own weigh-ins and daily steps, and the app ranks everyone from every angle.

**The look:** quiet and grown-up. Near-black stone, ivory and one muted bronze; Cormorant Garamond
for headings and figures, Inter for everything else; engraved initials instead of avatars. The
drama comes from the Physical: 100-style plaster torsos and a daily line from Gladiator, 300,
Rocky or the Stoics, not from decoration.

Add `?demo` to the URL (e.g. `https://bmg.sgj.luxe/?demo`) to preview with sample players
without touching the live data.

### The Pantheon and warrior ranks

Every warrior has a plaster torso on a plinth. It **carves itself** as they lose weight:

| Rank | % body weight lost | Torso |
|---|---|---|
| Recruit | start | soft belly |
| Hoplite | 1%+ | chest appears |
| Gladiator | 3%+ | first abs |
| Spartan | 5%+ | four-pack |
| Champion | 8%+ | six-pack, obliques |
| Immortal | 12%+ | eight-pack, V-lines |

The leader's torso is cast in **gold**. Anyone who gained weight this week gets a
**cracked** statue, a nod to Physical: 100 smashing the losers' casts.

### Ranks (leaderboards)

| Board | What it ranks |
|---|---|
| **Standings** | Glory points, with ▲▼ movement versus 7 days ago |
| **Body %** | % of your own start weight lost, so different sizes fight fairly |
| **Weight lost** | Raw kg / lb |
| **This week** | This week's change, plus Honours for past weeks |
| **Steps** | Steps today, this week, or daily average against the shared goal |
| **Training** | Sessions logged this week and this campaign (gym, run, padel… typed freely) |
| **Streaks** | Days in a row hitting the step goal, and weeks in a row on the scales |
| **Targets** | Progress towards each person's own target weight |

The Arena (home) screen shows:
- a daily line from Gladiator, 300, Rocky or the Stoics
- how much the legion has lost in total
- your torso, rank and the % left to carve before your next rank
- the gap to the warrior above you
- the Pantheon
- last week's **Champion** and who **lost ground**
- today's steps
- **Absent from the scales**: 7+ days without a weigh-in
- the chronicle

### Glory

| | |
|---|---|
| Every 1% of body weight lost (or gained back) | ±10 |
| Each day the step goal is hit | +1 |
| Each week (Mon–Sun) with at least one weigh-in | +3 |
| Champion of the week (largest % drop) | +5 |

To change the values, edit `POINTS` near the top of the script in `index.html`. The rank
thresholds are in `TIERS` and the daily lines in `QUOTES`.

### Privacy

Each player picks whether to share their exact weight. When it's off, the squad sees
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
4. **Firestore → Rules:** paste the contents of `firestore.rules` and publish.
5. **Project settings → Your apps → Web (`</>`)**: register an app and copy the `firebaseConfig` object.
6. Paste it into `FIREBASE_CONFIG` at the top of the script in `index.html`, then commit.
7. Host it on **Vercel** (same as the other apps): import the repo with no framework preset and no build;
   `vercel.json` serves the root folder. Production is `https://bmg.sgj.luxe`.
8. Share the link in the group chat. On iPhone: Share → **Add to Home Screen**.

Once live, the app creates the group with a 10,000-step goal and a campaign starting today.
Anyone can change either under **Profile → Campaign & step goal**.

### Admin tools

Open `https://bmg.sgj.luxe/?admin` once on your phone to unlock **Profile → Manage the group** on that
device (`?admin=off`, or the row at the bottom of that section, hides it again). From there:

- **Campaign start & step goal**
- **Add a player** for someone who hasn't joined, with an optional back-dated first weigh-in
- **Update step totals**: enter a player's total steps since the campaign start, up to a date (also
  available to each player from **Log steps → Enter a running total instead**). Whatever their
  logged days don't account for is spread evenly over the unlogged days in that period, so weekly
  totals, averages and today's race keep working. Each full goal's worth of spread steps counts as
  one goal day. Totals must only go up over time; spread days show faded on the steps chart.
- **Catch up entries**: a day-by-day grid of steps and weight for any player, prefilled with what's
  already logged. Blank boxes are skipped, clearing a box deletes that entry, and nothing saves if
  any box looks wrong. Weight uses the device's units (`st lb` as two numbers, e.g. `14 2.5`).
- Tap any player to **edit their details**, including removing a forgotten PIN

It's an honour-system unlock, like the PINs: anyone who knows the link could use it.

### Campaigns

Only entries on or after the campaign start date count, and each person's baseline is
their first weigh-in on or after it. To start a new campaign, move the start date
forward. Old entries are kept.

## Data model (Firestore)

```
groups/belly-must-go               { name, stepGoal, startDate }
  players/{id}                     { name, emoji, targetKg, showWeight, pinHash, created, ts }
  weighins/{playerId}_{YYYY-MM-DD} { pid, date, kg, ts }      // one per player per day
  sessions/{playerId}_{date}_{rnd} { pid, date, text, ts }    // training, any number per day
  steps/{playerId}_{YYYY-MM-DD}    { pid, date, steps, ts }   // latest number wins
  stepTotals/{playerId}_{date}     { pid, date, total, ts }   // running total since campaign start
```

Weights are always stored in kg; kg / st & lb / lb is a per-device display setting.
