---
name: transcriptome-evolutionary-index-calculation
description: Use when single-cell RNA data (Scanpy AnnData) needs a gene age-weighted
  expression index per cell, stage, or cell type — e.g. to detect whether young gene
  age classes peak in early embryos — or when per-class partial TEI / relative expression
  profiles are required.
when_to_use_negative:
- Input lacks an orthomap or any per-gene category (gene age class or binned metric)
  — the TEI weighting cannot be computed without a gene-to-class mapping matched to
  the scRNA identifiers
- The goal is standard differential expression or clustering rather than evolutionary-age-weighted
  expression profiles — use standard Scanpy workflows instead
- Data is not a Scanpy-compatible scRNA object (oggmap's TEI calculation is implemented
  on Scanpy AnnData objects)
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: oggmap
  role: Provides orthomap2tei ID matching, get_TEI, get_pstrata (partial TEI), get_rematrix
    (relative expression per age class), and get_bins (arbitrary metric binning) used
    to compute the weighted expression index
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Hosts the scRNA AnnData objects on which oggmap calculates the gene age-weighted
    expression data
- name: OrthoFinder
  role: Source of orthogroup results from which oggmap extracts the orthomap (gene
    ages per orthogroup) used as weights
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:5ea0ca1e9722d8f665e2371efe5fe2e59b372248a283fd1cfc1f88bfa1f80227
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# transcriptome-evolutionary-index-calculation

## Summary

Computes the transcriptome evolutionary index (TEI) — a gene age-weighted expression index — per cell or per annotated stage/group from single-cell RNA data using oggmap's TEI module on Scanpy AnnData objects. Also covers partial TEI (per age-class contributions that sum to the global TEI) and relative expression per gene age class via get_rematrix.

## When to use

Use when single-cell RNA data (Scanpy AnnData) needs a gene age-weighted expression index per cell, stage, or cell type — e.g. to detect whether young gene age classes peak in early embryos — or when per-class partial TEI / relative expression profiles are required.

## When NOT to use

- Input lacks an orthomap or any per-gene category (gene age class or binned metric) — the TEI weighting cannot be computed without a gene-to-class mapping matched to the scRNA identifiers
- The goal is standard differential expression or clustering rather than evolutionary-age-weighted expression profiles — use standard Scanpy workflows instead
- Data is not a Scanpy-compatible scRNA object (oggmap's TEI calculation is implemented on Scanpy AnnData objects)

## Inputs

- Scanpy AnnData scRNA count object (e.g. Packer et al. 2019 C. elegans embryonic dataset, ~90,000 cells)
- orthomap table mapping gene/transcript IDs to gene age class (e.g. C. elegans orthomap from Sun et al. 2021)
- optional per-gene metric table (e.g. Tajima's D or nucleotide diversity per gene) for get_bins-based weighting
- annotated observation column (cell type or sampling timepoint) for grouped aggregation

## Outputs

- per-cell/per-stage TEI values (gene age-weighted expression index) annotated on the AnnData object
- stacked partial TEI table (age-class contributions summing to global TEI per stage) and stacked profile figure (e.g. nematode_partial_tei.png)
- relative expression per age class table (rows = observation groups, columns = gene age classes; e.g. relative_expression_per_age_class.csv)
- binning_map.tsv (gene-to-bin-group mapping) and binned_index_per_stage.tsv when non-age metric bins are used

## How to apply

Load the scRNA count object (AnnData via Scanpy) together with an orthomap (gene age class per gene, e.g. from Sun et al. 2021 for C. elegans) and use oggmap's orthomap2tei module to match gene/transcript IDs and their gene age class against the scRNA identifiers. Then call oggmap's get_TEI to compute the gene age-weighted expression index (TEI) per cell or per annotated stage. To visualize contributions of individual age classes, run get_pstrata to compute stacked partial TEI values such that each age class's contribution sums to the global TEI per stage, then plot the stacked profile across stages. For grouped summaries, use get_rematrix to compute mean/relative expression per gene age class grouped by any annotated observation (cell type or sampling timepoint), from raw counts by default or an implemented transformation (e.g. normalized/log-transformed layer), normalizing within each group so age-class shares sum to 1. oggmap also supports arbitrary gene-based metric bins (e.g. Tajima's D or nucleotide diversity bins via get_bins) in place of evolutionary gene age classes as weights.

## Related tools

- **oggmap** (Provides orthomap2tei ID matching, get_TEI, get_pstrata (partial TEI), get_rematrix (relative expression per age class), and get_bins (arbitrary metric binning) used to compute the weighted expression index) — https://github.com/kullrich/oggmap
- **Scanpy** (Hosts the scRNA AnnData objects on which oggmap calculates the gene age-weighted expression data)
- **OrthoFinder** (Source of orthogroup results from which oggmap extracts the orthomap (gene ages per orthogroup) used as weights) — https://github.com/davidemms/OrthoFinder

## Examples

```
from oggmap import orthomap2tei as o2t; adata = o2t.get_TEI(adata, gene_age_table, obs='embryonic_stage')
```

## Evaluation signals

- Partial TEI table exists and contains the expected age classes (e.g. age classes 11–13 for C. elegans), with young classes peaking in early-embryo stages
- Relative expression shares within each observation group sum to 1, with each value in (0,1], and the table has one column per gene age class
- Stacked partial TEI values per gene age class sum to the global TEI per stage
- Gene/transcript IDs in the orthomap (or binning map) are successfully matched against the identifiers in the scRNA dataset
- Rendered stacked partial TEI figure matches the reference pattern (e.g. Fig. 1e of Ma and Zheng 2023 re-analysis)

## Limitations

- oggmap v0.0.2 was developed using Python 3.10; Python 2.7x and Python <=3.10 are not supported (per methods statement)
- gene/transcript ID mismatches between the orthomap and the scRNA dataset must be resolved by the oggmap orthomap-to-scRNA matching step before TEI computation
- the article's extracted discussion content is only a release changelog, so no explicit limitations or data availability statements are documented there

## Evidence

- [abstract] can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit: "can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [methods] calculate and visualize partial TEI profiles: "calculate and visualize partial transcriptome evolutionary index (TEI) profiles on scRNA data sets."
- [methods] calculate mean/relative expression per gene age class: "- calculate mean/relative expression per gene age class."
- [methods] match gene and transcript IDs and their corresponding gene age class with the scRNA data: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [abstract] oggmap can visualize individual gene categories and their expression profiles: "oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles"
