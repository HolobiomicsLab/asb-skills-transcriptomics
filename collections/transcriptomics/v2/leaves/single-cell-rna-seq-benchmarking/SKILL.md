---
name: single-cell-rna-seq-benchmarking
description: Use when comparing wall-clock runtime or novel-cell-detection performance
  of scRNA-seq annotation tools as dataset size grows from ~1k to million-scale cells,
  e.g. showing whether a tool scales efficiently versus scmap-cluster/scPred.
when_to_use_negative:
- The goal is annotation accuracy on a single small dataset without a runtime or scaling
  comparison across dataset sizes
- The datasets lack labeled reference sets, since scPred and scmap-cluster require
  labeled references for the comparison splits
- Data are not single-cell transcriptomic count matrices (e.g. bulk RNA-seq or pre-computed
  feature tables)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scNovel
  role: 'Tool under benchmark: trained on reference sets (cross-entropy loss, lambda=0.5)
    and timed at the query stage with processing_unit=''gpu'' where available'
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: 'Preprocessing for all benchmarked methods: sc.pp.normalize_total(target_sum=1e4)
    followed by sc.pp.log1p on reference and query matrices'
- name: scmap-cluster
  role: Fastest baseline in the runtime comparison; run with default parameters on
    identical reference/query splits
- name: scPred
  role: Baseline comparison method (random forest classifier); its runtime is the
    reference point for the >120x speedup on the TM dataset
- name: PyTorch
  role: Backend for scNovel's neural network; Cudatoolkit recommended for GPU-accelerated
    benchmark runs
merged_aliases:
- single-cell-rna-seq-scaling-benchmarking
merged_alias_records:
- alias: single-cell-rna-seq-preprocessing
  slug: single-cell-rna-seq-preprocessing
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:b03cc31291a6f325fc4a1701dd339d64781a185cbb7ee35d039947e163366657
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - PyTorch
  - Python
  - SCANPY
  - scNovel
  - scPred
  - scSPARKL
  - scmap-cluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# single-cell-rna-seq-benchmarking

## Summary

Benchmarking runtime and detection performance of single-cell novel-cell discovery tools (scNovel, scmap-cluster, scPred) across datasets spanning ~1k cells (Darmanis) to the million-scale TM atlas, after Scanpy preprocessing (normalize_total to 1e4 + log1p).

## When to use

Use when comparing wall-clock runtime or novel-cell-detection performance of scRNA-seq annotation tools as dataset size grows from ~1k to million-scale cells, e.g. showing whether a tool scales efficiently versus scmap-cluster/scPred.

## When NOT to use

- The goal is annotation accuracy on a single small dataset without a runtime or scaling comparison across dataset sizes
- The datasets lack labeled reference sets, since scPred and scmap-cluster require labeled references for the comparison splits
- Data are not single-cell transcriptomic count matrices (e.g. bulk RNA-seq or pre-computed feature tables)

## Inputs

- scRNA-seq count matrices for reference and query sets (CSV format as in darmanis_counts.csv)
- Cell-type label vectors (CSV as in darmanis_label.csv)
- Benchmark datasets spanning size range: Darmanis (~1k cells), Zilionis (~60k cells), TM million-scale dataset

## Outputs

- Tabulated runtime per tool (scNovel, scmap-cluster, scPred) per dataset size
- Runtime scaling comparison showing scNovel's consistent runtime across sample sizes
- Speedup figures (e.g. up to 25% faster than scmap-cluster for >30k cells; >120x faster than scPred on TM)

## How to apply

Load reference and query scRNA-seq count matrices and labels (e.g. Darmanis ~1k cells, Zilionis ~60k cells, and the TM million-scale dataset) via Scanpy, applying sc.pp.normalize_total(target_sum=1e4) followed by sc.pp.log1p to both sets — the preprocessing applied identically to all benchmarked methods. Train scNovel on each reference set with cross-entropy loss and default novelty threshold lambda = 0.5, keeping processing_unit fixed (e.g. 'gpu') across datasets, and record total wall-clock runtime at the query stage. Run scmap-cluster and scPred on the same reference/query splits with default parameters and tabulate runtime per tool per dataset size; note that dataloader preparation dominates small-dataset runtimes, so judge scalability mainly from large-dataset behavior. Validate against the paper's reported benchmarks: scNovel's runtime stays relatively consistent across sizes, cuts runtime by up to 25% versus the fastest baseline scmap-cluster for datasets with >30k cells, and is over 120x faster than scPred on the TM dataset.

## Related tools

- **scNovel** (Tool under benchmark: trained on reference sets (cross-entropy loss, lambda=0.5) and timed at the query stage with processing_unit='gpu' where available) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Preprocessing for all benchmarked methods: sc.pp.normalize_total(target_sum=1e4) followed by sc.pp.log1p on reference and query matrices)
- **scmap-cluster** (Fastest baseline in the runtime comparison; run with default parameters on identical reference/query splits)
- **scPred** (Baseline comparison method (random forest classifier); its runtime is the reference point for the >120x speedup on the TM dataset)
- **PyTorch** (Backend for scNovel's neural network; Cudatoolkit recommended for GPU-accelerated benchmark runs)

## Examples

```
import scanpy as sc; import scNovel as sr; sc.pp.normalize_total(train_adata, target_sum=1e4); sc.pp.log1p(train_adata); pred_result = sr.scNovel(test_adata, train_adata, train_label, 'gpu')
```

## Evaluation signals

- scNovel's wall-clock runtime remains relatively consistent across datasets of varying sample sizes
- For datasets with more than 30k cells, scNovel reduces runtime by up to 25% compared to scmap-cluster (the fastest comparison model)
- On the TM million-scale dataset, scNovel is over 120 times faster than scPred
- All methods receive identical preprocessing (normalize_total to target_sum=1e4 + log1p) and the same processing_unit setting across datasets
- Small-dataset runtimes are dominated by dataloader preparation, so small-dataset timings are not used to judge scalability

## Limitations

- Dataloader preparation dominates runtime for small datasets (~1k cells), so timing comparisons at that scale are not informative about scalability
- GPU acceleration (processing_unit='gpu') is strongly recommended; CPU default runs will skew runtime comparisons
- Benchmark conclusions depend on the specific datasets (Darmanis, Zilionis, TM) and splits used; no accession IDs or dataset sources are given in the discussion section

## Evidence

- [abstract] We validate scNovel's performance on a million-scale dataset to illustrate the scalability of scNovel further.: "We validate scNovel's performance on a million-scale dataset to illustrate the scalability of scNovel further."
- [intro] First tool validated on million-level atlas: "our work is the first and only novel rare cell discovery tool validated on a million-level atlas"
- [discussion] Open-source package description: "we propose the first corresponding auto-detection model named scNovel, an open-source Python package that integrates neural network and barcode preprocessing"
- [readme] Runtime benchmark call: "pred_result = sr.scNovel(test_adata, train_adata, train_label, processing_unit)"
- [readme] Standard preprocessing applied: "sc.pp.normalize_total(train_adata, target_sum=1e4)"
