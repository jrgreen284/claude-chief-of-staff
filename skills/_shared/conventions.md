# Chief of Staff — shared conventions

Every skill and agent in this plugin follows this file. Read it before any vault read or write.

## 1. Locate the config

All user-specific values live in a single config file: `.cos/config.md` at the root of the user's Obsidian vault. Find it in this order and stop at the first hit:

1. A vault path given in the request or the skill's arguments — use `<path>/.cos/config.md`.
2. `./.cos/config.md` — Claude was launched from the vault root.
3. `./*/.cos/config.md` — the vault is one folder below the working directory.
4. `$COS_VAULT/.cos/config.md` — read the variable with `printenv COS_VAULT` only if 1–3 find nothing. If that command is blocked, skip this step.

Check 2 and 3 with the Read or Glob tools rather than shell commands, so the lookup works under strict permission settings.

If none exists, stop and tell the user: "No Chief of Staff config found — run `/chief-of-staff:setup` first." Do not guess paths.

The vault root is the folder that contains `.cos/`. Every path in the config is relative to the vault root.

## 2. Read the config

The config is YAML frontmatter. Keys used across the plugin:

| Key | Meaning |
|---|---|
| `user_name` | How to address the user. Use it wherever these instructions say "the user". |
| `timezone` | IANA zone (e.g. `Europe/London`). Convert every calendar time to it before display. |
| `spelling` | `British` or `American`. Apply throughout all output. |
| `start_time`, `wrap_up_time`, `finish_time` | Working-day start, wrap-up and finish (HH:MM). |
| `confidentiality` | Free-text rule about what must never be copied into reviews, briefs or summaries. Obey it everywhere. |
| `folders.*` | `reference`, `templates`, `company`, `clients_vendors`, `tasks`, `learning`, `daily`, `meetings`, `reviews` — vault-relative folder paths. |
| `about_me`, `career_goals` | Vault-relative paths to the user's profile and growth goals. |
| `task_files[]` | Each has `file` (inside `folders.tasks`), `label`, `scope` (one line on what belongs in it) and `id_prefixes[]`. |
| `organisations[]` | Each has `name`, `kind` (`company`, `client`, `vendor`) and `folder`. Each org's `people.md` lives at `<folder>/people.md`. |
| `projects[]` | Each has `id_prefix`, `folder` (where project notes for that prefix live — normally one prefix per client or vendor, pointing at that organisation's `Projects` folder) and `auto_create` (true = create a missing note from the project template; false = flag it). |
| `horizons.focus_plan_days`, `horizons.watch_days` | Look-ahead windows for the morning brief. |
| `due_date_defaults` | Days to add per priority when a new commitment has no due date (`High`, `Medium`, `Low`). |
| `standing_items[]` | Commitment Ids that must always appear in the Focus Plan until marked complete. |
| `integrations.microsoft_365` | `true` if the Microsoft 365 connector should be used. |

If a key the current task needs is missing, say which key is missing and carry on with the rest rather than failing the whole run.

## 3. Vault conventions

**Task files.** Each task file has `## Tasks` and `## Threads to Keep Warm`. Both tables use exactly these columns:

`Id | Name | Priority | Status | Due Date - Sprint | Description | Next Action / Notes`

Completed rows move to a sibling file named `<name>-archive.md` in the same folder, under `## Archive` with the same columns. Never leave `Complete` or `Done` rows in a live table.

**Daily notes** live at `<folders.daily>/YYYY-MM-DD.md`, built from `<folders.templates>/daily-prep-template.md`.

**Meeting notes** live at `<folders.meetings>/<Meeting Name> - YYYY-MM-DD.md`.

**Reviews** live at `<folders.reviews>/week-YYYY-WNN.md` and `<folders.reviews>/month-YYYY-MM.md`.

**Ids.** Always show a commitment's Name alongside its Id — never an Id on its own.

## 4. Obsidian markdown rules

Apply these to every file written in the vault:

- Frontmatter is flat YAML between `---` lines at the very top. Use `type`, `tags` (a YAML list) and `related` (a YAML list of quoted wikilinks, e.g. `- "[[work-company]]"`). Dates are `YYYY-MM-DD`.
- Link notes with `[[Note Name]]` (no `.md`), or `[[Note Name|display text]]`. Link a heading with `[[Note Name#Heading]]`.
- Callouts: `> [!note]`, `> [!warning]`, `> [!todo]` on the first line, content on following `>` lines.
- Tasks: `- [ ] item` and `- [x] item`.
- Tables need a header separator row. Escape any `|` inside a cell as `\|`, including in wikilink aliases.
- Never leave template HTML comments (`<!-- ... -->`) in a generated note.

If the `claude-obsidian:obsidian-markdown` skill is installed, you may consult it for anything these rules don't cover.

## 5. Microsoft 365

Calendar, email and Teams come from the Microsoft 365 connector in Claude (claude.ai → Settings → Connectors). Its tools are named `mcp__<server>__<tool>`; match on the tool suffix:

| Need | Tool suffix |
|---|---|
| Today's meetings | `outlook_calendar_search` |
| Unread / recent email | `outlook_email_search` |
| Teams chats (list) | `teams_list_chats` |
| Teams chat / channel messages | `chat_message_search`, `teams_list_channel_messages` |
| Who the user is | `get_me` |

If `integrations.microsoft_365` is `false`, or no tool with these suffixes is available, skip every Microsoft 365 step, write one line in the output saying "Microsoft 365 not connected — calendar, email and Teams skipped", and complete the rest of the task from the vault alone. Never fail a run because Microsoft 365 is missing.

Microsoft 365 access is **read-only** for this plugin. Never send, reply, create events, delete or move anything.

## 6. Graphify (required)

Graphify is a prerequisite. It keeps a knowledge graph of the vault at `<vault>/graphify-out/graph.json`, so agents can find the right notes without reading whole folders into context.

- To find a note, person, project or topic, run `graphify query "<keywords>"` from the vault root **first**. Read only the files it points to.
- Fall back to `grep -ril` / `find` only when the query returns no confident match.
- If the `graphify` command is missing, or `graph.json` doesn't exist, say so in one line ("Graphify not installed — run `/chief-of-staff:setup`" or "No vault graph — run `/graphify .` from the vault root"), then carry on with `grep`/`find`. Never fail a run because of it.
- Never rebuild the graph yourself during a routine; building it uses tokens. Suggest `/graphify . --update` when the graph looks stale.

## 7. Session memory

Agent session notes live in `<vault>/.cos/memory/`. Each agent reads `<agent-name>-last-session.md` at the start of a session if it exists, and overwrites it with a short summary (discussed, decided, left open) at the end. When the user says "remember this", append a dated line to `<vault>/.cos/memory/remembered.md`.

## 8. Universal rules

- Never overwrite or delete a vault file without the user's confirmation, except where a skill explicitly allows a direct edit.
- Never delete, send or publish anything without explicit confirmation.
- Trust what is in the vault — the user curates it.
- Be direct. No filler, no preamble.
