---
name: barcode-preprocessing-novelty-scoring
description: Use when a labeled scRNA-seq reference exists (e.g. healthy 54 cell types)
  and a query dataset may contain rare cell types absent from the reference (e.g.
  10 COVID-only types) that must be auto-detected.
when_to_use_negative:
- Query cells all belong to cell types already present in the reference — a plain
  auto-annotation tool (scPred, scmap-cluster, scLearn) suffices
- Input is not raw counts but already normalized/PCA-reduced features — the skill
  expects count matrices it normalizes itself (target_sum=1e4, log1p), and prior PCA-based
  feature extraction is exactly the limitation scNovel addresses
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: scNovel
  role: 'Main framework: trains the deep neural network classifier and applies barcode
    preprocessing to score query cells as known or novel'
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Preprocessing of count matrices (normalize_total to target_sum=1e4, log1p)
    and data loading before the scNovel call
- name: scLearn
  role: Baseline novel class detection method used for comparison in the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:15588c539531bd088ad2b9b4569c8c66751144e36cf0bcee407d6d42f8041f6d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scLearn
  - scNovel
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# barcode-preprocessing-novelty-scoring

## Summary

Apply scNovel's barcode preprocessing plus a cross-entropy-trained deep neural network to flag novel rare cell types in a query single-cell RNA-seq dataset against a labeled reference, thresholding predicted cell-type probabilities at lambda.

## When to use

Use when a labeled scRNA-seq reference exists (e.g. healthy 54 cell types) and a query dataset may contain rare cell types absent from the reference (e.g. 10 COVID-only types) that must be auto-detected.

## When NOT to use

- Query cells all belong to cell types already present in the reference — a plain auto-annotation tool (scPred, scmap-cluster, scLearn) suffices
- Input is not raw counts but already normalized/PCA-reduced features — the skill expects count matrices it normalizes itself (target_sum=1e4, log1p), and prior PCA-based feature extraction is exactly the limitation scNovel addresses

## Inputs

- Reference scRNA-seq expression matrix with per-cell type labels (e.g. CSV like darmanis_counts.csv / darmanis_label.csv)
- Query/test scRNA-seq expression matrix to be annotated (CSV)
- Label vector in pandas structure matching the reference cells
- processing_unit setting: 'cpu' (default) or 'gpu'

## Outputs

- Per-query-cell novelty predictions (label vs. novel) after thresholding cell-type probabilities at lambda
- Per-novel-cell-type metric table: FPR95, AUROC, AUPR against ground-truth labels
- Optionally, novel subtypes with differential gene expression analysis (e.g. three novel macrophage subtypes in the COVID-19 dataset)

## How to apply

Normalize reference and query count matrices identically with Scanpy (sc.pp.normalize_total with target_sum=1e4, then sc.pp.log1p) as in the package tutorial. Train the scNovel neural network on the reference with cross-entropy loss so the classifier outputs cell-type probability predictions; the task is described as two-stage — training, then a query stage that thresholds the probability at lambda (e.g. 0.5). Choose one of the four novel-cell-detection modes determined by the query stage option; the S^SIM mode (simultaneous T_minus and T_plus barcode preprocessing) is identified by ablation as best. For million-scale atlases (e.g. 1,462,702 cells), pass processing_unit='gpu'. Score each held-out novel type with FPR95, AUROC and AUPR against ground-truth labels.

## Related tools

- **scNovel** (Main framework: trains the deep neural network classifier and applies barcode preprocessing to score query cells as known or novel) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Preprocessing of count matrices (normalize_total to target_sum=1e4, log1p) and data loading before the scNovel call)
- **scLearn** (Baseline novel class detection method used for comparison in the benchmark)

## Examples

```
pred_result = sr.scNovel(test_adata, train_adata, train_label, 'gpu')
```

## Evaluation signals

- Metric table per novel type shows low FPR95 with high AUROC and high AUPR for distinct held-out types (e.g. Epi-Squamous, Epi-Secretory, Epi-Ciliated, Epi-AT2, Mast in the COVID atlas)
- Averaged AUROC should exceed 94% on benchmark datasets — scNovel is reported as the only method above 94%, up to 16.26% higher than the second-best method
- Types with closely related subtypes in the reference (Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2, DC-LAMP3) show relatively lower performance — an expected pattern, not a bug
- Both reference and query matrices must have received identical normalize_total(target_sum=1e4) + log1p preprocessing before the scNovel call
- On million-scale input, GPU processing_unit is used and the run completes (README progress bar finishes)

## Limitations

- The score function could be improved — the authors acknowledge a better design would improve scNovel performance
- Types with closely related subtypes in the reference (macrophage, neutrophil and DC-LAMP3 subtypes in the COVID atlas) are harder to detect as novel
- Current method uses scRNA-seq only; it does not incorporate other biology information such as single-cell chromatin accessibility
- No version numbers for scNovel or Python are given; README specifies NumPy > 1.20, Pandas > 1.2 and CUDA-enabled PyTorch is recommended

## Evidence

- [intro] Training/query two-stage design with probability thresholding: [evidence span withheld — anchor status: not_found]
- [intro] Four barcode preprocessing modes: "barcode preprocessing is utilized to boost model performance via four modes of novel cell detection"
- [abstract] Million-scale validation: "We validate scNovel's performance on a million-scale dataset to illustrate the scalability of scNovel further."
- [readme] Scanpy normalization before scNovel call: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [readme] GPU recommendation for large data: "We highly recommend setting as 'gpu' if your server supports."
