---
name: cluster-assignment-error-rate-evaluation
description: Use when comparing cell-clustering accuracy of methods that assign single
  cells (scRNA-seq/scATAC-seq) to known subpopulations, e.g. testing whether loop
  data reduces error as dropout increases.
when_to_use_negative:
- Ground-truth subpopulation labels are unavailable, so no reference exists against
  which to compute a classification error rate
- The task is deconvolution of bulk profiles (accuracy of subpopulation-specific expression/accessibility/loop
  profiles), not cluster-assignment accuracy of single cells
- A single deterministic clustering run suffices; the 50-initialization averaging
  protocol is designed for stochastic NMF-based methods sensitive to initialization
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: DC3
  role: Method under evaluation; its coupled-clustering assignments (scRNA-result.txt,
    scATAC-result.txt) are scored against true GM12878/K562 labels in the error-rate
    comparison against coupled NMF and plain NMF baselines
  repo: github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:7136105787d20b4c72932922ec1e4addfd785aa580d03d65ea8bc44fb7cf215b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cluster-assignment-error-rate-evaluation

## Summary

Evaluates clustering quality of single-cell methods (e.g., DC3 vs coupled NMF vs plain NMF) by computing the classification error rate between true subpopulation labels and predicted cluster assignments, averaged across modalities and independent runs. In the DC3 paper this quantifies how bulk HiChIP loop data improves coupled clustering under varying dropout.

## When to use

Use when comparing cell-clustering accuracy of methods that assign single cells (scRNA-seq/scATAC-seq) to known subpopulations, e.g. testing whether loop data reduces error as dropout increases.

## When NOT to use

- Ground-truth subpopulation labels are unavailable, so no reference exists against which to compute a classification error rate
- The task is deconvolution of bulk profiles (accuracy of subpopulation-specific expression/accessibility/loop profiles), not cluster-assignment accuracy of single cells
- A single deterministic clustering run suffices; the 50-initialization averaging protocol is designed for stochastic NMF-based methods sensitive to initialization

## Inputs

- scRNA-seq expression matrix E with ground-truth subpopulation labels (e.g., 146 GM12878/K562 cells, 73 per cell line)
- scATAC-seq openness matrix O with ground-truth subpopulation labels (e.g., 746 cells, 373 per cell line)
- Bulk HiChIP enhancer-promoter loop-count matrix (gene index, enhancer index, loop counts)
- DC3 clustering output files scRNA-result.txt and scATAC-result.txt per run

## Outputs

- Per-run cluster-assignment classification error rates versus true subpopulation labels
- Table of mean error rates with standard deviations per method per dropout rate (e.g., dropout_clustering_error_rates.csv)
- Figure-2b-style comparison of DC3 vs coupled NMF vs plain NMF across dropout rates [0, 0.5, 0.8, 0.9]

## How to apply

Build an in silico mixture with known ground-truth labels (e.g., GM12878 vs K562: 146 scRNA-seq cells with 73 per cell line, 746 scATAC-seq cells with 373 per cell line) plus a bulk HiChIP loop-count matrix. Simulate dropout per site (gene or enhancer) by Bernoulli draws at rates such as 0, 0.5, 0.8, and 0.9, zeroing dropped entries in the expression matrix E and openness matrix O. At each dropout rate, run each method (DC3 with scRNA-seq + scATAC-seq + bulk HiChIP; coupled NMF without loop data; plain NMF per data type) using 50 independent runs with different initializations to account for NMF initialization sensitivity. For each run, compute the error rate between true subpopulation labels and the predicted cluster assignments from DC3's clustering outputs (scRNA-result.txt and scATAC-result.txt), then average over scRNA-seq and scATAC-seq samples and over the 50 runs. Assemble mean error rates with standard deviations per method per dropout rate (e.g., into dropout_clustering_error_rates.csv); validation expects DC3 to reduce classification error by more than four folds relative to clustering without loop data at dropout rates of 80% or higher.

## Related tools

- **DC3** (Method under evaluation; its coupled-clustering assignments (scRNA-result.txt, scATAC-result.txt) are scored against true GM12878/K562 labels in the error-rate comparison against coupled NMF and plain NMF baselines) — github.com/SUwonglab/DC3

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Classification error is computed against known in silico mixture labels (GM12878 vs K562), so predicted clusters must align with the 73/146 and 373/746 per-cell-line splits before scoring
- Results are averaged over 50 independent runs with different initializations; report standard deviations alongside means to show run-to-run stability
- Invariant check: DC3's mean error rate should be more than four folds lower than coupled NMF (no loop data) at dropout rates of 80% or higher, reproducing the paper's Fig 2b finding
- Output table must cover all tested dropout rates (0, 0.5, 0.8, 0.9) and all three methods (DC3, coupled NMF, plain NMF) with errors averaged over both scRNA-seq and scATAC-seq samples
- At 0% dropout, plain NMF and coupled NMF errors should be comparable to DC3, since the loop-data advantage is expected mainly at high dropout

## Limitations

- Evaluation relies on an in silico mixture with known labels; error rates on real heterogeneous populations without ground truth cannot be computed this way
- Bernoulli dropout simulation is a specific degradation model; results at rates such as 80-90% may not transfer to other noise regimes
- DC3's cost function does not yet incorporate jointly-profiled single-cell data types (e.g., sci-CAR), which the authors deferred to future work
- NMF-based methods are initialization-sensitive, so single-run error estimates are unreliable; the protocol requires many independent runs

## Evidence

- [abstract] DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [discussion] DC3 enhances clustering of single-cell data: "We showed that DC3 can decompose bulk profiles into subpopulation-specific profiles and at the same time enhance clustering performance of the single-cell data."
- [readme] Loop counts improve clustering results: "At the same time, the subpopulation-specific loop counts in turn lead to improved clustering results in the scRNA-seq and scATAC-seq data."
- [readme] DC3 outputs clustering result files: "DC3 outputs 4 files: scATAC-result.txt the clustering results for scATAC-seq; scRNA-result.txt the clustering results for scRNA-seq"
