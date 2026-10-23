# Braun-Blanquet relevé (vegetation plot)

A relevé is a record of a vegetation plot. The author places a plot in a homogeneous stand of vegetation and lists every plant species in it, layer by layer. Each species gets a combined cover-abundance value and, traditionally, a sociability value. The method comes from Braun-Blanquet's *Pflanzensoziologie* and is the basis of European phytosociology and of vegetation-plot databases such as those managed in Turboveg. This case study maps the method as described in the English translation, [*Plant Sociology* (Braun-Blanquet 1932)](https://archive.org/details/plantsociologyst00brau), with the extended cover-abundance scale of [Barkman, Doing & Segal (1964)](https://natuurtijdschriften.nl/pub/539344) and the plot sizes proposed by [Chytrý & Otýpková (2003)](https://doi.org/10.1111/j.1654-1103.2003.tb02183.x). The header fields follow the [GBIF Relevé extension](http://rs.gbif.org/extension/gbif/1.0/releve_2016-05-10.xml), which reflects what vegetation-plot databases hold. An [illustrative example package](example/) with two relevés accompanies it.

## The protocol in brief

- **Plot placement:** the author chooses a stand that is homogeneous in structure and species composition, and places the plot in it. Placement is preferential, not random.
- **Plot size:** traditionally the "minimal area", found by doubling a small plot until few new species are added (Braun-Blanquet 1932, pp. 28–29). Sizes in practice range widely. Chytrý & Otýpková (2003) propose standards: 4 m² for aquatic and low vegetation, 16 m² for most grassland and heathland, 50 m² for scrub and 200 m² for woodland. Plots are best square or rectangular.
- **Layers:** four principal layers: tree, shrub, herb and moss (ground) layer. They may be subdivided, e.g. an upper and lower tree layer. "Each well-defined layer of vegetation must be estimated separately" (p. 34), so a species in several layers, such as beech in the tree, shrub and herb layers, is recorded once per layer. The height boundaries of the layers vary between authors and traditions.
- **Species list:** every vascular plant species in the plot, by layer. Bryophytes and lichens are recorded by some authors and not others. Vegetation-plot databases record whether each group was identified (`mossesIdentified`, `lichensIdentified` in the GBIF Relevé extension).
- **Cover-abundance:** one value per species per layer. The classic scale (1932) is `+`, `1`–`5`, with `r` added later. The extended scale of Barkman, Doing & Segal (1964) splits class 2:

  | Code | Meaning |
  |---|---|
  | `r` | one or very few individuals, cover negligible |
  | `+` | few individuals, cover below 1% |
  | `1` | numerous individuals, cover below 5% |
  | `2m` | very numerous (more than 100 individuals), cover below 5% |
  | `2a` | any number, cover 5–12.5% |
  | `2b` | any number, cover 12.5–25% |
  | `3` | cover 25–50% |
  | `4` | cover 50–75% |
  | `5` | cover 75–100% |

  Dropping the letters gives the classic scale (Barkman et al. 1964, p. 399).
- **Sociability:** how the shoots of a species are grouped, from 1 (growing singly) to 5 (great crowds, pure populations). Often written with the cover-abundance value, e.g. `2a.4`. Many modern surveys omit it.
- **Header data:** date, location, elevation, aspect, inclination, total cover and the cover and height of each layer, and often bedrock, soil and land use.
- **Species outside the plot:** the 1932 procedure ends by noting "the species present in the stand outside the sample area" (p. 29). Some authors keep this list, without values.
- **Syntaxon:** after fieldwork, the author usually assigns each relevé to a vegetation unit (an association), by comparing it with other relevés in a table.

## Event structure

```text
Relevé                   REL-001                survey     · relevé

Stand                    REL-002-STAND          context    · vegetation stand
├── Relevé                  REL-002             survey     · relevé
└── Species outside plot    REL-002-OUTSIDE     occurrence · species outside plot
```

- **Each relevé is one Event**, with `eventCategory` = `survey` and `eventType` = `relevé`. This agrees with GBIF's guidance that a relevé corresponds to a single Event ([Palpurina & Ingenloff 2025](https://doi.org/10.3897/biss.9.182514)). The author's relevé number goes in `fieldNumber`.
- **No parent for independent relevés.** Relevés in a study are placed one by one, in different stands, so a parent Event (a project or field campaign) would be invented. Grouping them is a matter for the dataset.
- **Layers are not Events.** Layers are vertical strata of the same plot, recorded at the same time by the same person. They are distinguished by SurveyTargets instead (see below).
- **The stand is an Event only when something is recorded for it.** The stand contains the plot, so it can be a parent. Relevé 2 has one because the author listed species outside the plot and estimated the stand's area. Those species go on a separate child Event with **no Survey**, so they can't be read as part of the plot list. They cannot be a child of the relevé, because they lie outside it. Relevé 1 has nothing recorded at stand level, so it has no stand Event.
- **Location:**
  - `decimalLatitude` and `decimalLongitude` are the plot centre.
  - `coordinateUncertaintyInMeters` combines the GPS accuracy with the plot's half-diagonal: 16 m for the 10 × 20 m plot and 8 m for the 4 × 4 m plot.
  - The plot's shape and dimensions go in `locationRemarks`. No `footprintWKT` is given, because the plot's orientation was not recorded.
  - Elevation goes in `minimumElevationInMeters` / `maximumElevationInMeters`.
- **Permanent plots:** a plot that is resurveyed keeps the same `locationID`, and each resurvey is a new relevé Event.

## Protocols

| `protocol_pk` | Used by |
|---|---|
| `P-BB-RELEVE` | Relevé, stand and outside-plot Events, Surveys, Occurrences (`occurrenceProtocol_fk`), header and sociability assertions |
| `P-BB-SYNTAXON` | Syntaxon assertions |

`P-BB-RELEVE` holds what a data user needs to read the values: the plot sizes, the layer height boundaries, the scale definitions, the sociability scale and the rules for bryophytes and lichens. Datasets with different conventions, such as different layer boundaries or the classic scale, need their own protocol rows. Each Occurrence points to the protocol through `occurrenceProtocol_fk`, so the scale behind its value is never ambiguous.

## Surveys and survey targets

Each relevé Event has one `survey`:

| Field | Value | Why |
|---|---|---|
| `surveySiteType` | `arealPlot` | |
| `sampleSizeValue` / `sampleSizeUnit` | plot area / `square metre` | |
| `totalAreaSampledValue` / `totalAreaSampledUnit` | plot area / `m²` | |
| `inventoryTypes` | `restrictedSearch` | A fixed area with a complete species list, and no time limit |
| `isAbundanceReported` | `true` | Cover-abundance is an ordinal quantity, not a count |
| `isAbsenceReported` | `false` | |
| `isLeastSpecificTargetCategoryQuantityInclusive` | `false` | See below |
| `isSamplingEffortReported` | `false` | No time is recorded |
| `hasNonTargetTaxa` | `false` | |
| `identifiedBy_fk` | the author | |

`event.isVegetationCoverReported` is `true` on each relevé Event.

### SurveyTargets by layer and group

The layers are part of the method: every species is recorded by layer, and a species absent from a layer is a meaningful non-detection (no beech seedlings in the herb layer, say). This is the same situation as the BBS distance bands, so each layer is a SurveyTarget. Bryophytes and lichens are split further, because whether each was identified is recorded separately.

| `surveyTarget_pk` | Descriptors |
|---|---|
| `ST-BB-TREE-LAYER` | include `taxon` = Tracheophyta · include `vegetation layer` = tree layer |
| `ST-BB-SHRUB-LAYER` | include `taxon` = Tracheophyta · include `vegetation layer` = shrub layer |
| `ST-BB-HERB-LAYER` | include `taxon` = Tracheophyta · include `vegetation layer` = herb layer |
| `ST-BB-BRYOPHYTES` | include `taxon` = Bryophyta, Marchantiophyta, Anthocerotophyta · include `vegetation layer` = moss layer · exclude `substrate` = bark of living trees |
| `ST-BB-LICHENS` | include `taxon` = Fungi · include `growthForm` = lichen · include `vegetation layer` = moss layer · exclude `substrate` = bark of living trees |

All five have `isSurveyTargetFullyReported = true`.

- **Vascular-plant layers are linked to every relevé**, including layers that are absent from the plot. A grassland relevé linked to the tree layer target says there were no trees in the plot, which is true.
- **Bryophyte and lichen targets are linked only when the group was identified.** This follows the BBS "looked for mammals" pattern. In the example, relevé 1 had its bryophytes identified, and neither relevé had its lichens identified. The flags are still kept as event assertions, so the lack of a target is explicit.
- **Layer heights are not descriptors.** The boundaries are set by the protocol, not per relevé, so the targets can be shared, and the measured heights of each layer are header assertions. Datasets whose layers are defined differently, or subdivided (upper and lower tree layer), need their own target rows.
- **`isLeastSpecificTargetCategoryQuantityInclusive = false`.** A species recorded in several layers has a separate value in each, and no record holds its total. Cover values of different layers can't simply be added either, because the layers overlap in projection.

## Occurrences

There is **one Occurrence per relevé × layer × species**:

| Relevé | DwC-DP |
|---|---|
| Relevé | `occurrence.event_fk` |
| Layer | `occurrence.surveyTarget_fk` |
| Cover-abundance value | `organismQuantity` (e.g. `2a`), with `organismQuantityType = Braun-Blanquet Scale` |
| Scale definition | `occurrenceProtocol_fk` = `P-BB-RELEVE` |
| Species, as written | `verbatimIdentification` |
| Species | `scientificName`, `taxonRank` |
| Recorded | `occurrenceStatus = detected` |
| Sociability | `occurrence-assertion` (`Braun-Blanquet sociability`) |

- **Codes are kept verbatim.** `organismQuantityType = Braun-Blanquet Scale` uses the example in the Darwin Core definition. The extended codes (`2m`, `2a`, `2b`) reduce to the classic scale by dropping the letter, and the protocol says which variant was used.
- **No percentages are stored as observations.** Converting codes to percentages (e.g. class midpoints, or ordinal transformations for analysis) is an analysis step with several competing conventions, so it is left to data users.
- **Combined notation is split.** A source value such as `2a.4` becomes `organismQuantity = 2a` and a sociability assertion of `4`.
- **No `notDetected` rows.** Relevés list only the species present. Non-detection follows from the fully reported targets.
- **Species outside the plot** are Occurrences on the outside-plot Event, without `surveyTarget_fk` or quantity.

An earlier GBIF publication of relevés ([Prioksko-Terrasnyi Biosphere Reserve](https://doi.org/10.3897/BDJ.9.e71266)) had to encode the layer in `locationID`, because a Darwin Core Archive has no place for it. In DwC-DP the layer is a defined SurveyTarget, so absence can be inferred per layer and `locationID` stays the identifier of the place.

## Assertions

| Attached to | `assertionType` | `assertionTypeIRI` | Unit / values |
|---|---|---|---|
| Relevé | `aspect` | `gbif:aspect` | compass point |
| Relevé | `inclination` | `gbif:inclinationInDegrees` | degrees |
| Relevé | `total cover` | `gbif:coverTotalInPercentage` | % |
| Relevé | `tree layer cover`, `shrub layer cover`, `herb layer cover`, `moss layer cover` | `gbif:coverTreesInPercentage`, … `coverCryptogamsInPercentage` | % |
| Relevé | `tree layer height`, `shrub layer height` | `gbif:treeLayerHeightInMeters`, `gbif:shrubLayerHeightInMeters` | m |
| Relevé | `herb layer height` | `gbif:herbLayerHeightInCentimeters` | cm |
| Relevé | `mosses identified`, `lichens identified` | `gbif:mossesIdentified`, `gbif:lichensIdentified` | `true` / `false` |
| Relevé | `bedrock` | | verbatim |
| Relevé | `syntaxon` | `gbif:syntaxonName` | syntaxon name with authorship |
| Stand | `stand area` | | m² |
| Occurrence | `Braun-Blanquet sociability` | | 1–5 |

`gbif:` stands for `http://rs.gbif.org/terms/1.0/`, the namespace of the GBIF Relevé extension. Using its terms as `assertionTypeIRI` lets a relevé published in DwC-DP be converted to and from the Relevé extension without losing meaning.

- **Layer covers and heights are kept as recorded.** Covers are only recorded for layers that are present. They are not calculated from the species values.
- **The syntaxon is an interpretation, not an observation.** It is made after fieldwork, may be revised, and depends on the classification used. It is an assertion with its own protocol, `P-BB-SYNTAXON`, an `assertionBy_fk` and an `assertionMadeDate` later than the relevé. A revised assignment is a new assertion row.

## Example

[`example/`](example/) contains two fictitious relevés by one author near Jena, Germany, in June 2025:

- **Relevé 1:** a 200 m² beech forest plot with all four layers.
  - *Fagus sylvatica* is recorded in the tree, shrub and herb layers.
  - Bryophytes were identified, so the bryophyte target is linked. Lichens were not.
  - Assigned to the *Galio odorati-Fagetum* after fieldwork.
- **Relevé 2:** a 16 m² calcareous grassland plot.
  - It has only a herb layer, plus a moss layer whose cover was estimated but whose species were not identified.
  - It includes a `2m` value (*Thymus pulegioides*).
  - It has a stand Event, with the stand's area and two species seen outside the plot.

The tables follow the ratified DwC-DP 1.0 schemas and have been checked with a validator.

## Open questions

- **Layer boundaries:**
  - The height limits in the example protocol (tree layer above 5 m, shrub layer 1–5 m) are an assumption, chosen for illustration.
  - Real datasets must state their own. Where a database stores finer layer codes (e.g. Turboveg's sub-layers, juveniles and seedlings), each code needs its own target, and the mapping of codes to layers needs documenting.
- **What counts as "in the plot":** authors differ on whether a species is recorded when its shoots overhang the plot but it is rooted outside, and on whether epiphytic bryophytes and lichens are included. The exclusion of epiphytes in the example is an assumption. The rule belongs in the protocol, and in the target descriptors where it changes what is in scope.
- **Juveniles:** some authors record seedlings and juveniles of trees separately from the herb layer, or mark them with a life stage. If so, they need their own target, or `lifeStage` on the Occurrence.
- **Plot size and the minimal area:** where a plot's size was set by the minimal area procedure, the smaller nested plots of the procedure are not usually kept. If they are, each nested plot would be a child relevé Event with its own Survey.
- **Phenology and vitality:** some authors add a phenological state or a vitality sign to each value. These would be further occurrence assertions.
- **Syntaxon references:** the syntaxon should ideally point to the classification used (e.g. a national vegetation overview or EuroVegChecklist), through `assertionReferences`.

## References

- Braun-Blanquet, J. (1932). *Plant Sociology: the study of plant communities*. Translated by G. D. Fuller & H. S. Conard. McGraw-Hill, New York. https://archive.org/details/plantsociologyst00brau
- Barkman, J. J., Doing, H. & Segal, S. (1964). Kritische Bemerkungen und Vorschläge zur quantitativen Vegetationsanalyse. *Acta Botanica Neerlandica* 13: 394–419. https://natuurtijdschriften.nl/pub/539344
- Chytrý, M. & Otýpková, Z. (2003). Plot sizes used for phytosociological sampling of European vegetation. *Journal of Vegetation Science* 14: 563–570. https://doi.org/10.1111/j.1654-1103.2003.tb02183.x
- GBIF Relevé extension (2016). http://rs.gbif.org/extension/gbif/1.0/releve_2016-05-10.xml
- Palpurina, S. & Ingenloff, K. (2025). Mapping nested vegetation plot data to Darwin Core and the Humboldt Extension: challenges and open questions. *Biodiversity Information Science and Standards* 9: e182514. https://doi.org/10.3897/biss.9.182514
