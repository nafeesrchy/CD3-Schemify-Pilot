# Sources — CD3 pilot: BGS (Breast Cancer Now Generations Study) derived variables

## Dictionary files

- `DerivedVariables_Schema.json` (copied to `CD3_Schemify_Pilot/BGS/raw_data/`, originally `Schema_and_Derivation_Utils/Questionnaire/R0/schemas/derived/DerivedVariables_Schema.json`) · JSON Schema, draft 2020-12 · 60 properties (`TCode` + 59 `R0_*` derived variables) describing one participant's R0 (baseline) analysis-ready derived variables. Each property carries `description`, most carry `x-description` (derivation logic in prose), `x-derivedFrom` (pointers into sibling pseudo-anonymised schemas, or for the 10 `R0_FamHist*` properties, legacy SAS script paths under `N:/BrBreakthrough/...`), `x-calculation` (formula, where applicable), `x-formerName` (legacy variable name). The file's own `$defs` are unresolved `$ref`s to sibling schemas (`AdminEvents`, `GeneralInformation`, `PhysicalDevelopment`, `Pregnancies`, `ContraceptiveHRT`, `BreastDisease`, `CancerRelatives`, `MH_Illnesses`, `AlcoholSmokingDiet`, `MenstrualMenopause`, `PhysicalActivity`) that are not present in this package — informational only (they document lineage), not resolvable or needed for conversion. Missingness in the source is inconsistent: most fields type-permit bare `null`; a `9999` "Not Applicable" const branch appears on 12 fields where a derivation can be structurally inapplicable (e.g. too young, never pregnant); no field carries an explicit source-documented generic-missing code. Registered 2026-09-15.

## External sources

- None consulted yet.

## The steward

- (to be filled from the intake interview)
