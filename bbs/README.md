# UK Breeding Bird Survey (BBS)

The [BTO/JNCC/RSPB Breeding Bird Survey](https://www.bto.org/our-science/projects/breeding-bird-survey) is the main scheme for monitoring the population changes of widespread breeding birds in the UK. Volunteers make standardised counts along fixed transects in randomly selected 1-km squares, every year.

This case study maps the field-recorded data (not the derived population trends) to DwC-DP. The protocol facts come from the [BBS Instructions (2018)](https://www.bto.org/file/bbsinstructions2018pdf). An [illustrative example package](example/) for one square and one season accompanies it.

## The protocol in brief

- **Site:** a randomly selected 1-km Ordnance Survey grid square. The route is two roughly parallel 1-km transects, each divided into five 200-m sections, numbered 1–10. The route should stay the same from year to year. It may be shortened if access is restricted, and a square where fewer than four sections can be covered is reported as uncoverable.
- **Visits:** an optional reconnaissance visit in March, then an Early visit (early April to mid May) and a Late visit (mid May to late June), at least four weeks apart. Visits ideally start between 06:00 and 07:00.
- **Bird counts:** every adult bird seen or heard is recorded with a two-letter BTO species code, by section and by the distance band where it was first noted:
  - **1:** less than 25 m from the line
  - **2:** 25–100 m
  - **3:** more than 100 m, including birds outside the square
  - **F:** in flight only, at any distance

  Positively identified juveniles are excluded, and no bird is counted twice. Displaying Skylarks and hovering Kestrels go in a distance band. Aerial-feeding hirundines and Swifts go in F unless they land. Date and start and finish times are recorded **per transect**, not per section.
- **Detection type (optional, online only):** song, call or visual for the first detection. If it is used, it must be recorded for all birds on that visit.
- **Weather:** one code (1–3) each for cloud, rain, wind and visibility, giving the overall conditions of the visit.
- **Habitat:** recorded once a year per section, for a 200 m × 50 m strip, separately from the bird count. It is recorded on the reconnaissance visit or after a count. A section can have up to two habitats, each with Level 1 and 2 codes (required) and up to two Level 3 and two Level 4 codes. Codes may be re-submitted at the Late visit if the habitat changes.
- **Mammals (optional):** the observer declares whether they looked for mammals on each visit, Y or N. Live mammals seen on the Early and Late visits are counted (code C). Other evidence from the season is recorded as presence only: D (dead), F (field signs), L (local knowledge) or S (seen on additional visits).
- **Colony counts:** pairs of colonially nesting birds (e.g. Rook) in the whole square, counted separately from the transects. The same birds may also appear in the transect counts.

## Event structure

```text
Square-season                SP1234-2025                   context  · site season
├── Reconnaissance visit     SP1234-2025-RECCE             context  · reconnaissance visit
│   └── Habitat recording ×10   …-RECCE-HAB-S01..S10       context  · habitat recording
├── Early visit              SP1234-2025-EARLY             survey   · site visit        (+ mammal Survey)
│   ├── Transect walk ×2        …-EARLY-T1, T2             context  · transect walk
│   │   └── Section count ×5    …-EARLY-T1-S01..S05        survey   · transect section count
│   └── Colony count            …-EARLY-COLONY             survey   · colony count
├── Late visit               SP1234-2025-LATE              (as Early; no mammal Survey if "looked for mammals" = N)
└── Mammal presence records  SP1234-2025-MAMMAL-PRESENCE   occurrence · season presence records
```

The structure rests on these choices:

- **The square-season is the root.** BBS is analysed per square per year, and the mammal presence codes refer to "this season" (January–July). The root's `eventDate` is therefore `2025-01/2025-07`, defined by the protocol rather than by the dates of its child Events.
- **Transect walks are an explicit level.** Times are recorded per transect, so `eventTime` (`06:42/07:30`) and `eventDurationValue` belong on the transect Event. The section Events carry only `eventDate`, because they have no times of their own and inventing them would be wrong.
- **Section count Events are where the bird Surveys live.** The section is the finest unit at which counts are made complete, so absence of detection can be inferred there.
- **Habitat recording is its own set of Events**, not assertions on the bird-count sections. The protocol tells observers not to record birds and habitat at the same time, and it describes a different area (a 50 m strip rather than the unbounded detection area). If there was no reconnaissance visit, the habitat Events sit under the visit after which they were recorded. Habitat changes submitted at the Late visit become a second set of habitat Events under the Late visit.
- **Physical places are identified by `locationID`, and visits by Events.** `BBS:SP1234:S03` is shared by every Event at section 3, in every year, so the repeat visits to one place can be followed over time. `footprintWKT` holds the route geometry in OSGB36 (`footprintSRS` = `EPSG:27700`).
- **The parent footprint must contain its children.** Routes may make minor intrusions into adjacent squares. Where that happens, the footprint of the square-level Events must be widened to the square plus the route. Band 3 birds can also lie outside the square, but they are Occurrences rather than Events, so they don't affect the Event footprints.

## Protocols

The `protocol` table gets one row per distinct method. Each row is referenced from `event.eventProtocol_fk`, `survey.samplingProtocol_fk` or `assertionProtocol_fk`, as appropriate.

| `protocol_pk` | Used by |
|---|---|
| `P-BBS-BIRDS` | Section count Events and their Surveys |
| `P-BBS-DETECTION` | Detection type occurrence assertions |
| `P-BBS-WEATHER` | Weather event assertions |
| `P-BBS-HABITAT` | Habitat recording Events and assertions |
| `P-BBS-MAMMALS` | Mammal Survey, mammal presence Event and assertions |
| `P-BBS-COLONY` | Colony count Event and Survey |

The special counting rules (displaying Skylarks, hovering Kestrels, aerial feeders, juveniles in mixed flocks) are recorded in `protocolDescription`. They are not encoded as data, because they adjust *how* a bird is classified rather than *which* birds are in scope.

## Surveys and survey targets

### Bird counts: one Survey per section count, four shared SurveyTargets

Each section count Event has one `survey`, with the following values:

| Field | Value |
|---|---|
| `surveySiteType` | `horizontalTransect` |
| `inventoryTypes` | `restrictedSearch` |
| `sampleSizeValue` / `sampleSizeUnit` | `200` / `metre` |
| `isAbundanceReported` | `true` |
| `isAbsenceReported` | `false` |
| `isLeastSpecificTargetCategoryQuantityInclusive` | `false` |
| `isSamplingEffortReported` | `false` |

Effort is not reported per section, because time is recorded only per transect and is already held on the transect Event.

The distance bands are modelled as **four SurveyTargets**, which every section Survey links to through `survey-survey-target`:

| `surveyTarget_pk` | Descriptors (`includeOrExclude` `surveyTargetType` = `surveyTargetValue`) |
|---|---|
| `ST-BBS-BAND-1` | include `taxon` = Aves · exclude `lifeStage` = juvenile · include minimum/maximum perpendicular distance = 0 / 25 m · exclude `behavior` = in flight |
| `ST-BBS-BAND-2` | as band 1, with 25 / 100 m |
| `ST-BBS-BAND-3` | as band 1, with minimum 100 m (no maximum) |
| `ST-BBS-BAND-F` | include `taxon` = Aves · exclude `lifeStage` = juvenile · include `behavior` = in flight |

All four have `isSurveyTargetFullyReported = true`.

Why four targets rather than one target ("adult birds") plus a distance-band assertion on each Occurrence?

- **The bands are part of the sampling design**, used by BTO for distance-sampling analyses. They are not incidental attributes.
- **`occurrence.surveyTarget_fk` holds the band**, through a key into a defined target, so it is not a free-text assertion.
- **Absence of detection can be inferred per band.** No Robin in band 1 of section 3 on the Early visit is a meaningful non-detection, distinct from no Robin anywhere in the section.
- **The bands are mutually exclusive**, so the total for a species in a section is the sum of its Occurrences. That is why `isLeastSpecificTargetCategoryQuantityInclusive` is `false`: no single record holds the total.

Two details of the descriptors need care:

- **Juveniles:** the exclusion is of *positively identified* juveniles only. Birds of unknown age are counted, so the target must not be described as "adults". `surveyTargetRemarks` states this explicitly.
- **Shared target rows:** they are valid only while every linked Survey reported all four bands completely. A dataset in which some visits recorded only some bands would need separate target rows with `isSurveyTargetFullyReported = false` for those visits.

### Mammals: a Survey only where the observer looked

The visit-level "looked for mammals" flag decides whether a Survey exists:

- **Y:** the visit Event gets a mammal `survey` linked to `ST-BBS-MAMMALS`, with one taxon descriptor per listed species (abridged in the example) and `isSurveyTargetFullyReported = true`. Count records (code C) are Occurrences on the **visit** Event, because mammals are counted per visit and not by section or band. A Y with no mammals seen therefore supports non-detection.
- **N:** no mammal Survey is created, so nothing about mammal absence can be inferred. The flag is still kept as an event assertion (`BBS looked for mammals` = `N`), so that the lack of a Survey is explicit rather than ambiguous.

The season presence codes (D, F, L, S) have no counts and no dates. They are Occurrences on a season-level `occurrence` Event with **no Survey**, each with an `occurrence-assertion` of `BBS mammal presence code`. D also sets `vitality = dead`. A species with several codes gets one Occurrence per code, which preserves the separate lines of evidence.

### Colony counts

A colony count is a different method with a different unit, so it gets its own Event (a child of the visit it was made on), its own Survey (`surveySiteType = arealPlot`, `inventoryTypes = openSearch`, 1 km²) and its own target. The target is `ST-BBS-COLONY` (include Aves, include `nesting habit` = colonial). Each count is an Occurrence with `organismQuantityType = breeding pairs`.

`isSurveyTargetFullyReported` is `false`, because the protocol does not list which species are colonial, so the absence of a colony record cannot safely be read as absence.

The same birds may appear in both the colony and the transect counts. The two are separate Surveys that are never summed, so no special flag is needed. `isLeastSpecificTargetCategoryQuantityInclusive` is about subcategories *within* one Survey and does not apply.

## Occurrences

There is **one Occurrence per visit × section × species × distance band**: the finest grain on the BBS count forms. Where detection type was recorded online, the source may hold one row per field record. In that case, keep those rows as separate Occurrences rather than summing them, since each has its own detection type.

| BBS | DwC-DP |
|---|---|
| Section count | `occurrence.event_fk` (section Event) |
| Distance band | `occurrence.surveyTarget_fk` |
| Count | `organismQuantity`, with `organismQuantityType = individuals` |
| BTO two-letter code | `verbatimIdentification` |
| Species | `scientificName`, `taxonRank`, and `taxonID` where available |
| Recorded | `occurrenceStatus = detected` |
| Detection type S/C/V | `occurrence-assertion` (`BBS detection type`) |

Occurrences are not created for individual birds, and Organisms are not created: a count conveys quantity, not identity. No `notDetected` Occurrences are created either. The forms have no zeros ("if there is no count, leave the box empty"), and non-detection follows from the Survey and its fully reported SurveyTargets.

## Assertions

| Attached to | `assertionType` | Values |
|---|---|---|
| Visit Event | `BBS cloud cover code`, `BBS rain code`, `BBS wind code`, `BBS visibility code` | 1–3 |
| Visit Event | `BBS detection type recorded` | `true` / `false` |
| Visit Event | `BBS looked for mammals` | `Y` / `N` |
| Habitat recording Event | `BBS habitat {1,2} level {1..4}` | BTO habitat codes, one row per code |
| Occurrence | `BBS detection type` | `S` / `C` / `V` |
| Occurrence | `BBS mammal presence code` | `C` / `D` / `F` / `L` / `S` |

The codes are kept verbatim, and their meanings are given in the `protocolDescription` of the referenced protocol. `event.reportedWeather` and `event.habitat` may additionally carry readable summaries, but they don't replace the coded assertions.

Route maps can be attached through `media` and `event-media` on the square-season Event. They are not included in the example.

## Example

[`example/`](example/) is a complete DwC-DP package for one fictitious square, SP1234, in 2025:

- **Early visit:** detection type recorded, mammals looked for (two counts), and a Rook colony count.
- **Late visit:** no detection type, and mammals not looked for, so it has no mammal Survey.
- **Season mammal presence:** records with codes D, F, L and S.
- **Habitat:** codes for all ten sections from the reconnaissance visit.

The tables follow the ratified DwC-DP 1.0 schemas and have been checked with a validator.

## Open questions

- **Detection type grain:** does BBS-Online store individual field records (with detection type) or only section × band totals? This sets the Occurrence grain.
- **Shortened routes:** how are unwalked sections recorded, and do they change between visits? Unwalked sections should have no section Event and Survey for that visit, and the route change should be documented on the square-season Event.
- **Weather timing:** is weather recorded per visit, as the 2018 instructions imply, or per transect?
- **Colonial species:** is there a defined list of colonial species? With one, `ST-BBS-COLONY` could be made fully reported.
- **Unsurveyed years:** should square-seasons for squares that were allocated but not surveyed be published, so that coverage of the sampling frame (and the stratification used for trends) can be reconstructed? This is out of scope here.
