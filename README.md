<!-- Public home of Dreven's command-line tool, published to npm as @dreven/cli. -->

> **npm:** [`@dreven/cli`](https://www.npmjs.com/package/@dreven/cli) · **Report a problem:** [open an issue](https://github.com/DrevenAI/cli/issues) or write to hello@drevenai.com · **Dreven:** https://drevenai.com
>
> This repository is the public home of the CLI: its README, its changes and its issues. Install it from npm; you do not need to clone anything.

# @dreven/cli

Dreven checks web pages for accessibility problems (WCAG). Most pages can be
checked from the Dreven app. A page you can only see after signing in cannot:
Dreven never asks for your password and never holds your session.

This package runs the check on your own computer instead, in a browser where
you are already signed in. It then sends the evidence (what the page looked
like and what was found) to your Dreven account, where you review the results
like any other audit.

What it never does:

- It never sends your password, cookies or session to Dreven.
- It never keeps anything on your computer after an audit, except an audit it
  could not send because your account had no pages left that month. That one
  is kept for 30 days so you can send it later, then deleted.
- It never audits a page unless you confirm you are allowed to (see
  "Permission" below).

An automated check does not prove a page is accessible. It finds problems and
gathers evidence; a person still reviews the results in Dreven.

## What you need

- Node.js 22 or newer.
- A Dreven account on a paid plan.
- Google Chrome or Microsoft Edge. The audit runs in the browser already
  installed on your computer, in a fresh window that shares nothing with your
  own browsing. With neither, install one, or run
  `npx playwright install chromium` for a private copy.

## Sign in to Dreven

You do not need to do anything first. The first time Claude Code, Codex or
the terminal asks Dreven to do something, a browser tab opens on Dreven's own
Allow page. In Codex, press Allow and ask again. To sign in ahead of time:

```
npx @dreven/cli login
```

A browser tab opens on Dreven's own Allow page. Choose your workspace and
press Allow. That is all: there is no key to copy. The sign-in is kept in
your computer's keychain (the macOS Keychain, the Windows Credential Locker,
or your Linux keyring), never in a file, and renews itself for 30 days. It
can import audits and read results, nothing else. End it with
`npx @dreven/cli logout`, or with Revoke in Dreven under Settings, MCP Servers.

An AI assistant without a terminal can start the same sign-in with the
runner's `sign_in_to_dreven` tool.

## Or use an API key (for CI)

Where nobody can press Allow, such as a GitHub Action, the runner takes a
Dreven API key with the "Import runs" permission instead (Settings, API
keys). It is read from the environment variable `DREVEN_API_KEY`, and from
nowhere else; when it is set it is used in place of the sign-in. Never type
it as part of a command: a command that contains a key is refused, so it
does not end up in your shell history.

```
export DREVEN_API_KEY="<your key>"
```

A key, like a sign-in, only goes to the Dreven deployment that made it. A
key from Dreven's live app only goes to app.drevenai.com, and a key from
Dreven's preview only to the preview; anything else is refused before the
audit starts. To sign in to the preview, set `DREVEN_URL` to
`https://app-preview.drevenai.com` before running `login`.

## Use it from Claude Code

```
claude mcp add dreven -- npx -y @dreven/cli mcp
```

Then ask Claude to audit a page. It will ask you to confirm you may audit the
site before it starts.

## Use it from Codex

```
codex mcp add dreven -- npx -y @dreven/cli mcp
```

Codex does not pass your shell's variables to the tool unless told to. Open
`~/.codex/config.toml`, find the `[mcp_servers.dreven]` section and add this
line to it:

```
env_vars = ["DREVEN_API_KEY", "DREVEN_STORAGE_STATE"]
```

## Use it from a terminal

```
npx @dreven/cli audit <address> --permission
npx @dreven/cli pages-left
npx @dreven/cli send <folder>
```

`pages-left` says how many pages your account has left to audit this month.
`send` sends an audit that was kept because the month's pages had run out.

## Send to my computer

When Dreven's start card finds that a page needs a sign-in, press **Send to
my computer**. Dreven opens the request in Claude, Claude Code or Codex; press
Enter there. The runner opens your browser on the page so you can sign in
yourself, finds the pages inside your signed-in session, audits them, and the
start card shows the audit arrive in your Audit Log. Nothing is copied.

The button sends the audit only to your own computer, signed in with
`npx @dreven/cli login`. A hand-off waits 30 minutes and is used once. You
can also take it from a terminal, or ask any assistant with Dreven added to
"run my Dreven hand-off":

```
npx @dreven/cli handoff
```

## Signing in to the page you audit

From Claude Code or Codex: when the page needs a sign-in, a browser window
opens on your computer. Sign in yourself, as you normally would, then press
**Continue audit** in the Dreven tab. The audit carries on inside that
session. Nothing about the session is saved.

From a terminal, save a session first by signing in yourself in the browser
that opens, then closing it. The browser that opens is Playwright's own; the first
time, install it with `npx playwright install chromium`.

```
npx playwright codegen --save-storage="$HOME/dreven-session.json" <address>
npx @dreven/cli audit <address> --storage-state "$HOME/dreven-session.json"
```

The saved file holds your session for that site. Keep it private, and delete
it when you are done.

## Permission

Before any audit you must confirm: "I own this site or have permission to
audit it." In a terminal, pass `--permission` or answer yes when asked. From
Claude Code or Codex, the assistant is told to ask you, and the audit is
refused unless you said yes. Dreven records the permission against your API
key.

## Pages count

Pages audited from your computer count toward the same monthly pages as
audits started in Dreven. A page already audited this month is free to audit
again.

## Help

`npx @dreven/cli --version` prints the version. Questions: https://drevenai.com

## Changes

- **0.2.5** (31 October 2026): signs in to Dreven's live app (app.drevenai.com) by default. A live key goes only to the live app, a preview key only to the preview. Links to this repository.
