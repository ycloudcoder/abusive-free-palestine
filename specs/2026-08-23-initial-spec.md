# Spec: "Free Palestine as Abuse" Incident Tracker

**Scope:** Dataset schema · Skill definition · Static web app contract  
**Philosophy:** KISS. Static files, no build step, no backend. Claude updates the dataset; a single HTML file presents it.

---

## 1. Goals and non-goals

### Goals
- Maintain a citable, versioned record of incidents in which "Free Palestine" (or close variants) was used as a vehicle for antisemitic abuse, threats, incitement, or celebration of violence against Jews.
- Make that record browsable, filterable, and exportable with no infrastructure beyond static file hosting (e.g. GitHub Pages, Netlify, Cloudflare Pages).
- Define a repeatable process by which Claude can find, evaluate, and append new incidents using a SKILL.md file.
- Seed the dataset with the Australian incidents already researched, to give the initial implementation something concrete to work with while the data model and web app take shape. The dataset is international and unbounded in date range from the outset.

### Non-goals
- Cataloguing every anti-Israel protest or every antisemitic incident. The dataset has a narrow, specific focus: incidents where the *slogan itself* was the instrument or accompaniment of harm.
- Real-time updates or a backend API.
- Automated scraping. Updates are Claude-assisted, human-approved.
- Taking a position on whether "Free Palestine" is inherently antisemitic. The dataset records instances where it *demonstrably was*, and is explicit about that scope.

---

## 2. Repository structure

```
/
├── data/
│   └── incidents.json        # The canonical dataset (JSON array)
├── schema/
│   └── incident.schema.json  # JSON Schema for validation
├── skills/
│   └── SKILL.md              # Instructions for Claude to update the dataset
├── index.html                # The entire web app (single file, no build)
└── README.md
```

No `package.json`, no bundler, no framework. `index.html` loads `data/incidents.json` via `fetch()`.

---

## 3. Dataset schema

File: `data/incidents.json` — a JSON object with a `meta` block and an `incidents` array, sorted by `date` descending.

### 3.0 Top-level structure

```jsonc
{
  "meta": {
    "last_updated": "2026-08-23",
    // ISO date of the most recent write to the file.

    "incident_count": 22,
    // Integer. Must match incidents array length. Updated on every write.

    "notes": "Seed dataset (v0.1): Australian incidents researched August 2026. International expansion pending.",
    // Free-text field. Use to record the current state of the dataset,
    // known gaps, or curation decisions. Update with each significant expansion.
    // Keep it short — two or three sentences at most per entry; append rather than overwrite.

    "primary_sources": [
      "ECAJ Annual Reports (ecaj.org.au)",
      "Menzies Research Centre antisemitism tracker (menziesrc.org)",
      "ADL Audit of Antisemitic Incidents (adl.org)",
      "Community Security Trust (cst.org.uk)",
      "RIAS Germany (report-antisemitism.de)",
      "Combat Antisemitism Movement (combatantisemitism.org)"
    ]
    // Array of strings. The main sources consulted to date.
    // Add new sources here when a research expansion covers them.
  },

  "incidents": [ /* incident objects — see 3.1 */ ]
}
```

### 3.1 Field definitions

```jsonc
{
  // REQUIRED FIELDS

  "id": "au-mel-2025-10-07-fitzroy-graffiti",
  // Stable, human-readable slug. Format: {country}-{city_abbr}-{YYYY-MM-DD}-{brief_label}
  // Use ISO 3166-1 alpha-2 for country (au, us, gb, gr, nl, ...).
  // No spaces; hyphens only. Must be unique across the dataset.

  "date": "2025-10-07",
  // ISO 8601. Use the most precise date known.
  // If only month known: "2025-10". If only year: "2025".
  // If a range: use the start date and explain in date_note.

  "date_display": "7 October 2025",
  // Human-readable string exactly as it should appear in the UI.

  "date_note": null,
  // Optional string. Use when date precision is limited or a range applies.
  // E.g. "incident reported over several days", "exact date unconfirmed; reported Nov 2024".

  "country": "AU",
  // ISO 3166-1 alpha-2.

  "location": "Fitzroy, Melbourne, Victoria",
  // Full location string: suburb/neighbourhood, city, state/region.
  // For non-Australian incidents, use city and country region as appropriate.

  "location_short": "Fitzroy, Melbourne",
  // Compact version for UI badges and map labels.

  "category": "graffiti_vandalism",
  // Enum — see Section 3.2.

  "summary": "Graffiti on anniversary of 7 October attacks: 'Free Palestine', 'Oct 7, do it again', 'Glory to Hamas'.",
  // One sentence. Active voice. Factual, not editorialised.
  // Should be self-contained enough to be understood without reading detail.

  "detail": "On the second anniversary of the Hamas attacks, 'Glory to Hamas' was painted on a billboard at Alexandra Parade and Brunswick Street, Fitzroy. 'Free Palestine' and 'Oct 7, do it again' appeared on a building near Smith Street. Similar graffiti appeared in Westgarth. PM Albanese called it 'abhorrent'; Premier Allan and Opposition Leader Ley condemned it. ECAJ co-CEO Alex Ryvchin called it despicable. Victoria Police and the AFP investigated whether it constituted a terrorism offence.",
  // Full account in plain prose. No markdown formatting inside this string.
  // Include: what happened, who was targeted, who was responsible (if known),
  // official responses, legal outcomes if any.
  // Use Australian English spelling.

  "slogan_recorded": true,
  // Boolean. Was "Free Palestine" (or a direct variant) explicitly documented
  // in the incident — in a police report, news article, court transcript,
  // witness account, or recorded footage? Set false if it is inferred or assumed.

  "slogan_verbatim": "Free Palestine; Oct 7, do it again",
  // The exact words as documented, or null if not precisely recorded.
  // Semicolons separate multiple phrases from the same incident.

  "accompanied_by": ["celebration_of_violence", "pro_hamas_content"],
  // Array of strings from the accompaniment enum — see Section 3.3.
  // Records what the slogan was paired with in this incident.

  "target": "property",
  // Who or what was the direct target of harm.
  // Enum: "individual_jew", "jewish_children", "jewish_institution",
  //       "jewish_business", "property", "jewish_community_event", "online"

  "sources": [
    {
      "name": "ABC News",
      "url": "https://www.abc.net.au/news/2025-10-07/october-7-commemorations-marred-by-antisemitic-graffiti/105860494",
      "type": "news"
    },
    {
      "name": "Canberra Times / AAP",
      "url": "https://www.canberratimes.com.au/story/9082765/investigation-into-terrorist-propaganda-graffiti/",
      "type": "news"
    }
  ],
  // Array of source objects. At least one required.
  // type enum: "news", "court_record", "police_statement", "ngo_report",
  //            "government_statement", "social_media", "advocacy"
  // Prefer primary sources (news, court, police) over advocacy where both exist.
  // Always include URL where available. Do not fabricate URLs.

  "legal_outcome": null,
  // String or null. Describes charges, convictions, sentences, or "under investigation".
  // E.g. "Kye Pickering charged 26 Jun 2025 with property damage, display of Nazi symbol, participating in criminal group."

  "caveat": null,
  // String or null. Populate when there is a specific, material reliability concern
  // the reader should know about — a disputed claim, contested attribution,
  // a subsequent police finding that complicates the account, etc.
  // Do not use for routine single-source incidents; credibility comes from
  // choosing reputable sources, and readers can draw their own judgements.
  // E.g. "The Allawah synagogue attack was later alleged by NSW Police to have
  // been staged by an organised-crime network, complicating grassroots motive attribution."

  "tags": ["october_7_anniversary", "melbourne", "investigation_terrorism"],
  // Free-form array of lowercase_underscore strings for cross-cutting concerns.
  // Use sparingly; prefer structured fields above.

  // AUDIT FIELDS — populated by the update process, not manually

  "added": "2026-08-23",
  // ISO date this entry was added to the dataset.

  "added_by": "research-session-2026-08-23",
  // Free string identifying the session or contributor.

  "last_updated": "2026-08-23"
  // ISO date of most recent edit to this entry.
}
```

### 3.2 Category enum

| Value | Meaning |
|---|---|
| `verbal_abuse` | Slogan shouted or directed at a Jewish person in a public or semi-public space |
| `graffiti_vandalism` | Slogan written on a building, vehicle, or object alongside antisemitic content |
| `physical_assault` | Slogan used immediately before, during, or after a physical attack |
| `protest_intimidation` | Slogan used in a protest context to intimidate or threaten Jews or Jewish institutions |
| `school` | Incident involving Jewish schoolchildren or occurring at or near a school |
| `workplace` | Incident directed at a Jewish person in a workplace context |
| `social_media` | Slogan used alongside antisemitic content in online posts |
| `criminal_conviction` | Incident that resulted in a criminal charge or conviction (may overlap with above) |
| `property_damage` | Slogan used alongside damage to Jewish property (distinct from graffiti alone) |

### 3.3 Accompaniment enum (for `accompanied_by` field)

| Value | Meaning |
|---|---|
| `death_threat` | Explicit threat to kill |
| `slurs` | Racial or ethnic slurs directed at Jews |
| `nazi_imagery` | Swastikas, Hitler references, Nazi salutes |
| `celebration_of_violence` | Explicit endorsement or celebration of attacks on Jews (e.g. "Oct 7 do it again") |
| `pro_hamas_content` | Explicit support for Hamas as an organisation |
| `physical_violence` | Accompanied an act of physical violence |
| `intimidation_of_children` | Directed at or in the presence of Jewish children |
| `religious_disruption` | Disrupted a Jewish religious service or observance |
| `doxxing` | Accompanied publication of identifying information |

---

## 4. Inclusion criteria

An incident **must** meet all of:

1. **Slogan nexus.** The words "Free Palestine" (or a close variant: "free, free Palestine", "Palestine libre", etc.) were explicitly used *in the same act* as the abuse, threat, or incitement. The slogan must be documented, not inferred.
2. **Antisemitic character.** The act was directed at Jews *as Jews* — not at Israeli government policy in the abstract. The test: would the act have occurred if the target were not Jewish?
3. **Documented by at least one credible source.** A news outlet, police statement, court record, or NGO incident report (ECAJ, ADL, CST, RIAS, etc.) must record the event. Unverified social media posts alone do not qualify; they may be cited as a corroborating source but not as the sole basis for inclusion.

The dataset has no hard date boundary and no geographic restriction. The initial seed covers Australian incidents, but the schema and skill are designed for global coverage from the outset.

An incident **should not** be included if:

- The "Free Palestine" slogan appeared at a protest that was otherwise non-threatening, even if the broader protest is controversial.
- The incident involves anti-Israel sentiment directed at Israeli government or military targets rather than Jews.
- The sole source is an unverified social media post or an outlet with a documented pattern of fabrication. Where motive or facts are genuinely disputed, populate the `caveat` field.

---

## 5. Skill definition (`skills/SKILL.md`)

```markdown
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

Draft each entry in the schema defined in spec.md. Key discipline:

- `id`: Generate as `{country}-{city}-{YYYY-MM-DD}-{2-3 word label}`, all
  lowercase, hyphens only. Check for collisions with existing IDs.
- `summary`: One sentence, active voice, factual, no editorialising.
- `detail`: Plain prose. Australian English spelling. No markdown inside the
  string. Include all material facts. Attribute quotes to their speaker.
- `slogan_verbatim`: Record exact words if documented; otherwise null.
- `sources`: At minimum one entry. Do NOT fabricate URLs. If you found an
  article but cannot confirm the URL, include the name and note URL as null
  with a comment.
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
```

---

## 6. Static web app spec (`index.html`)

A **single HTML file** with embedded CSS and JS. No external framework. No build step. Served from the same directory as the spec; loads `data/incidents.json` via `fetch()`.

### 6.1 Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  HEADER — title, one-line purpose, key stats (total incidents,  │
│           countries covered, date range)                        │
├──────────────────┬──────────────────────────────────────────────┤
│  FILTER PANEL    │  INCIDENT LIST                               │
│  (left sidebar   │                                              │
│  or top bar on   │  One card per incident:                      │
│  mobile)         │  - Date · Location badge · Category badge   │
│                  │  - Summary (always visible)                  │
│  • Country       │  - [Expand] → Detail + Sources + Caveat     │
│  • Category      │                                              │
│  • Accompaniment │  Sorted by date desc by default.            │
│  • Has caveat    │  Showing N of M total.                       │
│  • Free text     │                                              │
│    search        │                                              │
│                  │                                              │
│  [Export JSON]   │                                              │
└──────────────────┴──────────────────────────────────────────────┘
```

On mobile: filter panel collapses to a top bar with a "Filters" toggle button.

### 6.2 Interaction model

**Filtering:** All filters are additive (AND logic). Changing any filter immediately updates the displayed list. No page reload.

**Search:** Free-text search across `summary`, `detail`, `location`, `slogan_verbatim`. Case-insensitive, partial match.

**Expand/collapse:** Each card expands in place to show `detail`, `sources` (linked), `legal_outcome`, and `caveat` if present. Collapse on second click.

**Export JSON:** Downloads the full JSON objects of all currently filtered incidents.

### 6.3 Display rules

- If `caveat` is non-null: show a ⚠️ indicator on the collapsed card; display the caveat prominently inside the expanded card.
- `slogan_verbatim` if present: display in a styled blockquote inside the expanded card.
- Sources: render as labelled links (`<a href="..." target="_blank">`). If URL is null, render name only (no broken link).
- Dates: display `date_display` everywhere. Show `date_note` in muted styling directly beneath if present.
- `accompanied_by` values: render as small coloured badges (distinct colour per value).
- `legal_outcome` if non-null: render in a distinct highlighted block inside the expanded card.

### 6.4 Performance constraints

- Must load and render in under 2 seconds on a standard connection with up to 1,000 incidents.
- All filtering and search is done in-memory after initial fetch.
- No pagination required at this scale; use CSS `contain` and a virtual list only if the dataset grows beyond ~500 visible cards.

### 6.5 What to avoid

- No external CSS frameworks. Use a minimal hand-rolled stylesheet.
- No CDN dependencies.
- No localStorage, no cookies, no analytics.
- No service worker or PWA boilerplate.

### 6.6 Accessibility baseline

- Keyboard navigable (expand/collapse via Enter/Space).
- Focus visible on all interactive elements.
- `aria-expanded` on card toggle buttons.
- Colour is not the sole differentiator for any status (category, accompaniment type).
- Passes basic WCAG 2.1 AA contrast requirements.

---

## 7. Hosting

GitHub Pages or any static host. The repository root contains `index.html`; the data file is at `data/incidents.json` relative to the root. No server-side rendering, no serverless functions, no database.

Suggested update workflow:
1. Run the SKILL.md update process in Claude Code or Claude Desktop.
2. Review and approve new entries.
3. Commit the updated `data/incidents.json` to the repository.
4. The hosting platform redeploys automatically on push.

---

## 8. Open question for sign-off

**Qualitative framing on the page.** The page needs a short contextual note explaining the dataset's scope. Working draft — confirm final wording before build:

> *"This dataset records incidents in which the slogan 'Free Palestine' was explicitly used as a vehicle for antisemitic abuse, threats, or celebration of violence against Jews. It does not claim the slogan is inherently antisemitic. For context on how the slogan is interpreted and debated, see the [ADL backgrounder](https://www.adl.org/resources/backgrounder/slogan-free-palestine) and the [AJC explainer](https://www.ajc.org/news/what-does-free-palestine-really-mean). This record exists because those who mean it peacefully have reason to distinguish themselves clearly – for example, by saying 'Free Palestine. No more war! Peace for all!'"*

---

## 9. Suggested first tasks after spec sign-off

1. Confirm qualitative framing wording (see Section 8).
2. Write `incident.schema.json` for validation.
3. Produce `data/incidents.json` seeded with the Australian incidents already researched, formatted to this schema, with `meta` block populated.
4. Build `index.html`.
5. Extract `skills/SKILL.md` from this spec as a standalone file.
6. Test manual-mode workflow end-to-end with one new incident provided by URL.
7. Test search-mode workflow with one research pass.
8. Deploy to GitHub Pages.
