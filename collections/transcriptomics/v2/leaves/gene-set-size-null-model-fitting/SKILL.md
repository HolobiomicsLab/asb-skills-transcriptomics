---
name: gene-set-size-null-model-fitting
description: Use when scoring cell-type marker gene sets of widely varying sizes against
  scRNA-seq clusters (e.g., Miko scoring on a Cell Catalog or permuted marker sets)
  and raw enrichment scores must be corrected for gene set-size bias before significance
  calling.
when_to_use_negative:
- Input marker gene sets are all of nearly identical size and standardized scoring
  frameworks (e.g., GSVA, Seurat AddModuleScore) are already appropriate — the size-corrected
  null is designed for heterogeneous set sizes
- The goal is clustering or gene program discovery (SSN analysis) rather than marker-based
  cell-type annotation scoring
- Gene sets exceed the fitted size range (paper's null was fit for sizes 2–200 genes),
  so null expectations would be extrapolated
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMiko
  role: Provides nullScore(), mikoScore(), and sigScore() implementing the 3-step
    Miko scoring pipeline with gene set-size bias correction
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework hosting the normalized Seurat object that nullScore() and mikoScore()
    operate on
  repo: https://satijalab.org/seurat/
- name: sctransform
  role: SCTransform (glmGamPoi, v2 flavor) normalization producing the count matrix
    representation scored against the null model
  repo: https://github.com/satijalab/sctransform
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:f294a1a0f2f90ff08b967723163bfe452ab2bcfb46317deb7ac29c65c957a60e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - sctransform
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Gene-set-size null model fitting

## Summary

Fits a gene set-size-dependent null model (via scMiko's nullScore()) that corrects for the bias whereby larger marker gene sets yield inflated enrichment scores, enabling fair comparison of variable-sized cell-type gene sets prior to Miko scoring.

## When to use

Use when scoring cell-type marker gene sets of widely varying sizes against scRNA-seq clusters (e.g., Miko scoring on a Cell Catalog or permuted marker sets) and raw enrichment scores must be corrected for gene set-size bias before significance calling.

## When NOT to use

- Input marker gene sets are all of nearly identical size and standardized scoring frameworks (e.g., GSVA, Seurat AddModuleScore) are already appropriate — the size-corrected null is designed for heterogeneous set sizes
- The goal is clustering or gene program discovery (SSN analysis) rather than marker-based cell-type annotation scoring
- Gene sets exceed the fitted size range (paper's null was fit for sizes 2–200 genes), so null expectations would be extrapolated

## Inputs

- Normalized/processed Seurat object (e.g., SCTransform glmGamPoi v2-flavor normalized gastrulation dataset)
- Cell-type-specific marker gene sets of variable size (e.g., Cell Catalog sets, or permuted marker sets at 0/25/50% replacement)
- Null-model parameters: n.replicate, nbin, min.gs.size, max.gs.size, step.size

## Outputs

- Fitted gene set-size-dependent null model object from nullScore()
- Size-corrected Miko scores (miko_score) for each gene set per cluster, ready for sigScore() significance testing

## How to apply

As the first step of the 3-step Miko scoring pipeline, run nullScore() on the normalized Seurat object to fit a null model that characterizes the expected score distribution as a function of gene set size. Sample random gene sets spanning the size range of the input marker sets — in the paper's benchmarks, n.replicate=10 random gene sets per size bin, with nbin=24 bins covering gene set sizes 2–200 genes at step.size=10 — and use the fitted size-dependent null to standardize observed mikoScore() enrichment values. This standardization is what allows sigScore() to call significant cluster enrichments at FDR < 0.05 with miko_score > 0 across marker sets ranging from a handful to hundreds of genes. The null model was validated in the permutation benchmark, where correctly size-corrected scoring achieved 100% sensitivity and 94% specificity at 0% permutation, degrading gracefully to 93%/96% at 25% gene permutation.

## Related tools

- **scMiko** (Provides nullScore(), mikoScore(), and sigScore() implementing the 3-step Miko scoring pipeline with gene set-size bias correction) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework hosting the normalized Seurat object that nullScore() and mikoScore() operate on) — https://satijalab.org/seurat/
- **sctransform** (SCTransform (glmGamPoi, v2 flavor) normalization producing the count matrix representation scored against the null model) — https://github.com/satijalab/sctransform

## Examples

```
null.model <- nullScore(object = so.query, n.replicate = 10, nbin = 24, min.gs.size = 2, max.gs.size = 200, step.size = 10)
```

## Evaluation signals

- Permutation benchmark sensitivity/specificity match Fig. 4d-e targets: 100%/94% at 0% permutation, 93%/96% at 25%, 54%/98% at 50% — sensitivity near 100% at 0% declining to ~54% at 50% while specificity stays ~94-98%
- Significance calls via sigScore() fall at FDR < 0.05 with miko_score > 0, and retained annotations pass the coherence filter (coherence_fraction >= 0.8)
- Random gene sets spanning the full size range (2–200 genes) are generated for all 24 size bins with n.replicate=10 per bin, so null expectations exist for every input gene set size
- Size-corrected scores should not systematically increase with gene set size; benchmarked AUROC against author-curated labels reaches 0.776, above GSVA (0.705) and SCINA (0.603)
- Results table (e.g., benchmark_permutation_results.csv) reproduces the benchmark trend when compared against author-curated cell-type labels

## Limitations

- The null model must be refit per dataset/normalization context; parameters used in the paper (nbin=24, sizes 2–200, step.size=10, n.replicate=10) may not cover gene sets outside that size range
- Marker-based annotations are only as good as the cell-type assigned to the original dataset; no validation of source catalog curations is provided
- Frequent-flier gene sets (flagged at threshold 0.75) can pass significance tests despite poor specificity, requiring post-hoc filtering beyond the null-corrected score

## Evidence

- [methods] The miko scoring pipeline is a 3-step workflow that is implemented in nullScore(), mikoScore(), and sigScore() functions.: "The miko scoring pipeline is a 3-step workflow that is implemented in nullScore(), mikoScore(), and sigScore() functions."
- [methods] In step 1 we run nullScore(), which fits a null model that corrects for gene set-size biases.: "In step 1 we run nullScore(), which fits a null model that corrects for gene set-size biases."
- [methods] In step 2, cell-type marker sets are scored using mikoScore().: "In step 2, cell-type marker sets are scored using mikoScore()."
- [methods] we identify which cell-type marker sets are significantly enriched in each cluster using sigScore().: "we identify which cell-type marker sets are significantly enriched in each cluster using sigScore()."
- [discussion] accepts or rejects candidate annotations using a hypothesis-testing framework: "facilitates unbiased scoring of a diverse set of variable-sized cell type-specific gene sets and accepts or rejects candidate annotations using a hypothesis-testing framework"
