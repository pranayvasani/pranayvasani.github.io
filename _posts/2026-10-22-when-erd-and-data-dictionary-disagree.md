---
tags: [AI, Data, Analytics, Healthcare]
---

# When the ERD and the Data Dictionary Disagree

Most data models ship as two documents that nobody checks against each other: an ERD exported to PDF, and a data dictionary in a spreadsheet. I built a way to make them check each other, and to turn the result into something you can click through. Two live examples and the full prompt are below.

## Two documents, one model

The ERD and the dictionary are usually written by different people, at different times, in different tools. Over a few releases they drift. A column gets renamed in the database but keeps its old name in the dictionary. A new table ships without a dictionary entry. A foreign key exists only as a sentence in a description.

None of this is visible while the two sit in separate files. It shows up later, when an analyst joins on the wrong key, an ETL fails on a constraint, or a client reads a definition that no longer matches the data.

## The idea: two witnesses, one record

Treat the diagram and the dictionary as two independent witnesses to the same model. Parse both into one structure, compare every table, column, type and key, and write down each disagreement with the evidence behind it. Then render the reconciled model as an explorer instead of a static picture.

Three rules make it trustworthy:

- **Every join says where it came from.** Documented means a foreign key constraint or an explicit marker. Inferred means a name match, shown dashed, with a confidence level. Nothing is promoted from guess to fact.
- **Count, don't estimate.** Table, column and join counts come from the parsed sources, per source and after the merge.
- **The disagreements are the product.** Every mismatch becomes an issue with a severity, a location and the file it came from, and the explorer has a tab for them.

The explorer itself is one self-contained HTML file. No server, no libraries, no network calls. You can email it, drop it in SharePoint or a GitHub Pages site, or open it on a plane.

## Try it on two public models

I ran the approach on two open healthcare data models, chosen because each publishes two independent descriptions of itself.

| Explorer | What the model is | Sources reconciled | Tables | Joins (documented / inferred) | Issues (high / med / low / info) |
| --- | --- | --- | --- | --- | --- |
| [OMOP CDM 5.4 Explorer](/erd-explorer/omop-cdm-5.4.html) | The OHDSI standard for real-world evidence from claims and EHR data | PostgreSQL DDL vs field-level and table-level dictionary CSVs | 39 | 178 / 12 | 2 / 16 / 50 / 2 |
| [Synthea CSV Explorer](/erd-explorer/synthea.html) | Synthetic patient records from MITRE's Synthea generator | Header lines in the exporter source code vs the wiki data dictionary | 19 | 43 / 2 | 3 / 19 / 10 / 0 |

What to try first:

1. Open the **Query paths** tab and pick "Patient journey" (OMOP) or "Claim line items" (Synthea). The diagram dims everything except that join path and zooms to it.
2. Hover any table, then any line. Lines show the join columns, cardinality, role and the evidence behind them.
3. Switch **Columns** to "Keys only" and **Layout** to "Focus" with 2 hops. Click a table to recentre on it.
4. Open **Data issues**. Click an issue to jump to the table and column it concerns.
5. Drag a few tables. Your arrangement is saved as a Custom layout, and **Export** gives you a PNG, CSVs or a layout file to share.

Each explorer is a single file, so you can also save it and open it offline.

## What reconciliation found

Both models are well maintained and widely used, and both still disagree with themselves. These are real findings from the current public sources, not planted examples.

**OMOP CDM 5.4**

- **A constraint that cannot be created.** VOCABULARY has no primary key and no unique index, yet two foreign keys reference `vocabulary.vocabulary_id`. PostgreSQL rejects a foreign key whose target is not unique, so that part of the shipped constraint script fails.
- **One table dominates the picture.** 118 of the 190 joins point at CONCEPT. Drawn naively, the diagram is a hairball. The explorer collapses those joins into a `→ concept` marker on each column and draws them only when you ask.
- **Joins that depend on another column.** `cost.cost_event_id` points at a drug exposure, a procedure or a visit, depending on `cost_domain_id`. The same pattern appears in `note`, `measurement` and `episode_event`. These are drawn as conditional, inferred joins, never as plain foreign keys.
- **Thinner documentation than it looks.** 104 columns have no user guidance text. Falling back to the ETL conventions field, 45 remain undocumented.

**Synthea**

- **An undocumented file.** The exporter writes `patient_expenses.csv`, and the dictionary has no section for it.
- **Columns added without documentation.** `NPI` was added to both `providers` and `organizations`.
- **Renames that change meaning.** The dictionary still documents `Start_Year` and `End_Year` in `payer_transitions`. The code writes `START_DATE` and `END_DATE`. Anyone reading the dictionary expects a year and gets a date.
- **A join that multiplies rows.** `claims_transactions.PATIENTINSURANCEID` points at `payer_transitions.MEMBERID`, which is not unique. Without a date filter, every transaction repeats once per coverage period.

## How it is built

The build runs in two stages with one intermediate record between them, so the page can never show a number the reconciliation did not produce.

![Build pipeline: physical model and data dictionary feed Stage A (extract and reconcile), which writes model.json; Stage B renders one HTML file, which is tested in a headless browser](/images/erd-build-pipeline.png)

Test failures go back to the render stage. The findings change only when a source changes, which also makes a rerun after each model release cheap: the diff between two model.json files is the release's documentation debt.

## The prompt

The whole build is driven by one prompt that works with any capable coding assistant that can run code and a headless browser. To use it, attach your two sources, fill in the placeholders, and say either "plan first" (it reports counts and mismatches, then waits) or "implement all". You can also [download it as a text file](/erd-explorer/erd-explorer-prompt-v2.txt).

This is version 2. Version 1 looked complete, but running it literally on these two models produced wrong findings. Each fix below became a rule.

| What went wrong with v1 | Effect | Rule added in v2 |
| --- | --- | --- |
| Type matching by prefix: "datetime" matched "date" | 22 false type mismatches in OMOP | Match type families longest prefix first; check the parser before reporting |
| Reserved words are quoted in DDL (`"offset"`) | A column reported missing that exists | Strip quoted identifiers before merging |
| Renamed columns counted on both sides | 5 renames reported as 10 missing columns | Pair one-sided columns by shared name token, flag as probable rename |
| No rule for lookup hubs | CONCEPT's 118 joins turned the diagram into a hairball | Collapse lookup hubs by default, never entity hubs like PERSON |
| Foreign keys described only in prose | Synthea's joins had no target column | A separate evidence tier: documented in prose, target is the referenced table's key |
| Every table on one screen | Unreadable at Fit for 39 tables | Names only above 25 tables, lanes packed to the screen shape |
| Downloads assumed to work | Blocked inside SharePoint, Teams and AI chat viewers | Every export shown in a dialog with Copy as well as Download |

{% raw %}
```text
ROLE
You are a data architect and front-end engineer. Build an interactive, standalone ERD explorer from two independent descriptions of the same data model: a physical model and a data dictionary. Reconcile them, then render the result.

INPUTS
- Physical model: {{MODEL_SOURCE}}. Any of: diagram PDF/image/Visio export, DDL (CREATE TABLE + PK/FK/unique scripts), or code that writes the tables (exporter headers). Source of tables, columns, types, nullability, PK/FK markers, join notes, and visual positions if present.
- Data dictionary: {{DICTIONARY_SOURCE}}. Any of: xlsx/csv, field-level and table-level spec CSVs, markdown/wiki pages. Source of table and column descriptions, types, required flags, and FKs written in prose.
- Optional: {{SUBJECT_AREAS}} (business grouping), {{QUERY_PATHS}} (common join paths), {{AUDIENCE}}, {{LOOKUP_TABLES}} (vocab/reference tables).
- Record the exact version of each source (file date, commit hash, or URL) and show it in the app.

WORKING RULES
1. Plan first. Before building, share: table count per schema and per source, hub tables (split into lookup hubs and entity hubs), how keys are marked in each source, mismatches found, proposed features, suggestions, and up to 3 decision questions. Wait for approval unless I say "implement all".
2. Never invent facts. Every relationship carries a source tier and evidence:
   - Documented: FK constraint, FK marker in the diagram, or dictionary FK field.
   - Documented (prose): dictionary text such as "Foreign key to the Provider". Resolve the target table from the text; the target column is the target's PK. Say so in the evidence.
   - Inferred: name match only, with confidence (high = exact match to a single-column PK; medium = role prefix stripped or column exists in one source only).
   Label table summaries "Derived". Never present a guess as documented.
3. Count, don't estimate. Report exact table, column and relationship counts per source and after merge.
4. Two stages. Stage A extracts both sources into one intermediate model.json (tables, columns, relationships, subject areas, query paths, issues, source versions) and runs the validation checklist against it in code. Stage B renders the HTML from model.json only. Never hand-edit numbers into the HTML.
5. Before reporting any mismatch, check whether it is your parser's fault. Re-read the raw source for that line. A false "missing column" or "type mismatch" is worse than a missed one.
6. Test before sharing. Run the HTML in a headless browser, exercise every control, confirm zero console errors, and review screenshots of each layout, light and dark, desktop and phone width.
7. Keep the response concise. No em dashes.

EXTRACTION
- Diagram PDF: use text extraction in raw order to get table blocks (schema.table header, then "PK/FK name type" lines). Use bounding-box extraction to capture each table header's x/y for the "Original" layout. If no source has coordinates, disable "Original" and say why in the app.
- DDL: parse CREATE TABLE, PRIMARY KEY, FOREIGN KEY and UNIQUE/INDEX scripts. Accept quoted identifiers ("offset", [order], `key`) and strip the quotes before merging; reserved words are often quoted.
- Code headers: concatenate split string literals before splitting on commas.
- Clean the extraction: fix ligature splits ("f i" to "fi"), split headers that run into other text, accept only real SQL types (varchar, char, text, int, bigint, numeric, decimal, float, date, datetime, timestamp, uuid, boolean) so annotation text is not read as columns. Strip HTML and annotations inside cells (e.g. "Start <br/> (Date, prior to v3)").
- Dictionary: one sheet per table, a long sheet, or one markdown section per table. Detect the header row (FIELD NAME / DATA TYPE / DESCRIPTION), the table description, the table-of-contents descriptions, and key markers (icons, Y/N columns, "Primary Key" in text).
- Descriptions: if the main description field is empty, fall back to a secondary field (e.g. ETL conventions) and record which field it came from. Flag a column only when every description field is empty.
- Merge on normalized names: case-, space- and underscore-insensitive ("Patient ID" = PATIENTID = patient_id). Keep columns that exist in only one source and flag them.
- Probable renames: when a table has columns present in only one source on each side, pair those that share a name token (FIPS vs "FIPS County Code", START_DATE vs Start_Year). Report one "Probable rename" issue per pair; never silently merge and never report them as two missing columns.

RELATIONSHIPS
- Store each one as: id, from table, from column(s), to table, to column(s), cardinality (N:1, 1:1, N:M), source tier, confidence, evidence, role, condition, flags.
- Model role-playing joins separately (Billing / Rendering / Referring / Supervising provider, primary / secondary payer, period start/end month, corporate parent, ancestor / descendant).
- Capture conditional joins (e.g. domain_id = 'Gender', CodeType = 'X'), bridge tables (N:M), and self-joins (preceding_visit_id, parent_id).
- Polymorphic keys (event_id plus a discriminator such as domain_id or *_field_concept_id): draw as inferred, conditional joins to named example targets, with the discriminator in the condition. Never draw them as plain FKs.
- Fan-out: flag any join whose target columns are not the full PK or a unique key. If an FK constraint targets a column with no PK or UNIQUE constraint, flag it high: most engines (e.g. PostgreSQL) reject that constraint, so the shipped script fails.
- Hubs: a lookup hub (vocabulary, code, reference table referenced by 15+ tables) is collapsed by default. Show "-> target" on the column row instead of a line; draw the lines when the user turns on "Show hub joins" or selects the referencing table. Never collapse entity hubs (person, patient, encounter): those joins are the model.
- Inference rule: exact match of a column to a single-column PK of another table, or name stem match (PROVIDERID, REFERRINGPROVIDERID -> providers.Id with role "referring"). Exclude generic names (ID, CODE, DESCRIPTION, NAME, TYPE).

DATA ISSUES (auto-generated plus manual review)
Every issue has: id, severity (high / med / low / info), category, table, column, message, evidence (which file, which source). Categories:
- Table or column missing from either source; column count mismatch.
- Type mismatch after normalizing numeric/decimal/float, varchar/char/text, int/integer/bigint. Match type families longest-prefix first ("datetime" must not match "date"). Treat date vs timestamp as a low "precision" note, not as equivalent and not as high.
- Nullability mismatch (required in dictionary vs NULL in DDL), PK flag mismatch, FK in one source only, FK target differs between sources.
- Join key type mismatch between the two sides of a relationship.
- No primary key; fan-out risk; constraint will fail; polymorphic join; undeclared join; isolated table.
- Name mismatches and probable renames.
- Dictionary content errors: shifted descriptions, copy-paste descriptions (identical text on different columns), swapped table descriptions, FK described in prose but the key marker missing, key marker with no resolvable target, "key" contradicted by "non-unique" in the text.

DELIVERABLE: one self-contained HTML file
- No external libraries, fonts, or network calls. Embed model.json as JSON in a script tag (escape "</"). Use vanilla JS and SVG. Include the [hidden] { display:none !important } reset and a full doctype shell so it works by double-click.
- Escape every string taken from the sources before inserting it into the DOM (descriptions can contain HTML).
- Canvas: pan, wheel zoom, pinch zoom, drag tables, Fit, zoom buttons. Draw crow's-foot ends; documented joins solid, inferred joins dashed in a second color. Anchor lines to the joined column row when it is visible. Hide column text below about 40% zoom.
- Hover a table: highlight it, its connected tables, the join lines, and the PK/FK join columns, and dim everything else. Show a tooltip with the summary and connected tables.
- Hover a line: show the join, cardinality, role, condition, confidence, source tier and evidence. Hover a column: show both sources' types, description, joins (capped list), and how many other tables have it.
- Touch: tap replaces hover. On phone width the filters become a drawer and the side panel a bottom sheet.
- Click a table: side panel with description, usage guidance, summary (Derived), PK, "References" and "Referenced by" lists (clickable, grouped by table when one table joins several times), data issues, and a filterable column table. Clicking a column shows its full description, both types, joins, and the other tables that contain it (clickable).
- Column modes: names only, keys only, all columns (truncate very wide tables on the canvas; the panel shows all). Default to names only above 25 tables so the first screen is legible.
- Layouts: Original (diagram positions), By subject area (lanes sized roughly square and packed to the screen's aspect ratio), Auto-arrange (deterministic force layout, ideal edge length scaled to box size, rectangle collision removal, hubs in the centre), Focus (selected table plus 1 or 2 hops; picks the most connected table if none is selected and shows it in the panel), Custom (dragging any table saves to Custom in localStorage, with JSON export/import for sharing). Wrap every storage call in try/catch.
- Filters: schema, subject area, documented/inferred, show hub joins, hide lookup tables, hide tables with no relationships. Show live counts (visible / total) and a status line (tables shown, joins drawn, hub joins collapsed).
- Tabs: Table (an overview with counts and most-connected tables when nothing is selected), Query paths (presets that highlight and zoom to a join path; switch away from Focus first; mark paths that use inferred joins), Data issues (grouped by category, high first, clickable, CSV export, plus the validation checklist results), Help (legend, interactions, sources with versions, caveats).
- Search: tables and columns, keyboard navigation, highlights matches on the canvas, jumps to the result and reveals the column row even if the current column mode hides it. Adjust filters if the target is hidden, and say so.
- Export menu: PNG of the visible diagram (embed the SVG styles with theme colors resolved, add margin for curved lines), relationships CSV, merged columns CSV, data issues CSV, layout JSON, import layout JSON. Show each export in a dialog with Copy and Download, because sandboxed viewers (Teams, SharePoint, Copilot, Claude) often block downloads. Use the host's save API when one exists.
- Light/dark theme (follow the system by default, with a toggle), accessible controls (labels, focus states, keyboard: / search, Esc clear, + - zoom, 0 fit, Enter selects a focused table), legend.

VALIDATION CHECKLIST (run in code in Stage A, re-run in the browser, report the results)
- Diagram and dictionary table sets match, and column counts are reconciled (every difference is an issue).
- Every relationship resolves to existing columns, ids are unique, and every preset path resolves and is connected by its own joins.
- Every table has a subject area.
- No reported mismatch is caused by parsing (spot-check each high issue against the raw source).
- All layouts, filters, search, presets, drag, save/export/import, and exports work with zero JS errors, at desktop and phone width.
- No table overlap in subject-area and auto layouts.
- Screenshots reviewed for overlap, readability at Fit, and tooltip size, in light and dark.
- Test harness pitfalls: at small zoom a click on a header can land on a column row; verify hover targets with elementFromPoint; CSS text-transform changes innerText.

OUTPUT
Share the file, then give 5 to 8 lines covering: counts (tables, columns, documented/inferred relationships, issues by severity), the top 3 real findings, key features, and caveats (inferred joins need validation against data, saved layouts are per browser, summaries are derived, issues compare documents not data).
```
{% endraw %}

## Running it on your own model

Start with "plan first". The plan step alone, with counts per source and the first list of mismatches, is often enough to open a useful conversation with whoever owns the model.

Keep these limits in mind:

- **Issues compare documents, not data.** A finding says the diagram and the dictionary disagree. Whether the database agrees with either is a separate check.
- **Inferred joins need validation.** Dashed lines are hypotheses. Test them with a join count against real data before anyone builds on them.
- **Summaries are derived.** Table summaries are generated from the model's structure and are labelled as such. They are not the source authors' words.
- **Saved layouts live in one browser.** Use Export, then Layout JSON, to share an arrangement with a colleague.
- **Keep client models private.** The output is a single file that carries the whole model. Treat it with the same care as the dictionary it came from.

If you try it on a model of your own, I would like to hear what it found.

## Sources

- [OHDSI CommonDataModel repository](https://github.com/OHDSI/CommonDataModel): v5.4 PostgreSQL DDL and the field-level and table-level CSVs, commit f6d2738a.
- [Synthea CSVConstants.java](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/export/CSVConstants.java): exporter header lines, commit bd8a52ba.
- [Synthea wiki: CSV File Data Dictionary](https://github.com/synthetichealth/synthea/wiki/CSV-File-Data-Dictionary).
