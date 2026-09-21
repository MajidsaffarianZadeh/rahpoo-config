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

- **top level** — the app id. `hokm`, `shelem`, … One per app, never renamed.
- **second level** — the market: `ca` Cafe Bazaar, `mk` Myket, `ps` Google Play, `all` fallback.
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

## How the apps read this

Three mirrors of this repo, tried in order until one answers:

```
https://majidsaffarianzadeh.github.io/rahpoo-config/update.json
https://cdn.jsdelivr.net/gh/MajidsaffarianZadeh/rahpoo-config@main/update.json
https://raw.githubusercontent.com/MajidsaffarianZadeh/rahpoo-config/main/update.json
```

At most one fetch every six hours, in the background, cached; a policy normally takes effect at
the next launch. Any failure — offline, filtered, malformed — is silent and the app behaves as if
this repo did not exist. jsDelivr may serve a stale copy for a few hours after a push.

Nothing here is secret: the repo is public and every file is world-readable.
