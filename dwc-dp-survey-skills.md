This helps guide LLMs to provide structured mappings to the Darwin Core Data Package for survey and monitoring protocols.

First understand:
1. Darwin Core conceptual model https://dwc.tdwg.org/cm/
2. Darwin Core Data Package (DwC-DP) schemas http://rs.tdwg.org/dwc-dp/1.0/table-schemas/ (raw JSON: https://github.com/tdwg/rs.tdwg.org/tree/master/dwc-dp/1.0/table-schemas)
3. The Event hierarchy guide https://dwc.tdwg.org/hierarchy/ and the isLeastSpecificTargetCategoryQuantityInclusive guide https://dwc.tdwg.org/inclusive/

Second, read all existing mappings from the repository to get a sense of how others are done.

Third, read the protocol's own primary documentation (field instructions, recording forms). Base the mapping on what the protocol actually records, cite it, and flag anything that is assumed rather than documented.

The goal is to provide a structural mapping in DwC-DP that organises the data faithfully in a way that satisfies the intent of the survey.

## Output

Each protocol gets its own folder containing a `README.md` and, where possible, an `example/` DwC-DP package. The README is in markdown and contains:

1. A concise summary of the protocol
2. The structure, with explanation of the choice made if appropriate covering
  a. the event structure
  b. the protocols
  c. the survey targets
  d. assertions, media etc that are relevant to connect
3. Open questions that could change the mapping

Write it as a reference document: state decisions and their reasons, not a first-person narrative. Keep it concise.

## Checks against the schemas

Verify every table and field name against the ratified JSON schemas; do not rely on memory. Common pitfalls:

- `occurrenceStatus` uses `detected` / `notDetected` (not `present` / `absent`).
- Sampling effort and sample size (`samplingEffortValue`, `sampleSizeValue`, …) are on `survey`, not `event`. Duration and time are on `event` (`eventTime`, `eventDurationValue`).
- `event.eventCategory` is required. Use the schema's examples (`survey`, `occurrence`, `context`, …) and put the narrower meaning in `eventType`.
- An Occurrence has a single `surveyTarget_fk`, so SurveyTargets used to classify counts must be mutually exclusive.
- `survey-target.isSurveyTargetFullyReported` is required and is a property of the target row: if completeness differs between surveys, they need separate target rows.

## Modelling guidance

- Model the original field records, not derived products (indices, trends, annual maxima).
- Follow the hierarchy guide: a parent Event must encompass its children in space and time, children do not inherit values, and parent values are not summarised from children. Put each value at the level where it was recorded, and don't invent values (e.g. times) for levels where they weren't.
- Use a stable `locationID` for places that are revisited, and separate Events for each visit.
- Activities that the protocol performs separately (e.g. habitat recording vs. counting) get separate Events, even at the same place.
- Create a Survey only where the protocol supports inferring absence. Optional components with a "did you look?" flag get a Survey only when the answer is yes. Keep the flag as an assertion either way.
- When a protocol classifies counts into disjoint categories that are part of the design (e.g. distance bands), consider one SurveyTarget per category so that non-detection can be inferred per category.
- Use one Occurrence per count at the finest source grain, with `organismQuantity`. Don't create one Occurrence or Organism per individual, and don't create `notDetected` rows the source doesn't contain.
- Keep source codes verbatim in assertions, with their meanings in the referenced `protocol`.

## Example packages

Write CSVs with all schema columns in schema order: the schemas match fields by position. Validate with `frictionless validate datapackage.json`. Frictionless requires every table referenced by a foreign key (e.g. `provenance`, `organism`) to exist, so add header-only CSVs in a scratch copy when validating. Mark example data as illustrative.
