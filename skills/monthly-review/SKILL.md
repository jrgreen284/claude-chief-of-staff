---
name: monthly-review
description: "Produce the monthly system-health review. Trigger when the user says \"monthly review\", \"month review\", or \"review the month\". Audits the config, task files, project notes, people files, daily-note habits and learning for drift, duplication and housekeeping, then proposes specific fixes. Writes to the reviews folder after confirmation."
---

# Monthly Review

You are the user's Chief of Staff running the monthly system-health review. It isn't a delivery summary; the weekly review covers that. This pass checks the system itself: what's consistent, what's drifted, what's duplicated and what needs pruning.

Every proposed action names the exact file and the exact change. Nothing gets changed until the user approves each item.

Read `../_shared/conventions.md` (relative to this skill's base directory) first, then load the config.

## Step 1 — Choose the target month

Default to the current month. The output file is `<reviews>/month-YYYY-MM.md`. If it already exists, continue from where it left off.

## Step 2 — Check the config against the vault

For every path in the config (`folders.*`, `about_me`, `career_goals`, `task_files[]`, `organisations[].folder`, `projects[].folder`), check that it exists. Flag:
- any missing path
- any organisation folder without a `people.md`
- any task file without its `-archive.md` sibling
- a missing `graphify-out/graph.json`, or one older than the newest note in the vault by more than 7 days (suggest `/graphify . --update`)

## Step 3 — Task file hygiene

Across every task file, flag:
- `Complete` or `Done` rows still sitting in live tables
- rows with no due date, or overdue by more than 14 days
- rows whose `Next Action / Notes` holds more than 2 actions or running commentary (that belongs in project notes)
- Ids whose prefix doesn't match their file's `id_prefixes`, and duplicate Ids
- Threads to Keep Warm untouched for 30+ days (no mention in any daily note this month), as candidates to close
- `standing_items` that are no longer in any task file

## Step 4 — Project notes

For each `projects[].folder`, flag:
- project notes with no matching live or archived commitment (orphans)
- live Tasks with a matching prefix but no project note
- project notes with no status update in 30+ days while their task is still active
- project notes that don't wikilink back to their task file

## Step 5 — People files

Flag:
- the same person appearing in more than one organisation's `people.md`
- entries with no Role or Relation
- people named in this month's daily or meeting notes who aren't in any people file

## Step 6 — Habits

Using this month's `<daily>/` notes:
- Count working days without a daily note.
- Count days where `## Tomorrow` was never filled in, meaning end-of-day capture didn't run.
- Check whether a weekly review exists for each completed week this month.

Report the counts only. Don't moralise.

## Step 7 — Learning

If `folders.learning` exists, list its subfolders or notes. Flag:
- courses or plans with no update in 30+ days
- plans marked done that still appear as open in a task file

Cross-check the active plans in `career_goals`.

## Step 8 — Draft

```markdown
---
type: review
period: month
month: YYYY-MM
tags:
  - review
---
# Monthly Review — Month YYYY
_Run: YYYY-MM-DD_

## Config and structure
## Task file hygiene
## Project notes
## People files
## Habits
## Learning
## Actions — awaiting confirmation
One line per action: [file path] — [exact proposed change] — awaiting yes/no
```

## Step 9 — Confirm before writing or acting

Show the full draft. Write the review file only once the user confirms it. Apply each proposed action only after an explicit yes for that item. Items with no response carry over to next month's review.

## Rules

- Read and report first. Make no writes, deletes or renames until the user approves.
- Each action is specific. Not "tidy the task files", but "move CO-012 Old Spike from `work-company.md` to `work-company-archive.md` — Status is Complete".
- When the config and the vault disagree, don't assume either one is right. Flag the difference.
- Obey the config's `confidentiality` rule and `spelling`.
