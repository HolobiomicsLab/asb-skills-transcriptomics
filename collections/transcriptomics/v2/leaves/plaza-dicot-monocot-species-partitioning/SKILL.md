---
name: plaza-dicot-monocot-species-partitioning
description: Use when reproducing or updating species-coverage counts for precomputed
  gene age class assignments, specifically when the PLAZA source must be split into
  dicot vs monocot species blocks alongside Ensembl and eggNOG counts.
when_to_use_negative:
- The goal is to build a de novo orthomap for a single query species (use plaza2orthomap/of2orthomap
  extraction, not coverage counting).
- The input is already an aggregated coverage table or the species partition (dicot/monocot)
  is already annotated — recomputing partitions would be redundant.
- The source data is not PLAZA (e.g. OrthoFinder or orthomcl output), where this dicot/monocot
  partitioning does not apply.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: oggmap
  role: Python package providing the precomputed PLAZA v5 gene age assignments (oggmap.datasets)
    and the plaza2orthomap subcommand used to extract orthomaps from PLAZA gene family
    data
  repo: https://github.com/kullrich/oggmap
- name: Python
  role: Runtime for loading oggmap datasets, enumerating distinct species, and writing
    coverage_counts.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:1d7c6ee524ad9096d32ee71ab68a3fc4f897c64fae81b504abdfd59fe34963c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - PLAZA
  - Python
  - eggNOG
  - oggmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# plaza-dicot-monocot-species-partitioning

## Summary

Enumerate the eukaryotic species with precomputed LCA-derived gene age assignments from PLAZA gene family data via oggmap, and partition them into dicot and monocot lineages so per-source coverage counts (PLAZA v5: 98 dicots + 52 monocots) can be aggregated with Ensembl and eggNOG blocks.

## When to use

Use when reproducing or updating species-coverage counts for precomputed gene age class assignments, specifically when the PLAZA source must be split into dicot vs monocot species blocks alongside Ensembl and eggNOG counts.

## When NOT to use

- The goal is to build a de novo orthomap for a single query species (use plaza2orthomap/of2orthomap extraction, not coverage counting).
- The input is already an aggregated coverage table or the species partition (dicot/monocot) is already annotated — recomputing partitions would be redundant.
- The source data is not PLAZA (e.g. OrthoFinder or orthomcl output), where this dicot/monocot partitioning does not apply.

## Inputs

- oggmap-shipped precomputed PLAZA v5 gene age class assignment files (via oggmap.datasets)
- oggmap-shipped Ensembl release-110 and eggNOG v6 gene age assignment files for the aggregate coverage table
- Local NCBI taxonomy database (taxadb.sqlite, downloadable via oggmap ncbitax)

## Outputs

- coverage_counts.csv with per-source rows (Ensembl release-110: 317 species; eggNOG v6: 1322 species; PLAZA v5: 98 dicots and 52 monocots) and a summed total >1000 eukaryotic species
- Distinct-species lists per PLAZA clade partition (dicots vs monocots)

## How to apply

Load oggmap's shipped PLAZA v5 precomputed gene-age assignment files through the oggmap Python package (oggmap.datasets module), whose plaza2orthomap subcommand 'extract orthomap from PLAZA gene family data for query species'. Parse each file to enumerate the distinct species possessing at least one lowest common ancestor (LCA)-derived gene age class assignment, rather than counting rows, to avoid double-counting isoforms. Partition the distinct species set into dicots and monocots using NCBI taxonomy lineage information (oggmap's qlin/ncbitax helpers against the taxadb.sqlite database). Verify the partition reproduces the reported PLAZA v5 figures (98 dicots, 52 monocots, 150 total) and combine with the Ensembl release-110 (317 species) and eggNOG v6 (1322 species) blocks into a single coverage_counts.csv table whose summed total exceeds 1000 eukaryotic species. The rationale is that oggmap 'provides extracted gene ages for more than thousand eukaryotic species', so per-source, per-clade counts are the natural validation unit.

## Related tools

- **oggmap** (Python package providing the precomputed PLAZA v5 gene age assignments (oggmap.datasets) and the plaza2orthomap subcommand used to extract orthomaps from PLAZA gene family data) — https://github.com/kullrich/oggmap
- **Python** (Runtime for loading oggmap datasets, enumerating distinct species, and writing coverage_counts.csv)

## Examples

```
from oggmap import qlin; qlin.get_qlin(q='Arabidopsis thaliana', dbname='taxadb.sqlite')
```

## Evaluation signals

- Dicot partition size equals 98 distinct species and monocot partition size equals 52 distinct species for PLAZA v5.
- Enumerated species are distinct (deduplicated by species), so partition sizes match the reported per-source counts exactly.
- Aggregated table (Ensembl release-110 + eggNOG v6 + PLAZA v5) sums to a total exceeding 1000 eukaryotic species.
- Each species in the PLAZA blocks carries at least one LCA-derived gene age class assignment (no empty-assignment species counted).
- coverage_counts.csv exists and its rows match the four reported source-block figures.

## Limitations

- Counts are tied to the shipped PLAZA v5 snapshot; changing PLAZA releases changes dicot/monocot totals and invalidates direct comparison to 98/52.
- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or Python <=3.10, so older environments may fail.
- The article provides no dataset accessions or DOIs for the shipped precomputed files, so provenance must be taken from the oggmap package itself.

## Evidence

- [methods] plaza2orthomap: "plaza2orthomap      extract orthomap from PLAZA gene family data for query species"
- [abstract] coverage claim: "oggmap provides extracted gene ages for more than thousand eukaryotic species"
- [readme] package scope: "is a python package to extract orthologous maps
(short: orthomap or in other words the evolutionary age of a given orthologous group) from OrthoFinder or eggNOG results"
- [readme] lineage query: [evidence span withheld — anchor status: external_doc]
- [methods] python version constraint: "oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`."
