---
name: baseline-method-default-parameter-benchmarking
description: Use when comparing novel cell detection/classification baselines on scRNA-seq
  reference-query splits, especially cross-platform settings where batch effects may
  degrade competitor methods like scLearn.
when_to_use_negative:
- The goal is hyperparameter tuning or performance optimization of a single method
  — this skill deliberately runs all methods at default parameters for fair baseline
  comparison
- The query set contains no novel cell types (all query cell types are present in
  the reference) — the benchmark design requires held-out novel types such as dendritic
  cells and megakaryocytes
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scNovel
  role: Primary method under evaluation; deep learning-based novel rare cell detection
    with barcode preprocessing
  repo: github.com/chuanyang-Zheng/scNovel
- name: scmap-cluster
  role: Baseline competitor performing nearest neighbor search between cells from
    different datasets
- name: scmap-cell
  role: Baseline competitor using approximate K-nearest neighbor search between reference
    and query
- name: scPred
  role: Baseline competitor using a random forest classifier to predict cell-type
    labels
- name: scLearn
  role: Baseline competitor previously achieving best novel class detection performance;
    benchmarks batch-effect sensitivity
- name: SCANPY
  role: Standardized preprocessing (normalize_total to 1e4, log1p) applied to all
    reference and query sets before benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:7e94d057a0ae459a51dad2f7bd75443f339ee61f6824bb4e8ecb7710ded732b4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# baseline-method-default-parameter-benchmarking

## Summary

Benchmarks novel rare cell detection methods (scNovel vs. scmap-cluster, scmap-cell, scPred, scLearn) on cross-platform PBMCbench settings using default parameters, scoring AUROC/AUPR/FPR95 to quantify robustness to batch effects.

## When to use

Use when comparing novel cell detection/classification baselines on scRNA-seq reference-query splits, especially cross-platform settings where batch effects may degrade competitor methods like scLearn.

## When NOT to use

- The goal is hyperparameter tuning or performance optimization of a single method — this skill deliberately runs all methods at default parameters for fair baseline comparison
- The query set contains no novel cell types (all query cell types are present in the reference) — the benchmark design requires held-out novel types such as dendritic cells and megakaryocytes

## Inputs

- PBMCbench reference/query scRNA-seq count matrices spanning 10Xv2, 10Xv3, Drop-Seq, Cel-Seq, InDrop and Seq-Well platforms
- Cell-type label vectors for reference and query sets (novel types dendritic cells and megakaryocytes held out from reference)
- scNovel Python package (pip install scNovel==1.1.0) with Scanpy, PyTorch, NumPy > 1.20, Pandas > 1.2

## Outputs

- Per-method, per-setting AUROC/AUPR/FPR95 table (e.g., pbmcbench_benchmark_results.csv)
- Novel-cell prediction scores/labels for each of the five methods on each cross-platform setting

## How to apply

Standardize all reference and query sets with Scanpy preprocessing (sc.pp.normalize_total with target_sum=1e4 followed by sc.pp.log1p) before running any method. Define two cross-platform settings — Cel-Seq as reference queried on 10Xv3, and 10Xv3 as reference queried on other platforms — holding out dendritic cells and megakaryocytes as the novel cell types. Run all five methods (scNovel, scmap-cluster, scmap-cell, scPred, scLearn) under their default parameter settings so differences reflect method robustness rather than tuning. Compute AUROC as the primary metric plus AUPR and FPR95 per method per setting, then assemble results into a single table (e.g., pbmcbench_benchmark_results.csv). Verify reproduction against reported values: scLearn reaches only 78.95% AUROC with a Cel-Seq reference and 80.61% with a 10Xv3 reference, and scNovel outperforms scmap-cluster by 6% and scLearn by 16%.

## Related tools

- **scNovel** (Primary method under evaluation; deep learning-based novel rare cell detection with barcode preprocessing) — github.com/chuanyang-Zheng/scNovel
- **scmap-cluster** (Baseline competitor performing nearest neighbor search between cells from different datasets)
- **scmap-cell** (Baseline competitor using approximate K-nearest neighbor search between reference and query)
- **scPred** (Baseline competitor using a random forest classifier to predict cell-type labels)
- **scLearn** (Baseline competitor previously achieving best novel class detection performance; benchmarks batch-effect sensitivity)
- **SCANPY** (Standardized preprocessing (normalize_total to 1e4, log1p) applied to all reference and query sets before benchmarking)

## Examples

```
import scNovel as sr; pred_result = sr.scNovel(test_adata, train_adata, train_label, 'gpu')
```

## Evaluation signals

- scLearn's AUROC reproduces the reported batch-effect-degraded values: 78.95% with a Cel-Seq reference and 80.61% with a 10Xv3 reference
- Relative gaps reproduce: scNovel outperforms scmap-cluster by ~6% and scLearn by ~16% in cross-platform settings
- All five methods ran with default parameters on identically preprocessed data (normalize_total 1e4 + log1p) — no method-specific tuning
- Output table contains AUROC, AUPR, and FPR95 for every method in both cross-platform settings (Cel-Seq reference and 10Xv3 reference)

## Limitations

- scLearn is highly sensitive to cross-platform batch effects, dropping to ~78.95–80.61% AUROC, which inflates the apparent advantage of batch-robust methods
- The article acknowledges scNovel's score function could be improved with a better design
- scNovel does not incorporate other biological information (e.g., single-cell chromatin accessibility data)

## Evidence

- [readme] scanpy preprocessing with default parameters: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [readme] scNovel API invocation from README: "pred_result = sr.scNovel(test_adata, train_adata, train_label, processing_unit)"
- [intro] scLearn as strongest prior baseline: "scLearn [27] achieves the best performance on novel class detection, compared to the other previous methods"
- [abstract] scNovel superiority claim underpinning the benchmark: "scNovel significantly outperforms previous state-of-the-art novel cell detection models"
- [readme] Installation command for the benchmarked package: "pip install scNovel==1.1.0"
