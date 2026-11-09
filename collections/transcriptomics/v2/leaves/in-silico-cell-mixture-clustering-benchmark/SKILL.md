---
name: in-silico-cell-mixture-clustering-benchmark
description: Use when evaluating whether incorporating bulk HiChIP loop data via DC3
  improves single-cell clustering over coupled NMF or plain NMF as scRNA-seq/scATAC-seq
  dropout rises, using an in silico two-subpopulation mixture with known ground-truth
  labels.
when_to_use_negative:
- Input lacks a third bulk datatype (HiChIP loop counts) and the goal is only per-datatype
  NMF clustering — the coupled-clustering baseline suffices without loop data
- Data are truly jointly profiled single-cell multiome (e.g., sci-CAR); the article
  notes such data types are not yet incorporated into DC3 and are deferred to future
  work
- There are no ground-truth subpopulation labels, since classification error between
  true and predicted cluster assignments cannot be computed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DC3
  role: Performs simultaneous deconvolution and coupled clustering of scRNA-seq, scATAC-seq,
    and bulk HiChIP data; the method under benchmark against coupled NMF and plain
    NMF baselines
  repo: github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:e26c073902fd73cd60eb3f34d148b72418d4bbf77705bfead8d76d49bc9674bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# in-silico-cell-mixture-clustering-benchmark

## Summary

Benchmarks DC3 coupled clustering (with bulk HiChIP loop data) against coupled NMF and per-datatype NMF on an in silico GM12878/K562 mixture, measuring cluster-assignment classification error as dropout in scRNA-seq and scATAC-seq increases. It quantifies how much third-datatype (HiChIP) information rescues clustering under high sparsity.

## When to use

Use when evaluating whether incorporating bulk HiChIP loop data via DC3 improves single-cell clustering over coupled NMF or plain NMF as scRNA-seq/scATAC-seq dropout rises, using an in silico two-subpopulation mixture with known ground-truth labels.

## When NOT to use

- Input lacks a third bulk datatype (HiChIP loop counts) and the goal is only per-datatype NMF clustering — the coupled-clustering baseline suffices without loop data
- Data are truly jointly profiled single-cell multiome (e.g., sci-CAR); the article notes such data types are not yet incorporated into DC3 and are deferred to future work
- There are no ground-truth subpopulation labels, since classification error between true and predicted cluster assignments cannot be computed

## Inputs

- scRNA-seq expression matrix E (genes x cells, 146 cells: 73 GM12878 + 73 K562)
- scATAC-seq openness matrix O (enhancers x cells, 746 cells: 373 GM12878 + 373 K562)
- bulk H3K27ac HiChIP enhancer-promoter loop-count matrix (gene index, enhancer index, loop count)
- gene symbol file and enhancer symbol file
- true subpopulation labels (GM12878 vs K562) for error-rate computation

## Outputs

- scATAC-result.txt (per-cell clustering assignments for scATAC-seq)
- scRNA-result.txt (per-cell clustering assignments for scRNA-seq)
- dropout_clustering_error_rates.csv (mean error rate +/- std per method per dropout rate)
- cluster-hichip.txt and cluster-specific-peaks-genes-pairs.txt (DC3 side outputs)

## How to apply

Build the in silico mixture from public GM12878 and K562 data: expression matrix E (146 cells, 73 per cell line), openness matrix O (746 cells, 373 per cell line), and a bulk HiChIP loop-count matrix restricted to genes in E and enhancers in O. For each dropout rate in [0, 0.5, 0.8, 0.9], apply Bernoulli dropout per gene/enhancer site, zeroing dropped entries. At each rate run DC3 (scRNA-seq + scATAC-seq + bulk HiChIP input), coupled NMF, and plain NMF, each with 50 independent initializations; DC3 is invoked with -k 2 and can auto-select lambda1/lambda2 if not specified. Compute the error rate between true GM12878/K562 labels and predicted clusters, averaged over scRNA-seq and scATAC-seq cells and over the 50 runs. The validation criterion is that DC3 reduces classification error by more than four folds relative to clustering without loop data at dropout of 80% or higher, matching the paper's reported result.

## Related tools

- **DC3** (Performs simultaneous deconvolution and coupled clustering of scRNA-seq, scATAC-seq, and bulk HiChIP data; the method under benchmark against coupled NMF and plain NMF baselines) — github.com/SUwonglab/DC3

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- DC3 classification error is reduced by more than four folds versus coupled NMF (clustering without loop data) at dropout rates of 80% or higher
- Each method/dropout-rate cell in dropout_clustering_error_rates.csv averages 50 independent runs with different initializations and reports standard deviation
- DC3 runs converge with -k 2 clusters matching the two known subpopulations (GM12878 and K562), yielding assignments for both scRNA-result.txt and scATAC-result.txt
- Error rates increase monotonically with dropout rate for all methods, with the DC3 advantage growing as dropout rises

## Limitations

- Benchmark uses a two-subpopulation in silico mixture from GM12878/K562; performance on more complex or in vivo heterogeneous populations requires separate validation
- DC3 does not yet support jointly profiled single-cell multiomics data types (e.g., sci-CAR); modifying its cost function for such data is future work
- Baseline plain NMF clusters each single-cell data type separately, so the comparison only covers the input combinations described in the paper

## Evidence

- [abstract] Introduce DC3 via joint analysis improving coupled clustering: "we introduce DC3 (De-Convolution and Coupled-Clustering) as a method for the joint analysis of various bulk and single-cell data"
- [discussion] Discussion claim that DC3 enhances single-cell clustering: "We showed that DC3 can decompose bulk profiles into subpopulation-specific profiles and at the same time enhance clustering performance of the single-cell data."
- [readme] README statement that subpopulation-specific loop counts improve clustering: "At the same time, the subpopulation-specific loop counts in turn lead to improved clustering results in the scRNA-seq and scATAC-seq data."
- [readme] README DC3 command-line parameters and outputs: "**DC3 outputs 4 files:** * scATAC-result.txt                       the clustering results for scATAC-seq"
