---
name: free-palestine-abuse-tracker
description: >
  Use this skill to find and add new incidents to the "Free Palestine as Abuse"
  dataset at data/incidents.json. Triggers: any request to update the tracker,
  add a new incident, search for new incidents, or review recent antisemitism
  news. Also use when given a URL or article and asked to assess whether it
  qualifies.
---

# Free Palestine Abuse Tracker — Update Skill

## What this dataset is

A citable record of incidents in which the slogan "Free Palestine" (or a close
variant) was explicitly used as a vehicle for antisemitic abuse, threats,
incitement, or celebration of violence against Jews. It is NOT a general
antisemitism tracker. The slogan nexus is the distinguishing feature. The
dataset is global and has no date boundary.

## Your role

When asked to update the tracker, you:
1. Search for new qualifying incidents.
2. Evaluate each against the inclusion criteria.
3. Draft new entries in the correct schema.
4. Present them to the user for approval before writing to the file.
5. Append approved entries to data/incidents.json.
6. Flag any existing entries that may need correction.

## Step 0 — Determine the mode

There are two ways to invoke this skill:

**Search mode (default):** The developer asks Claude to find and add new incidents,
without specifying a particular source. Claude conducts searches (see Step 1),
evaluates candidates (Step 2), drafts entries (Step 3), and presents them for
approval (Step 4) before writing (Step 5).

**Manual mode:** The developer provides a specific URL, article text, or incident
description and asks Claude to add it. In this mode, skip Steps 1 and 2 and go
directly to Step 3 using the provided material. Do not conduct independent searches.
Do not second-guess whether to include it — the developer has already decided.
Still present the drafted entry for approval (Step 4) before writing (Step 5).

---

## Step 1 — Searching for new incidents

### Primary sources to search

The guiding principle is reputable, independent sources — outlets, bodies, and
databases with editorial standards, accountability, and no financial or political
incentive to fabricate. Prefer primary sources (news reporting, court records,
police statements) over aggregators, and aggregators over single-account social
media. Good source categories include:

- National and regional Jewish community organisations that publish incident
  reports (e.g. ECAJ in Australia, ADL in the US, Community Security Trust in
  the UK, RIAS in Germany, CRIF in France)
- Mainstream news outlets with established editorial standards
- Court records, police statements, and government inquiries
- Wire services (AAP, Reuters, AP) for confirmed facts

When sources disagree or a finding is contested, record the dispute in `caveat`
rather than choosing one side.

### Search query principles

The goal is to surface incidents where the slogan was explicitly documented
alongside antisemitic harm — not every protest mention. Searches should therefore
combine the slogan term with harm-related terms (assault, attack, graffiti,
threat, conviction, abuse) and/or victim-targeting terms (Jewish, synagogue,
school). Example patterns (adapt as needed):

- `"free Palestine" [harm term] [location or country]`
- `"free Palestine" Jewish [target type: school / synagogue / community]`
- `site:[reputable-source-domain] "free Palestine" [harm term]`
- `[incident report body, e.g. ECAJ / ADL / CST] "free Palestine" [year]`

Do not limit searches by geography or date. Search broadly then apply the
inclusion criteria to filter.

### What to note during search

For each candidate, record:
- The exact words of the slogan as documented
- What the slogan was accompanied by (slurs, threats, violence, etc.)
- Who was targeted
- Date and location
- The source name and URL
- Whether a police report, charge, or conviction is mentioned

## Step 2 — Evaluating each candidate

Apply the three inclusion criteria in order. If any fails, exclude and note why.

1. **Slogan nexus:** Is "Free Palestine" or a close variant explicitly documented
   in the same act as the abuse? Do not include if the slogan was present at a
   protest that was otherwise peaceable, even if the broader event is controversial.

2. **Antisemitic character:** Was the act directed at Jews *as Jews*? Look for:
   Jewish-identifying clothing, Jewish schools or synagogues as targets,
   explicit Jewish slurs, or the targeting of visibly Jewish individuals.

3. **At least one credible source:** A news outlet, police statement, court record,
   or NGO incident report (ECAJ, ADL, CST, RIAS, etc.). Unverified social media
   posts alone do not qualify. Many incidents are under-reported; a single
   reputable source is sufficient for inclusion. Credibility comes from source
   quality, not source count.

**Caveat field:** Populate only when there is a specific, material reliability
concern — a disputed factual claim, contested motive attribution, or a subsequent
finding that significantly qualifies the account. Do not add boilerplate caveats
to every single-source entry.

**Contested or complex cases:** Flag to the user rather than deciding unilaterally.
Examples: incidents where motive is actively disputed by police or courts;
incidents later attributed to staged or foreign-directed operations.

## Step 3 — Drafting entries

Draft each entry in the schema defined in schema/incident.schema.json. Key discipline:

- `id`: Generate as `{country}-{city}-{YYYY-MM-DD}-{2-3 word label}`, all
  lowercase, hyphens only. Check for collisions with existing IDs.
- `summary`: One sentence, active voice, factual, no editorialising.
- `detail`: Plain prose. Australian English spelling. No markdown inside the
  string. Include all material facts. Attribute quotes to their speaker.
- `slogan_verbatim`: Record exact words if documented; otherwise null.
- `sources`: At minimum one entry. Do NOT fabricate URLs. If you found an
  article but cannot confirm the URL, include the name and note URL as null.
- `added`: Today's ISO date.
- `added_by`: "skill-update-{YYYY-MM-DD}".

## Step 4 — Presenting for approval

Before writing anything to disk, present a summary of:
- How many new incidents found
- A brief description of each (id, date, location, one-line summary)
- Any caveat flagged, if applicable
- Any existing entries flagged for correction

Ask: "Shall I add all, or review individually?"

## Step 5 — Writing to the file

Once approved:
1. Read the current `data/incidents.json`.
2. Append new entries to `incidents`.
3. Resort `incidents` by `date` descending (most recent first).
4. Update `meta.last_updated` to today's ISO date.
5. Update `meta.incident_count` to match the new array length.
6. If this update represents a meaningful expansion (new country, new source
   type, significant new batch), append a note to `meta.notes`.
7. Write back and validate the file is valid JSON before confirming completion.

## Step 6 — Ongoing maintenance flags

While searching, also note (but do not automatically change):
- Entries where legal outcomes have since been resolved or updated
- Entries where a caveat has been resolved, superseded, or needs updating
- Entries with null URLs where the article can now be confirmed

Report these to the user separately as "Maintenance notes."

## Scope notes

- The dataset covers incidents where the slogan is documented. Do not add
  incidents where "Free Palestine" is assumed or inferred.
- The dataset is global with no date boundary. Expand geographically and
  chronologically as research grows.
- The dataset is not exhaustive; it is representative. Prefer well-sourced,
  specific incidents over vague aggregated claims.
- In manual mode, trust the developer's judgement on inclusion. Your role is
  to format and record, not to gatekeep.
