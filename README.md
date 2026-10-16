# Belly Must Go ⚔️

The arena for the padel legion. Warriors log their own weigh-ins and daily marches
(steps), and the app ranks everyone from every angle.

**The look:** Physical: 100's plaster torsos on marble plinths, the bronze and crimson
of *300* and *Troy*, *Gladiator*'s arena and gold, and Rocky's grit. The palette is stone,
bronze, crimson and plaster. Titles are set in Roman capitals (Cinzel) and numbers in
heavy condensed type (Anton).

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

The leader's torso is cast in **gold**. Anyone who gained weight this battle gets a
**cracked** statue, a nod to Physical: 100 smashing the losers' casts.

### Ranks (leaderboards)

| Board | What it ranks |
|---|---|
| 🏛️ **Glory** | Overall points, with ▲▼ movement versus 7 days ago |
| ⚖️ **Body %** | % of your own start weight lost, so different sizes fight fairly |
| 🪓 **Weight shed** | Raw kg / lb (the heavyweights' board) |
| ⚔️ **This battle** | This week's change, plus the Hall of Champions for past battles |
| 🥾 **The march** | Steps today, this week, or daily average against the shared goal |
| 🥊 **Montage** | Days in a row hitting the step goal, and weeks in a row on the scales (*Shield wall*) |
| 🎯 **Conquest** | Progress towards each warrior's own target weight |

The Arena (home) screen shows:
- a daily line from Gladiator, 300, Troy or Rocky
- how much the legion has lost in total
- your torso, rank and the % left to carve before your next rank
- the gap to the warrior above you
- the Pantheon
- last battle's **Champion 🌿** and **Thumbs down 👎**
- today's march
- **Deserters 👀**: 7+ days without a weigh-in
- the battle chronicle

### Glory

| | |
|---|---|
| Every 1% of body weight carved off (or gained back) | ±10 |
| Each day the march goal is hit | +1 |
| Each battle (Mon–Sun week) with at least one weigh-in | +3 |
| Champion of the battle (largest % drop) | +5 |

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

**Training grounds (demo mode)** works straight away. With no Firebase config the app loads a sample legion
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
Anyone can change either under **Warrior → Campaign & march goal**.

### Campaigns

Only entries on or after the campaign start date count, and each person's baseline is
their first weigh-in on or after it. To start a new campaign, move the start date
forward. Old entries are kept.

## Data model (Firestore)

```
groups/belly-must-go               { name, stepGoal, startDate }
  players/{id}                     { name, emoji, targetKg, showWeight, pinHash, created, ts }
  weighins/{playerId}_{YYYY-MM-DD} { pid, date, kg, ts }      // one per player per day
  steps/{playerId}_{YYYY-MM-DD}    { pid, date, steps, ts }   // latest number wins
```

Weights are always stored in kg; kg / st & lb / lb is a per-device display setting.
