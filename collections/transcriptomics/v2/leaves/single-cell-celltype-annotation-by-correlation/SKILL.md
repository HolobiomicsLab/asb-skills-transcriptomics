---
name: single-cell-celltype-annotation-by-correlation
description: Use when you have a 10x 3' gene-cell-barcode count matrix (e.g. from
  Cell Ranger) and need per-cell type labels by matching expression profiles to reference
  populations, or to stratify cell types by host/donor genotype.
when_to_use_negative:
- Input is already an annotated feature table or cell-type label set rather than a
  gene-cell-barcode count matrix
- Cells of interest lack reference populations with adequate surface-marker or transcriptional
  coverage (e.g. erythrocyte precursors, where limited surface markers make assignment
  unreliable)
- A sample is a K=1 all-one-genotype call with mixture fraction below the ~4%-of-6,000-cells
  detection level, where genotype-stratified annotation is uninformative
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Seurat
  role: Receives the log-transformed gene-cell-barcode matrix and performs reference-correlation
    cell-type assignment
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: Cell Ranger Single-Cell Software Suite
  role: Produces the sample-demultiplexed, barcode-processed 3' gene counting matrix
    used as the annotation input
  repo: https://github.com/10XGenomics/cellranger
- name: Freebayes 1.0.2
  role: Calls de novo SNVs on the Cell Ranger genome BAM, enabling genotype-stratified
    cell-type composition for chimerism analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:6b7f7e923c43a584991bc8adda6eeb4a3c44e1eb65b6949be3d734e38a8925e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/single-cell-3prime-paper
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  - Seurat
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-celltype-annotation-by-correlation

## Summary

Annotate single-cell types by correlating each cell's log-transformed expression profile against reference populations in Seurat, as used on ~68k PBMCs and ~17,000 post-transplant BMMCs profiled with the 10x Genomics GemCode droplet platform.

## When to use

Use when you have a 10x 3' gene-cell-barcode count matrix (e.g. from Cell Ranger) and need per-cell type labels by matching expression profiles to reference populations, or to stratify cell types by host/donor genotype.

## When NOT to use

- Input is already an annotated feature table or cell-type label set rather than a gene-cell-barcode count matrix
- Cells of interest lack reference populations with adequate surface-marker or transcriptional coverage (e.g. erythrocyte precursors, where limited surface markers make assignment unreliable)
- A sample is a K=1 all-one-genotype call with mixture fraction below the ~4%-of-6,000-cells detection level, where genotype-stratified annotation is uninformative

## Inputs

- Gene-cell-barcode count matrix (Cell Ranger output) from 10x Genomics GemCode 3' scRNA-seq, log-transformed
- Set of reference cell populations with expression profiles for correlation
- Optional: per-cell host/donor genotype assignments from the de novo SNV-based Bayesian model for genotype-stratified composition tables

## Outputs

- Per-cell type annotations with correlation scores
- Cell-type composition tables (optionally stratified by genotype and pre/post-transplant timepoint)
- Per-cell genotype-by-type assignment table (e.g. aml_transplant_chimerism_results.csv)

## How to apply

Log-transform the Cell Ranger gene-cell-barcode matrix as input to Seurat. Identify the top 1,000 most variable genes based on their mean and dispersion (variance/mean), similar to the Macosko et al. approach. For each cell, compute correlation against reference populations (e.g. PBMC immune populations or erythroid/granulocyte/monocyte precursors in BMMC samples) and assign the type with the highest correlation. In transplant applications, compute per-genotype cell-type composition by combining these assignments with the SNV-based host/donor genotype calls (e.g. Freebayes SNVs with support from at least two cell barcodes, SNV Qual >=30, base Qual>=1). Treat assignments where the highest and second-highest correlations are close (e.g. cytotoxic T vs NK cells) as low-confidence.

## Related tools

- **Seurat** (Receives the log-transformed gene-cell-barcode matrix and performs reference-correlation cell-type assignment) — https://github.com/10XGenomics/single-cell-3prime-paper
- **Cell Ranger Single-Cell Software Suite** (Produces the sample-demultiplexed, barcode-processed 3' gene counting matrix used as the annotation input) — https://github.com/10XGenomics/cellranger
- **Freebayes 1.0.2** (Calls de novo SNVs on the Cell Ranger genome BAM, enabling genotype-stratified cell-type composition for chimerism analysis)

## Examples

```
Rscript -e 'library(Seurat); mat <- readMM("filtered_gene_bc_matrices/hg19/matrix.mtx"); mat <- log1p(mat); genes <- order(matvec(mat), decreasing=TRUE)[1:1000]'
```

## Evaluation signals

- Each cell receives the type whose correlation is highest, and the margin between the top two correlations is reported to flag low-confidence calls (small gaps such as cytotoxic T vs NK indicate uncertain assignment)
- Feature selection step reproducibly yields the top 1,000 most variable genes ranked by mean and dispersion (variance/mean)
- Cell-type composition tables recapitulate known population structure (e.g. large immune populations in 68k PBMCs)
- In AML027 post-transplant BMMCs, applying annotations to the two SNV-defined genotypes should reproduce compositions consistent with the clinical 12% donor chimerism (minor genotype ~13.8%)
- Pre/post-transplant composition shifts should show erythroid expansion and post-transplant monocytopenia as reported

## Limitations

- Cell assignment was low-confidence for some cells (e.g. cytotoxic T vs NK) because the difference between highest and second-highest correlation was small
- Erythrocyte precursors are hard to assess because of the limited number of surface markers for these populations, and clinical flow/morphology failed to detect the expanded erythroid precursor population
- Proof-of-principle chimerism analysis covered only two transplant patients; correlation-based annotation of chimerism is not yet clinically validated

## Evidence

- [discussion] Log-transformed matrix input to Seurat: "The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat."
- [discussion] Top 1000 variable genes: "The top 1,000 most variable genes were identified based on their mean and dispersion (variance/mean), which is similar to the approach used by Macoscko et al.7"
- [discussion] Low-confidence assignments: "the difference between the highest and second highest correlation was small for some cells (for example, the difference between cytotoxic T and NK cells)"
- [discussion] Cell Ranger matrix generation: "The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting"
- [abstract] Chimerism application: "we used sequence variation in the transcriptome data to determine host and donor chimerism at single-cell resolution"
- [readme] Repository README: "The R code used to analyse 68k PBMCs and purified PBMCs can be found here: https://github.com/10XGenomics/single-cell-3prime-paper."
