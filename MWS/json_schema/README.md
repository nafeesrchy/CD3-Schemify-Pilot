# MWS — Million Women Study baseline recruitment package

## 1. What this package describes

The Million Women Study (MWS) is a UK cohort study recruiting women through the NHS Breast Screening Programme from the mid-1990s onward, run by the Cancer Epidemiology Unit (CEU), University of Oxford. This package covers baseline data collected at recruitment.

**Grain**: one element is one participant's baseline recruitment-questionnaire record.

| Table | Mother file | Row grain | Categories |
|---|---|---|---|
| `recruitment` | `recruitment/recruitment.schema.json` | One participant's baseline record | 8 (below) |

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

65 variables total, drawn from three sheets of the source dictionary (`Basic information`, `Recruitment variables`, `Self-reported health at recruit`). Three later-wave sheets (3-year resurvey, dietary data at 3-year resurvey, 8-year resurvey) are out of scope — a different grain (follow-up wave, not baseline).

Category names and groupings deliberately match the CD3 target schema's own seven domains (Demographics, Socioeconomic, Anthropometric, Behavioral, Reproductive/Hormonal, Medical History, Screening) rather than an arbitrary local scheme, to make the later cross-cohort `/harmonize` mapping more direct.

## 2. Layout and composition

```
json_schema/
├── README.md                         ← this file
├── VARIABLES.csv                     ← inventory; coverage ground truth (kept forever)
├── common/defs.json                  ← shared sentinels
├── recruitment/
│   ├── recruitment.schema.json       ← mother file: array of row objects + all routing rules
│   └── categories/*.json             ← 8 category files
├── examples/                         ← toy fixtures (67 valid rows, 41 invalid rows + ledger)
├── tools/                            ← validator + requirements.txt
├── dictionary.html                   ← searchable data dictionary
├── playground.html                   ← toy-data viewer + live validator
└── assets/                           ← vendored rendering library
```

A row is the `allOf` union of its 8 category files, plus 11 routing conditionals (section 5). No category file closes itself with `additionalProperties` — the mother file's single `unevaluatedProperties: false` is the only place unknown columns are rejected, so every category can coexist under `allOf`.

## 3. Value-encoding conventions

- **Codes get `oneOf`**: a categorical variable — Yes/No, a labeled scale — is a `oneOf` of single-value `const` branches, each carrying its own `title` (the label). Never a bare `enum`.
- **Measures get `anyOf`**: a continuous or counted variable is an `anyOf` of one numeric branch (with plausibility bounds) plus one branch per sentinel that applies to it.
- **Bounds are plausibility bounds**, not the tightest bound that fits the data: they reject the impossible, not the merely rare. The source's own stated range wins whenever it states one; where it doesn't, a bound was chosen and logged as an explicit decision (`DECISIONS.md`, now folded into this file's history).
- **Title separator**: em dash (—). **Formatting**: 2-space indent, one key per line.
- **`$id` base**: `https://schemas.example.org/cd3-mws-pilot/` — a **placeholder**, not a real namespace. Replace it before publishing these schemas anywhere public; nothing dereferences it, it exists only so `$ref` resolution is unambiguous.

## 4. Sentinel semantics

| Code | Meaning | Where it applies |
|---|---|---|
| `-1` | Not on questionnaire | Adopted verbatim from the source's own code lists, wherever the source documents it for a specific field (varies by questionnaire version — Aqua/Blue/Other). |
| `-999` | Missing | Generic fallback for any field where a value could be absent for an unexplained reason. Applied package-wide, including *alongside* `-1` on fields that also have it — the two are independent: `-1` explains one specific reason a value is absent, `-999` covers everything else. |
| `-777` | Not applicable | A value is absent because routing puts the participant out of that field's universe (e.g. age at hysterectomy, when no hysterectomy was ever had). Always enforced by a real skip/applicability conditional pair in the mother file, never just asserted. |

A field may carry more than one of these simultaneously (non-overlapping) when more than one of these situations can genuinely occur for it.

## 5. Enforced routing rules

Every rule below is a real `if`/`then` pair in `recruitment.schema.json`'s `allOf`, checked by the validator — not documentation of intent.

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

## 6. Documented but not enforced

JSON Schema cannot compare one column's value against another's, so these real rules are recorded here and in the relevant category file's `$comment`, but nothing rejects a row that violates them:

- `RAGELB` (age at last birth) should be ≥ `RAGEFB` (age at first birth), when both are substantive.
- `ROCSTOP` (age stopped oral contraceptives) should be ≥ `ROCSTART`.
- `RHRTSTO` (age last used HRT) should be ≥ `RHRTSTA` (age started HRT).
- `RDEP5` (deprivation quintile) and `RDEP3` (deprivation tertile) are both derived from the same Townsend index and should be mutually consistent (e.g. `RDEP5` ∈ {1,2} implies `RDEP3=1`).

## 7. Known source issues handled

None identified in this pilot. The source dictionary's own sentinel codes, availability columns, and stated ranges were internally consistent everywhere they were checked.

## 8. Sources and provenance

- **Dictionary**: `millionwomenstudydatadictionary-v1-21.xlsx` (v1.21, dated 05/11/2024) — Excel, 6 sheets; this package reads `Basic information`, `Recruitment variables`, and `Self-reported health at recruit` (65 variables). Sourced from the CEU Oxford data dictionary; not redistributed in this repository (see `.gitignore` — raw dictionary files stay local).
- **External source consulted**: CEU Oxford, "Data access and sharing" — `https://www.ceu.ox.ac.uk/research/the-million-women-study/for-researchers/data-access-and-sharing` — confirms this dictionary is CEU's own published copy, and that any individual-level data (not the dictionary itself) requires a formal Data Access Policy application with identifiers removed. Consulted 2026-09-15.
- No real participant-level data was used anywhere in producing this package — every fixture row is authored from the dictionary's own stated ranges, code lists, and the steward's confirmations.

## 9. Validating and browsing

```
pip install -r tools/requirements.txt      # or use: uv run tools/validate.py …
python3 tools/validate.py summary .        # schemas, fixtures, coverage — everything
python3 tools/validate.py data . --file your_export.csv
```

Double-click `dictionary.html` to browse the data dictionary — keyword search is built in; the Semantic search switch fetches a small model once, then also finds related variables by meaning. Run `python3 -m http.server 8000` from this directory and open `playground.html` to try schemas against the toy fixtures live.

The `$id` namespace (`https://schemas.example.org/cd3-mws-pilot/`) is a placeholder — replace it before publishing these schemas anywhere public.

## 10. Open items to confirm with the data provider

- **No participant-identifier field.** Nothing in the source dictionary's `Basic information`, `Recruitment variables`, or `Self-reported health at recruit` sheets identifies a participant-ID column. The mother file asserts `uniqueItems: true` as standing policy, but it has no real key to anchor to, and standard JSON Schema can't enforce cross-row identifier uniqueness regardless. Resolving this needs contact with the MWS study team to find out what they use as a participant identifier in actual data extracts.
