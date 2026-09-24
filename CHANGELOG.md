# Changelog

## 0.1.1 — 2026-09-22

- **Graphify is now required.** Setup checks it is installed, offers to install it and offers to build the vault graph. Every agent and skill queries the graph before searching files. The monthly review flags a missing or stale graph. The `integrations.graphify` config key is removed.
- **Folders renamed:** `1) Internal` is now `1) Company` and `2) External` is now `2) Clients & Vendors`. The config keys are now `folders.company` and `folders.clients_vendors`, and the organisation kind `internal` is now `company`.
- **Task files renamed:** `work-client.md` is now `work-clients.md` (Client work, `CL`) and `work-internal.md` is now `work-company.md` (Company work, `CO`). Each task file now has a `scope` in the config, and setup asks for it.
- **Config template:** example organisations and projects are now commented out, and user-specific values start empty, so nothing example-only can leak into a real config.
- **Per-client Id prefixes:** each client and vendor now gets its own prefix and project folder, instead of one shared `CL` prefix routing everything to a single client. Setup uses an existing Jira key as the prefix where there is one.
- **Works under strict permissions:** setup writes templates with Read/Write instead of `cp`, and the config lookup checks the vault path in the request and the working directory before reading `COS_VAULT`.
- **Setup reports Microsoft 365 switched off** when the config disables it.
- **Personal task file removed,** along with hard stops. The morning brief now flags the finish time from the config only.
- **Packaging:** MIT licence, install guide, contributing guide, issue templates and an example daily note.

## 0.1.0 — 2026-09-21

First release.

- **Agents:** `ai-chief-of-staff`, `ai-executive-assistant`, `ai-mentor-and-coach`.
- **Skills:** `setup`, `morning-briefing`, `end-of-day-capture`, `commitments-review`, `meeting-prep`, `weekly-review`, `monthly-review`.
- **Config:** a single per-vault config file, `.cos/config.md`.
- **Microsoft 365:** optional, via the Claude Microsoft 365 connector (read-only).
