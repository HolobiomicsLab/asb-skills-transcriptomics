---
name: stacked-evolutionary-profile-visualization
description: Use when a scRNA dataset (Scanpy AnnData) has a gene age class annotation
  (orthomap) and you need to show which evolutionary age classes drive the global
  TEI across stages or cell types — e.g. testing whether youngest, species-specific
  classes peak in early embryos.
when_to_use_negative:
- Input lacks a gene age classification (no orthomap) — the stratification is meaningless
  without per-gene age classes.
- Data are bulk transcriptomics or already aggregated to a gene-age-by-condition feature
  table — use a direct expression-profile plot instead of per-cell TEI stratification.
- The goal is species-level phylostratigraphy in R (e.g. mypromaps/PhyloStratum workflows)
  rather than the oggmap/Scanpy Python stack.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3299
tools:
- name: oggmap
  role: Provides orthomap2tei ID matching, get_TEI for the gene age-weighted index,
    and get_pstrata for stacked partial TEI strata
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Holds the scRNA data as AnnData objects that oggmap annotates and computes
    TEI on
- name: OrthoFinder
  role: Source of orthogroup results from which oggmap extracts the gene ages (orthomap)
    used to stratify TEI
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:1565e1cf8b2f640e3729f784b36d86b148fb0359b7e46666f6f9d4bdd6d8f607
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# stacked-evolutionary-profile-visualization

## Summary

Visualize per-cell/stage contribution of each gene age class to the transcriptome evolutionary index (TEI) by computing stacked partial TEI strata with oggmap's get_pstrata, so that per-class partial TEI values sum to the global TEI and young age classes' embryonic peaks are directly visible.

## When to use

Use when a scRNA dataset (Scanpy AnnData) has a gene age class annotation (orthomap) and you need to show which evolutionary age classes drive the global TEI across stages or cell types — e.g. testing whether youngest, species-specific classes peak in early embryos.

## When NOT to use

- Input lacks a gene age classification (no orthomap) — the stratification is meaningless without per-gene age classes.
- Data are bulk transcriptomics or already aggregated to a gene-age-by-condition feature table — use a direct expression-profile plot instead of per-cell TEI stratification.
- The goal is species-level phylostratigraphy in R (e.g. mypromaps/PhyloStratum workflows) rather than the oggmap/Scanpy Python stack.

## Inputs

- Orthomap table assigning each gene/transcript ID to a gene age class (e.g. from OrthoFinder or eggNOG results via oggmap)
- Single-cell RNA-seq count data as a Scanpy AnnData object (e.g. GEO-deposited Packer et al. 2019 C. elegans embryonic dataset, ~90,000 cells)
- Cell/stage metadata ordering embryonic stages for the x-axis

## Outputs

- Partial TEI table (per stage, per gene age class, contributions summing to global TEI) saved as CSV
- Stacked partial TEI profile figure (e.g. nematode_partial_tei.png) showing age-class contributions across embryonic stages
- Global TEI values per cell/stage from oggmap's get_TEI

## How to apply

Load the pre-existing orthomap (gene age class per gene, e.g. Sun et al. 2021 C. elegans classes) and the scRNA dataset (~90,000 cells in Packer et al. 2019) into a Scanpy AnnData object. Match gene/transcript IDs and their gene age class with the scRNA data using oggmap's orthomap2tei module. Compute the global TEI per cell/stage with oggmap's get_TEI, then compute stacked partial TEI values with get_pstrata so each age class's contribution sums to the global TEI per stage. Calculate mean/relative expression per gene age class and plot the stacked partial TEI profile across ordered embryonic stages, saving the per-stage table to CSV and the figure (e.g. nematode_partial_tei.png). Validate that the table contains the target young age classes (11–13) and that they peak in early-embryo stages, matching the reference figure pattern.

## Related tools

- **oggmap** (Provides orthomap2tei ID matching, get_TEI for the gene age-weighted index, and get_pstrata for stacked partial TEI strata) — https://github.com/kullrich/oggmap
- **Scanpy** (Holds the scRNA data as AnnData objects that oggmap annotates and computes TEI on)
- **OrthoFinder** (Source of orthogroup results from which oggmap extracts the gene ages (orthomap) used to stratify TEI) — https://github.com/davidemms/OrthoFinder

## Examples

```
from oggmap import orthomap2tei; gene_age_annot = orthomap2tei.get_adata_homology(adata=adata, orthomap=orthomap); orthomap2tei.get_pstrata(adata=adata, obs_key='stage', gene_age_class='gene_age', keep='all')
```

## Evaluation signals

- Partial TEI table exists as CSV and contains the youngest age classes (11–13 in the C. elegans case).
- Per-stage partial TEI values across age classes sum to the global TEI computed by get_TEI (stratification invariant).
- Age classes 11–13 show peak partial TEI in early-embryo stages, confirming Ma and Zheng (2023).
- Rendered PNG matches the reference Fig. 1e stacked profile pattern (llm_judge on the rendered figure).
- Gene/transcript IDs from the orthomap successfully matched against AnnData var_names via orthomap2tei (no wholesale ID-mismatch dropout).

## Limitations

- oggmap v0.0.2 was developed with Python 3.10; Python 2.7x and versions <=3.10 other than 3.10 are not supported.
- Requires a pre-computed orthomap from a gene family resource (OrthoFinder, eggNOG, PLAZA) — gene ages are not inferred from the scRNA data itself.
- Gene/transcript ID formats must be reconciled between the orthomap and the scRNA dataset before TEI calculation can succeed.

## Evidence

- [abstract] oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles: "oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles"
- [methods] partial TEI profiles workflow step: "calculate and visualize partial transcriptome evolutionary index (TEI) profiles on scRNA data sets."
- [methods] mean/relative expression per gene age class: "- calculate mean/relative expression per gene age class."
- [methods] match gene and transcript IDs: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [readme] oggmap weighted expression from scRNA objects: "Oggmap results (gene ages per orthologous group) can be further used to calculate and visualize weighted expression
(transcriptome evolutionary index) from scRNA sequencing objects."
