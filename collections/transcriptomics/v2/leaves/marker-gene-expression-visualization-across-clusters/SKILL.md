---
name: marker-gene-expression-visualization-across-clusters
description: Use when scRNA-seq count matrices (raw vs imputed) must be compared at
  the cluster level by coloring t-SNE plots by cell type and overlaying curated marker
  genes (e.g., S100A8, FCER1A) to confirm subpopulation separation.
when_to_use_negative:
- The analysis goal is a numeric clustering benchmark (e.g., ARI) rather than the
  qualitative t-SNE/marker validation reported in this paper, which explicitly states
  'no numeric threshold reported in the paper'.
- The input is bulk RNA-seq data, which 'does not allow people to study cell-to-cell
  variation' and therefore has no cell clusters or marker overlays to visualize.
- Gene-level filtering has already removed most genes; scImpute's README recommends
  applying it to the whole-genome count matrix 'to ensure robust identification of
  dropouts', which this visualization workflow depends on.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scImpute
  role: produces the imputed count matrix (scimpute_count.csv) whose t-SNE embedding
    and marker overlays are compared against the raw data in this visualization skill
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:ee718b0cd80a0eedca3796144cdbfe7b3e652b19116324297353ec536c759686
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# marker-gene-expression-visualization-across-clusters

## Summary

Overlay known marker genes (e.g., S100A8, FCER1A for monocyte/dendritic clusters) and cell-type colors onto t-SNE embeddings computed separately from raw and scImpute-imputed scRNA-seq count matrices, to qualitatively verify whether imputation resolves subpopulation structure that dropout obscures.

## When to use

Use when scRNA-seq count matrices (raw vs imputed) must be compared at the cluster level by coloring t-SNE plots by cell type and overlaying curated marker genes (e.g., S100A8, FCER1A) to confirm subpopulation separation.

## When NOT to use

- The analysis goal is a numeric clustering benchmark (e.g., ARI) rather than the qualitative t-SNE/marker validation reported in this paper, which explicitly states 'no numeric threshold reported in the paper'.
- The input is bulk RNA-seq data, which 'does not allow people to study cell-to-cell variation' and therefore has no cell clusters or marker overlays to visualize.
- Gene-level filtering has already removed most genes; scImpute's README recommends applying it to the whole-genome count matrix 'to ensure robust identification of dropouts', which this visualization workflow depends on.

## Inputs

- raw 10x Genomics PBMC count matrix (genes x cells, ~4500 cells)
- imputed count matrix scimpute_count.csv produced by scImpute (drop_thre = 0.5, labeled=TRUE)
- character vector of cell-type labels for the 9 immune cell types

## Outputs

- side-by-side t-SNE plots (raw vs imputed) colored by the 9 cell types (PNG)
- S100A8 and FCER1A marker-expression overlay plots on the t-SNE embeddings (PNG)

## How to apply

Load the raw count matrix (rows = genes, columns = cells) and the imputed matrix written by scImpute as scimpute_count.csv, along with the cell-type label vector (here 9 immune cell types from the 10x Genomics PBMC dataset, ~4500 cells). Normalize the raw and imputed count matrices and run t-SNE dimension reduction separately on each, so the two embeddings are independently comparable. Render side-by-side t-SNE plots colored by the 9 cell types, and visually check the reported resolution gains: cytotoxic vs naive cytotoxic T cells split into two groups, and naive vs memory T cells distinguishable from other T cells. Then overlay S100A8 and FCER1A expression on both embeddings to reproduce the monocyte-cluster marker patterns reported for raw vs imputed data. Save the t-SNE comparison and marker-gene figures as PNG files for the validation record.

## Related tools

- **scImpute** (produces the imputed count matrix (scimpute_count.csv) whose t-SNE embedding and marker overlays are compared against the raw data in this visualization skill) — https://github.com/Vivianstats/scImpute

## Evaluation signals

- After imputation, cytotoxic and naive cytotoxic T cells separate into two distinct groups on the t-SNE plot, and naive vs memory T cells become distinguishable from other T cells (Fig. 6).
- S100A8 and FCER1A overlay plots show the reported monocyte-cluster marker patterns consistently in the raw vs imputed embeddings.
- t-SNE embeddings are computed separately on normalized raw and normalized imputed matrices, and both are colored with the same 9 cell-type labels.
- The t-SNE comparison figure and the marker-gene figures are saved as PNG files.
- Validation is qualitative by design: 'no numeric threshold reported in the paper' — any numeric pass/fail would not match the source.

## Limitations

- Validation is visual and qualitative; the paper reports no numeric threshold for the t-SNE separation of T-cell subpopulations.
- t-SNE on the imputed data is affected by upstream imputation quality: scImpute's imputation efficiency may degrade 'when dropout rates in the raw data are severely high (droplet-based technologies)'.
- Choice of the parameter K (number of cell populations) is left to the user without a definitive selection procedure, which affects cluster structure shown in the plots.
- No software version for scImpute is specified in the paper, complicating exact reproduction of the figures.

## Evidence

- [abstract] scImpute is shown to identify likely dropouts, enhance the clustering of cell subpopulations, improve the accuracy of differential expression analysis, and aid the study of gene expression dynamics.: "scImpute is shown to identify likely dropouts, enhance the clustering of cell subpopulations, improve the accuracy of differential expression analysis, and aid the study of gene expression dynamics."
- [intro] dropout phenomenon definition: "the “dropout” phenomenon where a gene is observed at a moderate expression level in one cell but undetected in another cell7"
- [readme] scImpute applied before dimension reduction: "`scImpute` can be applied to raw read count matrix before the users perform downstream analyses such as"
- [readme] scimpute function call with drop_thre threshold: "drop_thre = 0.5,          # threshold set on dropout probability"
- [readme] recommend whole-genome count matrix input: [evidence span withheld — anchor status: external_doc]
