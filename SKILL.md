---
name: tg-cli
description:
  "Read and send your own Telegram chats from the terminal via the bundled
  `bin/tg` command. Logs in as your Telegram **user account** over MTProto
  (Telethon) — not a bot — so it sees every chat your account sees. `tg login`
  authenticates once (phone + code + 2FA), `tg chats` lists conversations, `tg
  read <name>` shows a thread, `tg search <text>` searches, `tg send <name>
  <text>` sends one (dry-run unless `--yes`). Session and API credentials are
  stored chmod 600 in ~/.config/tg-cli. Use when the user wants to read, search,
  or send their personal Telegram messages."
---

# Telegram reader / sender CLI

Terminal access to your **own** Telegram account. It authenticates as your user
account over MTProto via [Telethon](https://docs.telethon.dev), so it can read
and send anything your account can — unlike a bot, which only sees chats it was
explicitly added to.

## How to invoke

Run the bundled launcher **`bin/tg`** (PEP 723 — `uv` resolves deps inline on
first run: `click` + `telethon`). Resolve `bin/tg` against this skill's
directory; from elsewhere use the absolute path.

## Prerequisite: log in (one-time)

1. Get an **api_id** and **api_hash** from <https://my.telegram.org> → *API
   development tools*. These identify the client app, not your account, and are
   reused across logins.
2. Run `bin/tg login`. It prompts for the api_id / api_hash (stored after the
   first time), then for your **phone number**, the **login code** Telegram
   sends you, and your **2FA password** if you have one set.

Credentials and the authenticated session are written `chmod 600` inside a
`chmod 700` `~/.config/tg-cli/` (override with `TG_CONFIG_DIR`, or inject creds
via `TG_API_ID` / `TG_API_HASH`).

## Security

The session file (`~/.config/tg-cli/session.session`) **is a credential** — it
grants full access to the account without the password. Keep it off synced
folders and out of git (the repo `.gitignore` blocks `*.session` and
`config.json`). If it ever leaks, revoke it instantly from the Telegram app:
**Settings → Devices → terminate session**. Enabling a 2FA password on the
account is strongly recommended.

## When to use

Trigger when the user wants to **read, search, or send their personal Telegram
messages** — "show my Telegram chat with X", "what did Y send me on Telegram",
"search my Telegram for Z", "message X on Telegram that …".

Do **not** use it on someone else's account. Sending is an outbound action:
confirm the exact recipient and text with the user before running `send --yes`.

## Commands

### `tg login [--api-id ID] [--api-hash HASH]`

Interactive one-time authentication (see above).

### `tg whoami [--json]`

Show the logged-in account (name, username, id).

### `tg chats [--limit N] [--json]`

List conversations, most recent first, each tagged `[dm]` / `[group]` /
`[channel]` with an unread count.

### `tg read <query> [--limit N] [--match N] [--json]`

Show messages from the chat whose name best matches `<query>` (case-insensitive
substring). Ambiguous matches print a numbered list — pick one with `--match N`.

```bash
bin/tg read alice              # most likely "Alice" chat
bin/tg read "Dev Team" --limit 200
bin/tg read alice --match 2    # 2nd match when ambiguous
```

### `tg search <text> [--chat QUERY] [--match N] [--limit N] [--json]`

Search messages containing `<text>`. With `--chat` it searches that one
conversation (single exact request); without it, a best-effort global search
across all chats.

### `tg send <query> <text> [--match N] [--peer P] [--yes]`

Send to the chat matching `<query>`. **Defaults to a dry-run** printing the
resolved recipient and text; pass `--yes` to actually send. Target an exact
recipient with `--peer @username` / phone / id, or disambiguate with `--match`.

```bash
bin/tg send alice "running late"          # dry-run: shows recipient + text
bin/tg send alice "running late" --yes    # actually sends
```

## Notes

- **Reading is non-destructive** — only `send --yes` writes anything.
- **Timestamps** are converted from Telegram's UTC to local time.
- Non-text messages render as a `[MediaType]` / `[ActionType]` placeholder.
- Every read command supports `--json` for structured output.
