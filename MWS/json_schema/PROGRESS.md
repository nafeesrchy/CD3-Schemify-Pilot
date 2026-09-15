# CD3 pilot — Million Women Study baseline variables — conversion progress

package: CD3_Schemify_Pilot/MWS/json_schema · started: 2026-09-15
grain: One element is one participant's baseline recruitment-questionnaire record (Basic information + Recruitment variables + Self-reported health at recruit sheets)
dictionary: millionwomenstudydatadictionary-v1-21.xlsx (3 of 6 sheets in scope) — full inventory in SOURCES.md

## How to continue

This conversion runs over several sittings — an agent session can hold only so
much at once, so the work is planned in units that each fit one session. Nothing
is lost between sittings: this file is the memory.

To continue at any time: open a fresh agent session in this package's directory
and invoke the skill again. The agent reads this file and proposes the next
unit. You can also ask for anything directly — a specific category, a change,
a question, the final review.

This is a learning/replication pilot alongside sibling BGS and OFH packages
(mirroring, not copying, `mgarciaclosas/cd3-schemify-pilot`'s own work on the
same three cohorts).

## Conventions

- sentinels: `-1` not_on_questionnaire (adopted verbatim from source) · `-999` missing_numeric (agent-decided fallback) · D003
- $id base: https://schemas.example.org/cd3-mws-pilot/ (replace before publishing) · D009
- grain: see header · title separator: — (em dash) · formatting: 2-space, one key per line · D009
- real data: none in repo · D004
- no participant-identifier field exists anywhere in the source dictionary — open item, affects how row uniqueness could ever be enforced · D007

## Categories

| # | category | file | vars | source slice | status | touched |
|---|---|---|---|---|---|---|
| 1 | Demographics | recruitment/categories/demographics.json | 5 | Basic information rows 8,9,10,11,14 | pending | 2026-09-15 |
| 2 | Socioeconomic | recruitment/categories/socioeconomic.json | 3 | Recruitment row 8; Basic information rows 12,13 | pending | 2026-09-15 |
| 3 | Body/Lifestyle | recruitment/categories/body_lifestyle.json | 8 | Recruitment rows 9-16 | pending | 2026-09-15 |
| 4 | Reproductive/Pregnancy | recruitment/categories/reproductive_pregnancy.json | 7 | Recruitment rows 17-23 | pending | 2026-09-15 |
| 5 | Breast Health | recruitment/categories/breast_health.json | 3 | Recruitment rows 24,35; Self-reported health row 8 | pending | 2026-09-15 |
| 6 | Gynae Surgery | recruitment/categories/gynae_surgery.json | 6 | Recruitment rows 25-30 | pending | 2026-09-15 |
| 7 | Contraceptive | recruitment/categories/contraceptive.json | 4 | Recruitment rows 31-34 | pending | 2026-09-15 |
| 8 | HRT | recruitment/categories/hrt.json | 7 | Recruitment rows 36-42 | pending | 2026-09-15 |
| 9 | Menstrual | recruitment/categories/menstrual.json | 3 | Recruitment rows 43-45 | pending | 2026-09-15 |
| 10 | Medical History | recruitment/categories/medical_history.json | 19 | Self-reported health rows 9-27 | pending | 2026-09-15 |

65 of 65 in-scope variables assigned to a category (VARIABLES.csv reconciles). No category files drafted yet.

## Package milestones

- [x] intake: sources registered · grain confirmed · categories confirmed (10, derived from source structure + one steward correction — REDUC moved out of Body/Lifestyle)
- [ ] common/defs.json + mother scaffold validate green — `common/defs.json` is written and structurally valid; the mother file (`recruitment/recruitment.schema.json`) is deliberately NOT yet created. `validate.py check` proved empirically that `allOf` must be non-empty (draft 2020-12 meta-schema) and every listed `$ref` must resolve — so a mother file can only validate green once it references at least one real category file. Per INTAKE.md step 6, ending intake before the first category exists is expected, not a shortfall; the mother file gets created as part of wiring the first category (CONVERT.md step 5)
- [ ] every category confirmed
- [ ] cross-category skip audit (D008 open — RCIGSPD routing)
- [ ] coverage audit 1:1
- [ ] pages current for the whole package
- [ ] review walked · cleanup decided

## Session log

- 2026-09-15 · intake · Registered the MWS dictionary's three in-scope sheets (Basic information, Recruitment variables, Self-reported health at recruit — scope widened mid-session from Recruitment-only on the steward's instruction), surveyed all 65 variables into VARIABLES.csv, ran the interview (grain, sentinel policy adopting sibling pilots' -1/-999 convention, no real data, CEU Oxford external source consulted), proposed and confirmed a 10-category table after the steward corrected REDUC's placement (Body/Lifestyle → Socioeconomic), installed render.py's assets/tools, scaffolded common/defs.json (validates green on its own). Attempted to also pre-scaffold the mother file with all 10 category refs; validate.py check proved this can't validate green with no category files yet (allOf can't be empty, and empty allOf items make the meta-schema check fail; a fully-wired allOf makes every ref unresolved) — removed the premature mother file rather than leave a false-green claim in this file. · next: convert demographics (5 vars, smallest category) — this both drafts the first category file and creates the mother file for the first time, wiring its one real $ref.
