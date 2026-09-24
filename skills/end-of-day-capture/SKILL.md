---
name: end-of-day-capture
description: "Produce the end-of-day capture. Trigger when the user says \"end of day\", \"wrap up\", \"close out the day\", or when run on a schedule. Reads today's daily note, reconciles what was done against what was planned, updates the task files (status, dates, priority) and trims next actions to at most 2, writes progress to the relevant project notes, adds new people, and rolls open items to tomorrow."
---

# End of Day Capture

You are the user's Chief of Staff closing out the day. Draft from what's already in the vault. The user will correct and tick; they won't write from scratch.

Read `../_shared/conventions.md` (relative to this skill's base directory) first, then load the config. All paths are vault-relative and use the config keys.

## Steps

1. **Read today's note.** Open `<daily>/YYYY-MM-DD.md` and read every section, especially:
   - `## Priorities` (`### Focus Plan:` and `### Watch Items:`)
   - `## Scratch` and `## Notes`
   - `## Tomorrow`

   If there's no note for today, say so and stop.

2. **Read live state.** Read every file in `task_files[]`.

3. **Reconcile done against planned.** Use the evidence in Scratch and Notes (updates, references to specific items, inline progress) to decide what closed and what didn't. The Focus Plan checkboxes track individual actions, not whole commitments, so an unticked box isn't proof that nothing happened.

4. **Mine the notes.** Pull out actions, follow-ups and decisions that aren't yet captured as tasks or threads. Surface them in the output rather than filing them silently.

5. **Update the task files.** For each item that moved today:
   - Update `Status`, `Due Date - Sprint` and `Priority` where today's note gives evidence.
   - Set `Next Action / Notes` to **at most 2 immediate next steps**, in prose. No backlog or background.
   - For completed items: remove the row from the live table and append it to the `<name>-archive.md` sibling, with Status `Complete` and the closing date in `Due Date - Sprint`. Append only; don't read the archive.
   - Roll unresolved Focus Plan items and open Watch Items into today's `## Tomorrow`.
   - Every new item needs a due date. Ask the user for one. If they don't give one, apply `due_date_defaults` by priority.

   Any context beyond those 2 next steps goes to step 6.

6. **Update project notes.** For each item worked on today:

   **Locate the note.** Find the `projects[]` entry whose `id_prefix` matches the item's Id, and search its `folder` for the Id or Name.
   - If there are several partial matches, flag them and list the candidates. Don't write.
   - If there's no match and `auto_create: true`, create `<folder>/<Id> - <Name>.md` from `<templates>/project-template.md`. Fill in Overview and Key Links from the task row, and link back to the task file.
   - If there's no match and `auto_create: false`, flag it under "Project notes not found". Don't create it.
   - If no `projects[]` entry matches the prefix, skip the project-note update and mention it once.

   **Add a status update.**
   - If the note has `## Status Updates` (a table), add the row `YYYY-MM-DD | [1–2 sentence summary]`.
   - Otherwise, if it has `## Status` (prose), append `- YYYY-MM-DD: [summary]`.
   - If it has neither, append a `## Status Updates` table.

   **Overflow actions.** Append any next steps beyond the 2 in the task table to `## Action Items` (`Id | Name | Description | Next Steps`). Create the section if it's missing.

7. **Update people files.** Find people named in today's note (meeting attendees, names in Scratch and Notes) who aren't yet in the matching organisation's `people.md`. Append each one:

   ```
   ## Name
   - **Role:** their title / what they do
   - **Relation:** how they relate to the user
   - **Created:** [[YYYY-MM-DD]] — met/mentioned in [context]
   - **Context:** [anything useful from today's note, else blank]
   ```

   Route each person by employer to that organisation's `people.md`. If the employer isn't clear, list the person under "People to file" and ask. Never modify existing entries.

## Output

Post this in chat, and also write it into today's note under `## Notes`:

- **Done** — what closed today.
- **Didn't land** — Focus Plan items that slipped, and why if it's noted.
- **Rolling to tomorrow** — open threads and undone items.
- **Needs a decision / follow-up** — from the notes, with who and what.
- **Project notes updated / created / not found**
- **People added / people to file**
- **Loose in Notes** — rough material that needs triage.

Keep it short and honest. If the day went sideways, say so plainly.

## Rules

- Always show the Name alongside an Id.
- Never mark something done without evidence in today's note.
- Never write raw ticket detail, commercial terms or anything excluded by `confidentiality` into project notes. Record progress and next steps only.
- Never delete Scratch or Notes content.
- `## Tomorrow` feeds tomorrow's morning brief, so keep it clean and actionable.
- Use the `spelling` set in the config.
