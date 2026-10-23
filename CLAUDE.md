# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A documentation-only repository (Markdown and example data, no code, build or lint) of case studies showing how to *structure* common biodiversity survey and monitoring protocols using the Darwin Core Data Package (DwC-DP), with an explanation of the design choices made. Many case studies are accompanied by examples.

Each case study is a folder (e.g. `bbs/`) with a `README.md`, an `example/` DwC-DP package (`datapackage.json` + CSVs) and an `example.zip` of that package. Each case study is listed in the "Case studies" table of the top-level `README.md`.

## Working on example packages

The only "test" is validating a package; the only build step is re-zipping it. Whenever an `example/` changes:

```sh
frictionless validate <case-study>/example/datapackage.json   # see the skills file for the empty-FK-table workaround
cd <case-study> && rm -f example.zip && zip -X -j -D example.zip example/*
```

CSV columns must contain every schema field in schema order (fields are matched by position). Check table and field names against the ratified JSON schemas, not memory.

Key references:
- Darwin Core Conceptual Model: https://dwc.tdwg.org/cm/
- DwC-DP table schemas: http://rs.tdwg.org/dwc-dp/1.0/table-schemas/

## Proposing a mapping for a new protocol

`dwc-dp-survey-skills.md` is the instruction set for LLMs proposing structural mappings for new protocols; read and follow it when asked to map a protocol (it covers output layout, schema pitfalls, modelling guidance and how to validate example packages). Update it with lessons learned when a new case study surfaces them.

When editing `dwc-dp-survey-skills.md` itself, remember it is read by other LLMs as instructions, so keep it self-contained and unambiguous.
