# UK PoMS Flower-Insect Timed Count (FIT Count)

The [FIT Count](https://ukpoms.org.uk/fit-counts) is the public-participation part of the UK Pollinator Monitoring Scheme (PoMS). A recorder watches a 50 × 50 cm patch of a chosen flower for ten minutes and counts the insects that visit it, in ten broad groups.

This case study maps one count, as captured on the [FIT Count field recording form (v6, 2023)](https://ukpoms.org.uk/sites/default/files/pdf/FIT%20Count%20survey%20form%20v6.pdf), to DwC-DP. An [illustrative example package](example/) with two counts accompanies it.

## The protocol in brief

- **When:** 1 April to 30 September, in dry, warm weather. The minimum temperature is 13 °C under less than half cloud, or 15 °C under half cloud or more.
- **Where:** anywhere a suitable target flower grows. The recorder gives a location name (town or village), an optional grid reference, and one habitat from a list of 12 (or a free-text "other").
- **Target flower:** preferably one of 14 listed species (e.g. Dandelion, Bramble, Lavender), otherwise any insect-attracting flower. The recorder notes:
  - the number of flowers in the patch, and the unit counted (individual flowers, heads, umbels or spikes)
  - the cover of the patch, in three classes
  - whether the patch sits within a larger patch of the same flower, within mixed flowers, or is isolated
- **Count:** for exactly 10 minutes, starting at a recorded time (British Summer Time), every insect that sits on or lands on the target flowers is counted, each individual once. Insects are tallied into ten groups:

  1. Bumblebees
  2. Honeybees
  3. Solitary bees
  4. Wasps (including ichneumon wasps)
  5. Hoverflies (including 'non-typical' hoverflies)
  6. Other flies
  7. Butterflies and moths
  8. Beetles (larger than 3 mm)
  9. Small insects (such as pollen beetles) less than 3 mm long
  10. Other insects (including any the recorder is unsure of)

  Spiders and caterpillars are not counted.
- **Weather:** sky (three classes), whether the patch was in sun (three classes) and wind (three classes).
- **Recorder:** self-assessed identification experience (four levels).
- **Photos:** a photo of the target flower is requested. Insect photos are optional and must not be taken during the count.

## Event structure

Each FIT Count is a single Event, with `eventCategory` = `survey` and `eventType` = `FIT Count`. There is no hierarchy.

- **No parent Event:** counts are independent. Recorders choose their own place and time, and nothing ties one count to another, so a parent would have to be invented.
- **Duration:** the 10-minute duration goes in `eventTime` = `11:20+01:00/11:30+01:00` and `eventDurationValue` = `10` minutes. The time zone is written out because the form specifies British Summer Time.
- **Location:**
  - `verbatimCoordinates` keeps the grid reference as given (`verbatimCoordinateSystem` = Ordnance Survey National Grid reference, `verbatimSRS` = `EPSG:27700`).
  - `decimalLatitude` and `decimalLongitude` are the centre of the grid square the reference describes, and `coordinateUncertaintyInMeters` follows from its precision: 71 m for a 6-figure (100 m) reference.
  - `locality` holds the location name.
  - The 50 cm patch is far smaller than the uncertainty, so no `footprintWKT` is given.
- **Habitat:** the single ticked habitat goes in `event.habitat`, using the form's wording.

The same structure would hold if FIT Counts were made as part of the structured PoMS 1-km square visits. Each count would then become a child of a square-visit Event, which is not covered here.

## Protocols

One `protocol` row, `P-FIT-COUNT`, describes the whole method. That covers the eligibility rules, the counting rules and the meaning of every categorical value on the form, and it references the form as its source. The event, the survey and every assertion point to it.

## Surveys and survey targets

Each FIT Count Event has one `survey` with the following values:

| Field | Value | Why |
|---|---|---|
| `surveySiteType` | `arealPlot` | A fixed 50 × 50 cm patch |
| `sampleSizeValue` / `sampleSizeUnit` | `0.25` / `square metre` | |
| `samplingEffortValue` / `samplingEffortUnit` | `10` / `personMinutes` | One recorder for exactly 10 minutes |
| `isSamplingEffortReported` | `true` | |
| `inventoryTypes` | `restrictedSearch` | Fixed time and area |
| `isAbundanceReported` | `true` | |
| `isAbsenceReported` | `false` | |
| `isLeastSpecificTargetCategoryQuantityInclusive` | `false` | See below |
| `hasNonTargetTaxa` | `true` | The target flower is reported but is not a target |
| `nonTargetTaxa` | the flower's name | |
| `areNonTargetTaxaFullyReported` | `false` | |
| `identifiedBy_fk` | the recorder | |

The recorder's self-assessed experience is a `survey-assertion` (`FIT recorder identification experience`). It qualifies how reliable this survey's identifications are, and DwC-DP has no agent-assertion table.

### Ten SurveyTargets, one per insect group

The ten groups are mutually exclusive, they are part of the method, and every one is always reported, so each group is a SurveyTarget. All ten are shared by every FIT Count Survey through `survey-survey-target`, and all have `isSurveyTargetFullyReported = true`. Every target has the descriptor `include behavior = sitting on or landing on target flowers`, plus the descriptors below.

| `surveyTarget_pk` | Further descriptors |
|---|---|
| `ST-FIT-BUMBLEBEES` | include taxon *Bombus* |
| `ST-FIT-HONEYBEES` | include taxon *Apis mellifera* |
| `ST-FIT-SOLITARY-BEES` | include taxon Anthophila · exclude *Bombus* · exclude *Apis mellifera* |
| `ST-FIT-WASPS` | include taxon Hymenoptera · exclude Anthophila |
| `ST-FIT-HOVERFLIES` | include taxon Syrphidae |
| `ST-FIT-OTHER-FLIES` | include taxon Diptera · exclude Syrphidae |
| `ST-FIT-LEPIDOPTERA` | include taxon Lepidoptera · exclude `lifeStage` larva (caterpillars) |
| `ST-FIT-BEETLES` | include taxon Coleoptera · include minimum body length 3 mm |
| `ST-FIT-SMALL-INSECTS` | include taxon Insecta · include maximum body length 3 mm |
| `ST-FIT-OTHER-INSECTS` | include taxon Insecta (residual group, explained in `surveyTargetRemarks`) |

Several groups are **not clean taxa**. "Solitary bees" means bees other than bumblebees and honeybees, "Other flies" means flies other than hoverflies, and "Other insects" means whatever was left over or uncertain. The SurveyTarget is what records these meanings, through its exclusions. An Occurrence's `scientificName` can only name the smallest taxon that contains the group (Anthophila, Diptera, Insecta), so on its own it is misleading: `Diptera, 3` here means three *non-hoverfly* flies.

This is also why `isLeastSpecificTargetCategoryQuantityInclusive` is `false`. The `Insecta` record from "Other insects" does not include the bees, flies and so on counted in the other groups, so the total of all insects is the sum of every Occurrence.

Spiders are excluded by the taxon descriptors already, since they are not Insecta. The form's mention of them is kept in the protocol.

## Occurrences

There is one Occurrence per **count × insect group** with a non-zero tally:

| FIT Count | DwC-DP |
|---|---|
| Insect group | `surveyTarget_fk` |
| Group label | `verbatimIdentification` |
| Smallest taxon containing the group | `scientificName`, `taxonRank` |
| Tally | `organismQuantity`, with `organismQuantityType = individuals` |
| Recorded | `occurrenceStatus = detected` |

A group left blank on the form is a non-detection. It is inferred from the fully reported SurveyTarget rather than stored as a row.

### The target flower is an Occurrence too

The flower patch is recorded as an Occurrence of the plant, without a `surveyTarget_fk`, because it was chosen rather than detected:

| Form field | DwC-DP |
|---|---|
| Flower | `scientificName`, with the form's name in `verbatimIdentification` (e.g. *Rubus fruticosus* agg., `taxonRank = speciesAggregate`) |
| Number of flowers in the patch | `organismQuantity` |
| Unit counted | `organismQuantityType` (`individual flowers`, `flower heads`, `flower umbels` or `flower spikes`) |
| Cover of the patch | `occurrence-assertion` (`FIT target flower cover of patch`) |
| Photo | `media` + `occurrence-media` |

Each insect-group Occurrence is then linked to the flower by an `organism-interaction` row:

| Field | Value |
|---|---|
| `subjectOccurrence_fk` | the insect-group Occurrence |
| `organismInteractionType` | `visited flower of` (an example value in the schema) |
| `relatedOccurrence_fk` | the flower Occurrence |
| `relatedOrganismPart` | `flower` |

This makes the plant–pollinator relationship explicit, so a data user can find the insect groups visiting Bramble without parsing assertions. The alternative, holding the flower as event assertions, would keep the plant's name and quantities out of the taxon fields, where they can't be searched as taxa.

## Assertions

| Attached to | `assertionType` | Values (form wording) |
|---|---|---|
| Event | `FIT sky` | All or mostly blue · Half blue and half cloud · All or mostly cloud |
| Event | `FIT patch sunshine` | Entirely in sunshine · Partly in sun and partly shaded · Entirely shaded |
| Event | `FIT wind strength` | Leaves still/moving occasionally · Leaves moving gently all the time · Leaves moving strongly |
| Event | `FIT patch setting` | In a larger patch of the same flower · In a larger patch of many different flowers · More or less isolated |
| Survey | `FIT recorder identification experience` | One of the four statements on the form |
| Flower Occurrence | `FIT target flower cover of patch` | Less than half · About half · More than half |

The patch setting is recorded on the Event because it describes the surroundings of the patch, not the flowers being watched.

## Example

[`example/`](example/) contains two fictitious counts:

- **`FIT-2025-000101`:** a garden in June, on Lavender counted as flower spikes. Five insect groups are recorded, and a photo of the flower is attached.
- **`FIT-2025-000102`:** a verge in July, on Bramble counted as individual flowers, by a less experienced recorder. Seven groups are recorded, including small insects and "other insects".

The tables follow the ratified DwC-DP 1.0 schemas and have been checked with a validator.

## Open questions

- **Stored zeros:** do the PoMS website and app store zero tallies explicitly for each group? If so, they could be published as `notDetected` Occurrences with `organismQuantity = 0`. This mapping currently relies on inference from the targets instead.
- **Overlapping groups:**
  - The size rule overlaps the taxonomic groups. A bee or fly under 3 mm could go in either its group or "Small insects", and ants could be read as "Wasps" or "Other insects".
  - Absence of detection in an overlapping group is therefore weaker than the targets suggest.
  - Guidance from PoMS on how recorders should resolve these overlaps would allow tighter descriptors.
- **Temperature:** the eligibility threshold depends on temperature, but the form does not record it. If the app captures temperature or derives it from weather data, it belongs in an event assertion.
- **Grid reference precision:** grid references vary in precision, and some are picked from an online map. The precision of each one should set `coordinateUncertaintyInMeters`.
