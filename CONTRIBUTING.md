# Contributing

Issues and pull requests are welcome.

## Testing changes locally

Load the plugin straight from your checkout, with no install needed:

```bash
claude --plugin-dir /path/to/claude-chief-of-staff
```

Point it at a throwaway vault, not your real one, and run `/chief-of-staff:setup` followed by the routine you changed.

Check the manifests before you open a PR:

```bash
claude plugin validate .
```

## Conventions

- **Spelling:** British English throughout (organisation, colour, behaviour).
- **Config keys:** everything a skill reads from the config is defined in `skills/_shared/conventions.md`. If you add a key, add it there and in `skills/setup/templates/config-template.md`.
- **Setup never overwrites files.** Keep it that way in any new scaffolding step.
- **Microsoft 365 stays read-only.** Don't add tools that send, reply, create or delete.
- **No real data.** Never commit a real vault, a `.cos/config.md`, `graphify-out/`, or names from your own work. Examples use fictional organisations such as Globex and Initech.

## Releasing

1. Bump `version` in `.claude-plugin/plugin.json`.
2. Add an entry at the top of `CHANGELOG.md`.
3. Commit, tag `vX.Y.Z` and push the tag.
