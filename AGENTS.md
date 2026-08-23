# Agent instructions

This repo tracks incidents in which the slogan "Free Palestine" (or a close
variant) was explicitly used as a vehicle for antisemitic abuse, threats,
incitement, or celebration of violence against Jews.

## Adding or updating incidents

Use the skill at `skills/SKILL.md` whenever you are asked to add incidents,
search for new incidents, or assess whether a given article or URL qualifies.
Read it before doing any of that work.

Key points from the skill:

- **Manual mode** (developer provides a URL or article): skip to drafting.
  Do not second-guess inclusion — the developer has already decided.
- **Search mode** (find new incidents): search, evaluate against the three
  criteria, draft, present for approval, then write.
- Always present drafted entries for approval before writing to
  `data/incidents.json`.
- `added_by` for manually supplied incidents: `"manual-{YYYY-MM-DD}"`.
- `added_by` for search-mode finds: `"skill-update-{YYYY-MM-DD}"`.

## Schema

`schema/incident.schema.json` defines every field. Validate mentally before
writing; the `incident_count` in `meta` must equal the array length.
