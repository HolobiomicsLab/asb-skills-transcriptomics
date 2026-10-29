---
name: marker-recovery-benchmarking-against-ground-truth
description: Use when simulated or ground-truth-annotated single-cell TE expression
  matrices exist for multiple methods and you must quantify per-cluster marker recovery
  rates and log2(Fold Change) deviations, as in the SoloTE vs scTE Fig 3 comparison.
when_to_use_negative:
- Real datasets with no embedded ground-truth TE-cluster labels — there is nothing
  to compute a recovery rate against
- Input is a raw BAM/CellRanger or STAR output rather than a processed TE expression
  matrix — run SoloTE or scTE first
- Single-method characterization without a comparator method or ground truth, where
  recovery-rate benchmarking is meaningless
tools:
- name: Seurat
  role: FindAllMarkers is run on each method's TE expression matrix to identify cell
    marker TEs per cluster under the adjusted-P/log2FC thresholds
- name: SoloTE
  role: Produces the locus-level gene+TE expression matrix that is benchmarked for
    marker recovery against the simulation ground truth
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Comparator method whose family-metagene TE expression matrices are benchmarked
    for marker recovery and log2(FC) underestimation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:8c9a90edc80ca80d6766bddb94aa177d5591630ea81c0103ed5b459f12f8c974
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - Seurat
  - SoloTE
  - scTE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# marker-recovery-benchmarking-against-ground-truth

## Summary

Benchmarking procedure that evaluates how well TE-expression methods (SoloTE vs scTE) recover cell-type marker TEs from simulated scRNA-Seq matrices against an embedded simulation ground truth, by running Seurat FindAllMarkers and comparing per-cluster recovery rates and average log2(FC) values.

## When to use

Use when simulated or ground-truth-annotated single-cell TE expression matrices exist for multiple methods and you must quantify per-cluster marker recovery rates and log2(Fold Change) deviations, as in the SoloTE vs scTE Fig 3 comparison.

## When NOT to use

- Real datasets with no embedded ground-truth TE-cluster labels — there is nothing to compute a recovery rate against
- Input is a raw BAM/CellRanger or STAR output rather than a processed TE expression matrix — run SoloTE or scTE first
- Single-method characterization without a comparator method or ground truth, where recovery-rate benchmarking is meaningless

## Inputs

- Simulated TE expression matrices per method (SoloTE and scTE) generated from hg38 and mm10 simulation ground truth
- Simulation ground truth containing expected TE-cluster labels and expected per-TE average log2(FC) values
- Cluster assignments (Seurat object) for the simulated single-cell data

## Outputs

- marker_recovery_log2fc_comparison.csv with per-cluster marker-recovery rates and log2(FC) deviations for SoloTE vs scTE
- Filtered FindAllMarkers marker lists per cluster (adjusted P-value ≤0.05, average log2FC ≥1) per method
- Per-method recovery-rate and log2(FC) deviation summaries usable for Fig 3-style comparison

## How to apply

Load the simulated TE expression matrices produced on hg38 and mm10 ground truth by each method being compared (SoloTE gives a combined gene and TE matrix at locus level; scTE summarizes TE expression at the family-metagene level). For each matrix, cluster cells as in the article and run the Seurat FindAllMarkers function in R per cluster, keeping only results with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1. Match recovered marker TEs against the expected TE-cluster labels embedded in the simulation ground truth and compute the marker-recovery rate for each cluster and method. Separately, compare the predicted average log2(FC) of recovered markers to the ground-truth log2(FC) to quantify underestimation — expected to be greater for scTE because family-metagene summarization loses the genomic locus from which each TE is expressed. Tabulate per-cluster marker-recovery rates and log2(FC) deviations across methods (output: marker_recovery_log2fc_comparison.csv).

## Related tools

- **Seurat** (FindAllMarkers is run on each method's TE expression matrix to identify cell marker TEs per cluster under the adjusted-P/log2FC thresholds)
- **SoloTE** (Produces the locus-level gene+TE expression matrix that is benchmarked for marker recovery against the simulation ground truth) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Comparator method whose family-metagene TE expression matrices are benchmarked for marker recovery and log2(FC) underestimation)

## Examples

```
markers <- Seurat::FindAllMarkers(sobj, only.pos = TRUE); markers <- subset(markers, p_val_adj <= 0.05 & avg_log2FC >= 1)
```

## Evaluation signals

- Every reported marker passed both filters: adjusted P-value ≤0.05 AND average log2(Fold Change) ≥1
- Marker-recovery rate is computed per cluster per method, matched against the TE-cluster labels embedded in the simulation ground truth
- Recovered-marker average log2(FC) values are checked against ground-truth log2(FC) — scTE should show severe underestimation in all simulated cases
- SoloTE should show consistently high marker recovery across simulated datasets except mm10 young TEs
- Output CSV contains paired per-cluster rows for both methods, enabling direct SoloTE-vs-scTE deviation comparison

## Limitations

- Benchmark depends on simulation fidelity: ground truth is only from simulated hg38 and mm10 data, so results may not transfer to real datasets
- SoloTE's high recovery does not extend to mm10 young TEs — an exception observed in the comparison
- Family-metagene summarization (scTE) caps achievable log2(FC) accuracy, so the benchmark measures both marker recovery and expression-value underestimation
- TEs are classified as young/old by divergence from the consensus (≤10% or >10%), so recovery rates should be stratified on this division

## Evidence

- [discussion] Marker analysis with Seurat FindAllMarkers: [evidence span withheld — anchor status: not_found]
- [discussion] Simulation ground truth genomes: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
- [intro] Rationale for scTE log2FC underestimation: "which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed"
- [discussion] Young/old stratification for benchmarking: "TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively"
