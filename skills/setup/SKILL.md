---
name: setup
description: "Set up or update the Chief of Staff for a vault. Trigger when the user says \"set up chief of staff\", \"configure chief of staff\", \"cos setup\", or when another Chief of Staff skill reports that no config was found. Interviews the user, writes .cos/config.md, and scaffolds the vault folders, templates, task files and people files without overwriting anything."
---

# Setup

You are configuring the Chief of Staff plugin for this user's Obsidian vault. Read `../_shared/conventions.md` (relative to this skill's base directory) first — it defines every config key.

The templates for this skill are in `templates/` inside this skill's base directory.

## Rule zero

**Never overwrite an existing file.** Before every write, check whether the target exists. If it does, skip it and list it under "Kept as-is" in the output. The only exception is `.cos/config.md`, which you may update after showing the user the diff and getting a yes.

## Step 0 — Check prerequisites

Graphify is required. The agents use its knowledge graph of the vault to find notes without reading whole folders, which keeps every routine's context small.

Run `command -v graphify` in Bash.

- **Found:** report the version (`graphify --version` if supported) and carry on.
- **Missing:** tell the user it's required and show the install commands:
  ```bash
  uv tool install graphifyy        # or: pipx install graphifyy
  graphify install --platform claude
  ```
  The PyPI package is `graphifyy` (two y's); the command it installs is `graphify`. Offer to run these for them. Only run them on an explicit yes. If they decline, carry on with setup, but list Graphify under **Still to do** in the output.

## Step 1 — Find or choose the vault

Look for an existing config using the search order in the conventions. If one is found, this is an **update run**: read it, show a short summary, and ask what the user wants to change (add an organisation, add a task file, change folders, toggle Microsoft 365). Then jump to Step 4.

If none is found, ask for the vault path. Offer the working directory as the default. Confirm the folder exists and looks like an Obsidian vault (it has a `.obsidian/` folder). If not, ask whether to continue anyway, because a new vault can be opened in Obsidian later.

## Step 2 — Detect existing structure

List the vault's top two folder levels. If the vault already has folders that match the purposes below (daily notes, meetings, reviews, tasks, reference, templates), propose mapping to them instead of the defaults. An existing vault should be adopted, not restructured.

Default folders:

| Key | Default |
|---|---|
| `reference` | `0) Reference Files` |
| `templates` | `0) Reference Files/Templates` |
| `company` | `1) Company` |
| `clients_vendors` | `2) Clients & Vendors` |
| `tasks` | `3) Tasks & Actions` |
| `learning` | `4) Learning` |
| `daily` | `5) Daily` |
| `meetings` | `6) Meetings` |
| `reviews` | `7) Reviews` |

## Step 3 — Interview

Use the AskUserQuestion tool where it's available and offer sensible defaults. Keep it short, in at most three rounds:

1. **About you.** Name, timezone (IANA, e.g. `Europe/London`), British or American spelling, and usual start, wrap-up and finish times.
2. **Work structure.**
   - Your own organisation (your company), plus any clients and vendors you work with. These become `organisations[]`, with folders `<company>/<Name>`, `<clients_vendors>/Clients/<Name>` and `<clients_vendors>/Vendors/<Name>`.
   - Id prefixes. Each client and vendor gets its own short prefix, so its tasks route to its own `Projects` folder. Default to the first three letters of the name in capitals; if they already use a ticket prefix for that organisation, such as a Jira project key, use that instead. Add every client and vendor prefix to the Client work task file's `id_prefixes`. Company work uses `CO`.
   - Task files. The default is two: `work-clients.md` (Client work) and `work-company.md` (Company work). For each task file, ask for a one-line **scope** — what belongs in it — offering the `scope` values in `templates/config-template.md` as defaults.
   - For each prefix, add a `projects[]` entry whose `folder` is that organisation's `Projects` folder (`CO` → the company's). Ask whether missing project notes should be auto-created (`auto_create`); default to true for `CO` and false for clients and vendors.
3. **Integrations and rules.**
   - Microsoft 365: is the connector enabled in Claude?
   - A one-line confidentiality rule, e.g. what must never appear in reviews.
   - Any standing items that must always show in the Focus Plan (usually none at first).

## Step 4 — Show the plan, then write

Show the user in one message:
- the config you are about to write (the full frontmatter)
- every folder and file you will create, marking those that already exist as "keep"

Wait for an explicit yes. Then create, skipping anything that exists:

1. `.cos/config.md` — built from `templates/config-template.md`. Fill every empty value with the user's answer and replace each `[]` list that has commented examples with the user's entries. Delete the commented `# e.g.` example blocks once their list is filled. Keep the body text. Before writing, check that no key is left empty; if one is, ask for it rather than writing a blank.
2. `.cos/memory/` — an empty folder.
3. Every folder in `folders.*`, each org folder, and each project folder. Create them with a single `mkdir -p` listing every quoted path, because the Write tool can't create empty folders.
4. In `folders.templates`: read `daily-prep-template.md`, `project-template.md` and `people-template.md` from this skill's `templates/` folder and write each one unchanged with the Write tool. Don't use `cp` — use Read and Write for every template in this step and the ones below, so setup works under strict permission settings.
5. For each task file:
   - `<tasks>/<file>` from `task-file-template.md`, with `[Label]` and `[scope]` filled from that task file's `label` and `scope` in the config.
   - `<tasks>/<name>-archive.md` from `task-archive-template.md`, with `[Label]` filled in.
6. `<org folder>/people.md` for each organisation, from `people-template.md`, with `[Organisation]` filled in.
7. `about_me` from `about-me-template.md`, and `career_goals` from `career-goals-template.md`.

## Step 5 — Check Microsoft 365

If `integrations.microsoft_365` is false, report "Microsoft 365 switched off in the config — the Chief of Staff will run vault-only. Set `integrations.microsoft_365: true` and re-run setup to connect it later." and skip the rest of this step.

If it is true, call the Microsoft 365 connector's `get_me` tool, and if that's unavailable, `outlook_calendar_search` for today. Report one of:
- "Microsoft 365 connected as <name/email>."
- "Microsoft 365 connector not available in this session. Enable it at claude.ai → Settings → Connectors, then restart Claude Code. The Chief of Staff will run vault-only until then."

Never change the config because of this check. The user may connect it later.

## Step 6 — Build the vault graph

Skip this step if Graphify isn't installed (Step 0).

If `<vault>/graphify-out/graph.json` already exists, report it and move on. Otherwise explain that the agents need a graph of the vault, that building it reads every note once and uses tokens in proportion to the vault's size, and that later refreshes with `--update` only re-read changed notes. Offer to build it now. On an explicit yes, invoke the `graphify` skill with the vault root as the path. On a no, list "Run `/graphify .` from the vault root" under **Still to do**.

## Step 7 — Permissions (offer, don't do)

Explain that the skills write to the vault, and that allowing this avoids a prompt on every edit. Show the snippet:

```json
{
  "permissions": {
    "allow": ["Edit(<vault path>/**)", "Write(<vault path>/**)"]
  }
}
```

Offer to add it to `<vault>/.claude/settings.local.json`. Only write it on an explicit yes, and merge it into any existing file rather than replacing it.

## Output

End with:

- **Created** — folders and files written.
- **Kept as-is** — anything that already existed.
- **Microsoft 365** — the Step 5 result.
- **Graphify** — installed or not, and whether the vault graph was built.
- **Still to do** — anything declined above (omit if empty).
- **Next steps:**
  - Fill in `about-me.md` and `career-goals.md`.
  - Add a few commitments to your task files.
  - Say "morning brief" to start your first day.
