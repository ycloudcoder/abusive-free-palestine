# "Free Palestine" as Abuse — Incident Tracker

A citable, structured record of incidents in which the slogan "Free Palestine"
(or a close variant) was explicitly used as a vehicle for antisemitic abuse,
threats, incitement, or celebration of violence against Jews.

## Repo structure

```
data/incidents.json           The dataset — edit this to add or update incidents
schema/incident.schema.json   JSON Schema defining every field
skills/data-updates.md        Curation workflow, inclusion criteria, and field guidance
index.html                    Single-page browser UI — reads data/incidents.json at runtime
```

## Adding incidents

Read `skills/data-updates.md` first. It covers the two modes (manual and
search), the three inclusion criteria, step-by-step write instructions, and
field-by-field guidance. The key constraint: the slogan must be explicitly
documented in the same act as the antisemitic harm — this is not a general
antisemitism tracker.

## Running locally

Serve the repo root with any static file server, e.g.:

```
python3 -m http.server
```

Then open `http://localhost:8000`.
