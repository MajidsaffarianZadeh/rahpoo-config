# rahpoo-config

Static configuration for the Rahpoo apps and games. No server, no build step: the files here are
the API. Editing one and pushing it is a deployment.

## `update.json`

Tells an installed build whether it is behind. Every app reads the same file and looks up its own
key; an app that is not listed is not governed.

```json
{
  "hokm":   { "all": { "min": 0, "latest": 126 }, "mk": { "min": 0, "latest": 120 } },
  "shelem": { "all": { "min": 0, "latest": 30 } }
}
```

- **top level** — the app id. `hokm`, `shelem`, `tarneeb`, `callbreak`, `spades`, `hearts`, `haftkhabis`, `nikipoo`, `tizbin`, `temsah`. One per app,
  never renamed.
- **second level** — the market: `ca` Cafe Bazaar, `mk` Myket, `all` fallback. Google Play is
  `ps` for the Java card games (`hokm`, `shelem`, `tarneeb`, `callbreak`, `spades`, `hearts`, `haftkhabis`) and `gp` for the nikipoo family — each app is keyed the way its own
  build spells the market, so check before you type it.
- **`min`** — the oldest `versionCode` still allowed to run. Below it, the app blocks and asks
  the player to update.
- **`latest`** — what that market actually has. Above the installed build, the app suggests an
  update, dismissibly.
- **`title`, `message`, `force_message`** — optional texts that replace the app's own.
- **`0`** — say nothing.

### The one rule

Raise `min` only after that exact `versionCode` is **live and approved in that exact market**.
Bazaar, Myket and Play approve on different days; that is why each has its own entry. Forcing a
version a market has not published leaves its users behind a button that leads nowhere.

Clients also refuse to honour a `min` above `latest`, but do not lean on it.

## The nikipoo family is different — `min` only

`nikipoo`, `tizbin` and `temsah` carry **no `latest`**, on purpose. Their client asks the store
itself whether a newer build exists — `BazaarUpdater` on Bazaar, Play In-App Updates on Play, the
`myket://check-update` intent on Myket — because a hand-kept number was always either a version
still in review or one approved in a single market. The store knows what is actually live; we do
not have to.

What the store cannot say is *"this build may no longer run"*. That is the only thing left in
this file for them, and it is `0` in the normal case. Touch it for a build that is actively
harmful — a crash loop, a broken protocol, a payload schema an old build silently ignores.

Two things that will lock an install base if you get them wrong:

- **`versionCode` bases differ**: `nikipoo` is `50000+`, `tizbin` and `temsah` are `10000+`.
  A number written under the wrong app id locks that app completely.
- **Their Play key is `gp`, not `ps`.**

Their client ignores `min` whenever the store reports the installed build is already the newest,
so an early `min` cannot strand anyone. **That guard does not work on Myket**, which answers
nothing at all — for `mk`, the one rule above still applies in full.

## How the apps read this

Three mirrors of this repo, tried in order until one answers:

```
https://majidsaffarianzadeh.github.io/rahpoo-config/update.json   ← needs Pages switched on
https://cdn.jsdelivr.net/gh/MajidsaffarianZadeh/rahpoo-config@main/update.json
https://raw.githubusercontent.com/MajidsaffarianZadeh/rahpoo-config/main/update.json
```

At most one fetch every six hours, in the background, cached; a policy normally takes effect at
the next launch. Any failure — offline, filtered, malformed — is silent and the app behaves as if
this repo did not exist. jsDelivr may serve a stale copy for a few hours after a push.

⚠️ **GitHub Pages is not enabled on this repo yet** — the first mirror above returns 404, so only
two of the three actually serve. Settings → Pages → Deploy from branch → `main` → `/ (root)`.

This repo also hosts the other static files the apps read (`promo.json`, `config.json`,
`news.json`, `level_tuning.json`). The nikipoo client already fetches `level_tuning.json` and
`promo.json` from here and falls back silently to its bundled copies while they are missing.

Nothing here is secret: the repo is public and every file is world-readable.
