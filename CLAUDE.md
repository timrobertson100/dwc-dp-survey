# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A documentation-only repository (Markdown, no code, build, lint or tests) of case studies showing how to *structure* common biodiversity survey and monitoring protocols using the Darwin Core Data Package (DwC-DP), with an explanation of the design choices made. Many case studies are accompanied by examples.

Each case study is a folder (e.g. `bbs/`) with a `README.md` and an `example/` DwC-DP package (`datapackage.json` + CSVs).

Key references:
- Darwin Core Conceptual Model: https://dwc.tdwg.org/cm/
- DwC-DP table schemas: http://rs.tdwg.org/dwc-dp/1.0/table-schemas/

## Proposing a mapping for a new protocol

`dwc-dp-survey-skills.md` is the instruction set for LLMs proposing structural mappings for new protocols; read and follow it when asked to map a protocol (it covers output layout, schema pitfalls, modelling guidance and how to validate example packages). Update it with lessons learned when a new case study surfaces them.

When editing `dwc-dp-survey-skills.md` itself, remember it is read by other LLMs as instructions, so keep it self-contained and unambiguous.
