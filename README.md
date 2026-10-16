# Belly Must Go 🎾

A weight-loss and step-count leaderboard for the padel crew. Everyone logs their own
weigh-ins and daily steps, and the app ranks the squad across every angle:

| Board | What it ranks |
|---|---|
| 🏆 **Overall** | Belly Points (see below), with ▲▼ movement versus 7 days ago |
| ⚖️ **Body %** | % of your own start weight lost, so different sizes compete fairly |
| 📉 **Weight lost** | Raw kg / lb lost (the big lads' board) |
| 📅 **This week** | Change since last week's final weigh-in, plus a weekly roll of honour |
| 🥾 **Steps** | Today, this week, or daily average against the shared step goal |
| 🔥 **Streaks** | Consecutive step-goal days and consecutive weeks weighed in |
| 🎯 **Targets** | Progress towards each person's own target weight |

The home screen has the squad total, your rank and the gap to the person above you,
**Biggest Loser** and **Hall of Shame 🍩** awards for last week, today's step race,
a **Dodging the scales 👀** list (7+ days without a weigh-in) and a banter feed.

### Belly Points

| | |
|---|---|
| Every 1% of body weight lost (or gained back) | ±10 |
| Each day the step goal is hit | +1 |
| Each week with at least one weigh-in | +3 |
| Biggest Loser of the week (largest % drop, Mon–Sun) | +5 |

Change the values in `POINTS` near the top of the script in `index.html`.

### Privacy

Each player picks whether to share their exact weight. When it's off, the squad sees
only changes (kg and %). Anyone determined could still work out the number from
those two, and the app says so when you set it.
An optional 4-digit PIN stops people logging as each other by accident. It's an
honour-system lock, not real security.

---

## Running it

It's one static `index.html` with no build step.

**Demo mode** works straight away. With no Firebase config the app loads a sample crew
and saves to the browser only. Open `index.html` via any static server
(`python3 -m http.server`) or the hosted URL.

**Going live (shared data for the whole squad)** takes about 10 minutes:

1. Create a project at <https://console.firebase.google.com> (Analytics not needed).
2. **Build → Firestore Database → Create database** (production mode, `europe-west2` London).
3. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable.**
4. **Firestore → Rules:** paste the contents of `firestore.rules` and publish.
5. **Project settings → Your apps → Web (`</>`)**: register an app and copy the `firebaseConfig` object.
6. Paste it into `FIREBASE_CONFIG` at the top of the script in `index.html`, then commit.
7. Host it. **GitHub Pages:** repo **Settings → Pages → Deploy from branch → `main` / root**.
   Vercel also works, the same way as the other apps.
8. Share the link in the group chat. On iPhone: Share → **Add to Home Screen**.

Once live, the app creates the group with a 10,000-step goal and a season starting today.
Anyone can change either under **Me → Season & step goal**.

### Seasons

Only entries on or after the season start date count, and each person's baseline is
their first weigh-in on or after it. To run a fresh challenge, move the start date
forward. Old entries are kept.

## Data model (Firestore)

```
groups/belly-must-go               { name, stepGoal, startDate }
  players/{id}                     { name, emoji, targetKg, showWeight, pinHash, created, ts }
  weighins/{playerId}_{YYYY-MM-DD} { pid, date, kg, ts }      // one per player per day
  steps/{playerId}_{YYYY-MM-DD}    { pid, date, steps, ts }   // latest number wins
```

Weights are always stored in kg; kg / st & lb / lb is a per-device display setting.
