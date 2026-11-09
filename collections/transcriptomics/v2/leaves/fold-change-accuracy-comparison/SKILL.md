---
name: fold-change-accuracy-comparison
description: Use when two or more single-cell expression quantification tools must
  be benchmarked on simulated data against a known ground truth of cluster-marker
  TEs and their log2(FC) values.
when_to_use_negative:
- Input is real (non-simulated) scRNA-Seq data with no embedded ground-truth marker
  labels — recovery rates cannot be computed
- The aim is a marker analysis itself rather than benchmarking a quantification method's
  accuracy against a known truth
- The compared tools do not differ in expression summarization granularity (e.g. both
  locus-level), since the expected underestimation signal arises from family-metagene
  summarization
tools:
- name: Seurat
  role: FindAllMarkers is used per cluster on each method's TE expression matrix to
    recover cell-marker TEs
- name: SoloTE
  role: Produces the locus-specific TE expression matrix being benchmarked (one arm
    of the comparison)
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Produces the family-metagene TE expression matrix serving as the comparison
    baseline
- name: BEDTools
  role: Upstream dependency used by SoloTE to assess read overlap with TE annotation
    before marker benchmarking
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:af1527c0b89162bb9391c445cb1408fba248f6414c5cc449f6d27bfdb26be39c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Seurat
  - SoloTE
  - scTE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fold-change-accuracy-comparison

## Summary

Benchmark skill for evaluating how accurately a single-cell TE quantification method recovers expected cell-marker transposable elements and reproduces ground-truth average log2(fold change) values from simulated scRNA-Seq matrices, as used to compare SoloTE against scTE.

## When to use

Use when two or more single-cell expression quantification tools must be benchmarked on simulated data against a known ground truth of cluster-marker TEs and their log2(FC) values.

## When NOT to use

- Input is real (non-simulated) scRNA-Seq data with no embedded ground-truth marker labels — recovery rates cannot be computed
- The aim is a marker analysis itself rather than benchmarking a quantification method's accuracy against a known truth
- The compared tools do not differ in expression summarization granularity (e.g. both locus-level), since the expected underestimation signal arises from family-metagene summarization

## Inputs

- Simulated TE expression matrices from each quantification method (e.g. SoloTE locus-level output and scTE family-metagene output) built on hg38 and mm10 simulation ground truth
- Simulation ground-truth annotations embedding expected TE-cluster labels and ground-truth log2(FC) values
- Clustered Seurat object or cluster assignment labels per cell

## Outputs

- Per-cluster marker-recovery rates for each method (markers with adjusted P-value ≤0.05 and avg log2FC ≥1)
- Per-cluster deviation of predicted average log2(FC) from ground-truth log2(FC)
- Comparison table, e.g. marker_recovery_log2fc_comparison.csv

## How to apply

Load the simulated TE expression matrices (generated from hg38 and mm10 simulation ground truth) produced by each method being compared — e.g. locus-level matrices from SoloTE and family-metagene matrices from scTE. For each matrix, run Seurat's FindAllMarkers per cluster, keeping only markers with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1. Match recovered marker TEs against the expected TE-cluster labels embedded in the simulation ground truth and compute a per-cluster marker-recovery rate for each method. Then compare predicted average log2(FC) of recovered markers against ground-truth log2(FC) to quantify underestimation, which the article attributes to scTE summarizing TE expression at the family-metagene level. Tabulate per-cluster recovery rates and log2(FC) deviations per method as a CSV (e.g. marker_recovery_log2fc_comparison.csv).

## Related tools

- **Seurat** (FindAllMarkers is used per cluster on each method's TE expression matrix to recover cell-marker TEs)
- **SoloTE** (Produces the locus-specific TE expression matrix being benchmarked (one arm of the comparison)) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Produces the family-metagene TE expression matrix serving as the comparison baseline)
- **BEDTools** (Upstream dependency used by SoloTE to assess read overlap with TE annotation before marker benchmarking) — https://github.com/arq5x/bedtools2

## Examples

```
markers <- FindAllMarkers(seurat_obj, only.pos = TRUE); markers <- subset(markers, p_val_adj <= 0.05 & avg_log2FC >= 1)
```

## Evaluation signals

- Marker set is filtered exactly to adjusted P-value ≤0.05 AND average log2(Fold Change) ≥1 before computing recovery rates
- Every recovered marker TE must be matched against the TE-cluster labels embedded in the simulation ground truth; recovery rate is per cluster and per method
- scTE-derived markers are expected to show severely underestimated average log2(FC) in all cases; deviation direction should be consistently negative relative to ground truth
- SoloTE is expected to show consistently high marker recovery across simulated datasets, with the exception of mm10 young TEs (young/old defined by ≤10% vs >10% divergence from consensus)
- Output table contains per-cluster rows with method, recovery rate, and log2(FC) deviation, saved as marker_recovery_log2fc_comparison.csv

## Limitations

- Underestimation of log2(FC) by scTE stems from its family-metagene summarization, which loses the genomic locus of TE expression; results are specific to this granularity difference
- Simulations cover only hg38 and mm10; behavior in organisms with a greater proportion of transpositionally-active, genetically intact, multi-mapping TEs may differ
- FindAllMarkers-based recovery depends on clustering quality and the applied significance/log2FC cutoffs, so recovery rates are conditional on those thresholds

## Evidence

- [discussion] marker analysis was carried out using the "FindAllMarkers" function of Seurat, only keeping the results having adjusted P-value ≤0.05 and average log2(Fold Change) ≥1: [evidence span withheld — anchor status: not_found]
- [intro] which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed: "which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed"
- [discussion] we simulated data from the human genome hg38 version, and from the mouse genome mm10 version: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
- [abstract] SoloTE, a pipeline that outperforms the previous approach in terms of computational resources and by allowing the inclusion of locus-specific TE activity in scRNA-Seq expression matrixes: "SoloTE, a pipeline that outperforms the previous approach in terms of computational resources and by allowing the inclusion of locus-specific TE activity in scRNA-Seq expression matrixes"
