---
name: out-of-cell-type-generalization-testing
description: Use when benchmarking surface protein imputation (e.g. CITE-seq PBMC
  ADT prediction) for generalization to held-out cell types, rather than random cell-level
  holdout splits.
when_to_use_negative:
- Input data is not UMI-based count expression — cTP-net can only apply to count-based
  (UMI) input and TPM/RPKM accuracy is untested
- The question is standard within-training-cell-type accuracy; the random 10%/90%
  holdout validation scheme is the appropriate design instead
- Cells are outside the hematopoietic lineage — generalization there was not tested
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cTP-net
  role: Imputes surface protein abundances from scRNA-seq data in the out-of-cell-type
    benchmark scenario
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises scRNA-seq data (default parameters) before cTP-net imputation; omitting
    it retracts performance by ~0.02 in correlation
- name: Seurat v3
  role: Provides the anchor-based transfer-learning baseline against which cTP-net
    is compared, and the LogNormalize transformation of denoised counts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:138cdf95516889d1a0df8d65a996dc7278e2d684fd78639d5bcc1e89fb2aa192
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat v3
  - cTP-net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# out-of-cell-type-generalization-testing

## Summary

A benchmark scheme for cTP-net protein imputation in which cells are partitioned by cell type and all cells of one type are iteratively held out for testing, measuring whether predictions generalize to cell types absent from the training data.

## When to use

Use when benchmarking surface protein imputation (e.g. CITE-seq PBMC ADT prediction) for generalization to held-out cell types, rather than random cell-level holdout splits.

## When NOT to use

- Input data is not UMI-based count expression — cTP-net can only apply to count-based (UMI) input and TPM/RPKM accuracy is untested
- The question is standard within-training-cell-type accuracy; the random 10%/90% holdout validation scheme is the appropriate design instead
- Cells are outside the hematopoietic lineage — generalization there was not tested

## Inputs

- CITE-PBMC paired RNA/ADT count matrix with per-cell surface-protein measurements
- Per-cell type labels for partitioning cells by cell type
- Pretrained cTP-net model file (model_file_path) and target protein panel (dprotein)
- Seurat v3 object for running the anchor-based transfer-learning baseline

## Outputs

- Per-protein, per-cell-type Pearson correlation matrix of imputed vs. measured protein levels for each held-out cell type
- cTP-net vs. Seurat v3 comparison heatmap of imputation correlations (Fig. 2a, Supplementary Fig. 5)
- Within-cell-type scatter plots of CITE-seq-measured vs. imputed protein abundance (Fig. 2b–c)

## How to apply

Instead of the random 10% testing / 90% training split, partition the CITE-PBMC paired RNA/ADT cells by their cell-type labels and iteratively hold out all cells of one type as the test set, training on the remaining types. Preprocess as in the standard cTP-net workflow: denoise scRNA-seq with SAVER-X (default parameters), apply Seurat LogNormalize to denoised counts, and transform surface protein ADT counts to relative abundance (geometric mean log-ratio). Run both cTP-net and the Seurat v3 anchor-based transfer-learning baseline on each split, then compute per-protein, per-cell-type Pearson correlations between imputed and CITE-seq-measured protein levels, restricting comparisons to the same cells as the validation-set holdout scheme to account for subset variation. Render comparison heatmaps and within-cell-type scatter plots (Fig. 2a–c, Supplementary Fig. 5) of imputation correlations across protein targets and splits.

## Related tools

- **cTP-net** (Imputes surface protein abundances from scRNA-seq data in the out-of-cell-type benchmark scenario) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises scRNA-seq data (default parameters) before cTP-net imputation; omitting it retracts performance by ~0.02 in correlation)
- **Seurat v3** (Provides the anchor-based transfer-learning baseline against which cTP-net is compared, and the LogNormalize transformation of denoised counts)

## Examples

```
out <- cTPnet(data, data_type, model_file_path, dprotein)
```

## Evaluation signals

- In the out-of-cell-type scenario, cTP-net correlation entries in the comparison heatmap substantially exceed the Seurat v3 entries, matching the paper's finding that cTP-net vastly improves upon Seurat v3 for cell types absent from training
- Per-protein, per-cell-type Pearson correlations between imputed and CITE-seq-measured levels are computed on the same cells as the validation-set holdout scheme to control for subset variation
- cTP-net predictions generalize to cell types absent from, but related to, the training data
- Preprocessing matches the reported workflow: SAVER-X denoising with default parameters, Seurat LogNormalize, and relative abundance (geometric mean log-ratio) transformation of protein counts

## Limitations

- Comparisons in the benchmark are limited to the same cells as in the validation set of the holdout scheme due to variation between subsets
- Generalization was not tested on cells outside the hematopoietic lineage
- cTP-net can only apply to count-based (UMI) expression input; accuracy for TPM/RPKM metrics is unknown
- cTP-net requires SAVER-X denoising — without it prediction performance retracts by 0.02 in correlation

## Evidence

- [discussion] Out-of-cell-type split design: "instead of selecting the test set cells randomly, we partition the cells by their cell types"
- [discussion] Generalization finding: "the predictions generalize to cell types that are absent from, but related to those in the training data"
- [discussion] Same-cell comparison constraint: "we limit comparisons to the same cells that were in the validation set in the holdout scheme to account for variations between subsets."
- [discussion] Baseline comparison: "We compared cTP-net with an anchor-based transfer-learning method developed in Seurat v3"
- [readme] SAVER-X prerequisite: "(1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
