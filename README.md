# claude-profiles

Use multiple Claude accounts side by side on one Mac. Each profile gets its own
sign-in and chats, its own Claude Desktop icon in the Dock, and its own Claude
Code command. Claude Code settings can be shared with your main account or kept
separate.

It's a single Bash script with no dependencies. Read it before installing.

## Install

You need macOS, Claude Desktop in `/Applications`, and Claude Code
(`curl -fsSL https://claude.ai/install.sh | bash`).

```bash
git clone https://github.com/ishaba/claude-profiles ~/.local/share/claude-profiles
~/.local/share/claude-profiles/claude-profiles add work
```

`add` asks how to handle your Claude Code settings. Then:

1. Drag `~/Applications/Claude work.app` to the Dock and open it.
2. Sign in to your second account (see "Signing in to Desktop").
3. In the terminal, run `claude-work` and sign in there too (`/login`).
4. Run `claude-profiles check`.

`add` puts the `claude-profiles` and `claude-work` commands in `~/.local/bin`,
where Claude Code installs itself. If you installed Claude Code another way and
that folder isn't in your `PATH`, `check` tells you what to add.

Your main Claude (`/Applications/Claude.app` and `claude`) is left untouched.

## Signing in to Desktop

Choose "Continue with email" and type in the code from the email. Browser and
Google sign-in, and the link in the email, all return to the app through a
`claude://` link. macOS sends that link to your main Claude, so the profile
never gets signed in.

If you'd rather sign in through the browser, quit your main Claude first (⌘Q),
sign in to the profile, then reopen it. The profile stays signed in, and both
run side by side from then on.

Signing in from the terminal (`claude-work`, then `/login`) works either way.

## Commands

```
claude-profiles add <name> [--color <color>] [--icon <image>] [--settings shared|copy|empty]
claude-profiles check
claude-profiles remove <name>
claude-<name> [claude arguments]
```

- `add` creates a profile. Running it again repairs the launcher without
  touching the profile's sign-in or chats.
- `--color` tints the icon orange, red, yellow, green, teal, blue, purple, or
  pink. `--icon` uses your own image (png, jpg, icns).
- `--settings` controls what the profile takes from your main Claude:
  CLAUDE.md, settings.json, skills, commands, agents, hooks, and plugins.
  - `shared`: symlinked, so a change in one shows up in both.
  - `copy`: copied once, then independent.
  - `empty`: start fresh.

  With `shared` and `copy`, MCP servers are copied too, including any keys in
  their settings. Sign-in, chats, history, and memory are never carried over.
- `check` lists your profiles with their accounts, and under each one any
  problem with how to fix it.
- `remove` deletes the profile but keeps its sign-in and chats on disk. To
  delete those too, remove `~/.claude-<name>` and, in
  `~/Library/Application Support`, `Claude-<name>` and `Claude-<name>-3p`.
  Then in Keychain Access delete the profile's "Claude Code-credentials" entry.
## How it works

A profile is just a name. Every path is derived from it, so there's no registry
to keep in sync. For `work`:

| What | Where |
|---|---|
| Dock launcher | `~/Applications/Claude work.app` |
| App copy, icon, and log | `~/Library/Application Support/claude-profiles/work/` |
| Desktop sign-in, chats, and settings | `~/Library/Application Support/Claude-work` |
| Claude Code sign-in, settings, and memory | `~/.claude-work` |
| Command | `~/.local/bin/claude-work`, a symlink to the script |

Desktop keeps its data in the folder passed with `--user-data-dir`. Claude Code
uses `CLAUDE_CONFIG_DIR` for its files and to keep its Keychain credentials
separate.

Each profile runs its own copy of `Claude.app`, because that's the only way to
get a separate Dock icon. On APFS the copy takes a second and almost no extra
space. Only the launcher creates or changes it, since macOS lets an app modify
apps it created itself. That's why clicking the launcher after a Claude update
is enough to rebuild the copy and restore its icon, with no extra permissions.

To stop the copy from updating itself and restarting on the wrong account, the
script writes Claude's "Block auto-updates" policy to `Claude-work-3p/`. It
applies to this profile only. If that folder already exists, it's left alone,
since it may hold third-party provider settings.

`claude-work` stays quiet when everything is fine, and warns you if the profile
is signed in to the same account as your main Claude.

## Limitations

- macOS only, and unofficial: `--user-data-dir` is an Electron flag and
  `CLAUDE_CONFIG_DIR` is an undocumented Claude Code variable. If Anthropic
  changes either, the script will need an update.
- Keep the launcher in the Dock, not the running app's icon. That icon starts
  the copy without the profile's data folder, so it opens your main account.
  If that happens, quit the app and reopen it from the launcher.
- One subscription covers one seat, so two profiles need two different
  accounts.

## Migrating from claude-multiprofile

Your existing profile folders are reused, so you won't need to sign in again.

1. Quit this profile's Claude app.
2. If the data folder name has capital letters, rename it and the `-3p` folder
   next to it:

   ```bash
   cd ~/Library/Application\ Support
   mv Claude-WORK Claude-work.tmp && mv Claude-work.tmp Claude-work
   mv Claude-WORK-3p Claude-work-3p.tmp && mv Claude-work-3p.tmp Claude-work-3p
   ```

3. Remove the `claude-multiprofile` block from `~/.zshrc`, or its alias will
   override the new command.
4. Install claude-profiles as above (`add work`). It replaces the old launcher
   and leaves `~/.claude-work` as is.
5. Once everything works, remove the old Dock icons, run
   `npm uninstall -g claude-multiprofile`, and move
   `~/Library/Application Support/claude-multiprofile` to the Trash.

## Credits

Ideas from [claude-multiprofile](https://github.com/jmdarre-v/claude-multiprofile)
(MIT): copying the app for a separate icon, blocking auto-updates through `-3p`,
copying MCP servers, and the email sign-in workaround.
