# Installing Chief of Staff

This guide takes you from nothing to your first morning brief. Most of it is one-off. Once the plugin is installed, the `setup` skill handles the rest interactively.

## 1. Claude Code, logged in with a claude.ai account

Install [Claude Code](https://docs.claude.com/en/docs/claude-code) and start it with `claude`. Log in with `/login` using your **claude.ai account** (Pro, Max, Team or Enterprise).

This matters for Microsoft 365. Connectors such as Microsoft 365 are set up on claude.ai and only show up in Claude Code when it's logged in with that same account. If Claude Code is using an API key (`ANTHROPIC_API_KEY` or a Console login), the connector won't appear and the Chief of Staff will run vault-only. Run `/status` to check which login is active.

## 2. Obsidian and a vault

Install [Obsidian](https://obsidian.md) and create a vault, or open an existing one. Setup adopts whatever folders you already have instead of restructuring them, and it never overwrites a file.

## 3. Graphify (required)

Graphify keeps a knowledge graph of your vault, so the agents find the right notes without loading whole folders.

```bash
uv tool install graphifyy        # or: pipx install graphifyy
graphify install --platform claude
```

The package is `graphifyy` (two y's), and the command it installs is `graphify`. You can skip this step, because setup checks for Graphify and offers to install it.

## 4. Microsoft 365 (optional)

Skip this step if you only want to run from the vault. You can connect Microsoft 365 later and re-run setup.

1. Go to [claude.ai](https://claude.ai) → **Settings → Connectors**.
2. Find **Microsoft 365** and select **Connect**.
3. Sign in with your work or personal Microsoft account and approve the permissions.
4. Restart Claude Code so that it picks up the connector.

On a company Microsoft tenant, the sign-in may stop at "Need admin approval". In that case your IT admin has to approve the Claude app for your organisation. Until then the Chief of Staff runs vault-only.

The plugin only reads from Microsoft 365. It never sends, replies to, creates or deletes anything there.

## 5. Install the plugin

In Claude Code:

```
/plugin marketplace add jrgreen284/claude-chief-of-staff
/plugin install chief-of-staff@chief-of-staff
```

Restart Claude Code if the new skills don't show up in `/help`.

## 6. Run setup

Start Claude Code in your vault folder, or in the folder above it, and run:

```
/chief-of-staff:setup
```

Setup:

- checks Graphify and offers to install it
- asks for your name, timezone, working hours, organisations, task files and Id prefixes
- shows you the config and every file it will create, then waits for a yes
- writes `.cos/config.md` and scaffolds any missing folders, templates, task files and people files
- checks the Microsoft 365 connection and tells you who you're connected as
- offers to build the vault graph and to allow vault edits without a prompt each time

Anything you decline is listed under **Still to do** at the end, and you can re-run setup at any time to finish it.

## 7. First day

Fill in `about-me.md` and `career-goals.md` in your reference folder, and add a few commitments to your task files. Then say:

```
morning brief
```

## Troubleshooting

**Setup says the Microsoft 365 connector isn't available.** Run `/status` and check that you're logged in with your claude.ai account, not an API key. Check the connector shows as connected on claude.ai, then restart Claude Code. Run `/mcp` to see which connectors the session loaded.

**`graphify: command not found` after installing.** uv's tool folder isn't on your `PATH`. Run `uv tool update-shell` (or `pipx ensurepath`) and open a new terminal.

**A permission prompt on every edit.** Accept setup's offer to allow vault edits, or add this to `<vault>/.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": ["Edit(/path/to/vault/**)", "Write(/path/to/vault/**)"]
  }
}
```

**The skills can't find the config.** Run Claude Code from the vault folder, name the vault path in your request, or set `COS_VAULT`. See [Daily use](README.md#daily-use).
