# Sources — CD3 pilot: Million Women Study recruitment variables

## Dictionary files

- `millionwomenstudydatadictionary-v1-21.xlsx` (copied to `CD3_Schemify_Pilot/MWS/raw_data/`, v1.21, dated 05/11/2024) · Excel, 6 sheets (`Basic information`, `Recruitment variables`, `Self-reported health at recruit`, `3-year resurvey`, `Dietary data at 3-year resurvey`, `8-year resurvey`). This pilot's scope: `Basic information` (7 vars), `Recruitment variables` (38 vars), `Self-reported health at recruit` (20 vars) — 65 variables total, all sharing one grain (one row per participant at baseline recruitment). The three later-wave sheets (`3-year resurvey`, `Dietary data at 3-year resurvey`, `8-year resurvey`) are out of scope by the steward's explicit choice — different grain (follow-up waves, not baseline). Parse notes, all three in-scope sheets share the same layout: title/version on rows 1-2, sheet label row 3, real header on row 4 (`Field Name` split into `Short`/`Long` sub-columns on row 6, `Description`, `Valid range` split into `From`/`To` on row 6, `Availability for questionnaire versions:` split into `Aqua`/`Blue`/`Other` on row 5, with version-year ranges on row 7 and numeric availability fractions on row 6 under each availability sub-column, `Code list`), data starting row 8. Code lists are packed into a single cell as newline-separated `code=label` pairs (e.g. `1=Tertiary (college or university)\n2=Secondary...`), not one row per code — contrast with the BGS/OFH-style sibling packages, which use one row per code. `RAGE` (Basic information) is the one field seen so far with a one-sided valid range (`From=48`, no `To`). Extracted via direct XML parsing of the xlsx zip (no Python/openpyxl available on this machine at time of registration; a real Python + openpyxl/pandas has since been installed, available for later sessions). Registered 2026-09-15.

## External sources

- CEU Oxford — "Data access and sharing" · `https://www.ceu.ox.ac.uk/research/the-million-women-study/for-researchers/data-access-and-sharing` (steward gave a slightly stale URL missing `/for-researchers/`; this is the live redirect target) · confirms this dictionary is the same one CEU publishes for researcher access, and that any individual-level data (not the dictionary itself) requires a formal Data Access Policy application with identifiers removed · steward-approved · consulted 2026-09-15.

## The steward

- Directing this CD3 learning/replication pilot (mirroring, not copying, the sibling `mgarciaclosas/cd3-schemify-pilot` work on the same cohorts). No first-hand knowledge of MWS beyond what the dictionary documents (steward's own words: "none").
- Ask here: nothing pending — steward confirmed grain, scope, real-data ruling, and the external source; the sentinel-code convention (D002-equivalent) is still open, see DECISIONS once scaffolded.
- Not here / stays open: RCIGSPD's implied routing from RSMOKE (no explicit sentinel in source for non-smokers).
