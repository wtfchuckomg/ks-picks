# Friday Night Picks — setup

Three files:

- `index.html` — the whole app (no build step, no dependencies to install)
- `firestore.rules` — the security rules that enforce the deadline lock and keep picks private
- `SETUP.md` — this file

Total setup time: about 15 minutes, all in a browser.

---

## 1. Firebase project

In the [Firebase console](https://console.firebase.google.com):

1. **Create a project** (or open the one you already have).
2. **Build → Firestore Database → Create database.** Pick a location near you.
   Start in **production mode** — the rules in step 3 replace whatever it starts with.
3. **Build → Authentication → Get started**, then enable two sign-in providers:
   - **Anonymous** — this is what players use. It costs them nothing and asks
     them for nothing; it just gives each browser a stable id so the server can
     tell one player's card from another's.
   - **Google** — this is only for you. It's how the server knows you're the
     commissioner.
4. **Project settings (gear icon) → Your apps → Web (`</>`)**. Register an app,
   and copy the `firebaseConfig` object it shows you.

## 2. Paste your config

Open `index.html` and replace the placeholder block near the top:

```js
const firebaseConfig = {
  apiKey:            "PASTE_API_KEY",
  ...
};
```

Just below it, check `ADMIN_EMAILS`. It's already set to `chuck8303@gmail.com` —
add any co-commissioner emails there too.

You can also change `LEAGUE_NAME`, and `SEASON_PCT_MODE`:

- `"all"` (default) — season percentage counts every game of every scored week,
  so a week you skip counts as zeros. Rewards showing up every week.
- `"played"` — only counts the weeks you actually submitted.

## 3. Publish the rules

Firestore → **Rules** tab. Delete what's there, paste the whole contents of
`firestore.rules`, and **Publish**.

**The email in the rules must match `ADMIN_EMAILS` in `index.html`.** It appears
once, in the `isAdmin()` function. If you added a co-commissioner in step 2, add
them to the list here too:

```
request.auth.token.email in ['chuck8303@gmail.com', 'someone@else.com']
```

## 4. Put it online

The app is one file, so anything that serves a static file works. Two free options:

**GitHub Pages** (uses `git`, which you already have):

```bash
git init && git add index.html && git commit -m "Friday Night Picks"
```

Then create an empty repo on github.com, push to it, and in the repo's
**Settings → Pages** set Source to your `main` branch. Your URL will be
`https://<username>.github.io/<repo>/`.

**Or Cloudflare Pages / Netlify** — both let you drag the folder onto their
dashboard and hand you a URL, no command line.

Either way, take the final domain and add it in Firebase under
**Authentication → Settings → Authorized domains**, or sign-in will be refused.

## 5. Test it locally first (optional)

```bash
cd /Users/cc/Desktop/PICKS && python3 -m http.server 8000
```

Open <http://localhost:8000>. `localhost` is authorized by Firebase out of the
box, so this works before you deploy anything.

---

## Running a week

1. Open the app, click **Sign in as commissioner**, sign in with Google, then
   claim your own name from the roster so you can play too.
2. **Commissioner** tab → set the week number, a label, and **Picks lock at**.
   Type the 15 matchups as *away team* at *home team*.
3. **Save week** keeps it as a draft nobody can see. **Publish to players** makes
   it live and starts the countdown.
4. Send everyone the link. They tap their name once, tap a winner in each game,
   and hit Submit. They can change picks freely until the deadline — and not one
   second after.
5. After the games, **Enter results** → tap each winner → **Post scores**.
   Standings update instantly for everybody.

## The site switcher

The dark strip at the very top links between your two sites: **Media Rankings**
goes to kansasmediarankings.com, **Pick 'Ems** is this one and shows as active.
To change either, edit the two `.sitelink` anchors near the top of the `<body>`
in `index.html`.

Because that bar can navigate away mid-card, the browser now warns before
leaving if someone has tapped picks they never submitted.

## Appearance

The site is white, always. It does not follow the visitor's dark-mode setting —
someone with their phone in dark mode still sees the light design, so what you
see is what everyone sees. The nav strip and the submit bar are dark on purpose;
that is the design, not a theme.

## Branding and link previews

`brand/` holds the artwork and the generated sizes:

| File | Used for |
|---|---|
| `header.png` | the original lockup, kept as the master |
| `header-web.png` | the masthead on the site |
| `helmet.png` | the original helmet, kept as the master |
| `icon-32/192.png` | browser tab icon |
| `icon-180.png` | iPhone home-screen icon |
| `og.jpg` | the image shown when the link is posted |

Upload the whole `brand/` folder with `index.html`.

The link-preview tags point at `https://picks.kansasmediarankings.com/brand/og.jpg`
as an absolute address, because Twitter and Facebook cannot resolve a relative
one. If the site ever moves, edit the `og:` and `twitter:` tags at the top of
`index.html`.

**Previews are cached hard.** Once a platform has scraped your link it keeps
that copy for a long time. To force a refresh after changing the artwork, run
the URL through Twitter's Card Validator or Facebook's Sharing Debugger.

## Team logos

`logos/` holds 167 Kansas school logos plus `teams.json`, the manifest the app
reads. Upload the whole folder to the repo alongside `index.html`.

Then, once only: **Commissioner → Teams → Import Kansas schools.** That loads
215 schools into the library. After that, typing a school into a slate shows
its logo automatically — there is nothing to do per week.

Crests fall back in three steps, so the board never looks broken:

1. the school's logo, if one is set
2. the school's colour behind its initials, if you set a colour
3. plain initials

Coverage is 215 of roughly 340 Kansas football programs. The gap is mostly
small and 8-man schools that FieldLevel doesn't carry, and 48 schools in the
library whose logo came back as a generic placeholder and was dropped. A school
that is missing still works — it is added automatically when you save the week
and shows initials until a logo exists.

To add one by hand: Commissioner → Teams → paste an image URL, or drop a file
into `logos/` and enter `logos/yourfile.png`.

## If you mess up

Almost nothing is permanent. Working from least to most drastic:

| What went wrong | Fix |
|---|---|
| Typo in a team name | Fix the text, **Save week**. Picks stay attached — they're stored per game slot, not per team name. |
| Wrong lock time | Change it and **Save week**. Works after publishing too. If it already locked, setting a later time reopens it. |
| Published before you were ready | **Unpublish** — hides it from players again. Available until the week is scored. |
| Entered the wrong winner | Tap the right one and **Post scores** again. It overwrites the results and everyone's scores. No need to clear anything. |
| Scored too early | Same as above — fix the winners and post again. |
| Someone claimed the wrong name | Roster → **Reset**, then they claim the right one. |
| Everything is a mess | **Clear all picks** — see below. |

**Clear all picks** deletes every submitted card for that week, removes its
scores from the standings, and clears the winners you entered. The games and
the lock time stay, so the week is immediately open for picking again. It
tells you exactly what it's about to delete before it does anything.

This is the one action that can't be undone. Everything else in the table
above is just editing.

**One thing to be careful about:** picks are stored against slot numbers
(game 1, game 2, ...), not team names. Fixing a spelling is safe. But if you
*reorder* the games or swap in a different matchup after people have already
picked, their picks stay on the slot and will now point at the wrong team.
If you need to reshuffle a slate that people have picked, clear the week
first.

## What's actually enforced (not just hidden)

- **The lock is server-side.** A pick write is rejected unless the server's own
  clock is still before that week's deadline. Changing the clock on a phone,
  or editing the page in dev tools, does nothing.
- **Picks are readable only by their owner and by you.** This is a database rule,
  not a UI choice — other players' cards are never sent to a player's browser.
- **Only your Google account can** create weeks, move a deadline, enter results,
  post scores, or view everyone's cards.
- The **schedule itself is readable** by any signed-in player, including a week
  still in draft. The games aren't secret; the picks are.

## Odds and ends

- **Someone got a new phone / cleared their browser.** Their claim lives in that
  browser. Commissioner tab → Roster → **Reset**, and they can claim their name
  again on the new device.
- **Ties.** Whoever has the most correct wins the week; a tie means shared wins,
  and everyone tied gets credit in the Wins column. There's no tiebreaker game —
  say the word if you want one added.
- **Fewer than 15 games** in a week is fine; leave rows blank and it'll ask you to
  confirm before publishing.
- **Cost.** A pool of this size sits far inside Firebase's free tier.
