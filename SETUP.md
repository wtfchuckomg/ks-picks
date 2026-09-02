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
