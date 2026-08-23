# "Free Palestine" as Abuse — Incident Tracker

A citable, structured record of incidents in which the slogan "Free Palestine"
(or a close variant) was explicitly used as a vehicle for antisemitic abuse,
threats, incitement, or celebration of violence against Jews.

This is **not** a general antisemitism tracker. The distinguishing feature is
the documented co-occurrence of the slogan with antisemitic harm — verbal
abuse, physical assault, graffiti, school bullying, workplace harassment, and
so on. Incidents where the slogan appeared in an otherwise peaceable context
are excluded.

## Dataset

All data lives in [`data/incidents.json`](data/incidents.json), validated
against [`schema/incident.schema.json`](schema/incident.schema.json). Each
entry records:

- Date, location, and category of the incident
- A one-sentence summary and full prose account
- The slogan as verbatim documented
- What it was accompanied by (slurs, threats, physical violence, etc.)
- Who was targeted
- Source(s)
- Legal outcome, if any
- Caveats, if there is a material reliability concern

The dataset is global with no date boundary. It is representative, not
exhaustive — well-sourced, specific incidents are preferred over vague
aggregated claims.

## Adding incidents

See [`skills/SKILL.md`](skills/SKILL.md) for the full curation workflow,
inclusion criteria, and field-by-field guidance. AI agents working in this
repo should read that file before adding or searching for incidents.

## Sources consulted to date

See the `meta.primary_sources` field in `data/incidents.json`.

## Note on the name

The repository name reflects the phenomenon being documented: the use of a
political slogan as a weapon against Jewish people.
