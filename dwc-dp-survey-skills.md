This helps guide LLMs to provide structured mappings to the Darwin Core Data Package for survey and monitoring protocols.

First understand:
1. Darwin Core conceptual model https://dwc.tdwg.org/cm/
2. Darwin Core Data Package (DwC-DP) schemas http://rs.tdwg.org/dwc-dp/1.0/table-schemas/ (raw JSON: https://github.com/tdwg/rs.tdwg.org/tree/master/dwc-dp/1.0/table-schemas)
3. The Event hierarchy guide https://dwc.tdwg.org/hierarchy/ and the isLeastSpecificTargetCategoryQuantityInclusive guide https://dwc.tdwg.org/inclusive/

Second, read all existing mappings from the repository to get a sense of how others are done.

Third, read the protocol's own primary documentation (field instructions, recording forms, and FAQs or data-entry guides, which often answer data questions such as how zeros, incomplete visits and repeat visits are handled). If a paper leaves out method detail, check whether the authors published the data (e.g. on GBIF): the dataset's metadata and records often fill the gaps and show how the data are actually structured. Base the mapping on what the protocol actually records, cite it, and flag anything that is assumed rather than documented.

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

Add a row for the new case study to the "Case studies" table in the repository's top-level `README.md`.

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
- When recording categories are not clean taxa (e.g. "solitary bees", "other flies", "other insects / unsure"), define each with include/exclude descriptors on its SurveyTarget. Give the Occurrence the smallest taxon that contains the group as `scientificName`, with the label in `verbatimIdentification`. Say clearly that the name alone is misleading without the target, and set `isLeastSpecificTargetCategoryQuantityInclusive = false` when a broad residual group doesn't include the others.
- The dimensions of a fixed detection zone (e.g. a transect "recording box") are SurveyTarget descriptors. Zones of a different shape or size need separate target rows. The area goes in `survey.totalAreaSampledValue`.
- Recorded aggregates of hard-to-separate species (e.g. "Small / Essex skipper") use the smallest containing taxon plus `identificationRemarks`. Check whether the aggregate includes or excludes the separated records when setting `isLeastSpecificTargetCategoryQuantityInclusive`.
- Sightings the protocol puts outside the count (outside the zone, in notes) go on a separate child Event with no Survey, so they can't be read as part of the count.
- Keep values the recorder wrote down, even summaries such as "average sunshine", as recorded. Don't publish values the scheme estimates later (e.g. counts for missed weeks) as observations.
- Organisms that define the sampling unit rather than being detected (e.g. the flower being watched, a host plant) are Occurrences without a `surveyTarget_fk`. Link the detected organisms to them with `organism-interaction` (e.g. `visited flower of`), and flag them through `hasNonTargetTaxa`.
- Physical samples are `material`: a bulk sample has `collectionEvent_fk` set to the sampling Event, and derived material (e.g. a DNA extract) has `derivedFromMaterialEntity_fk`. Properties of the whole sample, such as biomass, go on the material (`measuredMassInGrams`), not on Occurrences.
- For DNA metabarcoding:
  - use one Occurrence per sample × ASV
  - put read counts in `nucleotide-analysis` (not `organismQuantity`), and set `isAbundanceReported = false`
  - link each Occurrence to its analysis and sequence through an `identification` row (`identificationType = nucleotideAnalysis`)
  - accept names at the rank the match supports, and use `taxonFormula` with `identification-taxon` for ambiguous matches
  - use one `molecular-protocol` row per extract (it is a sample-level record)
- Values derived later by the researchers (e.g. GIS land cover) can be assertions, but give them their own protocol and `assertionBy` so they aren't mistaken for field observations.
- Keep source codes verbatim in assertions, with their meanings in the referenced `protocol`.
- Keep locations as given (`verbatimCoordinates`, `verbatimSRS`). Derive decimal coordinates with a `coordinateUncertaintyInMeters` that reflects the source precision (e.g. a grid square's half-diagonal). Include the time zone in `eventTime` when the protocol specifies one.
- Don't invent a parent Event for independent records just to have a hierarchy.

## Example packages

Write CSVs with all schema columns in schema order: the schemas match fields by position. Mark example data as illustrative.

Also publish the package as `example.zip` next to the `example/` folder, containing just the files at the top level (e.g. `zip -X -j -D example.zip example/*`). Re-create it whenever the example changes.

Validate with `frictionless validate datapackage.json`. Frictionless reports an error for any table that a schema's foreign key points to but the package doesn't include (e.g. `provenance`, `organism`), even if the key is never used. To validate, add empty copies of those tables (header row only) in a temporary copy of the package. Don't commit them, and don't describe this workaround in the case study README.
