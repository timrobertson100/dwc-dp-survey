# Car net sampling with DNA metabarcoding (Insektmobilen)

Flying insects are caught in a large net mounted on a car roof while the car is driven along a fixed route. Each drive's catch is a bulk sample, which is identified by DNA metabarcoding rather than by sorting. This case study maps the design of the Danish *Insektmobilen* citizen science project, as described in [Svenningsen et al. (2021)](https://doi.org/10.1098/rsbl.2020.0833). The paper refers to its supplement and an earlier paper for the net and route details. Those details are taken from the project's [published GBIF dataset](https://doi.org/10.15468/m48e3e), which also shows how the authors themselves published the data. An [illustrative example package](example/) for one route on one day accompanies it.

The case brings in two things the other case studies don't have: **material** (the bulk sample and its DNA extract) and **DNA-derived identifications**.

## The protocol in brief

- **Routes:** 5 km routes designed by hand, placed in forest, urban, farmland, wetland and grassland. Each route's dominant land cover and biogeographic region are later derived by GIS.
- **Net:**
  - **Shape:** funnel-shaped, mounted on the car roof, with a detachable sampling bag at the rear.
  - **Dimensions:** the front is 75 cm high and 100 cm wide, the net is 140 cm long, and the bag is 29 cm wide.
  - **Mesh:** about 0.3 mm, and 2 × 1 mm on the bottom fabric.
- **Sampling:**
  - **Driving:** the volunteer drives the route from start to end and back again, so about 10 km, at no more than 50 km/h.
  - **Timing:** each route is driven once between 12:00 and 15:00 and once between 17:00 and 20:00 on the same day, in June. In the paper, 151 volunteers took 365 samples on 218 routes.
  - **Weather:** at least 15 °C, average wind no more than 6 m/s, and no rain. Volunteers record temperature and wind in classes.
  - **Samples:** each drive's sampling bag is one sample. It is put in 96% ethanol and stored at −20 °C.
- **Laboratory:**
  - **Extraction and sequencing:** non-destructive bulk DNA extraction, so the insects are kept. The COI marker is amplified with the fwh primers (fwhF2 / fwhR2n), and sequenced on Illumina with paired reads. The protocol is on [protocols.io](https://dx.doi.org/10.17504/protocols.io.bmunk6ve).
  - **Bioinformatics:** demultiplexing with cutadapt, ASVs and chimera removal with DADA2, and removal of redundant ASVs with LULU.
  - **Taxonomic assignment:** the GBIF sequence ID tool (BLAST against BOLD). The paper accepts names at a 99% match or better across the full query. The GBIF dataset reports a match type for each ASV.
- **Outputs:** read counts per ASV per sample. The paper converts these to presence/absence for species analyses and rarefies them for diversity. The GBIF dataset also gives each sample's biomass.

## Event structure

```text
Route sampling day     R0001-2018-06-12            context · route sampling day
├── Midday drive          …-MIDDAY                 survey  · car net drive   → bulk sample → DNA extract
└── Evening drive         …-EVENING                survey  · car net drive   → bulk sample → DNA extract
```

- **The route sampling day is the parent.** Both drives happen on the same day on the same route. This level holds the GIS-derived route attributes: dominant land cover and biogeographic region, as `event-assertion` rows whose protocol is `P-LANDCOVER` and whose `assertionBy` is the research team. They are not observed in the field, and this marks them as such.
- **Each drive is the sampling Event and holds the Survey.** One drive produces one sample, so the drive is the finest unit at which anything is detected.
  - **Time:** `eventTime` gives the drive's start and end in local summer time (`13:02+02:00/13:23+02:00`), and `eventDurationValue` gives its length.
  - **Weather:** the volunteer's temperature and wind classes are event assertions.
  - **Place:** the route is in `footprintWKT`, and `locationID` stays the same for the route across days and years.
- **Lab steps are not Events.** The lab steps (extraction, PCR, sequencing) are described by the molecular protocol rather than modelled as Events. `nucleotide-analysis.event_fk` points to the **drive**, because the sequences are evidence of what was flying there and then. The dates of lab work are not needed to interpret the occurrences.

## Protocols

| `protocol_pk` | Used by |
|---|---|
| `P-CARNET` | Drive Events and Surveys: net design, route driving, timing and weather criteria |
| `P-LANDCOVER` | GIS-derived route assertions |
| `P-SEQID` | Identifications: the GBIF sequence ID tool and the meaning of its match types |

The lab and bioinformatics method goes in `molecular-protocol`, described below, not in `protocol`.

## Surveys and survey targets

Each drive has one `survey`:

| Field | Value | Why |
|---|---|---|
| `surveySiteType` | `horizontalTransect` | |
| `sampleSizeValue` / `sampleSizeUnit` | `10000` / `metre` | The route out and back |
| `samplingEffortValue` / `samplingEffortUnit` | drive duration / `minutes of driving` | |
| `isSamplingEffortReported` | `true` | |
| `isAbundanceReported` | **`false`** | Read counts are not organism counts |
| `isAbsenceReported` | `false` | |
| `hasMaterialSamples` / `materialSampleTypes` | `true` / `bulk sample \| DNA extract` | |
| `hasVouchers` / `voucherInstitutions` | `true` / `Natural History Museum of Denmark` | |
| `hasNonTargetTaxa` | `true` when non-insect ASVs are reported | |
| `areNonTargetTaxaFullyReported` | `false` | |

One SurveyTarget, `ST-CARNET-INSECTS`, is shared by every drive Survey. It has two descriptors: include `taxon` = Insecta, and include `behavior` = in flight. It has `isSurveyTargetFullyReported = true`, because every ASV the pipeline retains is reported. That is an honest statement of what was *reported*, but it is weak evidence of absence:

- **Catchability:** the net only catches insects flying at roof height. The paper notes that Dictyoptera, Megaloptera, Raphidioptera and Strepsiptera were never detected.
- **Detectability:** the primers amplify some groups better than others, and the reference database has gaps, particularly for small Diptera and Hymenoptera.

`surveyTargetRemarks` says this explicitly. A data user wanting stronger non-detection claims needs the molecular protocol as well as the flag.

The primers also amplify non-insects (spiders, mites, and DNA from gut contents). These are reported as Occurrences without a `surveyTarget_fk`, with `hasNonTargetTaxa = true`.

## Material

Each drive produces two `material` records:

| Material | Key fields |
|---|---|
| Bulk sample `…-BULK` | `collectionEvent_fk` = drive · `materialEntityCategory = preserved` · `materialEntityType = bulk sample` · `preparations = whole organisms (96% ethanol)` · `measuredMassInGrams` (biomass) · `disposition = in collection` |
| DNA extract `…-DNA` | `derivedFromMaterialEntity_fk` = bulk sample · `materialEntityCategory = DNA extract` · `disposition = in collection` |

The extraction is non-destructive, so the bulk sample stays `in collection` and is a voucher for every occurrence detected in it. Individual specimens can later be picked from it for morphological identification or reference barcoding. Those specimens would become further `material` rows with `isPartOfMaterialEntity_fk` set to the bulk sample.

Biomass is mass of the whole catch, so it belongs on the bulk sample (`measuredMassInGrams`). It is not an Occurrence quantity.

## DNA-derived data

| Table | One row per | Holds |
|---|---|---|
| `molecular-protocol` | DNA extract | `assayType = metabarcoding`, `source_mat_fk` = extract, `target_gene = COI`, primer names, sequences and reference, `seq_meth`, `lib_layout`, `chimera_check`, `otu_class_appr` (DADA2 + LULU), `otu_db` (reference database and version), `nucl_acid_ext` and `sop` (protocols.io), DNA `concentration` |
| `nucleotide-sequence` | ASV | the ASV sequence. The ID is its MD5 hash (the convention in the GBIF dataset), so the same ASV in different samples is the same row |
| `nucleotide-analysis` | sample × ASV | `event_fk` = drive, `materialEntity_fk` = DNA extract, `nucleotideSequence_fk`, `molecularProtocol_fk`, `readCount`, `processedTotalReadCount`, `rawTotalReadCount` |
| `occurrence` | sample × ASV | `event_fk` = drive, `surveyTarget_fk`, `occurrenceStatus = detected`, accepted name and rank |
| `identification` | occurrence | `identificationType = nucleotideAnalysis`, `occurrence_fk`, `nucleotideAnalysis_fk`, `nucleotideSequence_fk`, `identificationProtocol_fk = P-SEQID`, the match details in `identificationRemarks` |
| `identification-taxon` | candidate taxon | the candidates when a match is ambiguous |

A single `molecular-protocol` row shared by all samples would be smaller. It is one row per extract because the schema makes it a sample-level record: it has `samp_name` and `source_mat_fk`, and holds the extract's concentration.

The design rests on these choices:

- **One Occurrence per sample × ASV, not per species.** Several ASVs can match the same species, and some ASVs match nothing named. Merging to species is an analysis step, and keeping ASVs keeps every Occurrence tied to exactly one sequence and one read count. A data user can aggregate by `scientificName`, but cannot split an aggregate.
- **Reads stay in `nucleotide-analysis`.** `occurrence.organismQuantity` is left empty. Reads measure DNA, not individuals, and depend on primer bias and sequencing depth. GBIF's guide for publishing DNA-derived data in a Darwin Core Archive puts reads in `organismQuantity` with `organismQuantityType = DNA sequence reads`. That is the only option in an archive, but in DwC-DP the reads have a proper home. `isAbundanceReported = false` on the Survey says the same thing.
- **The accepted name reflects the strength of the match.**
  - **Exact matches** (identity of 99% or more) are accepted at species.
  - **Close matches** (90–99%) are accepted at genus, keeping the best-matching species in `verbatimIdentification`. In the example, the *Chironomus* ASV is a 95.1% match to *C. riparius*.
  - **Ambiguous matches** use `taxonFormula = A or B` and list both candidates in `identification-taxon`, with the shared genus accepted on the Occurrence. In the example this is *Lucilia caesar* or *L. illustris*.
- **Identifications can be redone.** A re-run against a newer reference database is a new `identification` row with `isAcceptedIdentification = true`, with the old row set to `false`. Occurrences and sequences don't change.

## Assertions

| Attached to | `assertionType` | Values |
|---|---|---|
| Route sampling day | `dominant land cover` | farmland · forest · urban (50% or more) · mixed. Derived by GIS |
| Route sampling day | `biogeographic region` | e.g. Funen, East Jutland. Derived |
| Drive | `temperature class` | the volunteer's class, e.g. `20-25` °C |
| Drive | `wind class` | the volunteer's class, e.g. `Light breeze 1.6-3.3 m/s` |

## Example

[`example/`](example/) contains one fictitious route on 12 June 2018:

- **Midday drive:**
  - seven ASVs, including an ambiguous *Lucilia* match, a close-match *Chironomus*, and a non-target spider
  - biomass of 47 mg
- **Evening drive:**
  - four ASVs, all also found at midday, so they reuse the same `nucleotide-sequence` rows
  - biomass of 112 mg

The sequences are random and the read counts are made up. Only the structure is meant to be realistic.

The tables follow the ratified DwC-DP 1.0 schemas and have been checked with a validator.

## Open questions

- **Detection threshold:** the paper does not say whether ASVs with very few reads in a sample were removed. That threshold defines what "detected" means, and so what `isSurveyTargetFullyReported` covers. It should be stated in the molecular protocol.
- **Controls:**
  - Neither the paper nor the GBIF dataset says how extraction and PCR negatives and positive controls are handled.
  - If control results are published, they would be further `nucleotide-analysis` rows on controls recorded as `material`, linked from `molecular-protocol.neg_cont_type` and `pos_cont_type`.
- **Biomass:** is it wet mass after ethanol, or dry mass? `measuredMassInGrams` doesn't say, so the method belongs in `materialEntityRemarks` or a `material-assertion`.
- **Taxonomic assignment:** the paper's 2020 assignment and the dataset's re-assignment against BOLD (2024) may differ. Which is the accepted identification should be stated. Both could be kept as separate `identification` rows.
- **Route design:** stops, turns and road surface affect the catch. Are route-level variables (number of stops, unpaved length) available, so they can be published as route assertions?

## References

- Svenningsen, C. S. et al. (2021). Detecting flying insects using car nets and DNA metabarcoding. *Biology Letters* 17: 20200833. https://doi.org/10.1098/rsbl.2020.0833
- Insektmobilen – National citizen science and DNA metabarcoding survey of flying insects in June 2018 and 2019. Natural History Museum of Denmark, via GBIF. https://doi.org/10.15468/m48e3e
- Vamos, E., Elbrecht, V. & Leese, F. (2017). Short COI markers for freshwater macroinvertebrate metabarcoding. *Metabarcoding and Metagenomics* 1: e14625. https://doi.org/10.3897/mbmg.1.14625
