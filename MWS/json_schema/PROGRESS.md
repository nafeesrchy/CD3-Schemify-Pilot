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
| 1 | Demographics | recruitment/categories/demographics.json | 5 | Basic information rows 8,9,10,11,14 | confirmed | 2026-09-16 |
| 2 | Socioeconomic | recruitment/categories/socioeconomic.json | 3 | Recruitment row 8; Basic information rows 12,13 | confirmed | 2026-09-16 |
| 3 | Anthropometric | recruitment/categories/anthropometric.json | 3 | Recruitment rows 9-11 | rendered, pending steward review | 2026-09-16 |
| 4 | Behavioral | recruitment/categories/behavioral.json | 5 | Recruitment rows 12-16 | rendered, pending steward review | 2026-09-16 |
| 5 | Reproductive/Pregnancy | recruitment/categories/reproductive_pregnancy.json | 7 | Recruitment rows 17-23 | pending | 2026-09-15 |
| 6 | Breast Health | recruitment/categories/breast_health.json | 3 | Recruitment rows 24,35; Self-reported health row 8 | pending | 2026-09-15 |
| 7 | Gynae Surgery | recruitment/categories/gynae_surgery.json | 6 | Recruitment rows 25-30 | pending | 2026-09-15 |
| 8 | Contraceptive | recruitment/categories/contraceptive.json | 4 | Recruitment rows 31-34 | pending | 2026-09-15 |
| 9 | HRT | recruitment/categories/hrt.json | 7 | Recruitment rows 36-42 | pending | 2026-09-15 |
| 10 | Menstrual | recruitment/categories/menstrual.json | 3 | Recruitment rows 43-45 | pending | 2026-09-15 |
| 11 | Medical History | recruitment/categories/medical_history.json | 19 | Self-reported health rows 9-27 | pending | 2026-09-15 |

65 of 65 in-scope variables assigned to a category (VARIABLES.csv reconciles). Now 11 categories (Body/Lifestyle split into Anthropometric + Behavioral per D016). No category files drafted for the remaining 9 yet.

## Package milestones

- [x] intake: sources registered · grain confirmed · categories confirmed (10, derived from source structure + one steward correction — REDUC moved out of Body/Lifestyle)
- [x] common/defs.json + mother scaffold validate green — mother file created when Demographics was wired in; `validate.py check` green (3 files, 6 refs resolved)
- [ ] every category confirmed
- [ ] cross-category skip audit (D008 open — RCIGSPD routing)
- [ ] coverage audit 1:1
- [ ] pages current for the whole package
- [ ] review walked · cleanup decided

## Session log

- 2026-09-15 · intake · Registered the MWS dictionary's three in-scope sheets (Basic information, Recruitment variables, Self-reported health at recruit — scope widened mid-session from Recruitment-only on the steward's instruction), surveyed all 65 variables into VARIABLES.csv, ran the interview (grain, sentinel policy adopting sibling pilots' -1/-999 convention, no real data, CEU Oxford external source consulted), proposed and confirmed a 10-category table after the steward corrected REDUC's placement (Body/Lifestyle → Socioeconomic), installed render.py's assets/tools, scaffolded common/defs.json (validates green on its own). Attempted to also pre-scaffold the mother file with all 10 category refs; validate.py check proved this can't validate green with no category files yet (allOf can't be empty, and empty allOf items make the meta-schema check fail; a fully-wired allOf makes every ref unresolved) — removed the premature mother file rather than leave a false-green claim in this file. · next: convert demographics (5 vars, smallest category) — this both drafts the first category file and creates the mother file for the first time, wiring its one real $ref.
- 2026-09-15 · convert demographics · Drafted all 5 properties (RYOB, RYSUBMIT, RAGE, RREGION, RQTYPE), creating the mother file for the first time with its one real $ref. Logged 3 plausibility-bound decisions (D010-D012) — RYOB's upper bound and RYSUBMIT's whole bound are derived from stated source facts, RAGE's upper bound and RYOB's lower bound are pure agent-decided ceilings/floors since the source states none. Applied the D003 sentinel policy (-999) uniformly across all 5 fields. Authored 13 toy_valid rows (every bound, every categorical level, every sentinel) and 8 toy_invalid rows (one per relevant violation kind: range-break x2, unknown-level x2, invented-sentinel, wrong-type/null, missing-key, undeclared-column). Hit and fixed a real Windows bug: render.py's dictionary command crashed with a UnicodeDecodeError reading its own template under Windows' default cp1252 codec — fixed by setting PYTHONUTF8=1, not by editing the script. validate.py summary green (3 files · 6 refs resolve · 13/13 toy_valid pass · 8/8 toy_invalid caught on the right column · coverage 5/65 converted, 0 mismatches). Both pages rendered and spot-checked via a local HTTP server. · next: present demographics to the steward for confirmation; if confirmed, convert socioeconomic (3 vars) next.
- 2026-09-16 · confirm demographics + convert socioeconomic · Steward confirmed Demographics as rendered (D013). Drafted socioeconomic (REDUC, RDEP5, RDEP3) — all three are plain coded fields, no continuous bounds, so no new plausibility-bound decisions. Logged D014 (not-enforceable): RDEP5/RDEP3 both derive from the same Townsend index and should be mutually consistent, which JSON Schema can't check — captured as a category-file $comment. Extended toy_valid to 13 rows (unchanged count, packed new coverage into existing rows) and toy_invalid to 11 rows (added 3 unknown-level cases for the new fields). validate.py summary green: 4 files, 10 refs resolve, 13/13 valid pass, 11/11 invalid caught, coverage 8/65. Both pages rebuilt clean. · next: present socioeconomic to the steward for confirmation; if confirmed, convert body_lifestyle (8 vars) next.
- 2026-09-16 · confirm socioeconomic + re-plan + convert anthropometric/behavioral · Steward confirmed Socioeconomic (D015). Steward re-planned the still-undrafted Body/Lifestyle category into Anthropometric (RHEIGHT, RWGT, RBMI) and Behavioral (RSMOKE, RCIGSPD, RALCNUM, REXANY, REXSTREN) — cleaner split matching the CD3 target schema's own domain names (D016). Drafted both: logged D017 (RHEIGHT/RWGT are `number` not `integer`, since a feet/inches-to-metric conversion can be fractional), D018 (RBMI's 10-80 plausibility bound, source states none), D019 (RALCNUM's 0-200 plausibility bound, source states none). REXANY uses the source's own -1 (not_on_questionnaire) sentinel rather than the generic -999, since the source documents it for this specific field; REXSTREN has no such code so uses -999. Extended fixtures (still 13 valid rows, packed in; invalid grew to 15, +4 new cases). validate.py summary green: 6 files, 20 refs resolve, 13/13 valid pass, 15/15 invalid caught, coverage 16/65. Both pages rebuilt clean. · next: present anthropometric + behavioral to the steward for confirmation; if confirmed, convert reproductive_pregnancy (7 vars) next.
