# Sources — CD3 pilot: Million Women Study recruitment variables

## Dictionary files

- `millionwomenstudydatadictionary-v1-21.xlsx` (copied to `CD3_Schemify_Pilot/MWS/raw_data/`, v1.21, dated 05/11/2024) · Excel, 6 sheets (`Basic information`, `Recruitment variables`, `Self-reported health at recruit`, `3-year resurvey`, `Dietary data at 3-year resurvey`, `8-year resurvey`); only `Recruitment variables` read for this pilot. Parse notes: title/version on rows 1-2, sheet label row 3, real header on row 4 (`Field Name` split into `Short`/`Long` sub-columns on row 6, `Description`, `Valid range` split into `From`/`To` on row 6, `Availability for questionnaire versions:` split into `Aqua`/`Blue`/`Other` on row 5, with version-year ranges on row 7 and numeric availability fractions on row 6 under each availability sub-column, `Code list`), data rows 8-45 (38 variables, one row per variable). Code lists are packed into a single cell as newline-separated `code=label` pairs (e.g. `1=Tertiary (college or university)\n2=Secondary...`), not one row per code — contrast with the BGS/OFH-style sibling packages, which use one row per code. Extracted via direct XML parsing of the xlsx zip (no Python/openpyxl available on this machine at time of registration). Registered 2026-09-15.

## External sources

- None consulted yet.

## The steward

- (to be filled from the intake interview)
