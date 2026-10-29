---
name: seurat-marker-detection-findallmarkers
description: Use when you have a gene+TE (or feature-by-cell) expression matrix with
  cluster identities and need per-cluster marker TEs/genes with log2FC and adjusted
  P-values, or when benchmarking a quantification method's marker-recovery rate against
  known cluster labels.
when_to_use_negative:
- The expression matrix has no cluster identities or cluster labels defined, since
  FindAllMarkers requires pre-defined clustering
- The input is bulk RNA-Seq TE expression (e.g., SQuIRE/Telescope-style analysis)
  rather than a cell-by-feature scRNA-Seq matrix
- The matrix was produced by an index that includes non-TE repeats such as Satellites
  without filtering, since marker calls would be confounded by non-TE elements
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: FindAllMarkers is run on the Seurat object to identify per-cluster marker
    TEs/genes, keeping adjusted P-value ≤0.05 and average log2(FC) ≥1
- name: SoloTE
  role: Produces the locus-specific gene+TE expression matrix (Seurat-compatible)
    whose markers are detected and benchmarked against scTE
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Comparator method that summarizes TE expression at family-metagene level;
    its markers and log2(FC) values are recovered for the same comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:fd3435d4ed220160cd19d940634afef032a486260aab779633e62581965d7fd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# seurat-marker-detection-findallmarkers

## Summary

Detect cell-marker features (e.g., transposable elements) per cluster from a Seurat single-cell expression matrix using FindAllMarkers, filtering to adjusted P-value ≤0.05 and average log2(Fold Change) ≥1, then score marker recovery against ground-truth cluster labels as done when benchmarking SoloTE vs scTE on simulated hg38/mm10 scRNA-Seq data.

## When to use

Use when you have a gene+TE (or feature-by-cell) expression matrix with cluster identities and need per-cluster marker TEs/genes with log2FC and adjusted P-values, or when benchmarking a quantification method's marker-recovery rate against known cluster labels.

## When NOT to use

- The expression matrix has no cluster identities or cluster labels defined, since FindAllMarkers requires pre-defined clustering
- The input is bulk RNA-Seq TE expression (e.g., SQuIRE/Telescope-style analysis) rather than a cell-by-feature scRNA-Seq matrix
- The matrix was produced by an index that includes non-TE repeats such as Satellites without filtering, since marker calls would be confounded by non-TE elements

## Inputs

- Gene + TE expression matrix per cell (Seurat-compatible, e.g., from SoloTE output or scTE output)
- Cluster/cell-type identity assignments per cell (e.g., from simulated ground truth or prior Seurat clustering)
- Ground-truth TE–cluster marker labels from the simulation (hg38 and mm10)
- Ground-truth average log2(Fold Change) values per expected marker

## Outputs

- Per-cluster marker table with adjusted P-values and average log2(Fold Change) (FindAllMarkers output, filtered)
- Per-cluster marker-recovery rate for each method (SoloTE vs scTE)
- Per-marker log2(FC) deviation from ground truth
- marker_recovery_log2fc_comparison.csv

## How to apply

Load the TE expression matrix — here produced by SoloTE (locus-level summarization of uniquely mapped reads with MAPQ ≥255, family-level for multi-mappers) or by scTE (family-metagene level) — as a Seurat object with cluster identities. Run Seurat's FindAllMarkers across all clusters, then retain only markers with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1. Match recovered marker TEs against the TE–cluster labels embedded in the simulation ground truth (hg38 and mm10 versions, TEs split into young ≤10% and old >10% divergence from consensus) and compute a per-cluster marker-recovery rate per method. Additionally compare each recovered marker's predicted average log2(FC) to the ground-truth value to quantify underestimation, which is severe for scTE because family-level metagenes dilute locus-specific signal. Tabulate per-cluster recovery rates and log2(FC) deviations for SoloTE vs scTE into marker_recovery_log2fc_comparison.csv.

## Related tools

- **Seurat** (FindAllMarkers is run on the Seurat object to identify per-cluster marker TEs/genes, keeping adjusted P-value ≤0.05 and average log2(FC) ≥1)
- **SoloTE** (Produces the locus-specific gene+TE expression matrix (Seurat-compatible) whose markers are detected and benchmarked against scTE) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Comparator method that summarizes TE expression at family-metagene level; its markers and log2(FC) values are recovered for the same comparison)

## Examples

```
markers <- Seurat::FindAllMarkers(object = seurat_obj)
```

## Evaluation signals

- All reported markers satisfy adjusted P-value ≤0.05 and average log2(Fold Change) ≥1
- Marker-recovery rate is computed per cluster per method against the TE–cluster labels embedded in the simulation ground truth
- Expected qualitative result: SoloTE shows consistently high marker recovery across simulated datasets, except mm10 young TEs
- Expected qualitative result: scTE severely underestimates average log2(FC) of markers in all cases, consistent with family-metagene summarization
- Output file marker_recovery_log2fc_comparison.csv contains per-cluster recovery rates and log2(FC) deviations for both methods

## Limitations

- In the benchmark, marker recovery failed for mm10 young TEs, so recovery rates are not uniform across TE age classes (young TEs defined as divergence from consensus ≤10%)
- scTE family-metagene summarization inherently loses genomic-locus information, so its log2(FC) values cannot be interpreted as locus-specific marker magnitudes
- Marker calls depend on upstream matrix construction: reads assigned to known genes are discarded by default (unless the 'dual' option is used), which can affect TEs fully contained within genes

## Evidence

- [discussion] Marker analysis filtering thresholds: [evidence span withheld — anchor status: not_found]
- [intro] Benchmark scope including cell marker TE detection: "First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs."
- [discussion] Seurat-compatible TE expression matrix output: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67"
- [discussion] Simulation genomes used for the benchmark: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
