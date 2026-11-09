---
name: novel-rare-cell-detection-benchmarking
description: Use when a labeled scRNA-seq reference exists but the query set may contain
  cell types absent from the reference, or when comparing novel-cell-detection methods
  (scNovel vs scmap/scPred/scLearn) with AUROC, AUPR and FPR95 across C^novel = 1–4
  settings.
when_to_use_negative:
- There is no labeled reference set of known cell types — scNovel's training stage
  requires a reference with cross-entropy-trained cell-type labels
- The goal is generic auto-annotation of cell types already present in the reference,
  not identifying novel/rare types absent from the reference (the article notes prior
  annotation tools 'paid little attention to novel rare cell discovery')
- The data are single-cell chromatin accessibility or other non-transcriptomic modalities
  — the paper states the current method does not incorporate such biology information
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scNovel
  role: 'Primary method: cross-entropy-trained neural network with barcode preprocessing
    and novelty threshold lambda = 0.5 for novel rare cell detection'
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Standardized preprocessing (normalize_total target_sum=1e4, log1p), Leiden
    clustering of novel cells, and UMAP visualization
- name: scmap-cluster
  role: 'Baseline: nearest-neighbor-search-based reference mapping for novel cell
    detection, run with default parameters'
- name: scmap-cell
  role: 'Baseline: approximate K-nearest-neighbor search between reference and query
    datasets, run with default parameters'
- name: scPred
  role: 'Baseline: random-forest cell-type predictor, run with default parameters'
- name: scLearn
  role: Strongest prior baseline for novel class detection; second-best method against
    which scNovel's ~11.75% average AUROC gain is measured
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:24e72ba2e690a2062a398cce0c8ed26cf1049624e906aa4f45fcaa7d67543db3
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# novarare-cell-detection-benchmarking

## Summary

Benchmark novel rare cell detection in single-cell transcriptomics by running scNovel (a deep neural network with barcode preprocessing) against scmap-cluster, scmap-cell, scPred and scLearn, scoring AUROC/AUPR/FPR95 with novel cells as positives across datasets of different scales and sequencing protocols.

## When to use

Use when a labeled scRNA-seq reference exists but the query set may contain cell types absent from the reference, or when comparing novel-cell-detection methods (scNovel vs scmap/scPred/scLearn) with AUROC, AUPR and FPR95 across C^novel = 1–4 settings.

## When NOT to use

- There is no labeled reference set of known cell types — scNovel's training stage requires a reference with cross-entropy-trained cell-type labels
- The goal is generic auto-annotation of cell types already present in the reference, not identifying novel/rare types absent from the reference (the article notes prior annotation tools 'paid little attention to novel rare cell discovery')
- The data are single-cell chromatin accessibility or other non-transcriptomic modalities — the paper states the current method does not incorporate such biology information

## Inputs

- Reference scRNA-seq expression matrix (cells x genes, e.g. CSV loadable via sc.read_csv) with a corresponding label vector in pandas structure
- Query scRNA-seq expression matrix to be annotated (e.g. darmanis_counts.csv format)
- AnnData objects for large atlas queries (e.g. GSE171524 with 116,314 cells; 1,462,702-cell COVID-19 immune atlas)
- Ground-truth labels marking held-out novel cell types (e.g. 10 COVID-only types: Epi-AT2, Epi-Ciliated, Epi-Secretory, Epi-Squamous, Mast, Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2, DC-LAMP3)

## Outputs

- Per-cell novelty flags and cell-type probability predictions (novel if score > lambda = 0.5)
- Per-dataset/per-C^novel AUROC, AUPR and FPR95 table (benchmark_performance_table.csv)
- Cross-platform PBMCbench results table (pbmcbench_benchmark_results.csv)
- Leiden clusters of detected novel cells plus UMAP figures (e.g. figure6_novel_detection.png) and marker-expression summaries (e.g. CD163-high Macrophage clusters 3, 5, 12)

## How to apply

Preprocess reference and query matrices in Scanpy with normalize_total(target_sum=1e4) followed by log1p, matching the package tutorial. Train the scNovel classifier with cross-entropy loss on the reference so it outputs per-cell-type probability predictions, then run the query stage in S^SIM mode (simultaneous T_minus and T_plus barcode preprocessing, the best ablation mode) flagging cells as novel when the novelty score exceeds threshold lambda = 0.5. Run the baselines scmap-cluster, scmap-cell, scPred and scLearn on the same splits with default parameters. Treat novel-class cells as positives and compute AUROC (primary), AUPR and FPR95 per dataset/C^novel combination, then average over C^novel and run a one-tailed t-test for significance. For scalability checks, apply the same pipeline to million-scale references (e.g. the 1,462,702-cell COVID-19 immune atlas with the healthy group's 54 cell types as reference) using GPU processing, and Leiden-cluster the detected novel cells followed by marker checks (e.g. CD163 for Macrophage subtypes).

## Related tools

- **scNovel** (Primary method: cross-entropy-trained neural network with barcode preprocessing and novelty threshold lambda = 0.5 for novel rare cell detection) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Standardized preprocessing (normalize_total target_sum=1e4, log1p), Leiden clustering of novel cells, and UMAP visualization)
- **scmap-cluster** (Baseline: nearest-neighbor-search-based reference mapping for novel cell detection, run with default parameters)
- **scmap-cell** (Baseline: approximate K-nearest-neighbor search between reference and query datasets, run with default parameters)
- **scPred** (Baseline: random-forest cell-type predictor, run with default parameters)
- **scLearn** (Strongest prior baseline for novel class detection; second-best method against which scNovel's ~11.75% average AUROC gain is measured)

## Examples

```
import scNovel as sr
pred_result = sr.scNovel(test_adata, train_adata, train_label, processing_unit='gpu')
```

## Evaluation signals

- scNovel should be the only method whose averaged AUROC exceeds 94% (96.33% AUROC at C^novel = 1), with the lowest FPR95 and highest AUROC/AUPR among compared methods
- Averaged AUROC gain over the second-best method scLearn should be about 11.75% (up to 16.26%)
- Cross-platform PBMCbench checks: scLearn degrades to 78.95% AUROC with a Cel-Seq reference and 80.61% with a 10Xv3 reference, while scNovel outperforms scmap-cluster by ~6% and scLearn by ~16%
- On GSE171524, B cell and T cell should rarely be flagged novel while AT1 and AT2 are successfully detected as novel; CD163-high novel clusters indicate candidate Macrophage subtypes
- Million-scale run: Epi types and 17-cell Mast should show low FPR95 with high AUROC/AUPR, while Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2 and DC-LAMP3 show relatively lower performance when close subtypes exist in the reference

## Limitations

- The score function could be improved — the authors acknowledge 'a better score function could improve scNovel performance'
- The current method does not incorporate other biology information such as single-cell chromatin accessibility data
- Method performance degrades when closely related subtypes of a novel type already exist in the reference (seen for Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2, DC-LAMP3)
- README/package naming is inconsistent: the README introduces the framework as 'scRare' while the install command is 'pip install scNovel==1.1.0', and the README citation differs from the published author list

## Evidence

- [abstract] In this work, we introduce scNovel, a powerful deep learning-based neural network that specifically focuses on novel rare cell discovery.: "In this work, we introduce scNovel, a powerful deep learning-based neural network that specifically focuses on novel rare cell discovery."
- [intro] Two-stage training/query design: "During the training stage, we adopt cross-entropy loss to train the model to output cell types’ probability prediction after the classifier. During the query stage"
- [discussion] Barcode preprocessing and Python package: "an open-source Python package that integrates neural network and barcode preprocessing"
- [readme] README preprocessing commands: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [readme] README install and requirements: "pip install scNovel==1.1.0"
