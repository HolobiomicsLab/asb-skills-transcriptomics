---
name: spearman-rank-correlation-analysis
description: Use when you have ranked gene lists from two or more SVG/HVG identification
  methods on the same dataset and need to quantify rank agreement via Spearman correlations
  plus top-n (n = 10, 20, 50, 100, 200) overlap counts.
when_to_use_negative:
- The goal is to identify or rank SVGs themselves on a new dataset rather than to
  compare existing rankings — run nnSVG (or a baseline) directly instead
- Only raw counts and spatial coordinates are available with no per-method ranked
  gene lists — the correlations require ranked outputs (e.g., nnSVG rank/LR_stat columns)
  from each method first
- Comparing methods across different tissues or spot sets, where gene rankings are
  not defined over a shared gene universe and set overlaps are not meaningful
edam_operation: null
tools:
- name: nnSVG
  role: Source of the reference SVG rankings (LR_stat-based rank in rowData) compared
    against baselines
  repo: https://bioconductor.org/packages/nnSVG
- name: SPARK-X
  role: Baseline SVG method whose fixed length-scale parameterization is expected
    to show lower top-n overlap with nnSVG
- name: Bioconductor
  role: Framework for the R implementation and SpatialExperiment inputs used to extract
    rankings
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:0e4f4615bde3c159aff83b38c358918d46dfce7b24b8a1c163ea7bed45c1628d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - SPARK-X
  - nnSVG
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# spearman-rank-correlation-analysis

## Summary

Quantify agreement between SVG gene rankings (e.g., nnSVG vs. HVGs, Moran's I, SPARK-X) using Spearman rank correlations and top-n set overlaps at n = 10, 20, 50, 100, 200 across datasets. This skill tests whether rankings from a new SVG method reproduce those of baseline methods rather than re-detecting SVGs.

## When to use

Use when you have ranked gene lists from two or more SVG/HVG identification methods on the same dataset and need to quantify rank agreement via Spearman correlations plus top-n (n = 10, 20, 50, 100, 200) overlap counts.

## When NOT to use

- The goal is to identify or rank SVGs themselves on a new dataset rather than to compare existing rankings — run nnSVG (or a baseline) directly instead
- Only raw counts and spatial coordinates are available with no per-method ranked gene lists — the correlations require ranked outputs (e.g., nnSVG rank/LR_stat columns) from each method first
- Comparing methods across different tissues or spot sets, where gene rankings are not defined over a shared gene universe and set overlaps are not meaningful

## Inputs

- Per-dataset ranked gene lists from nnSVG (rank column from rowData of a SpatialExperiment object)
- Per-dataset ranked gene lists from baseline methods: HVGs (e.g., scran), Moran's I, SPARK-X
- Datasets: Visium human DLPFC (STexampleData), ST mouse olfactory bulb, seqFISH mouse embryo

## Outputs

- Spearman rank correlation values for each dataset-method pair
- Top-n (n = 10, 20, 50, 100, 200) set overlap counts for each dataset-method pair
- Summary table nnsvg_baseline_agreement.csv stratified by dataset and method pair

## How to apply

Load the per-dataset ranked gene lists produced by nnSVG and each baseline (HVGs, Moran's I, SPARK-X) for the Visium DLPFC, ST mouse OB, and seqFISH datasets into R. For each dataset-method pair, compute the Spearman rank correlation between the nnSVG ranking and the baseline ranking using R's rank-based correlation. In parallel, compute the overlap of the top-ranked gene sets at n = 10, 20, 50, 100, and 200 between nnSVG and each baseline. Assemble correlations and overlap counts into one summary table stratified by dataset (Visium DLPFC, ST mouse OB, seqFISH) and method pair, and save it as nnsvg_baseline_agreement.csv. Rationale: nnSVG rankings derive from LR statistics and ranks stored in rowData of the SpatialExperiment object, and agreement analysis reveals whether methods with flexible gene-specific length scales (nnSVG) recover the same top genes as baselines, with SPARK-X expected to diverge because it fits a fixed combination of covariance functions and length scale parameters across all genes.

## Related tools

- **nnSVG** (Source of the reference SVG rankings (LR_stat-based rank in rowData) compared against baselines) — https://bioconductor.org/packages/nnSVG
- **SPARK-X** (Baseline SVG method whose fixed length-scale parameterization is expected to show lower top-n overlap with nnSVG)
- **Bioconductor** (Framework for the R implementation and SpatialExperiment inputs used to extract rankings) — https://bioconductor.org/packages/nnSVG

## Examples

```
R: cor(nnsvg_ranks, hvgs_ranks, method = "spearman")  # plus length(intersect(top_n_nnsvg, top_n_baseline)) at n = 10, 20, 50, 100, 200
```

## Evaluation signals

- The summary table contains all method-pair comparisons (nnSVG vs. HVGs, Moran's I, SPARK-X) for each of the three datasets (Visium DLPFC, ST mouse OB, seqFISH)
- nnSVG's top-n overlap with HVGs and Moran's I exceeds its overlap with SPARK-X, as reported for the seqFISH mouse embryo dataset at n = 10, 20, 50, 100, 200
- The largest nnSVG vs. Moran's I rank mismatches occur for genes at small length scales
- Overlaps are computed at exactly n = 10, 20, 50, 100, and 200, matching the workflow specification
- Output file is saved as nnsvg_baseline_agreement.csv with dataset and method-pair stratification

## Limitations

- SPARK-X fits a fixed combination of covariance functions and length scale parameters across all genes, reducing its flexibility to identify SVGs with different spatial ranges and thereby limiting agreement with nnSVG
- Transcriptome-wide datasets with >=10,000 spatial locations require runtimes of several hours with 10 processor cores on an HPC cluster, so generating the ranked lists that feed this analysis is costly; runtime depends on the number of genes and can be reduced with more stringent gene filtering
- Small negative values in estimated LR statistics occur mainly for lower-ranked genes, which can introduce noise at the ranking tail that correlations are sensitive to

## Evidence

- [readme] LR_stat ranks feed the correlation analysis: [evidence span withheld — anchor status: external_doc]
- [discussion] nnSVG output lists are the comparison objects: "nnSVG can be used to generate lists of top SVGs during exploratory unsupervised analyses of SRT datasets"
- [discussion] SPARK-X divergence rationale: "thereby leading to reduced flexibility to identify SVGs with different spatial ranges in expression"
- [readme] Ranked lists are produced by running nnSVG on a SpatialExperiment: "set.seed(123)
spe <- nnSVG(spe, n_threads = 1)"
