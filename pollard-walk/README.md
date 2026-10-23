# Butterfly transect / Pollard walk (UKBMS)

A Pollard walk is a fixed-route butterfly transect, walked weekly through the flight season, on which every butterfly within a fixed "recording box" is counted. The method was introduced by Pollard (1977) and is the basis of butterfly monitoring schemes across Europe. This case study maps the version used by the [UK Butterfly Monitoring Scheme](https://ukbms.org/guidance-recording-forms) (UKBMS), drawing on the following documents:

- **G1** and **G2**: summary and field guidance notes
- **G3** and **G4**: site details guidance and the habitat classification
- **F1** and **F2**: the site details form and the weekly recording form
- the UKBMS FAQ

An [illustrative example package](example/) for one transect and one season accompanies it.

## The protocol in brief

- **Site:** a fixed route of about 1–2 km (45–60 minutes), divided into 5–15 numbered sections. Ideally the sections are of similar length, with each marking a change in habitat or management. The route must not change. A changed route is set up as a new transect, and added sections become a separate "extension" transect.
- **Site details (form F1):** for each section, the length, a grid reference near its mid-point, and up to four habitat codes and four management codes, covering 10 m either side of the line. Habitat codes come from the G4 classification (codes 1–40, based on EUNIS). Management codes are M1–M13. The form also records the site's grid reference, transect width, land use, conservation status and type of recorder.
- **When:** once a week through the 26 UKBMS weeks (week 1 is 1–7 April, week 26 is 23–29 September), with optional weeks before and after. Walks are made ideally between 10:45 and 15:45.
  - **Weather:** walks need at least 13 °C, with at least 60% sunshine below 17 °C, no rain, and wind no stronger than Beaufort 5.
  - **Sub-optimal weather:** walks may still be entered, with their weather, but are excluded from trend analysis.
  - **Several walks in one week** are all submitted.
- **Counting:**
  - **The recording box:** one nominated recorder walks at a steady pace, counting every adult butterfly within 2.5 m either side of the line and 5 m ahead. On sea cliffs and woodland rides, 5 m on one side only is allowed.
  - **What not to do:** no waiting at hotspots, no counting butterflies behind, and avoid double counting.
  - **Hard identifications:** Small and Essex skipper may be recorded as an aggregate. Mixed whites may be netted and the count apportioned by the sample's proportions.
  - **Outside the box:** sightings outside the box go in the notes, not the count.
  - **Complete walks only:** walks are submitted only if every section was covered.
- **Weather recording:** % sunshine per section (to the nearest 10%), and per walk: shade temperature, average sunshine, average wind speed (Beaufort 0–6) and wind direction. Start and finish times are recorded for the walk, not per section.
- **Zero counts:** a walk with no butterflies is still submitted ("a negative result is still a result"), and the online system fills in the zeros.
- **Other taxa:** moths, dragonflies and other species may optionally be entered as well.

## Event structure

```text
Transect season            T0001-2025                       context    · transect season
├── Site details           T0001-2025-SITE                  context    · site description
│   └── Section details ×6    …-SITE-S01..S06               context    · section description
├── Weekly walk            T0001-2025-W10-2025-06-05        context    · transect walk
│   ├── Section count ×6      …-W10-2025-06-05-S01..S06     survey     · transect section count
│   └── Notes (optional)      …-OUTSIDE                     occurrence · incidental records
└── … one walk Event per walk (weeks 3, 10 and 14 in the example)
```

- **The transect-season is the root.** UKBMS indices are calculated per site per year. `eventDate` is the year, because walks are allowed outside weeks 1–26, so a narrower range defined by the protocol wouldn't contain them all. The root carries the site's identity: `locationID` = `UKBMS:T0001`, the site name in `locality`, the grid reference in `verbatimCoordinates`, and the route in `footprintWKT` (`EPSG:27700`).
- **Each walk is an Event**, identified by week and date, so two valid walks in the same week are two Events. The walk holds what is recorded once per walk: date, `eventTime` (start/finish), duration, and the weather and week-number assertions.
- **Section count Events hold the Surveys.** Counts and sunshine are recorded per section, and the box is defined per section. Every walk has an Event for every section, because only complete walks are submitted.
- **Section details are their own Events**, as with BBS habitat, rather than being attached to every count. They describe the place for the season, are recorded separately from the counts, and cover a different strip (10 m either side, not the 2.5 m box). If they are updated between seasons, the next season gets its own site-details Events.
- **Sections are identified by `locationID`** (`UKBMS:T0001:S03`), which stays the same across walks and years. A route change means a new transect, so a section's `locationID` always refers to the same stretch of ground.

## Protocols

| `protocol_pk` | Used by |
|---|---|
| `P-UKBMS-TRANSECT` | Walk and section count Events, section Surveys, week-number assertions |
| `P-UKBMS-WEATHER` | Weather assertions (walk and section) |
| `P-UKBMS-SITE` | Site and section details Events and their assertions |

The `protocolDescription` of `P-UKBMS-TRANSECT` holds the minimum weather criteria. Whether a walk met them is deliberately not stored as a flag. The form doesn't record one, and data users can work it out from the recorded temperature, sunshine and wind.

## Surveys and survey targets

Each section count Event has one `survey`:

| Field | Value | Why |
|---|---|---|
| `surveySiteType` | `horizontalTransect` | |
| `sampleSizeValue` / `sampleSizeUnit` | section length / `metre` | |
| `totalAreaSampledValue` / `totalAreaSampledUnit` | length × 5 / `square metre` | |
| `inventoryTypes` | `restrictedSearch` | |
| `isAbundanceReported` | `true` | |
| `isAbsenceReported` | `false` | |
| `isLeastSpecificTargetCategoryQuantityInclusive` | `false` | See the skipper aggregate below |
| `isSamplingEffortReported` | `false` | Time is recorded per walk, and is held on the walk Event |

The recording box is the SurveyTarget. Unlike the BBS distance bands, the box has no internal categories, so each section Survey links to one target:

| `surveyTarget_pk` | Descriptors (all `include`) |
|---|---|
| `ST-UKBMS-BOX-BOTH-SIDES` | `taxon` = Papilionoidea · `lifeStage` = adult · maximum perpendicular distance either side = 2.5 m · maximum distance ahead = 5 m |
| `ST-UKBMS-BOX-ONE-SIDE` | `taxon` = Papilionoidea · `lifeStage` = adult · maximum perpendicular distance, one side only = 5 m · maximum distance ahead = 5 m |

- **`isSurveyTargetFullyReported = true` for both.** "Every butterfly within the recording box is recorded", so a species not recorded in a section on a walk was not detected there. This holds for walks with no butterflies at all: their Surveys exist and have no Occurrences.
- **Papilionoidea is the true butterfly superfamily, which includes the skippers (Hesperiidae).**
- **Box width belongs to the target.** The box is set when the transect is established and may differ by section (the one-sided box in the example is section 6, a woodland ride) or be wider (e.g. 10 m). Transects whose boxes differ need their own target rows. F1's "transect width" maps to these descriptors rather than to a field of its own.
- **Single-species transects** use the same structure, with the taxon descriptor narrowed to the target species. Walks then cover only that species' flight period.

### The Small / Essex skipper aggregate

The F2 form has rows for Small skipper, Essex skipper and "Small / Essex skipper". The aggregate is for individuals that were not separated. It does not total the other two rows.

| Record | DwC-DP |
|---|---|
| `verbatimIdentification` | `Small / Essex skipper` |
| `scientificName` / `taxonRank` | *Thymelicus* / genus |
| `identificationRemarks` | the two candidate species |

Because the aggregate excludes the separated individuals, `isLeastSpecificTargetCategoryQuantityInclusive = false`: the *Thymelicus* total in a section is the sum of all three rows. *Thymelicus* also contains Lulworth skipper, which is why the remark names the two species.

Counts of whites apportioned from a netted sample are estimates, not direct identifications. If the source marks them, keep that with `identificationRemarks` or an `occurrence-assertion`.

### Other taxa and sightings outside the box

- **Moths, dragonflies and other taxa** entered optionally are Occurrences on the section Event with no `surveyTarget_fk`. The Survey records them with `hasNonTargetTaxa = true` and `areNonTargetTaxaFullyReported = false`, so their absence elsewhere means nothing. The Wider Countryside Butterfly Survey variant asks whether moths and dragonflies were recorded at all. Where the answer is yes, they would get their own fully reported SurveyTarget.
- **Sightings outside the recording box** (from the notes) are Occurrences on a separate `incidental records` child of the walk, with **no Survey**, so they can never be mistaken for transect counts.

## Occurrences

There is **one Occurrence per walk × section × species** (or aggregate):

| UKBMS | DwC-DP |
|---|---|
| Section count | `occurrence.event_fk` (section Event) |
| Recording box | `occurrence.surveyTarget_fk` |
| Count | `organismQuantity`, with `organismQuantityType = individuals` |
| Species (F2 row) | `verbatimIdentification`, `scientificName`, `taxonRank` |
| Recorded | `occurrenceStatus = detected` |

Zero counts are not stored as rows: absence of detection follows from the fully reported target. UKBMS's own system does store zeros. If those are what is being exported, they can be published as `notDetected` Occurrences with `organismQuantity = 0` without changing anything else.

Missed weeks have no Event. UKBMS estimates values for missing weeks when computing indices, but those estimates are derived data and are not observations.

## Assertions

| Attached to | `assertionType` | Unit / values |
|---|---|---|
| Walk Event | `UKBMS week number` | 1–26, or 0, −1… / 27… outside the main season |
| Walk Event | `shade temperature` | °C |
| Walk Event | `average sunshine` | % |
| Walk Event | `average wind speed` | Beaufort 0–6 |
| Walk Event | `wind direction` | compass point |
| Section count Event | `sunshine` | %, nearest 10 |
| Section details Event | `UKBMS section length` | m |
| Section details Event | `UKBMS habitat code {1..4}` | G4 code 1–40, primary first |
| Section details Event | `UKBMS management code {1..4}` | M1–M13, primary first |
| Transect season Event | `UKBMS type of recorder` | BC / V / W / M |

"Average sunshine" is recorded on the form by the recorder, so it is kept as recorded on the walk. It is not recalculated from the section values.

The land use and conservation status from F1 would be further assertions on the transect-season Event. The F1 route map can be attached with `media` and `event-media`.

## Example

[`example/`](example/) contains one fictitious six-section, 1000 m transect (`T0001`) in 2025:

- **Week 3:** a valid walk with no butterflies, so it has Surveys and no Occurrences.
- **Week 10:** includes an optional moth record (Cinnabar) as a non-target Occurrence.
- **Week 14:** includes a Small / Essex skipper aggregate alongside separated Small and Essex skippers, and a Silver-washed Fritillary seen outside the recording box.

Section 6 uses the one-sided box.

The tables follow the ratified DwC-DP 1.0 schemas and have been checked with a validator.

## Open questions

- **Sub-optimal weather walks:** should they be published? If so, should the scheme's own assessment of whether a walk met the criteria be published with them, as an event assertion, rather than left for users to work out?
- **Several recorders:** for transects walked by a team, `recordedBy` is the nominated recorder on each walk. Is that recorded per walk in the UKBMS database, or only per site?
- **Apportioned counts:** are counts apportioned from netted samples flagged in the source data?
- **Section history:** how are changes to section details over time (habitat and management codes) kept? This decides whether site-details Events can be published per season or only as the current state.

## References

- Pollard, E. (1977). A method for assessing changes in the abundance of butterflies. *Biological Conservation* 12: 115–134.
- Pollard, E. & Yates, T. J. (1993). *Monitoring Butterflies for Ecology and Conservation*. Chapman & Hall.
- [UKBMS guidance and recording forms](https://ukbms.org/guidance-recording-forms) (G1–G4, F1, F2, FAQ).
