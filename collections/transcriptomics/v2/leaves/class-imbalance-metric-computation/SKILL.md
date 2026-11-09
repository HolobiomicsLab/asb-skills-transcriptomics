---
name: class-imbalance-metric-computation
description: Use when evaluating a cell-type annotation or novelty detector on scRNA-seq
  data where novel/rare cells form a small positive class and you need threshold-free
  ranking metrics (AUROC, AUPR) plus a tail metric (FPR95).
when_to_use_negative:
- The task is generic cell-type annotation of known classes with no held-out novel
  cell types — AUROC/FPR95 novelty metrics are undefined without a positive novel
  class
- Positive and negative classes are balanced and both error types matter equally —
  accuracy or macro-F1 may be more informative than AUROC/FPR95
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scNovel
  role: Primary detector whose novel-cell probability predictions are scored with
    AUROC/AUPR/FPR95; benchmarked as the only method with averaged AUROC above 94%
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Used for total-count normalization (target_sum=1e4) and log1p transformation
    of the scRNA-seq inputs prior to scoring
- name: scLearn
  role: Second-best baseline whose AUROC is used as the reference for the reported
    11.75% average improvement
- name: scPred
  role: Comparison method run with default parameters on the same data splits before
    metric computation
- name: scmap-cluster
  role: Comparison method run with default parameters on the same data splits before
    metric computation
- name: scmap-cell
  role: Comparison method run with default parameters on the same data splits before
    metric computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:8b822bbfe217b003287c624213eecb84ef6eb7a93396abcdbd5258557601c5f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scLearn
  - scNovel
  - scPred
  - scmap-cell
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# class-imbalance-metric-computation

## Summary

Computes AUROC, AUPR, and FPR95 for novel rare cell detection under severe class imbalance, treating novel-class cells as positives, and averages per-dataset metrics across C^novel settings to benchmark detectors like scNovel against scmap-cluster, scmap-cell, scPred, and scLearn.

## When to use

Use when evaluating a cell-type annotation or novelty detector on scRNA-seq data where novel/rare cells form a small positive class and you need threshold-free ranking metrics (AUROC, AUPR) plus a tail metric (FPR95).

## When NOT to use

- The task is generic cell-type annotation of known classes with no held-out novel cell types — AUROC/FPR95 novelty metrics are undefined without a positive novel class
- Positive and negative classes are balanced and both error types matter equally — accuracy or macro-F1 may be more informative than AUROC/FPR95

## Inputs

- Per-cell novelty scores or cell-type probability predictions from scNovel, scmap-cluster, scmap-cell, scPred, or scLearn on the query set
- Binary ground-truth labels marking novel-class cells as positives for each dataset/C^novel combination (C^novel = 1 to 4)
- Preprocessed scRNA-seq query/reference matrices (Scanpy, normalized to target_sum=1e4 then log1p)

## Outputs

- benchmark_performance_table.csv: averaged AUROC, AUPR, and FPR95 per method across datasets and C^novel settings (mirroring Appendix Table 2)
- One-tailed t-test significance results for scNovel's advantage over the second-best method

## How to apply

Preprocess each scRNA-seq dataset (e.g., Zilionis, Darmanis, TM, PBMCbench spanning 10Xv2, 10Xv3, Drop-Seq, Cel-Seq, InDrop, Seq-Well) with Scanpy total-count normalization to target_sum=1e4 followed by log1p, then collect each method's per-cell novelty scores or class probabilities on the query set. Treat novel-class cells as positives and compute AUROC (the primary metric), AUPR, and FPR95 — the false positive rate when 95% of novel cells are recalled — for each dataset and each C^novel setting (1, 2, 3, 4). Average the per-dataset metrics over all C^novel values and run a one-tailed t-test to assess statistical significance of the advantage of the proposed method over the second-best. Correct application is confirmed when the comparison table (mirroring Appendix Table 2) shows scNovel as the only method with averaged AUROC above 94% (96.33% at C^novel=1) and an average gain of 11.75% AUROC over scLearn, with the lowest FPR95 and highest AUPR among the compared methods.

## Related tools

- **scNovel** (Primary detector whose novel-cell probability predictions are scored with AUROC/AUPR/FPR95; benchmarked as the only method with averaged AUROC above 94%) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Used for total-count normalization (target_sum=1e4) and log1p transformation of the scRNA-seq inputs prior to scoring)
- **scLearn** (Second-best baseline whose AUROC is used as the reference for the reported 11.75% average improvement)
- **scPred** (Comparison method run with default parameters on the same data splits before metric computation)
- **scmap-cluster** (Comparison method run with default parameters on the same data splits before metric computation)
- **scmap-cell** (Comparison method run with default parameters on the same data splits before metric computation)

## Examples

```
pred_result = sr.scNovel(test_adata, train_adata, train_label, 'gpu'); # then compute AUROC/AUPR/FPR95 treating novel-class cells as positives
```

## Evaluation signals

- scNovel's averaged AUROC exceeds 94% (the only such method), reaching 96.33% at C^novel=1
- scNovel gains on average 11.75% AUROC over the second-best method scLearn, and up to 16.26% in the best case
- scNovel achieves the lowest FPR95 and the highest AUPR among all compared methods on the averaged table
- The assembled CSV mirrors Appendix Table 2, with one row per method and metrics averaged over all C^novel = 1 to 4 settings
- One-tailed t-test on per-dataset averaged metrics confirms the advantage is statistically significant

## Limitations

- A better score function could improve scNovel performance (the article acknowledges the score function design as a limitation, which directly bounds achievable AUROC/FPR95)
- Novel-class positives must be known in the ground truth for AUROC/AUPR/FPR95; metrics cannot be computed on truly unlabeled query data
- Results were validated on specific benchmark datasets (Zilionis, Darmanis, TM, PBMCbench); performance on other scales, protocols, or imbalance degrees is not guaranteed by this table

## Evidence

- [other] scNovel significantly improves the averaged performance over previous models on the novel rare cell discovery task, achieving the lowest FPR95 score, the highest AUROC and the highest AUPR among the compared methods.: [evidence span withheld — anchor status: not_found]
- [other] Compute AUROC (primary metric), AUPR and FPR95 for each method on each dataset/C^novel combination, treating novel-class cells as positives.: [evidence span withheld — anchor status: not_found]
- [abstract] scNovel significantly outperforms previous state-of-the-art novel cell detection models, reaching the most AUROC performance (the only one method whose averaged AUROC results are above 94%).: "reaching the most AUROC performance(the only one method whose averaged AUROC results are above 94%"
- [readme] The README reports up to 16.26% more AUROC compared to the second-best method via two newly designed score functions.: "proposed two new score functions, scRare especially obtains an outperformance on novel rare cell detection((up to 16.26% more AUROC compared to the second-best method)"
- [readme] The benchmark applies standardized Scanpy preprocessing before metric computation.: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
