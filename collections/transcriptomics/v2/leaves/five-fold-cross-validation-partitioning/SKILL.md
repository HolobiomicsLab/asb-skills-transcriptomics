---
name: five-fold-cross-validation-partitioning
description: Use when comparing cross-modality translation methods on paired scRNA-seq/scATAC-seq
  datasets and a by-cell resampling design is needed so per-fold metrics can feed
  paired statistical tests (one-sided paired Wilcoxon signed-rank).
when_to_use_negative:
- Data are unpaired across modalities — use scButterfly's diagonal/unpaired training
  mode instead of by-cell fold splitting
- A single train/test split suffices and no paired statistical comparison across folds
  is intended
- The evaluation target is perturbation-response prediction or cell-type annotation
  rather than cross-validation benchmarking of translation methods
tools:
- name: scButterfly
  role: Provides load_data, which consumes the five-fold train_id/test_id/validation_id
    cell-ID lists to realize the cross-validation partitions within the translation
    pipeline
  repo: https://github.com/BioX-NKU/scButterfly
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:c5d040e62754fda1c54ea6f5c6eb696878e23bf716eb4957c848cc2c826c4976
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - JAMIE
  - Polarbear
  - Python
  - scButterfly
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# five-fold-cross-validation-partitioning

## Summary

A benchmarking protocol that partitions each cell into five disjoint folds so that every held-out fold is translated in both directions (RNA→ATAC and ATAC→RNA), yielding 140 paired evaluations (7 datasets × 5 folds × 2 directions) for fair statistical comparison of cross-modality translators such as scButterfly-B/T/C against BABEL, Polarbear, and JAMIE.

## When to use

Use when comparing cross-modality translation methods on paired scRNA-seq/scATAC-seq datasets and a by-cell resampling design is needed so per-fold metrics can feed paired statistical tests (one-sided paired Wilcoxon signed-rank).

## When NOT to use

- Data are unpaired across modalities — use scButterfly's diagonal/unpaired training mode instead of by-cell fold splitting
- A single train/test split suffices and no paired statistical comparison across folds is intended
- The evaluation target is perturbation-response prediction or cell-type annotation rather than cross-validation benchmarking of translation methods

## Inputs

- Raw count AnnData objects (n_obs × n_vars) for paired scRNA-seq (cells × genes) and scATAC-seq (cells × peaks) profiles
- Seven paired datasets: BMMC, MB, CL, MCC, MK, MDS, PBMC
- Per-dataset fold assignments: lists of cell IDs for train_id, test_id, and optionally validation_id

## Outputs

- Five disjoint train/test partitions per dataset
- Per-fold translated profiles in both directions (A2R_predict, R2A_predict)
- Fold-level clustering metrics (AMI, ARI, HOM, NMI) aggregated to a benchmark_clustering_metrics.csv table (n=140 evaluations)
- Wilcoxon signed-rank test p-values for scButterfly vs. each baseline

## How to apply

Split the cells of each paired scRNA-seq/scATAC-seq dataset (BMMC, MB, CL, MCC, MK, MDS, PBMC) into five disjoint, non-overlapping folds, holding out one fold at a time while training on the remainder. Pass the fold splits as train_id/test_id (and an optional validation_id, which defaults to 20% of train cells if None) into butterfly.load_data so every method — scButterfly-B/T/C, BABEL, Polarbear, JAMIE — is trained and tested on identical partitions. For each fold, preprocess with RNA_data_preprocessing and ATAC_data_preprocessing, train via construct_model/train_model (plus augmentation for scButterfly-T with cell_type labels or scButterfly-C with MultiVI cluster labels), and translate held-out cells with test_model. Cluster the translated profiles and compute AMI, ARI, HOM, and NMI against ground-truth labels with calculate_cluster_index. The disjointness of folds is essential: it makes the per-fold metrics paired samples, enabling one-sided paired Wilcoxon signed-rank tests over the n=140 evaluations.

## Related tools

- **scButterfly** (Provides load_data, which consumes the five-fold train_id/test_id/validation_id cell-ID lists to realize the cross-validation partitions within the translation pipeline) — https://github.com/BioX-NKU/scButterfly

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id)
```

## Evaluation signals

- Folds are disjoint: every cell appears in the test fold of exactly one of the five partitions per dataset
- All methods (scButterfly-B/T/C, BABEL, Polarbear, JAMIE) are evaluated on identical fold splits and both translation directions
- Total evaluation count equals 7 datasets × 5 folds × 2 directions = 140 per metric
- Per-fold AMI/ARI/HOM/NMI values fall in [0, 1] and can be paired across methods for Wilcoxon testing
- Aggregated table is written to benchmark_clustering_metrics.csv and reproduces Fig. 2b/c comparisons

## Limitations

- Cross-cell-type and cross-organ translation performance is flagged in the paper's discussion as needing improvement, so held-out folds containing novel cell types may degrade translation quality
- Held-out cells are translated on training-derived models, so results reflect generalization to new cells from the same dataset, not to entirely new biological contexts or batch effects
- scButterfly-T requires cell_type labels in adata.obs and scButterfly-C trains an extra MultiVI model, both adding training time per fold

## Evidence

- [abstract] superiority over baselines in five-fold CV: "we provide compelling evidence of scButterfly’s superiority over baseline methods in preserving cellular heterogeneity while translating datasets of various contexts"
- [readme] fold splits passed to load_data: [evidence span withheld — anchor status: external_doc]
- [readme] validation default of 20% of train cells: [evidence span withheld — anchor status: external_doc]
- [methods] clustering metric computation: "calculate_cluster.calculate_cluster_index"
