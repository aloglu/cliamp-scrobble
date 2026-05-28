# cliamp-scrobble

Last.fm scrobbling for [`cliamp`](https://www.cliamp.stream/).

`cliamp-scrobble` waits until you have actually listened to a configurable amount of a track, then sends the scrobble to Last.fm. It also adds keybindings for loving/unloving tracks and temporarily toggling scrobbling.

## Features

- Scrobbles after **50%** of a track by default.
- Configurable scrobble threshold.
- `*` loves or un-loves the current track.
- Loving a not-yet-scrobbled current track immediately scrobbles it too.
- `&` toggles automatic scrobbling for the current `cliamp` session.
- Retryable scrobble failures are cached and retried later.
- No Last.fm now-playing updates.

## Quick Start

### 1. Install

```bash
cliamp plugins install aloglu/cliamp-scrobble
```

Restart `cliamp` after installing.

<details>
<summary>Other install methods</summary>

Install from a local checkout:

```bash
git clone https://github.com/aloglu/cliamp-scrobble.git
cd cliamp-scrobble
cliamp plugins install .
```

Manual file install:

```bash
mkdir -p ~/.config/cliamp/plugins
cp lastfm-scrobbler.lua ~/.config/cliamp/plugins/
cp lastfm-auth ~/.config/cliamp/plugins/
chmod +x ~/.config/cliamp/plugins/lastfm-auth
```

Restart `cliamp` after either method.

</details>

### 2. Create a Last.fm API App

Create an API app here:

<https://www.last.fm/api/account/create>

You will need:

- API key
- shared secret

> [!IMPORTANT]
> Treat the shared secret and session key like passwords.

### 3. Authenticate

Run the helper:

```bash
~/.config/cliamp/plugins/lastfm-auth --write-config
```

The helper will:

- ask for your API key and shared secret
- open Last.fm in your browser
- ask you to approve the app
- write the plugin config to `~/.config/cliamp/config.toml`

If the browser cannot open automatically:

```bash
~/.config/cliamp/plugins/lastfm-auth --no-open
```

<details>
<summary>Manual Last.fm authentication without the helper</summary>

Add this config block after you get a session key:

```toml
[plugins.cliamp-scrobble]
api_key = "$LASTFM_API_KEY"
api_secret = "$LASTFM_API_SECRET"
session_key = "$LASTFM_SESSION_KEY"
enabled = true
threshold = 0.5
poll_secs = 2
```

Manual session-key flow:

1. Create a Last.fm API app at <https://www.last.fm/api/account/create>.
2. Copy the API key and shared secret.
3. Call `auth.getToken` with your API key and a valid API signature.
4. Open `https://www.last.fm/api/auth/?api_key=YOUR_API_KEY&token=TOKEN`.
5. Approve access in Last.fm.
6. Call `auth.getSession` with the approved token and a valid API signature.
7. Copy `session.key` into `~/.config/cliamp/config.toml`.

Caveats:

- Auth tokens expire after 60 minutes.
- Auth tokens are single-use.
- API signatures sort parameters alphabetically, concatenate `name + value`, append the shared secret, and MD5-hash the result.
- Do not include `format`, `callback`, or `api_sig` itself in the signature string.
- Session keys usually stay valid until you revoke the app in Last.fm settings.

</details>

## Configuration

Default config:

```toml
[plugins.cliamp-scrobble]
api_key = "$LASTFM_API_KEY"
api_secret = "$LASTFM_API_SECRET"
session_key = "$LASTFM_SESSION_KEY"
enabled = true
threshold = 0.5
poll_secs = 2
```

`cliamp` supports environment-variable interpolation in `config.toml`, so secrets can stay in your shell environment.

### Scrobble Timing

`threshold` decides when a track is scrobbled.

| Value | Meaning |
| --- | --- |
| `0.5` | scrobble after 50% |
| `0.75` | scrobble after 75% |
| `1.0` | scrobble after the full track |

Example:

```toml
threshold = 0.75
```

`poll_secs` controls how often playback progress is checked:

```toml
poll_secs = 2
```

> [!NOTE]
> Lower `poll_secs` values are more responsive. Higher values do less polling.

## Usage

Inside `cliamp`:

| Key | Action |
| --- | --- |
| `*` | love or un-love the current track |
| `&` | toggle automatic scrobbling for this session |

The `&` toggle only affects automatic scrobbling. You can still love or un-love tracks with `*` while scrobbling is disabled.

If `*` loves the current track before the scrobble threshold, the plugin immediately scrobbles that play session and marks it as handled.

## Behavior

- Forward seeks do not immediately count as listened time.
- Tracks shorter than 30 seconds are ignored.
- Successful scrobbles show a `cliamp` message with the updated Last.fm play count when available.
- Retryable failures are stored in `~/.config/cliamp/lastfm-scrobbler-cache.json`.
- Plugin logs are written to `~/.config/cliamp/plugins.log`.
