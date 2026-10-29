---
name: orthomap-gene-age-coverage-counting
description: Use when the task requires quantifying species coverage of precomputed
  gene age / orthomap assignments in oggmap, e.g. verifying that oggmap covers 317
  (Ensembl release-110), 1322 (eggNOG v6), and 98 dicots + 52 monocots (PLAZA v5)
  eukaryotic species.
when_to_use_negative:
- The goal is to generate a NEW orthomap for a single query species from your own
  OrthoFinder, eggNOG, or PLAZA results — use the oggmap subcommands (of2orthomap,
  plaza2orthomap) instead of counting shipped coverage files.
- The goal is to calculate gene age-weighted expression or the transcriptome evolutionary
  index (TEI) on scRNA data — this skill only counts species coverage and produces
  no expression values.
- The input is a homologous-group file not shipped with oggmap (custom or newer-source
  releases); counting its species would not reproduce the reported Ensembl release-110
  / eggNOG v6 / PLAZA v5 figures.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0621
tools:
- name: oggmap
  role: Python package whose datasets module ships the precomputed gene-age (orthomap)
    files that are parsed and counted; also provides ncbitax/qlin for taxonomy support
  repo: https://github.com/kullrich/oggmap
- name: Python
  role: Runtime used to load the oggmap datasets, enumerate distinct species per source,
    and write coverage_counts.csv (oggmap v0.0.2 requires Python 3.10)
- name: eggNOG
  role: Source of homologous groups underlying the precomputed eggNOG v6 gene-age
    assignments (1322 species) counted by this skill
- name: PLAZA
  role: Source of gene family data underlying the precomputed PLAZA v5 gene-age assignments
    partitioned into dicots (98) and monocots (52)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:40feef44c80ec3a715287813dc77384c68378458c8dc85650b389552c7f4f335
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - PLAZA
  - Python
  - eggNOG
  - oggmap
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# orthomap-gene-age-coverage-counting

## Summary

Enumerates how many eukaryotic species have precomputed gene age class (orthomap) assignments shipped with oggmap across three sources — Ensembl release-110, eggNOG v6, and PLAZA v5 (dicots vs monocots) — by loading oggmap's packaged gene-age files and counting distinct species with at least one LCA-derived age class, then validating against the reported coverage figures.

## When to use

Use when the task requires quantifying species coverage of precomputed gene age / orthomap assignments in oggmap, e.g. verifying that oggmap covers 317 (Ensembl release-110), 1322 (eggNOG v6), and 98 dicots + 52 monocots (PLAZA v5) eukaryotic species.

## When NOT to use

- The goal is to generate a NEW orthomap for a single query species from your own OrthoFinder, eggNOG, or PLAZA results — use the oggmap subcommands (of2orthomap, plaza2orthomap) instead of counting shipped coverage files.
- The goal is to calculate gene age-weighted expression or the transcriptome evolutionary index (TEI) on scRNA data — this skill only counts species coverage and produces no expression values.
- The input is a homologous-group file not shipped with oggmap (custom or newer-source releases); counting its species would not reproduce the reported Ensembl release-110 / eggNOG v6 / PLAZA v5 figures.

## Inputs

- oggmap shipped precomputed gene-age assignment file for Ensembl release-110 (expect 317 species)
- oggmap shipped precomputed gene-age assignment file for eggNOG v6 (http://eggnog6.embl.de/, expect 1322 species)
- oggmap shipped precomputed gene-age assignment file for PLAZA v5 gene family data (expect 98 dicots and 52 monocots)
- Optional: local NCBI taxonomy database (taxadb.sqlite) for lineage-aware species resolution

## Outputs

- coverage_counts.csv with per-source eukaryotic species counts (Ensembl release-110, eggNOG v6, PLAZA v5 dicots, PLAZA v5 monocots)
- Aggregated total eukaryotic species coverage figure (>1000)

## How to apply

Load oggmap's shipped precomputed gene-age assignment files for the three sources via the oggmap Python package's datasets module (Ensembl release-110, eggNOG v6 from eggnog6.embl.de, and PLAZA v5 gene family data), rather than re-running orthology inference. Parse each source file and enumerate the distinct species that possess at least one lowest common ancestor (LCA)-derived gene age class assignment, since oggmap assigns gene ages by taking the LCA of orthologous groups. Partition the PLAZA v5 counts into dicots (expected 98) and monocots (expected 52), and aggregate the Ensembl release-110 and eggNOG v6 counts per source block. Sum the per-source counts to obtain total eukaryotic species coverage and write the assembled table to coverage_counts.csv. Validate per-source row counts against the reported figures (317, 1322, 98, 52) and confirm the summed total exceeds 1000 eukaryotic species.

## Related tools

- **oggmap** (Python package whose datasets module ships the precomputed gene-age (orthomap) files that are parsed and counted; also provides ncbitax/qlin for taxonomy support) — https://github.com/kullrich/oggmap
- **Python** (Runtime used to load the oggmap datasets, enumerate distinct species per source, and write coverage_counts.csv (oggmap v0.0.2 requires Python 3.10))
- **eggNOG** (Source of homologous groups underlying the precomputed eggNOG v6 gene-age assignments (1322 species) counted by this skill)
- **PLAZA** (Source of gene family data underlying the precomputed PLAZA v5 gene-age assignments partitioned into dicots (98) and monocots (52))

## Evaluation signals

- Per-source row counts match the reported figures exactly: Ensembl release-110 = 317 species, eggNOG v6 = 1322 species, PLAZA v5 = 98 dicots and 52 monocots
- The summed per-source total exceeds 1000 eukaryotic species, consistent with the abstract's claim that 'oggmap provides extracted gene ages for more than thousand eukaryotic species'
- Each enumerated species in a source block has at least one non-empty LCA-derived gene age class assignment (no species counted purely from metadata)
- PLAZA v5 block partitions cleanly into dicot and monocot sub-blocks with no overlap or unclassified species
- coverage_counts.csv is produced and its rows are keyed by source block (Ensembl, eggNOG, PLAZA-dicot, PLAZA-monocot) with consistent column semantics

## Limitations

- Coverage figures are tied to specific shipped resource versions (Ensembl release-110, eggNOG v6, PLAZA v5); counting files from other releases will not reproduce these numbers
- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or Python <=3.10, so the counting environment must match
- Species lineage resolution relies on the local NCBI taxonomy database (~150MB download via oggmap ncbitax); a stale or missing taxadb.sqlite can affect species enumeration

## Evidence

- [abstract] oggmap provides extracted gene ages for more than thousand eukaryotic species: "oggmap provides extracted gene ages for more than thousand eukaryotic species"
- [methods] extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species.: "extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [methods] oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`.: "oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`."
- [methods] plaza2orthomap extract orthomap from PLAZA gene family data for query species: "plaza2orthomap              extract orthomap from PLAZA gene family data for query species"
- [readme] extract orthologous maps (orthomap / evolutionary age of a given orthologous group) from OrthoFinder or eggNOG results: [evidence span withheld — anchor status: external_doc]
- [readme] The command downloads or updates the local NCBI taxonomy database (~150MB) stored as taxadb.sqlite: [evidence span withheld — anchor status: external_doc]
