---
name: species-coverage-summary-tabulation
description: Use when you need to quantify how many species are covered by oggmap's
  precomputed gene age assignments across Ensembl, eggNOG, or PLAZA sources, or to
  verify reported coverage totals before relying on shipped orthomaps.
when_to_use_negative:
- You need gene age assignments for a species not present in the shipped precomputed
  files — instead extract a fresh orthomap with of2orthomap or plaza2orthomap from
  OrthoFinder, eggNOG, orthomcl, or PLAZA results.
- Your goal is gene-age-weighted expression (TEI) analysis on scRNA data rather than
  coverage statistics — this tabulation skill does not touch Scanpy objects or expression
  data.
- The input is raw orthogroup output (OrthoFinder/eggNOG results) rather than oggmap's
  already-extracted gene age class files — use the appropriate orthomap extraction
  sub-command first.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0621
tools:
- name: oggmap
  role: Python package whose oggmap.datasets module ships the precomputed gene age
    class assignment files for Ensembl release-110, eggNOG v6, and PLAZA v5 that are
    parsed and tabulated
  repo: https://github.com/kullrich/oggmap
- name: eggNOG
  role: Source of homologous groups from which one precomputed gene age coverage block
    (eggnog_6.0, 1322 species) is derived
- name: PLAZA
  role: Source of gene family data from which the PLAZA v5 coverage block (98 dicots
    + 52 monocots) is derived
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:cfa2f20ff1e88a8d82ee5d9ae525289fb96f5ee644ad059be743f2b631fe4d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - PLAZA
  - Python
  - eggNOG
  - oggmap
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# species-coverage-summary-tabulation

## Summary

Enumerate and tabulate the eukaryotic species coverage of oggmap's precomputed gene age class assignments by loading the shipped Ensembl release-110, eggNOG v6, and PLAZA v5 files, counting distinct species with at least one LCA-derived gene age assignment per source, and writing the per-source counts (317 Ensembl species; 1322 eggNOG species; 98 PLAZA dicots + 52 PLAZA monocots) to a summary table.

## When to use

Use when you need to quantify how many species are covered by oggmap's precomputed gene age assignments across Ensembl, eggNOG, or PLAZA sources, or to verify reported coverage totals before relying on shipped orthomaps.

## When NOT to use

- You need gene age assignments for a species not present in the shipped precomputed files — instead extract a fresh orthomap with of2orthomap or plaza2orthomap from OrthoFinder, eggNOG, orthomcl, or PLAZA results.
- Your goal is gene-age-weighted expression (TEI) analysis on scRNA data rather than coverage statistics — this tabulation skill does not touch Scanpy objects or expression data.
- The input is raw orthogroup output (OrthoFinder/eggNOG results) rather than oggmap's already-extracted gene age class files — use the appropriate orthomap extraction sub-command first.

## Inputs

- oggmap precomputed gene age class assignment files for Ensembl release-110 (shipped via oggmap.datasets)
- oggmap precomputed gene age class assignment files for eggNOG v6 (eggnog_6.0)
- oggmap precomputed gene age class assignment files for PLAZA v5 gene family data (dicot and monocot blocks)

## Outputs

- Per-source species coverage counts table (e.g. coverage_counts.csv) with rows for Ensembl release-110 (317 species), eggNOG v6 (1322 species), PLAZA v5 dicots (98), and PLAZA v5 monocots (52)
- Summed total eukaryotic species coverage count (>1000)

## How to apply

Load the precomputed gene-age assignment files shipped with oggmap (via the oggmap.datasets module) for each source block: Ensembl release-110, eggNOG v6 (eggnog_6.0), and PLAZA v5 gene family data. Parse each file and enumerate the distinct species that possess at least one lowest common ancestor (LCA)-derived gene age class assignment, so that a species with many annotated genes is counted only once. For PLAZA v5, partition the species counts into dicots and monocots; for Ensembl and eggNOG, aggregate the counts per source block. Sum the per-source counts to obtain the total eukaryotic species coverage (which should exceed 1000, matching the paper's claim) and assemble the counts into a single table, e.g. coverage_counts.csv, with one row per source and columns for species counts.

## Related tools

- **oggmap** (Python package whose oggmap.datasets module ships the precomputed gene age class assignment files for Ensembl release-110, eggNOG v6, and PLAZA v5 that are parsed and tabulated) — https://github.com/kullrich/oggmap
- **eggNOG** (Source of homologous groups from which one precomputed gene age coverage block (eggnog_6.0, 1322 species) is derived)
- **PLAZA** (Source of gene family data from which the PLAZA v5 coverage block (98 dicots + 52 monocots) is derived)

## Examples

```
pip install oggmap; python -c "from oggmap import datasets; ens, egg, plaza = datasets.ensembl_110(), datasets.eggnog_v6(), datasets.plaza_v5()"
```

## Evaluation signals

- Ensembl release-110 block counts exactly 317 distinct species with at least one LCA-derived gene age class assignment.
- eggNOG v6 block counts exactly 1322 distinct species.
- PLAZA v5 block partitions into exactly 98 dicot species and 52 monocot species.
- The summed per-source total exceeds 1000 eukaryotic species, consistent with the paper's claim that oggmap provides gene ages for more than a thousand eukaryotic species.
- Species are de-duplicated per source: a species with many gene age assignments is counted once, and the table written to coverage_counts.csv matches the reported per-source row counts.

## Limitations

- Coverage figures are tied to specific source releases (Ensembl release-110, eggNOG v6, PLAZA v5) and will not match if files from other releases are parsed.
- oggmap v0.0.2 was developed using Python 3.10; Python 2.7x and Python <=3.10 outside that version are not supported, so the oggmap.datasets loading code requires a compatible environment.
- The species counts reflect species with at least one LCA-derived gene age assignment, not full gene-by-gene coverage per species.

## Evidence

- [abstract] coverage claim: "oggmap provides extracted gene ages for more than thousand eukaryotic species"
- [methods] LCA assignment mechanism: "extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [methods] PLAZA source: "plaza2orthomap      extract orthomap from PLAZA gene family data for query species"
- [methods] eggNOG source version: "extract orthologous maps from `eggnog_6.0 <http://eggnog6.embl.de/>`_ results."
- [readme] installation context for parsing scripts: "oggmap is a python package to extract orthologous maps
(short: `orthomap` or in other words the evolutionary age of a given orthologous group)"
