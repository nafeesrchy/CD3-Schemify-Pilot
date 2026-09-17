# MWS — Million Women Study data package

## 1. What this package describes

The Million Women Study (MWS) is a UK cohort study recruiting women through the NHS Breast Screening Programme from the mid-1990s onward, run by the Cancer Epidemiology Unit (CEU), University of Oxford. This package covers all three waves in the source dictionary: the baseline recruitment questionnaire, the 3-year ("yellow") follow-up resurvey, and the 8-year ("lilac") follow-up resurvey.

| Table | Mother file | Row grain | Categories | Vars |
|---|---|---|---|---|
| `recruitment` | `recruitment/recruitment.schema.json` | One participant's baseline record | 8 (below) | 65 |
| `resurvey_3yr` | `resurvey_3yr/resurvey_3yr.schema.json` | One participant's 3-year follow-up record | 12 (below) | 149 |
| `resurvey_8yr` | `resurvey_8yr/resurvey_8yr.schema.json` | One participant's 8-year follow-up record | 7 (below) | 38 |

### `recruitment` categories

| # | Category | File | Vars |
|---|---|---|---|
| 1 | Demographics | `categories/demographics.json` | 5 |
| 2 | Socioeconomic | `categories/socioeconomic.json` | 3 |
| 3 | Anthropometric | `categories/anthropometric.json` | 3 |
| 4 | Behavioral | `categories/behavioral.json` | 5 |
| 5 | Reproductive/Hormonal | `categories/reproductive_hormonal.json` | 21 |
| 6 | Breast Health | `categories/breast_health.json` | 3 |
| 7 | Gynae Surgery | `categories/gynae_surgery.json` | 6 |
| 8 | Medical History | `categories/medical_history.json` | 19 |

Drawn from three sheets of the source dictionary (`Basic information`, `Recruitment variables`, `Self-reported health at recruit`).

### `resurvey_3yr` categories

| # | Category | File | Vars |
|---|---|---|---|
| 1 | Demographics | `categories/demographics.json` | 2 |
| 2 | Medical History | `categories/medical_history.json` | 21 |
| 3 | Reproductive/Hormonal | `categories/reproductive_hormonal.json` | 7 |
| 4 | Medications | `categories/medications.json` | 20 |
| 5 | Behavioral | `categories/behavioral.json` | 9 |
| 6 | Anthropometric | `categories/anthropometric.json` | 7 |
| 7 | Early Life | `categories/early_life.json` | 8 |
| 8 | Parental Mortality | `categories/parental_mortality.json` | 12 |
| 9 | Family History | `categories/family_history.json` | 27 |
| 10 | Socioeconomic | `categories/socioeconomic.json` | 15 |
| 11 | Wellbeing/Sleep | `categories/wellbeing_sleep.json` | 7 |
| 12 | Dietary | `categories/dietary.json` | 14 |

Drawn from two sheets of the source dictionary (`3-year resurvey`, `Dietary data at 3-year resurvey`), merged into one table since both describe the same wave and grain.

### `resurvey_8yr` categories

| # | Category | File | Vars |
|---|---|---|---|
| 1 | Demographics | `categories/demographics.json` | 2 |
| 2 | Medical History | `categories/medical_history.json` | 16 |
| 3 | Gynae Surgery | `categories/gynae_surgery.json` | 2 |
| 4 | Reproductive/Hormonal | `categories/reproductive_hormonal.json` | 4 |
| 5 | Behavioral | `categories/behavioral.json` | 9 |
| 6 | Anthropometric | `categories/anthropometric.json` | 2 |
| 7 | Family History | `categories/family_history.json` | 3 |

Drawn from a single sheet (`8-year resurvey`) — no separate dietary sheet exists for this wave. Noticeably smaller than `resurvey_3yr`: it drops Medications, Early Life, Parental Mortality, Socioeconomic, Wellbeing/Sleep, and Dietary entirely, and shrinks both Gynae Surgery (to hysterectomy only) and Family History (to breast-cancer-only across mother/father, plus a sister field not asked about in either other table).

All 6 sheets in the source dictionary are now in scope, across the three tables above.

Category names and groupings in `recruitment` deliberately match the CD3 target schema's own seven domains (Demographics, Socioeconomic, Anthropometric, Behavioral, Reproductive/Hormonal, Medical History, Screening) rather than an arbitrary local scheme, to make the later cross-cohort `/harmonize` mapping more direct. `resurvey_3yr` and `resurvey_8yr` reuse the same domain names where a category matches, and each introduces its own where its questionnaire asks about things the others didn't (`resurvey_3yr`: Medications, Early Life, Parental Mortality, Family History, Wellbeing/Sleep; `resurvey_8yr` reuses names throughout but with much narrower category contents) — category sets are not required to match across tables.

## 2. Layout and composition

```
json_schema/
├── README.md                                  ← this file
├── VARIABLES.csv                              ← inventory; coverage ground truth (kept forever)
├── common/defs.json                           ← shared sentinels
├── recruitment/
│   ├── recruitment.schema.json                ← mother file: array of row objects + routing rules
│   └── categories/*.json                      ← 8 category files
├── resurvey_3yr/
│   ├── resurvey_3yr.schema.json                ← mother file: array of row objects + routing rules
│   └── categories/*.json                      ← 12 category files
├── resurvey_8yr/
│   ├── resurvey_8yr.schema.json                ← mother file: array of row objects + routing rules
│   └── categories/*.json                      ← 7 category files
├── examples/
│   ├── recruitment/                           ← toy fixtures (67 valid, 41 invalid + ledger)
│   ├── resurvey_3yr/                          ← toy fixtures (12 valid, 93 invalid + ledger)
│   └── resurvey_8yr/                          ← toy fixtures (9 valid, 40 invalid + ledger)
├── tools/                                     ← validator + requirements.txt
├── dictionary.html                            ← searchable data dictionary, all three tables
├── playground-recruitment.html                ← toy-data viewer + live validator, recruitment
├── playground-resurvey_3yr.html                ← toy-data viewer + live validator, resurvey_3yr
├── playground-resurvey_8yr.html                ← toy-data viewer + live validator, resurvey_8yr
└── assets/                                    ← vendored rendering library
```

A row in any table is the `allOf` union of that table's category files, plus that table's routing conditionals (section 5). No category file closes itself with `additionalProperties` — each mother file's single `unevaluatedProperties: false` is the only place unknown columns are rejected, so every category in a table can coexist under that table's `allOf`. All three tables share `common/defs.json`'s sentinels but are otherwise independent schemas — there is currently no field linking a row in one table back to the same participant's row in another (see section 10).

## 3. Value-encoding conventions

- **Codes get `oneOf`**: a categorical variable — Yes/No, a labeled scale — is a `oneOf` of single-value `const` branches, each carrying its own `title` (the label). Never a bare `enum`.
- **Measures get `anyOf`**: a continuous or counted variable is an `anyOf` of one numeric branch (with plausibility bounds) plus one branch per sentinel that applies to it. A handful of fields mix both — a numeric range alongside the source's own labeled special codes (e.g. `F1MENOAG`'s 77/99, `F1ALCNUM`'s 97/99) — those use `oneOf`, since the branches are mutually exclusive by construction.
- **Bounds are plausibility bounds**, not the tightest bound that fits the data: they reject the impossible, not the merely rare. The source's own stated range wins whenever it states one; where it doesn't, a bound was chosen and logged as an explicit decision (`DECISIONS.md`, kept as the full working record) — and reused across tables for the same measurement type wherever one already exists, rather than re-derived from scratch each time (e.g. `F2WGT`/`F2BMI` reuse bounds already confirmed for `resurvey_3yr`'s and `recruitment`'s equivalent fields).
- **Title separator**: em dash (—). **Formatting**: 2-space indent, one key per line. Each table's category titles carry a distinct prefix ("MWS Recruitment — {category}", "MWS 3-year resurvey — {category}", "MWS 8-year resurvey — {category}") so a category list spanning multiple tables (e.g. three different "Demographics") is never ambiguous.
- **`$id` base**: `https://schemas.example.org/cd3-mws-pilot/` — a **placeholder**, not a real namespace. Replace it before publishing these schemas anywhere public; nothing dereferences it, it exists only so `$ref` resolution is unambiguous.

## 4. Sentinel semantics

| Code | Meaning | Where it applies |
|---|---|---|
| `-1` | Not on questionnaire | Adopted verbatim from the source's own code lists, wherever the source documents it for a specific field. Each table has its own questionnaire-version scheme: `recruitment` uses Aqua/Blue/Other; `resurvey_3yr` uses 4 numeric codes (9903/9907/9909/0012, tracked by `F1QTYPE`); `resurvey_8yr` uses only 2 (0512/0605, tracked by `F2QTYPE`), and uniquely, `F2QTYPE`'s own code values literally *are* the version tags rather than small ordinals — a third, distinct convention. Some fields carry `-1` even though the source's own availability columns mark them available on every version (e.g. `resurvey_3yr`'s `F1MDOLD`) — kept exactly as the source coded it, most likely a source documentation inconsistency. Conversely, some fields genuinely unavailable on early versions document no `-1` at all (e.g. `resurvey_3yr`'s `F1MHTCM`, and 4 mother fields in its Family History category) — also kept as given, not invented. |
| `-999` | Missing | Generic fallback for any field where a value could be absent for an unexplained reason. Applied package-wide, including *alongside* `-1` on fields that also have it — the two are independent: `-1` explains one specific reason a value is absent, `-999` covers everything else. |
| `-777` | Not applicable | A value is absent because routing puts the participant out of that field's universe (e.g. age at hysterectomy, when no hysterectomy was ever had). Always enforced by a real skip/applicability conditional pair in the mother file, never just asserted — except where a checkbox-style trigger field has no explicit "No" code to key an if/then off of (see `resurvey_8yr`'s `F2HYST`/`F2HYSTAG`, section 5), where it is included as a legal value but deliberately left unenforced by steward's choice. |

A field may carry more than one of these simultaneously (non-overlapping) when more than one of these situations can genuinely occur for it. Many fields across `resurvey_3yr` and `resurvey_8yr` are checkbox-style ("tick all that apply" or "tick if this applies," e.g. `resurvey_3yr`'s 19 medication fields and 27 family-history fields, `resurvey_8yr`'s 15 illness/operation fields): the source documents only `1=Yes` for these, with no `2=No` — an unchecked box that is genuinely in-universe falls to `-999`, and this residual ambiguity (confirmed-not-this vs. genuinely-unrecorded) is not resolvable from the schema alone; see section 6.

## 5. Enforced routing rules

Every rule below is a real `if`/`then` pair in the relevant mother file's `allOf`, checked by the validator — not documentation of intent.

### `recruitment.schema.json`

1. **RANYCHLD** (ever had children) gates the whole pregnancy/breastfeeding block: No → `RNUMFTP`, `RAGEFB`, `RAGELB`, `RBFYN`, `RBFNUM`, `RBFDUR` are all `not_applicable`.
2. **RBFYN** (ever breastfed), nested within #1: No → `RBFNUM`, `RBFDUR` are `not_applicable`.
3. **ROC** (ever used oral contraceptives): No → `ROCSTART`, `ROCSTOP`, `ROCDUR` are `not_applicable`.
4. **RHTNPC** (HRT never/past/current): Never → `RHRTSTA`, `RHTDUR`, `RHRTSTO`, `RHRTMP`, `RHTTYPE`, `RHTLU` are all `not_applicable`.
5. **RHTNPC**, narrower: Current (still using, hasn't stopped) → `RHRTSTO` (age last used), `RHTLU` (time since last used) are `not_applicable`, even though the participant has used HRT.
6. **RPSTOPYN** (periods stopped): No or Irregular → `RPSTOPAG` is `not_applicable`.
7. **RHYST** (hysterectomy): No → `RHYSTAG` is `not_applicable`.
8. **ROOPH** (bilateral oophorectomy): No → `ROOPHAGE` is `not_applicable`. (`ROOPH=3`, "Not sure", deliberately does **not** trigger this — left unconstrained, an explicit steward call.)
9. **RSTER** (sterilised): No → `RSTERAGE` is `not_applicable`.

Each rule above is paired with its applicability half (in-universe ⇒ the field must carry a real value or an item-missing code, never `not_applicable`) — both halves are separate conditionals in the mother file, both fixture-tested.

`RCIGSPD` (cigarettes per day) was investigated for a similar rule but does **not** have one: the real question is asked independent of smoking status, and a non-smoker answers the field's own `1=None` directly — there is no not-applicable case.

### `resurvey_3yr.schema.json`

1. **F1HTNPC** (HRT never/past/current): Never → `F1HRTSTA`, `F1HTTYPE`, `F1HTDUR`, `F1HTLU` are all `not_applicable`. (`F1HTNPC=4`, "Ever, status not known", deliberately does **not** trigger this — left unconstrained, an explicit steward call, same treatment as `ROOPH=3` above.)
2. **F1HTNPC**, narrower: Current → `F1HTLU` (time since last used) is `not_applicable`, even though the participant has used HRT.
3. **F1SMOKE**: Never → `F1CIGSPD` (cigarettes per day) is `not_applicable`. Unlike `RCIGSPD` above, the real resurvey questionnaire's smoking question is "have you ever been a smoker?" — a Never answer genuinely skips the follow-up, so this **does** need the rule (initially drafted without one by pattern-matching on the identical code list, then corrected).
4. **F1MALIVE** (mother still alive): Yes → `F1MDAGE` and all 7 mother cause-of-death fields (`F1MDHRT`, `F1MDSTR`, `F1MDPNU`, `F1MDOLD`, `F1MDCABR`, `F1MDCAWO`, `F1MDCAOV`) are `not_applicable`; No (deceased) → `F1MAGE` is `not_applicable`. (`F1MALIVE=3`, "Do not know", deliberately does **not** trigger either half — left unconstrained, an explicit steward call.) The 7 cause-of-death fields are not required to carry a real value even when the mother has died — see section 6 for the residual ambiguity this leaves.
5. **F1PART** (currently has a partner): No → `F1PARTSM` (partner smokes) is `not_applicable`.
6. **F1MED** (used any medication, not HRT, in the last 4 weeks): No → all 19 specific medication fields are `not_applicable`. Same residual ambiguity as #4 in the other direction — see section 6.

Each rule above is paired with its applicability half where one exists (rules #4's `F1MAGE`/`F1MDAGE` halves, #1, #2, #3, #5) — both halves are separate conditionals in the mother file, both fixture-tested. Rules #4's cause-of-death half and #6 are single-direction by design (see section 6).

Two relationships were investigated and found to need **no** rule: `F1ALCNUM`/`F1ALCEAT`/`F1ALCDAY` (a non-drinker answers `0` directly, a real value, not a sentinel) and `F1MOBFRQ`/`F1MOBYRS` (same reasoning — never having used a mobile phone is answered as `0` years). The 9 social-activity checkboxes, Family History's 27 mother/father fields, and medical_history's 20 illness/treated-now checkboxes have no gating field at all and were confirmed independent before drafting.

### `resurvey_8yr.schema.json`

1. **F2HTNPC** (HRT never/past/current): Never → `F2HTTYPE`, `F2HTDUR`, `F2HTLU` are all `not_applicable`. (`F2HTNPC=4`, "Ever, status not known", deliberately does **not** trigger this — same treatment as the other two tables' equivalent field.) Note this table has no "age started HRT" field at all, unlike the other two.
2. **F2HTNPC**, narrower: Current → `F2HTLU` (time since last used) is `not_applicable`.
3. **F2ALCNUM** (alcoholic drinks per week): the steward gave the real printed instruction directly — "If you have less than one drink a week, please go to question 60" — so `F2ALCNUM=0` → `F2ALCEAT` (drinks with meals) and `F2ALCDAY` (days per week) are both `not_applicable`. Confirmed *not* to extend to the tea/coffee questions, which are asked of every participant regardless. This is a genuinely different answer from `resurvey_3yr`'s equivalent fields (rule-free, see above) — the two waves' questionnaires route differently, and this was checked directly with the steward rather than assumed either way.
4. **F2SMOKE**: Never → `F2CIGSPD` (cigarettes per day) is `not_applicable`; Past or Current → `F2CIGSPD` must be real. The steward gave the real question wording directly — "About how many cigarettes do you/did you smoke on average each day? (if you are an ex-smoker, how many did you smoke on average when you smoked?)" — explicitly covering **both** current and past smokers, a different shape from both `RCIGSPD` (no skip at all) and `F1CIGSPD` (Never-only skip) above.

Each rule above is paired with its applicability half. `F2HYST`/`F2HYSTAG` (hysterectomy / age at hysterectomy) was investigated and confirmed to match `RHYST`/`RHYSTAG`'s semantic relationship (rule #7 above), but `F2HYST` is checkbox-style with no explicit "No" code to key an enforceable rule off of — the steward explicitly declined to enforce either half rather than risk conflating "no hysterectomy" with "genuinely unrecorded"; `F2HYSTAG` keeps `not_applicable` as a legal value, just not one any conditional pins or requires. Gynae Surgery, Medical History, and Family History in this table have no gating field among their own checkbox items and were confirmed independent before drafting.

## 6. Documented but not enforced

JSON Schema cannot compare one column's value against another's, so these real rules are recorded here and in the relevant category file's `$comment`, but nothing rejects a row that violates them:

- `RAGELB` (age at last birth) should be ≥ `RAGEFB` (age at first birth), when both are substantive.
- `ROCSTOP` (age stopped oral contraceptives) should be ≥ `ROCSTART`.
- `RHRTSTO` (age last used HRT) should be ≥ `RHRTSTA` (age started HRT).
- `RDEP5` (deprivation quintile) and `RDEP3` (deprivation tertile) are both derived from the same Townsend index and should be mutually consistent (e.g. `RDEP5` ∈ {1,2} implies `RDEP3=1`).
- `resurvey_3yr`'s three dietary macronutrients (`F1NPROT`, `F1NTFAT`, `F1NCARB`) roughly sum to the energy total (`F1NKJ`) via standard conversion factors, but the source doesn't document its exact factors or rounding, so no approximate identity is enforced.
- `resurvey_3yr`'s `F1MALIVE`-gated cause-of-death block: a downstream analyst cannot distinguish "all 7 fields blank because the mother is alive" from "all but one blank because she died of an unlisted or unrecorded cause" without cross-referencing `F1MALIVE` directly — the schema pins the *alive* case to `not_applicable`, but an unchecked box within the *deceased* universe still falls to generic `-999`, indistinguishable from a genuine gap. The same residual ambiguity applies to `resurvey_3yr`'s `F1MED`-gated medication block in the other direction.
- `resurvey_8yr`'s `F2HYST`/`F2HYSTAG`: the same semantic relationship as `RHYST`/`RHYSTAG` holds, but is not enforced at all (see section 5) — a downstream analyst should treat `F2HYSTAG` as potentially meaningful even when `F2HYST` is missing, and vice versa, since neither field constrains the other.

## 7. Known source issues handled

- **`resurvey_3yr`'s `-1` sentinel is inconsistently documented across otherwise-similar fields.** Most version-gated fields (unavailable on the earliest two questionnaire versions) carry a source-documented `-1='Not on questionnaire'` code that lines up with their own availability columns — but a few don't, despite the same unavailability (`F1MHTCM` vs. its `F1FHTCM` counterpart; 4 of 13 mother fields in Family History vs. their father counterparts; `F1MDOLD` even more unusually carries `-1` despite being marked available on *every* version). Every one of these was transcribed exactly as the source coded it, asymmetries included, rather than corrected or made consistent — per this pilot's rule against inventing metadata the source doesn't state.
- No other defects, corrected typos, or coding inversions were identified in any of the three tables.

## 8. Sources and provenance

- **Dictionary**: `millionwomenstudydatadictionary-v1-21.xlsx` (v1.21, dated 05/11/2024) — Excel, 6 sheets, all now in scope. `recruitment` reads `Basic information`, `Recruitment variables`, and `Self-reported health at recruit` (65 variables); `resurvey_3yr` reads `3-year resurvey` and `Dietary data at 3-year resurvey` (149 variables); `resurvey_8yr` reads the single `8-year resurvey` sheet (38 variables). Sourced from the CEU Oxford data dictionary; not redistributed in this repository (see `.gitignore` — raw dictionary files stay local).
- **External source consulted**: CEU Oxford, "Data access and sharing" — `https://www.ceu.ox.ac.uk/research/the-million-women-study/for-researchers/data-access-and-sharing` — confirms this dictionary is CEU's own published copy, and that any individual-level data (not the dictionary itself) requires a formal Data Access Policy application with identifiers removed. Consulted 2026-09-15.
- No real participant-level data was used anywhere in producing this package — every fixture row is authored from the dictionary's own stated ranges, code lists, and the steward's confirmations.

## 9. Validating and browsing

```
pip install -r tools/requirements.txt      # or use: uv run tools/validate.py …
python3 tools/validate.py summary .        # schemas, fixtures, coverage — everything
python3 tools/validate.py data . --file your_export.csv
```

Double-click `dictionary.html` to browse the data dictionary — all three tables' categories are in one page, keyword search is built in, and the Semantic search switch fetches a small model once, then also finds related variables by meaning. Run `python3 -m http.server 8000` from this directory and open `playground-recruitment.html`, `playground-resurvey_3yr.html`, or `playground-resurvey_8yr.html` to try that table's schema against its own toy fixtures live.

The `$id` namespace (`https://schemas.example.org/cd3-mws-pilot/`) is a placeholder — replace it before publishing these schemas anywhere public.

## 10. Open items to confirm with the data provider

- **No participant-identifier field, in any of the three tables.** Nothing in any of the six source sheets this package reads identifies a participant-ID column. All three mother files assert `uniqueItems: true` as standing policy, but none has a real key to anchor to, and standard JSON Schema can't enforce cross-row identifier uniqueness regardless. There is also currently no way to link a participant's row in one table to their row in another. Resolving this needs contact with the MWS study team to find out what they use as a participant identifier in actual data extracts.
