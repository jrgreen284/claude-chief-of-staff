---
name: meeting-prep
description: "Prep the user for a specific meeting by name. Trigger when the user says \"prep me for X\" or \"meeting prep for X\". Searches the vault for prior meeting notes, related project notes and live commitments tied to the topic, posts a prep brief, and creates a stub note for today's meeting."
argument-hint: "<meeting name or topic>"
---

# Meeting Prep

You are the user's Chief of Staff prepping them for a meeting. The brief comes entirely from what's already in the vault. The only write is a new meeting-note stub; existing files are never edited. Make no Microsoft 365 calls. Reuse whatever today's daily note already holds.

Read `../_shared/conventions.md` (relative to this skill's base directory) first, then load the config.

## Steps

1. **Take the meeting or topic name** from the request (or `$ARGUMENTS`).

2. **Search `<meetings>/`.** Files are named `<Name> - YYYY-MM-DD.md`. Rank the matches in this order:
   1. exact substring in the filename
   2. word overlap with the filename
   3. a content grep for the topic

   For a recurring series, prefer the latest instance.

3. **If there's more than one plausible match, ask which one.** Show the candidates (name and date) and wait. A wrong guess wastes prep time.

4. **Extract from the matched notes.** Pull out open actions and every commitment Id. An Id is any configured `id_prefixes` value followed by `-` and a number.

5. **Cross-reference the task files.** Look up each Id in the task files for its current Status, Priority, Due and Description. The task file is current truth; the meeting note is only a snapshot.

6. **Search project notes.** Look in each `projects[].folder` for filename or topic matches, and for a frontmatter field whose value matches any Id from step 4.

7. **Check today's daily note.** If `## Meetings` lists this meeting, reuse its time, attendees and prep pointer.

8. **Nothing found?** Run a graphify query for the topic (see the conventions) and fold any hits into "Related project notes". If that finds nothing either, say that little was found. Never error out.

## Output — post in chat

```
# Prep — [Meeting Name]

**Last occurrence:** [date] — [[meeting note]]
**Today's slot:** [time] | [location] | [attendees]   ← only if it's in today's daily note

## What happened last time
## Open actions from last time
## Live commitment status
## Related project notes
## Walk in ready to

---
*Meeting note created → [[Meeting Name - YYYY-MM-DD]]*
```

If today's note already exists, print `*(meeting note already exists — skipped)*` instead.

## Create the meeting note

After posting the brief, create `<meetings>/<Meeting Name> - <today YYYY-MM-DD>.md`. Skip this if the file already exists.

```markdown
---
date: YYYY-MM-DD
meeting: [Full meeting name]
attendees:
  - [names if known, else leave the placeholder]
tags:
  - meeting
---

# [Meeting Name] — DD Month YYYY

**Date:** DD Month YYYY
**Time:** [from the daily note if available, else blank]
**Attendees:** [names if known, else blank]

---

## Previously Discussed

[3–5 bullets from "What happened last time" and "Open actions from last time"]

## Prep Items for Today

[Bullets from "Walk in ready to"]

## Agenda

- [ ] 
- [ ] 

## Notes

## Actions
```

Fill it from the brief you've just produced. Don't search again.

## Rules

- Only create new files in `<meetings>/`. Never edit existing vault files.
- Ask rather than guess when the meeting name is ambiguous.
- Always show the Name alongside an Id.
- Obey the config's `confidentiality` rule and `spelling`.
