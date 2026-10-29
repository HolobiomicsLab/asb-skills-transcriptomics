---
name: auroc-performance-computation
description: Use when comparing novel cell type detection or cell-type classification
  methods on single-cell RNA-seq query/reference pairs and you need threshold-independent
  performance metrics such as AUROC, AUPR and FPR95 per method per setting.
when_to_use_negative:
- When the task is standard cell-type annotation accuracy (e.g. accuracy or macro-F1
  over known labels) rather than novel/rare cell detection against held-out types
- When no ground-truth novel cell type labels exist to define positives/negatives,
  since AUROC/AUPR/FPR95 cannot be computed without them
- When methods are evaluated without a common preprocessing pipeline, as batch effects
  can confound metric comparison (scLearn dropped to 78.95–80.61% AUROC under cross-platform
  batch effects)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scNovel
  role: Primary novel rare cell detection method whose probability predictions are
    scored with AUROC/AUPR/FPR95
  repo: github.com/chuanyang-Zheng/scNovel
- name: scmap-cluster
  role: Baseline nearest-neighbor annotation method scored in the same AUROC/AUPR/FPR95
    comparison
- name: scmap-cell
  role: Baseline K-nearest-neighbor method scored in the same AUROC/AUPR/FPR95 comparison
- name: scPred
  role: Baseline random-forest cell-type classifier scored in the same AUROC/AUPR/FPR95
    comparison
- name: scLearn
  role: Baseline method scored in the same AUROC/AUPR/FPR95 comparison; sensitive
    to cross-platform batch effects
- name: SCANPY
  role: Standardized preprocessing (normalize_total to 1e4 + log1p) applied to all
    sets before metric computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:529b8953dd31154aad37eb11d7a141cac6c2b1aade7e0d2f55444b89d5a128a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - scLearn
  - scNovel
  - scPred
  - scmap-cell
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# auroc-performance-computation

## Summary

Computes AUROC (primary), AUPR and FPR95 to quantify how well a novel rare cell detection method separates held-out novel cell types from known reference types, as used to benchmark scNovel against scmap-cluster, scmap-cell, scPred and scLearn on PBMCbench cross-platform settings.

## When to use

Use when comparing novel cell type detection or cell-type classification methods on single-cell RNA-seq query/reference pairs and you need threshold-independent performance metrics such as AUROC, AUPR and FPR95 per method per setting.

## When NOT to use

- When the task is standard cell-type annotation accuracy (e.g. accuracy or macro-F1 over known labels) rather than novel/rare cell detection against held-out types
- When no ground-truth novel cell type labels exist to define positives/negatives, since AUROC/AUPR/FPR95 cannot be computed without them
- When methods are evaluated without a common preprocessing pipeline, as batch effects can confound metric comparison (scLearn dropped to 78.95–80.61% AUROC under cross-platform batch effects)

## Inputs

- Per-query-cell novelty/novel-cell probability scores from each method (scNovel, scmap-cluster, scmap-cell, scPred, scLearn)
- Binary ground-truth labels marking held-out novel cell types (e.g. dendritic cells, megakaryocytes) as positives
- Preprocessed Scanpy AnnData/normalized count matrices for PBMCbench reference and query sets (10Xv2, 10Xv3, Drop-Seq, Cel-Seq, InDrop, Seq-Well)

## Outputs

- Per-method, per-setting AUROC (primary metric), AUPR and FPR95 values
- Aggregated benchmark results table, e.g. pbmcbench_benchmark_results.csv

## How to apply

After running each method (scNovel, scmap-cluster, scmap-cluster/cell, scPred, scLearn) under default parameters on the evaluation setting — e.g. Cel-Seq as reference queried on 10Xv3, or 10Xv3 as reference queried on other platforms, with dendritic cells and megakaryocytes held out as novel types — score each query cell's novelty probability. Treat held-out novel cell types as positives and compute AUROC as the primary metric, alongside AUPR and FPR95 (false positive rate at 95% recall). Apply a standardized Scanpy preprocessing (total-count normalization to 1e4 followed by log1p) to all reference and query sets before scoring so metric differences reflect the methods, not preprocessing. Assemble per-method, per-setting AUROC/AUPR/FPR95 values into a single results table (e.g. pbmcbench_benchmark_results.csv) and verify against reference values such as scLearn reaching only 78.95% AUROC with a Cel-Seq reference and 80.61% with a 10Xv3 reference.

## Related tools

- **scNovel** (Primary novel rare cell detection method whose probability predictions are scored with AUROC/AUPR/FPR95) — github.com/chuanyang-Zheng/scNovel
- **scmap-cluster** (Baseline nearest-neighbor annotation method scored in the same AUROC/AUPR/FPR95 comparison)
- **scmap-cell** (Baseline K-nearest-neighbor method scored in the same AUROC/AUPR/FPR95 comparison)
- **scPred** (Baseline random-forest cell-type classifier scored in the same AUROC/AUPR/FPR95 comparison)
- **scLearn** (Baseline method scored in the same AUROC/AUPR/FPR95 comparison; sensitive to cross-platform batch effects)
- **SCANPY** (Standardized preprocessing (normalize_total to 1e4 + log1p) applied to all sets before metric computation)

## Examples

```
from sklearn.metrics import roc_auc_score, average_precision_score; auroc = roc_auc_score(is_novel, novelty_scores); aupr = average_precision_score(is_novel, novelty_scores)
```

## Evaluation signals

- AUROC is the primary metric; cross-platform reference values include scLearn at 78.95% AUROC (Cel-Seq reference, Figure 4A) and 80.61% (10Xv3 reference, Figure 4B)
- Reproduced scNovel should outperform scmap-cluster by ~6% and scLearn by ~16% in cross-platform experiments
- All methods must be run under default parameters with identical Scanpy preprocessing (normalize_total target_sum=1e4, log1p) before scoring
- Results table must contain per-method, per-setting AUROC, AUPR and FPR95 for every cross-platform pairing evaluated
- Held-out novel types (dendritic cells, megakaryocytes) must be excluded from the reference so they serve as positives for metric computation

## Limitations

- AUROC can mask behavior under extreme class imbalance typical of rare cell detection; AUPR and FPR95 are computed alongside it
- Metric comparisons are only fair under identical preprocessing — batch effects depressed scLearn to ~79–81% AUROC in cross-platform settings
- The paper notes a better score function could improve scNovel performance, implying metric values are sensitive to the novelty scoring design

## Evidence

- [other] Compute AUROC (primary metric) plus AUPR and FPR95 for each method in each cross-platform setting.: [evidence span withheld — anchor status: not_found]
- [other] scLearn reaches only 78.95% AUROC with the Cel-Seq reference and 80.61% with the 10Xv3 reference: [evidence span withheld — anchor status: not_found]
- [abstract] scNovel significantly outperforms previous state-of-the-art novel cell detection models, reaching the most AUROC performance: "scNovel significantly outperforms previous state-of-the-art novel cell detection models, reaching the most AUROC performance"
- [readme] README: Scanpy total-count normalization to 1e4 followed by log1p: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [discussion] Discussion: score function limitation affecting metric values: "Firstly, a better score function could improve scNovel performance."
