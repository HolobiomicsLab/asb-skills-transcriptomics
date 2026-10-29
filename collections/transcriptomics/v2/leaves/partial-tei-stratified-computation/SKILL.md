---
name: partial-tei-stratified-computation
description: Use when scRNA-seq data (Scanpy AnnData, .h5ad) must be decomposed into
  gene-age-stratified expression contributions — e.g. to test whether the youngest
  age classes (C. elegans classes 11–13) peak in early embryos.
when_to_use_negative:
- The goal is a single global TEI trendline per cell without decomposing into age-class
  contributions — plain get_TEI suffices
- No orthomap/gene age annotations exist for the species and none can be extracted
  — gene ages are the prerequisite for stratification
- The input is bulk expression data rather than a Scanpy scRNA object; oggmap's TEI
  functions operate on Scanpy AnnData
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: oggmap
  role: Provides orthomap2tei module with get_TEI and get_pstrata to compute global
    and stacked partial TEI values, plus mean/relative expression per gene age class
    on AnnData
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: 'Container toolkit: hosts the scRNA count matrix as an AnnData object on which
    the TEI calculations are performed'
- name: OrthoFinder
  role: Upstream orthogroup inference whose results feed of2orthomap to build the
    gene age classes used for stratification
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:6b89ef8a134ecbc133a57f26a75c665fc18c69634fcc16304fec6dac8e023593
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - Scanpy
  - oggmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# partial-tei-stratified-computation

## Summary

Compute per-cell/per-stage partial transcriptome evolutionary index (TEI) profiles, where the expression-weighted contributions of individual gene age classes sum to the global TEI, using oggmap on Scanpy scRNA objects. This reveals which evolutionary age strata (e.g. youngest, species-specific classes) drive expression at each cell type or developmental stage.

## When to use

Use when scRNA-seq data (Scanpy AnnData, .h5ad) must be decomposed into gene-age-stratified expression contributions — e.g. to test whether the youngest age classes (C. elegans classes 11–13) peak in early embryos.

## When NOT to use

- The goal is a single global TEI trendline per cell without decomposing into age-class contributions — plain get_TEI suffices
- No orthomap/gene age annotations exist for the species and none can be extracted — gene ages are the prerequisite for stratification
- The input is bulk expression data rather than a Scanpy scRNA object; oggmap's TEI functions operate on Scanpy AnnData

## Inputs

- Scanpy AnnData object (.h5ad) with a gene-by-cell scRNA count matrix (e.g. Packer et al. 2019 ~90,000 C. elegans embryonic cells)
- Orthomap table mapping gene/transcript IDs to gene age classes (e.g. pre-existing C. elegans orthomap from Sun et al. 2021, or OrthoFinder/eggNOG-derived oggmap output)
- Cell/stage metadata column in adata.obs grouping cells for stratified plotting (e.g. embryonic stage or cell type)

## Outputs

- CSV table of partial TEI values per cell type/stage and gene age class (partial strata summing to the global TEI)
- Stacked partial-TEI profile figure (e.g. nematode_partial_tei.png) showing each age class's contribution across stages
- Global TEI values per cell/stage from oggmap's get_TEI

## How to apply

First obtain an orthomap (gene ages per orthogroup) for the query species, either from a pre-existing one (e.g. the Sun et al. 2021 C. elegans orthomap) or by extracting one from OrthoFinder results with oggmap. Load the scRNA dataset (e.g. the ~90,000-cell Packer et al. 2019 nematode embryo dataset) into a Scanpy AnnData object, then match gene and transcript IDs and their gene age class to the data using oggmap's orthomap2tei module. Compute the global TEI per cell/stage with oggmap's get_TEI, then compute stacked partial TEI values with oggmap's get_pstrata so each age class's contribution sums to the global TEI per stage. Calculate mean/relative expression per gene age class, plot the stacked partial TEI profile across stages, and save the table to CSV and the figure (e.g. nematode_partial_tei.png). Validation hinges on the additive property: partial strata must sum to the global TEI, and expected biology (young classes high in early embryos) should be checked on the rendered figure.

## Related tools

- **oggmap** (Provides orthomap2tei module with get_TEI and get_pstrata to compute global and stacked partial TEI values, plus mean/relative expression per gene age class on AnnData) — https://github.com/kullrich/oggmap
- **Scanpy** (Container toolkit: hosts the scRNA count matrix as an AnnData object on which the TEI calculations are performed)
- **OrthoFinder** (Upstream orthogroup inference whose results feed of2orthomap to build the gene age classes used for stratification) — https://github.com/davidemms/OrthoFinder

## Examples

```
from oggmap import orthomap2tei; partial_tei = orthomap2tei.get_pstrata(adata=adata, gene_age_tax_dict=gene_age_tax_dict)
```

## Evaluation signals

- Additivity check: the stacked partial TEI values per cell type/stage must sum to the global TEI computed by get_TEI
- The partial TEI table exists and contains the full set of expected age classes (including classes 11–13 for C. elegans)
- Young age classes 11–13 must show peak partial TEI in early-embryo stages, matching the Ma and Zheng (2023) pattern and the Fig. 1e rendering
- Rendered PNG (llm_judge) reproduces the published stacked partial-TEI profile figure
- Every gene in the AnnData with expression is matched to an age class via orthomap2tei ID matching; unmatched-gene counts should be reported/low

## Limitations

- ID matching between transcript/gene identifiers in the orthomap and the scRNA dataset is error-prone; mismatched IDs silently drop genes from stratified sums
- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or versions <=3.10, constraining the environment
- Gene age assignment depends on the chosen orthology resource (OrthoFinder, eggNOG 6.0, PLAZA) and the query species lineage from NCBI taxonomy; age-class granularity changes with the source

## Evidence

- [methods] calculate and visualize partial TEI profiles: "calculate and visualize partial transcriptome evolutionary index (TEI) profiles on scRNA data sets."
- [methods] match gene/transcript IDs to age classes: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [methods] per-class expression summary: "- calculate mean/relative expression per gene age class."
- [abstract] category-level expression visualization: "oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles"
- [readme] weighted expression from orthomaps: [evidence span withheld — anchor status: external_doc]
