---
name: simulated-umi-subsampling-dropout-benchmarking
description: Use when quantifying how accurately a gene-expression imputation or dropout-correction
  method recovers spatial expression of marker genes from deliberately degraded (subsampled-UMI)
  ST data against a known ground truth.
when_to_use_negative:
- The spatial data lack paired ground-truth expression for the target genes — simulated
  subsampling requires un-degraded measurements to score against
- The method under test cannot be re-run on degraded input (e.g., it has no imputation
  or decomposition mode), making MAE/PCC of corrected expression undefined
- The goal is comparing spot-level cell-type deconvolution proportions (RCTD, Cell2location,
  CARD) rather than recovering full gene-expression values after dropout
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SpatialScope
  role: 'Method under test: performs RCTD-inspired singlet/doublet classification,
    cell type identification, and conditional score-based generative-model decomposition
    with annealed Langevin dynamics to produce dropout-corrected expression'
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Baseline alignment-based imputation method against which SpatialScope's MAE
    and PCC are compared at each subsampling level
- name: RCTD
  role: Source of the singlet/doublet classification strategy that replaces nucleus
    segmentation for Slide-seq spots (at most two cell types per 10 μm spot) prior
    to decomposition
  repo: https://github.com/dmcable/spacexr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:2b30b3d431e54ca72efbe10b9a1cd3cfc4846eb39d1ac136cbe244d2fae2d9c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - RCTD
  - SpatialScope
  - Tangram
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Simulated UMI subsampling dropout benchmarking

## Summary

A benchmarking protocol that simulates technical dropouts in spatial transcriptomics data by subsampling UMIs of high-capture marker genes, then compares dropout-correction methods (SpatialScope vs. Tangram) using mean absolute error (MAE) and Pearson's correlation (PCC) of corrected versus ground-truth expression.

## When to use

Use when quantifying how accurately a gene-expression imputation or dropout-correction method recovers spatial expression of marker genes from deliberately degraded (subsampled-UMI) ST data against a known ground truth.

## When NOT to use

- The spatial data lack paired ground-truth expression for the target genes — simulated subsampling requires un-degraded measurements to score against
- The method under test cannot be re-run on degraded input (e.g., it has no imputation or decomposition mode), making MAE/PCC of corrected expression undefined
- The goal is comparing spot-level cell-type deconvolution proportions (RCTD, Cell2location, CARD) rather than recovering full gene-expression values after dropout

## Inputs

- Slide-seq V2 mouse cerebellum ST dataset (filtered to 20,141 genes, 8,952 spots, spots with ≥100 total UMI counts)
- Mouse cerebellum snRNA-seq reference (24,387 genes, 15,609 cells, 19 cell types) for the decomposition method under test
- List of 22 high-capture marker genes for UMI subsampling

## Outputs

- Subsampled ST datasets at 50, 100, and 200 UMI count levels per marker gene
- Posterior-mean de-noised, dropout-corrected single-cell expression matrices for each method and subsampling level
- Comparison table of MAE and PCC (corrected vs. ground truth) for SpatialScope and Tangram across the three subsampling settings (Fig. 5b-d)

## How to apply

Start from a quality-filtered ST dataset — for the Slide-seq V2 cerebellum run, filter genes with zero counts across all spots and spots with total UMI counts below 100, yielding 20,141 genes and 8,952 spots. Generate dropout conditions by subsampling UMIs of 22 high-capture marker genes to fixed total-count levels (50, 100, and 200 counts) so the original data serve as ground truth. Re-run the method under test (SpatialScope: cell type identification with the 24,387-gene/15,609-cell/19-type mouse cerebellum snRNA-seq reference and Potts spatial smoothness prior at ν=10, then gene expression decomposition via conditional score-based generative model with annealed Langevin dynamics posterior sampling) and the baseline (Tangram) on each subsampled dataset. Compute MAE and PCC of corrected versus ground-truth expression for both methods at every subsampling level and assemble a comparison table. The benchmark succeeds when the tested method attains the lowest MAE and highest PCC across all subsampling settings, reproducing the Fig. 5b–d comparison, with output covering 10,975 cells across 8,952 spots.

## Related tools

- **SpatialScope** (Method under test: performs RCTD-inspired singlet/doublet classification, cell type identification, and conditional score-based generative-model decomposition with annealed Langevin dynamics to produce dropout-corrected expression) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Baseline alignment-based imputation method against which SpatialScope's MAE and PCC are compared at each subsampling level)
- **RCTD** (Source of the singlet/doublet classification strategy that replaces nucleus segmentation for Slide-seq spots (at most two cell types per 10 μm spot) prior to decomposition) — https://github.com/dmcable/spacexr

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git; docker pull xiaojs95/spatialscope
```

## Evaluation signals

- SpatialScope achieves the lowest MAE and highest PCC versus Tangram at all three subsampling levels (50, 100, 200 counts), reproducing Fig. 5b-d
- The output covers 10,975 cells across 8,952 spots (6,929 singlet and 2,023 doublet spots) after RCTD-inspired classification
- Subsampled marker-gene datasets reproduce exactly 22 genes at total counts of 50, 100, and 200 per subsampling condition
- PCC between corrected and ground-truth expression is high (and MAE low) for all evaluated marker genes, confirming dropout recovery rather than noise injection
- Cell type identification used the expected feature set: top 1000 highly variable genes plus 50 marker genes per cell type after total-counts normalization and log(1+x) transformation

## Limitations

- Dropout is simulated by fixed-UMI subsampling of 22 high-capture marker genes only, which may not reflect naturally occurring dropout across the whole transcriptome
- Training the deep generative models is time-consuming (hours or days), making repeated benchmark runs expensive
- No software version numbers or computational environment details are reported, hindering exact reproduction

## Evidence

- [methods] Filtering before simulation: "We filtered out genes that have zero counts across all spots and filtered out spots with total UMI counts less than 100"
- [methods] Posterior sampling used for correction: [evidence span withheld — anchor status: not_found]
- [intro] Motivation for benchmarking against Tangram: "methods developed to infer unmeasured gene expressions, such as Tangram18, gimVI19 and SpaGE20 are not sufficiently accurate"
- [abstract] Expected outcome of the corrected expression: "SpatialScope not only enhances seq-based ST data to achieve single-cell resolution, but also accurately infers transcriptome-wide expression levels for image-based ST data"
- [readme] SpatialScope framework grounding: "A unified approach for integrating spatial and single-cell transcriptomics data by leveraging deep generative models"
