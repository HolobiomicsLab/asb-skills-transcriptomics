---
name: cross-method-auroc-evaluation
description: Use when comparing novel cell detection or open-set cell-type classifiers
  on scRNA-seq benchmarks where novel-class cells are positives and imbalance makes
  AUROC/AUPR/FPR95 the appropriate metrics.
when_to_use_negative:
- When the task is generic closed-set cell-type annotation accuracy (e.g., accuracy
  or macro-F1) rather than novel rare cell detection under class imbalance
- When only a single method is being run — the evaluation requires side-by-side comparison
  of scNovel and the four baselines on the same splits
- When datasets cannot be uniformly preprocessed (normalize_total to target_sum=1e4
  + log1p), since metrics are only comparable across methods on identical splits
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scNovel
  role: Method under evaluation; trained with cross-entropy loss and run in S^SIM
    query mode with novelty threshold lambda=0.5
  repo: github.com/chuanyang-Zheng/scNovel
- name: scmap-cluster
  role: Baseline method run with default parameters on the same splits
- name: scmap-cell
  role: Baseline method run with default parameters on the same splits
- name: scPred
  role: Baseline method run with default parameters on the same splits
- name: scLearn
  role: Baseline method (previously best on novel class detection) run with default
    parameters on the same splits
- name: SCANPY
  role: 'Standardized preprocessing: total-count normalization to target_sum=1e4 followed
    by log1p'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:35e25800026c78e67cef7cd113f8955e2ea86a8bc4a5ae03abdc676b26f74be3
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cross-method-auroc-evaluation

## Summary

Benchmark novel rare cell detection methods (scNovel vs scmap-cluster, scmap-cell, scPred, scLearn) on scRNA-seq data by computing AUROC (primary), AUPR, and FPR95 per dataset and per number of novel classes (C^novel = 1–4), then averaging and significance-testing the results.

## When to use

Use when comparing novel cell detection or open-set cell-type classifiers on scRNA-seq benchmarks where novel-class cells are positives and imbalance makes AUROC/AUPR/FPR95 the appropriate metrics.

## When NOT to use

- When the task is generic closed-set cell-type annotation accuracy (e.g., accuracy or macro-F1) rather than novel rare cell detection under class imbalance
- When only a single method is being run — the evaluation requires side-by-side comparison of scNovel and the four baselines on the same splits
- When datasets cannot be uniformly preprocessed (normalize_total to target_sum=1e4 + log1p), since metrics are only comparable across methods on identical splits

## Inputs

- Preprocessed scRNA-seq benchmark datasets (Zilionis, Darmanis, TM, PBMCbench subsets; 10Xv2, 10Xv3, Drop-Seq, Cel-Seq, InDrop, Seq-Well protocols) with labeled reference and query splits
- Method predictions/scores per dataset and C^novel setting (1–4) from scNovel, scmap-cluster, scmap-cell, scPred, scLearn
- Novel-class cell labels used to define positives

## Outputs

- Per-dataset, per-C^novel table of AUROC, AUPR, and FPR95 for each method (benchmark_performance_table.csv mirroring Appendix Table 2)
- Averaged performance metrics per method across C^novel values
- One-tailed t-test significance assessment of scNovel's advantage

## How to apply

Preprocess every scRNA-seq dataset identically with Scanpy: total-count normalization to target_sum=1e4 followed by log1p transformation. For each benchmark dataset (e.g., Zilionis, Darmanis, TM, PBMCbench spanning 10Xv2, 10Xv3, Drop-Seq, Cel-Seq, InDrop, Seq-Well protocols) and each C^novel setting (1, 2, 3, 4), run each method — scNovel with cross-entropy training, S^SIM query mode and default novelty threshold lambda=0.5; the four baselines at default parameters — on identical data splits. Compute AUROC (primary metric), AUPR, and FPR95 treating novel-class cells as positives. Average per-dataset metrics over all C^novel values and run a one-tailed t-test to assess statistical significance of the winning method's advantage. Assemble the averaged table (mirroring Appendix Table 2) and validate against reported expectations, e.g., scNovel as the only method with averaged AUROC above 94% (96.33% at C^novel=1) and an average gain of 11.75% AUROC over the second-best method scLearn.

## Related tools

- **scNovel** (Method under evaluation; trained with cross-entropy loss and run in S^SIM query mode with novelty threshold lambda=0.5) — github.com/chuanyang-Zheng/scNovel
- **scmap-cluster** (Baseline method run with default parameters on the same splits)
- **scmap-cell** (Baseline method run with default parameters on the same splits)
- **scPred** (Baseline method run with default parameters on the same splits)
- **scLearn** (Baseline method (previously best on novel class detection) run with default parameters on the same splits)
- **SCANPY** (Standardized preprocessing: total-count normalization to target_sum=1e4 followed by log1p)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(test_adata, target_sum=1e4); sc.pp.log1p(test_adata); import scNovel as sr; pred = sr.scNovel(test_adata, train_adata, train_label, 'gpu')
```

## Evaluation signals

- scNovel's averaged AUROC exceeds 94% (the only method to do so), reaching 96.33% AUROC at C^novel=1
- scNovel gains on average 11.75% AUROC over the second-best method scLearn, and up to 16.26% in the best case
- scNovel achieves the lowest FPR95 and the highest AUPR among the compared methods across benchmark datasets
- All methods are evaluated on identical data splits after identical Scanpy preprocessing (target_sum=1e4, log1p), so per-cell metric differences are attributable to the methods
- One-tailed t-test on averaged per-dataset metrics confirms the advantage is statistically significant

## Limitations

- The score function design is acknowledged as improvable; a better score function could improve detection performance
- The method does not incorporate other biological modalities (e.g., single-cell chromatin accessibility data) for novel rare cell discovery
- No accession IDs or dataset version numbers are given in the main text sections; benchmark comparability depends on faithfully reproducing the same data splits

## Evidence

- [full_text] Task card: benchmark protocol: [evidence span withheld — anchor status: not_found]
- [full_text] Task card: preprocessing standard: [evidence span withheld — anchor status: not_found]
- [abstract] Article: scNovel advantage: "scNovel significantly outperforms previous state-of-the-art novel cell detection models, reaching the most AUROC performance(the only one method whose averaged AUROC results are above 94%"
- [intro] Article: baseline context: "scLearn [27] achieves the best performance on novel class detection, compared to the other previous methods"
- [intro] Article: training stage: "During the training stage, we adopt cross-entropy loss to train the model to output cell types’ probability prediction after the classifier."
- [readme] README: preprocessing: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
