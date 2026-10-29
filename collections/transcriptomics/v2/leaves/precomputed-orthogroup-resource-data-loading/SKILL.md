---
name: precomputed-orthogroup-resource-data-loading
description: Use when you need to enumerate which eukaryotic species have precomputed
  LCA-derived gene age assignments in oggmap's shipped Ensembl, eggNOG, or PLAZA resource
  files rather than re-running orthology inference.
when_to_use_negative:
- You need gene ages for a query species not covered by the shipped precomputed resources
  — instead extract an orthomap yourself from OrthoFinder or eggNOG results (of2orthomap,
  eggnog2orthomap, plaza2orthomap).
- Your goal is scRNA analysis (gene age-weighted expression, TEI) rather than cataloguing
  resource species coverage.
- Input is already a parsed per-species orthomap table — re-enumerating from the shipped
  datasets would duplicate work.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0080
tools:
- name: oggmap
  role: Python package whose datasets module ships the precomputed gene age class
    assignment files for Ensembl release-110, eggNOG v6, and PLAZA v5
  repo: https://github.com/kullrich/oggmap
- name: eggNOG
  role: source of homologous groups from which one of the precomputed orthomaps is
    derived (eggnog_6.0)
- name: PLAZA
  role: source of gene family data from which the precomputed dicot and monocot orthomaps
    are derived (plaza2orthomap input resource)
- name: OrthoFinder
  role: alternative orthogroup resource whose results oggmap can consume when precomputed
    coverage is insufficient
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:89593cd03e1bf15c4380ad9d1112eabe597c111a078183db1739e329a2b2c339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - PLAZA
  - Python
  - eggNOG
  - oggmap
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# precomputed-orthogroup-resource-data-loading

## Summary

Load and enumerate oggmap's shipped precomputed gene-age (orthomap) assignments from Ensembl release-110, eggNOG v6, and PLAZA v5 sources via the oggmap.datasets Python module, to quantify per-source eukaryotic species coverage.

## When to use

Use when you need to enumerate which eukaryotic species have precomputed LCA-derived gene age assignments in oggmap's shipped Ensembl, eggNOG, or PLAZA resource files rather than re-running orthology inference.

## When NOT to use

- You need gene ages for a query species not covered by the shipped precomputed resources — instead extract an orthomap yourself from OrthoFinder or eggNOG results (of2orthomap, eggnog2orthomap, plaza2orthomap).
- Your goal is scRNA analysis (gene age-weighted expression, TEI) rather than cataloguing resource species coverage.
- Input is already a parsed per-species orthomap table — re-enumerating from the shipped datasets would duplicate work.

## Inputs

- oggmap.datasets module with shipped precomputed gene-age assignment files for Ensembl release-110
- Precomputed eggNOG v6 (eggnog_6.0) orthologous map results shipped with oggmap
- Precomputed PLAZA v5 gene family / homologous group orthomaps shipped with oggmap (dicots and monocots)

## Outputs

- coverage_counts.csv table with per-source distinct species counts (Ensembl: 317, eggNOG: 1322, PLAZA: 98 dicots + 52 monocots) and summed total
- Enumerated list of distinct species possessing at least one LCA-derived gene age class assignment per source

## How to apply

Install oggmap (Python 3.10, via PyPI or docker) and use the oggmap.datasets module to load the shipped precomputed gene-age assignment files for Ensembl release-110, eggNOG v6 (eggnog_6.0), and PLAZA v5 gene family data. Parse each file to enumerate distinct species with at least one LCA-derived gene age class assignment, treating species-level uniqueness (not per-gene rows) as the counting unit. Partition PLAZA v5 counts into dicots and monocots (98 and 52 respectively) while aggregating Ensembl release-110 (317 species) and eggNOG v6 (1322 species) per source block. Sum per-source counts and write coverage_counts.csv, validating against the reported figures and confirming the total exceeds 1000 eukaryotic species.

## Related tools

- **oggmap** (Python package whose datasets module ships the precomputed gene age class assignment files for Ensembl release-110, eggNOG v6, and PLAZA v5) — https://github.com/kullrich/oggmap
- **eggNOG** (source of homologous groups from which one of the precomputed orthomaps is derived (eggnog_6.0))
- **PLAZA** (source of gene family data from which the precomputed dicot and monocot orthomaps are derived (plaza2orthomap input resource))
- **OrthoFinder** (alternative orthogroup resource whose results oggmap can consume when precomputed coverage is insufficient) — https://github.com/davidemms/OrthoFinder

## Examples

```
pip install oggmap; python -c "from oggmap import datasets; ensembl, eggnog, plaza = datasets.load_ensembl_orthomaps(), datasets.load_eggnog_orthomaps(), datasets.load_plaza_orthomaps()"
```

## Evaluation signals

- Per-source row counts match reported figures: Ensembl release-110 = 317 species, eggNOG v6 = 1322 species, PLAZA v5 = 98 dicots and 52 monocots.
- Summed per-source total exceeds 1000 eukaryotic species, consistent with the article's claim that 'oggmap provides extracted gene ages for more than thousand eukaryotic species'.
- Each counted species has at least one lowest common ancestor (LCA)-derived gene age class assignment (species-level deduplication, not raw row counts).
- coverage_counts.csv is written with per-source blocks including the PLAZA dicot/monocot partition.

## Limitations

- Precomputed coverage is fixed to specific resource versions (Ensembl release-110, eggNOG v6, PLAZA v5); other releases require regenerating orthomaps.
- Species coverage differs per source (317 vs 1322 vs 98+52), so a species of interest may be absent from one source despite presence in another.
- oggmap v0.0.2 requires Python 3.10 and does not support Python 2.7x or <=3.10 versions, per the methods section.

## Evidence

- [abstract] oggmap provides extracted gene ages for more than thousand eukaryotic species: "oggmap provides extracted gene ages for more than thousand eukaryotic species"
- [methods] extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class: "extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [methods] oggmap v0.0.2 developed with Python 3.10: "oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`."
- [methods] plaza2orthomap extracts orthomap from PLAZA gene family data: "plaza2orthomap      extract orthomap from PLAZA gene family data for query species"
- [readme] PyPI installation of oggmap: "$ pip install oggmap"
