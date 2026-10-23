# Case studies for structuring Survey protocols in Darwin Core Data Package

The [Darwin Core Conceptual Model](https://dwc.tdwg.org/cm/) and its implementation through the [Darwin Core Data Package schemas](http://rs.tdwg.org/dwc-dp/1.0/table-schemas/) provides a rich structure for capturing data. 

This repository provides case studies that demonstrate how to _structure_ common survey protocols using DwC-DP, with explanation of the design choices taken. Many are accompanied with examples.

## Case studies

| Protocol | Summary | Example |
|---|---|---|
| [UK Breeding Bird Survey (BBS)](./bbs/) | Repeated line-transect bird counts by section and distance band in 1-km squares, with habitat, weather, optional mammal recording and colony counts | [example](./bbs/example/) |
| [Butterfly transect / Pollard walk (UKBMS)](./pollard-walk/) | Weekly fixed-route butterfly counts within a 5 m recording box, by section, with weather and section habitat/management | [example](./pollard-walk/example/) |
| [Car net sampling with DNA metabarcoding (Insektmobilen)](./insektmobil/) | Roof-mounted car nets driven along fixed routes; bulk samples identified by COI metabarcoding, with material, sequences and DNA-based identifications | [example](./insektmobil/example/) |
| [UK PoMS Flower-Insect Timed Count (FIT Count)](./fit-count/) | 10-minute counts of insects visiting a 50 × 50 cm patch of a target flower, in ten insect groups | [example](./fit-count/example/) |

In addition, the [dwc-dp-survey-skills.md](./dwc-dp-survey-skills.md) provides instructions to aid LLMs in helping propose structural mappings for new protocols.

Contributions to this repository are very welcome through pull requests.