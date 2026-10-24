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

- **top level** — the app id. `hokm`, `shelem`, `tarneeb`, `callbreak`, `spades`, `hearts`, `haftkhabis`, `twentynine`, `twentyone`, `nikipoo`, `tizbin`, `temsah`. One per app,
  never renamed.
- **second level** — the market: `ca` Cafe Bazaar, `mk` Myket, `all` fallback. Google Play is
  `ps` for the Java card games (`hokm`, `shelem`, `tarneeb`, `callbreak`, `spades`, `hearts`, `haftkhabis`, `twentynine`, `twentyone`) and `gp` for the nikipoo family — each app is keyed the way its own
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

## The cameras — `camera`, `min` only

The twelve kids' cameras (`com.rahpoo.c.*`) read `update.json` from 1.6.4 (versionCode 1604).
Like the nikipoo family they carry no `latest`: on Bazaar the client asks Bazaar itself
(`BazaarUpdater`), and `min` is only for a build that must stop.

- One key, `camera`, covers all twelve: they share one `versionCode`.
- `camera_<last segment of the package>` (e.g. `camera_tomjerry`) overrides it for one app;
  inside each entry the market wins over `all`.
- Their Play key is **`ps`**.
- Myket answers nothing, and the twelve are often on different versions there (an app stuck in
  review), so on `mk` raise `min` only once that exact `versionCode` is live for **every** app the
  key covers, or override the stragglers with their own `camera_<id>` entry.

## `camera-promo.json`

What the twelve camera apps (`com.rahpoo.c.*`, from 1.6.1) advertise: the single icon on the
camera screen (`slot`) and the all-apps dialog (`list`), per market.

```json
{
  "schemaVersion": 1,
  "character": { "com.rahpoo.c.bobi": "com.rahpoo.bobi", ... },
  "markets": {
    "ca": { "live": [...], "slot": ["com.rahpoo.nikipoo", "@character"], "list": [...] },
    "mk": { ... }
  },
  "apps": { "com.rahpoo.x": { "fa": "…", "en": "…", "icon": "https://…/x.png", "kind": "mini" } }
}
```

- **`live`** — the packages whose **current** version on that store is a working one. This is
  the switch to flip when a review lands. A package missing here is never shown on that market,
  whatever `slot` and `list` say. Leave a Nikipoo character app out while the store still serves
  its old pre-2.x build: that build needs nikipoo.com and does not work.
- **`slot`** — candidates in order. The camera shows the first one not yet installed. If all of
  them are installed, it shows the first one and simply opens it.
- **`list`** — the dialog, in this order. `@character` is the Nikipoo School character app
  mapped from the camera in `character`. `@cameras` is the twelve cameras. The app you are in
  is always dropped.
- **`apps`** — only for a package the camera build does not already know. It needs a title and
  an icon url, or it is skipped. The build knows Nikipoo School, the twelve character apps,
  tizbin, temsah, aseman and the cameras.
- `kind` (`school`, `character`, `mini`, `camera`) picks the words of the slot's dialog.
- **`adUnlock`** (camera 1.6.5+) — whether a locked item, and the full version with 30 ads, can be
  opened by watching an ad on that market. Missing: `false` where the build sells (ca, mk), `true`
  where it cannot (ps, none). Majid turned it off on 2026-10-24 because it ate the sales.
- **`paidFeatures`** (camera 1.6.5+) — features that need the full version on that market:
  `background`, `strip`, `colour_in`, `challenge`. Each works once for free first. Missing: all three where the
  build sells, none on ps. `[]` frees them all.
- A market with no entry (and no `all` entry) shows no promotion at all. That covers irapps,
  ch and ps today.
- If the file does not parse, or its `schemaVersion` is not 1, the camera ignores it and uses
  its bundled copy (`app/src/main/assets/camera-promo.json` in the camera repo). Keep that copy
  in step when you ship a camera release.

## `house-ads.json`

House ads: our own apps and games advertised in the ad slots, beside Adivery (Tapsell is gone).
Every Rahpoo app and Parsa's games read it. The design and its reasons are in
`Rahpoo/docs/house-ads-design.md`. The creatives live on `nikipoo.ir/ads/`, not here.

- **`share`**: per format (`banner`, `rect`, `interstitial`, `video`, `rewarded`), the part of
  the slots that go to a house ad even when Adivery has one. With `fillWhenEmpty`, a slot Adivery
  cannot fill always gets a house ad.
- **`variety`**: per-user caps, all counted on the phone. A same-app cooldown, a daily cap per
  app, a rest for a creative after a few views, and a rest for an app tapped twice without an
  install.
- **`groups`**: the first draw is a group, weighted. The second is an app inside it, so twelve
  cameras do not crowd out one Nikipoo School.
- **`hosts`**: which audience an app shows. Adult hosts are named one by one; `*` is `kids`.
- **`apps[pkg].audience`**: who may see that app. A kids host shows only `kids` apps. An adult
  host shows `adult` and `parent` apps. **Never put a card game in front of a child.**
- **`markets.<m>.live`**: the packages with a public, working page on that store. A package goes
  in only after its page answers. A Nikipoo character app goes in only once its store build is
  2.x (the same list as `camera-promo.json`).
- **`apps[pkg].creatives`**: paths under `base`; `a.webp|b.webp` is the portrait|landscape pair.
  An app with no creative for a format is never drawn for that format.
- **Every new app or game**, once published on any store, gets its entry, its creatives and its
  `live` line as part of that release.

## `nakhand.json`

Ad switches and pacing for «بخندی، باختی» (`com.rahpoo.nakhand`). Numbers and switches only; the
game keeps only keys it knows, with the type of its own default, so a typo cannot break it.

```json
{
  "ads": { "on": true, "interstitialEvery": 3, "interstitialFromLevel": 3, "bribesPerRound": 1,
           "lifeAdsPerDay": 5, "leaveEvery": 1, "banner": true },
  "markets": { "bazaar": { "ads": { "on": false } }, "myket": { "ads": { "on": false } } }
}
```

- **`ads.on`** — master switch; off also hides the rewarded buttons (bribe, life). Ads need both
  this and real ad ids in the build. Without this file the game's default is **off**.
- **`markets`** — `play`, `bazaar`, `myket`; a market block overrides the shared one. Bazaar and
  Myket stay off until the build carries Adivery ids.
- Turning ads on reaches every installed build, test APKs with real AdMob ids included: never
  tap a real ad on your own phone.

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
