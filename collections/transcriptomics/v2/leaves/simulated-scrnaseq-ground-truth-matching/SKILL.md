---
name: simulated-scrnaseq-ground-truth-matching
description: Use when simulated single-cell expression matrices (with known TE-cell
  labels and ground-truth log2(FC)) must be scored against a tool's recovered markers,
  e.g. benchmarking SoloTE vs scTE in Fig 3.
when_to_use_negative:
- Input is real (non-simulated) scRNA-Seq data with no ground-truth TE-cell labels
  or known log2(FC) values — matching cannot be computed
- You only need to rank/cluster cells (e.g. UMAP dimensional reduction comparison),
  not assess marker recovery against truth
- The expression matrix is gene-level only and contains no TE features to match against
  ground truth
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: FindAllMarkers per cluster to identify cell-marker TEs with adjusted P-value
    ≤0.05 and average log2(FC) ≥1
- name: SoloTE
  role: Produces the locus-level simulated TE expression matrix being benchmarked
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Baseline method producing family-metagene TE matrices whose log2(FC) underestimation
    is quantified
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:d2a4ecdd467a1ccd2b147267fbe9cdb49087fc587c5a7dcbc922c77392bf9a18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# simulated-scrnaseq-ground-truth-matching

## Summary

Benchmarks TE expression quantification methods (SoloTE vs scTE) by matching cell-marker TEs recovered from simulated hg38/mm10 scRNA-Seq matrices against the TE-cluster labels and log2(FC) values embedded in the simulation ground truth. Quantifies marker-recovery rates and the family-metagene underestimation of log2(FC).

## When to use

Use when simulated single-cell expression matrices (with known TE-cell labels and ground-truth log2(FC)) must be scored against a tool's recovered markers, e.g. benchmarking SoloTE vs scTE in Fig 3.

## When NOT to use

- Input is real (non-simulated) scRNA-Seq data with no ground-truth TE-cell labels or known log2(FC) values — matching cannot be computed
- You only need to rank/cluster cells (e.g. UMAP dimensional reduction comparison), not assess marker recovery against truth
- The expression matrix is gene-level only and contains no TE features to match against ground truth

## Inputs

- Simulated TE expression matrix from SoloTE (locus-level, hg38/mm10)
- Simulated TE expression matrix from scTE (family-metagene level, hg38/mm10)
- Simulation ground truth with expected TE-cluster labels and ground-truth log2(FC) per marker TE

## Outputs

- Per-cluster marker-recovery rates for each method
- Per-marker log2(FC) deviations from ground truth (underestimation quantified)
- marker_recovery_log2fc_comparison.csv

## How to apply

Load the simulated TE expression matrices produced by each method (SoloTE locus-level output and scTE family-metagene output) derived from the hg38 and mm10 simulation ground truth. For each matrix, run Seurat's FindAllMarkers per cluster and keep only markers with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1. Match the recovered marker TEs against the expected TE-cluster labels embedded in the simulation ground truth and compute the per-cluster marker-recovery rate for each method. Then compare predicted average log2(FC) of recovered markers to the ground-truth log2(FC) to quantify underestimation — expected to be severe for scTE because it summarizes TE expression at the family-metagene level, losing locus information. Optionally stratify TEs as young (divergence from consensus ≤10%) vs old (>10%), since SoloTE recovery is high across simulated datasets except mm10 young TEs. Tabulate per-cluster recovery rates and log2(FC) deviations as marker_recovery_log2fc_comparison.csv.

## Related tools

- **Seurat** (FindAllMarkers per cluster to identify cell-marker TEs with adjusted P-value ≤0.05 and average log2(FC) ≥1)
- **SoloTE** (Produces the locus-level simulated TE expression matrix being benchmarked) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Baseline method producing family-metagene TE matrices whose log2(FC) underestimation is quantified)

## Examples

```
markers <- FindAllMarkers(seurat_obj, only.pos = TRUE); markers <- subset(markers, p_val_adj <= 0.05 & avg_log2FC >= 1)
```

## Evaluation signals

- Every kept marker row satisfies adjusted P-value ≤0.05 AND average log2(Fold Change) ≥1
- Each recovered marker TE maps to exactly one expected TE-cluster label in the simulation ground truth; unrecovered labels appear as zero-recovery clusters
- SoloTE marker-recovery rate is high across all simulated datasets except mm10 young TEs (divergence from consensus ≤10%)
- scTE recovered markers show systematically lower predicted average log2(FC) than the ground-truth log2(FC), consistent with family-metagene summarization
- Output CSV contains per-cluster recovery rates and log2(FC) deviations for both methods under a consistent column schema

## Limitations

- SoloTE shows reduced marker recovery for mm10 young TEs (divergence from consensus ≤10%), because most copies of transpositionally-active TEs can be genetically intact and multi-mapping
- scTE indexes do not filter non-TE repeats such as Satellites, which can contaminate the matrix being benchmarked
- Results depend on the hg38/mm10 simulation design; recovery rates do not generalize to other genomes without regenerating ground truth

## Evidence

- [discussion] marker analysis was carried out using the "FindAllMarkers" function of Seurat, only keeping the results having adjusted P-value ≤0.05 and average log2(Fold Change) ≥1: [evidence span withheld — anchor status: not_found]
- [intro] which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed: "which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed"
- [discussion] we simulated data from the human genome hg38 version, and from the mouse genome mm10 version: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
- [discussion] TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively: "TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively"
- [discussion] Compatible output matrix description from README: a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67"
