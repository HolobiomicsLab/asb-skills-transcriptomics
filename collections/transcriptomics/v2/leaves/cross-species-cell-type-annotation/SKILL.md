---
name: cross-species-cell-type-annotation
description: Use when training labels exist only in one species (e.g., mouse Tabula
  Muris) and you must annotate scRNA-seq cells from another species (e.g., human pancreas)
  using a shared one-to-one orthologue gene space.
when_to_use_negative:
- Query cells are of types not represented in the reference — the null-dataset/conf_score
  machinery, not the plain Bet classifier, is needed to control false positives
- Data are not UMI counts or full-length read counts with the paper's normalization
  (TPM + log or size-factor 10,000 + add-one + log), since SciBet is technology-dependent
  on matched preprocessing
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SciBet
  role: Supervised multinomial classifier used for SelectGene (ΔS), Learn (Laplace-smoothed
    reference), and Bet (MLE prediction) across species
  repo: https://github.com/PaulingLiu/scibet
- name: HomoloGene
  role: NCBI database (Build 68) used to identify one-to-one homologous genes between
    human and mouse for gene-space conversion
- name: Seurat v3
  role: Preprocessing scheme (size-factor 10,000, add-one, log normalization) applied
    to UMI count data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:94bc10dd4f37e9556dfb534cdc39ffeedc27704aa84281924478c1a5f2bfab99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - HomoloGene
  - SciBet
  - Seurat v3
  - Tabula Muris
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-species-cell-type-annotation

## Summary

Cross-species cell type annotation trains a SciBet multinomial classifier on annotated mouse scRNA-seq data (Tabula Muris) restricted to one-to-one human-mouse orthologues from HomoloGene Build 68, then predicts human cell types via maximum likelihood estimation. It demonstrates that a supervised classifier transfers across species when the gene space is orthologue-restricted.

## When to use

Use when training labels exist only in one species (e.g., mouse Tabula Muris) and you must annotate scRNA-seq cells from another species (e.g., human pancreas) using a shared one-to-one orthologue gene space.

## When NOT to use

- Query cells are of types not represented in the reference — the null-dataset/conf_score machinery, not the plain Bet classifier, is needed to control false positives
- Data are not UMI counts or full-length read counts with the paper's normalization (TPM + log or size-factor 10,000 + add-one + log), since SciBet is technology-dependent on matched preprocessing

## Inputs

- Annotated mouse scRNA-seq training matrix (Tabula Muris) with cell type labels
- NCBI HomoloGene Build 68 human-mouse orthologue lookup table
- Human scRNA-seq query expression matrices (e.g., human pancreas datasets)

## Outputs

- Per-cell predicted cell type labels mapped from mouse-trained reference classes
- Optional per-cell confidence scores from conf_score for filtering low-confidence assignments
- Sankey diagram linking predicted mouse-derived labels to known human pancreas cell types (cf. Fig. 3b)

## How to apply

Load the annotated Tabula Muris training matrix and the NCBI HomoloGene Build 68 lookup table; retain only one-to-one human-mouse orthologue pairs and intersect gene names to obtain a human-gene-space training set. Normalize expression per technology: TPM with pseudo-value one and log normalization for full-length data, or Seurat v3-style size-factor 10,000, add-one, log normalization for UMI data. Select informative genes with SelectGene using the ΔS entropy statistic (default k=500) and build the multinomial reference with Learn (Laplace smoothing, add-one). Classify each human query cell with Bet, assigning the type whose mean expression vector maximizes the multinomial likelihood (MLE, equivalent to minimal KL divergence); optionally apply conf_score cutoffs in the 0.001–0.5 range to withhold low-confidence assignments. Aggregate predictions per dataset and render a Sankey diagram linking mouse-derived labels to known human cell types.

## Related tools

- **SciBet** (Supervised multinomial classifier used for SelectGene (ΔS), Learn (Laplace-smoothed reference), and Bet (MLE prediction) across species) — https://github.com/PaulingLiu/scibet
- **HomoloGene** (NCBI database (Build 68) used to identify one-to-one homologous genes between human and mouse for gene-space conversion)
- **Seurat v3** (Preprocessing scheme (size-factor 10,000, add-one, log normalization) applied to UMI count data)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/scibet")
```

## Evaluation signals

- ~92% of human pancreas cells should be assigned the correct orthologous cell type, matching the Fig. 3b cross-species result
- Gene space after orthologue intersection must contain only one-to-one human-mouse orthologues shared between Tabula Muris and the HomoloGene lookup table
- Feature selection should retain the default top 500 ΔS-ranked informative genes
- Per-cell confidence scores, if computed, should fall within the 0.001–0.5 cutoff range used to withhold low-confidence assignments
- Sankey diagram should show most query mass flowing to the orthologous mouse-derived cell type labels

## Limitations

- Accuracy depends on restricting to one-to-one orthologues; one-to-many or lineage-specific genes are excluded and may carry discriminative signal
- No seed or randomization parameters were reported for the repeated splits and permutation tests, complicating exact reproduction
- For query cell types absent from the reference, plain classification can produce false positives unless the null-dataset confidence score is applied

## Evidence

- [discussion] Cross-species mapping via HomoloGene one-to-one conversion: "the gene names were then converted to human gene names to obtain a test-set-compatible training set for classification"
- [discussion] HomoloGene Build 68 orthologue identification: "We used the HomoloGene databases provided by NCBI (Build 68) to identify homologous genes between human and mouse"
- [discussion] MLE assignment over informative genes: "We assign cell type j to cell y with maximum likelihood estimation (MLE)"
- [readme] SciBet installed from GitHub: "devtools::install_github("PaulingLiu/scibet")"
- [discussion] False positive control via confidence score cutoffs: "we did not assign cell types to the cells with lowest score C with a series of cutoffs from 0.001 to 0.5."
