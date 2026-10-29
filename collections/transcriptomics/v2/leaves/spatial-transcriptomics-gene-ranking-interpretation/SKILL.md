---
name: spatial-transcriptomics-gene-ranking-interpretation
description: Use when you have per-gene SVG rankings (e.g., LR_stat and rank columns
  from nnSVG's rowData output) and need to interpret them, select top-ranked or significant
  genes, or assess agreement between methods' rankings.
when_to_use_negative:
- The goal is identifying spatial domain marker genes (mean-level changes between
  domains) rather than genes with continuous spatial variation — the article explicitly
  focuses on SVGs, not domain marker genes
- The task is grouping SVGs into gene groups or metagenes — nnSVG identifies individual
  SVGs only, and metagene grouping is stated as future work
- You need effect sizes that separate technical from biological variance — the prop_sv
  definition does not distinguish these components
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: nnSVG
  role: Generates ranked SVG lists with LR_stat, rank, pval, padj, and prop_sv stored
    in rowData; its gene-specific length scale (phi) explains rank mismatches with
    Moran's I
  repo: https://bioconductor.org/packages/nnSVG
- name: SPARK-X
  role: Baseline method whose rankings are compared against nnSVG; expected to show
    lower top-n overlap due to its fixed covariance functions and length scales
- name: Bioconductor
  role: Framework hosting nnSVG and SpatialExperiment objects in which ranking results
    are stored and inspected
  repo: https://bioconductor.org/packages/nnSVG
- name: R
  role: Language used to compute Spearman rank correlations and top-n set overlaps
    between method rankings
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:b63423142e6c39e897f69baaf7ecc602c2159b0150d51e81659f0ad5fa702101
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - SPARK-X
  - nnSVG
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 29
---

# spatial-transcriptomics-gene-ranking-interpretation

## Summary

Interprets and validates ranked lists of spatially variable genes (SVGs) by examining nnSVG's likelihood-ratio (LR) statistics, gene-specific length scale parameters (phi), effect sizes (prop_sv), and by comparing rankings against baseline methods (HVGs, Moran's I, SPARK-X) via Spearman rank correlations and top-n set overlaps.

## When to use

Use when you have per-gene SVG rankings (e.g., LR_stat and rank columns from nnSVG's rowData output) and need to interpret them, select top-ranked or significant genes, or assess agreement between methods' rankings.

## When NOT to use

- The goal is identifying spatial domain marker genes (mean-level changes between domains) rather than genes with continuous spatial variation — the article explicitly focuses on SVGs, not domain marker genes
- The task is grouping SVGs into gene groups or metagenes — nnSVG identifies individual SVGs only, and metagene grouping is stated as future work
- You need effect sizes that separate technical from biological variance — the prop_sv definition does not distinguish these components

## Inputs

- SpatialExperiment Bioconductor object with rowData containing LR_stat, rank, pval, padj, prop_sv, and phi columns from nnSVG
- Per-dataset ranked gene lists from baseline methods (HVGs, Moran's I, SPARK-X) for Visium DLPFC, ST mouse OB, and seqFISH datasets
- Numeric matrix of log-transformed normalized counts (logcounts) and spatial coordinates as alternative input format

## Outputs

- Summary table of Spearman rank correlations and top-n (10, 20, 50, 100, 200) overlap counts per dataset and method pair (e.g., nnsvg_baseline_agreement.csv)
- Filtered set of top-ranked or significant SVGs (e.g., top 100 by rank, or padj <= 0.05) for downstream clustering or validation

## How to apply

Load per-dataset ranked gene lists from nnSVG and baseline methods (HVGs, Moran's I, SPARK-X) into R. Rank genes by nnSVG's LR_stat (stored in rowData of the SpatialExperiment object) and compare method pairs using Spearman rank correlation. Compute overlap of top-ranked gene sets at n = 10, 20, 50, 100, and 200 between nnSVG and each baseline, and assemble results into a summary table stratified by dataset (Visium DLPFC, ST mouse OB, seqFISH). Interpret effect size as prop_sv (proportion of spatial variance), and select genes either by an arbitrary top-n cutoff (e.g., top 100) or by adjusted p-values (padj <= 0.05) from the LR test. Expect nnSVG to agree more closely with HVGs and Moran's I than with SPARK-X, since SPARK-X fits fixed covariance combinations and length scales across all genes; the largest nnSVG vs Moran's I rank mismatches should occur for genes at small length scales.

## Related tools

- **nnSVG** (Generates ranked SVG lists with LR_stat, rank, pval, padj, and prop_sv stored in rowData; its gene-specific length scale (phi) explains rank mismatches with Moran's I) — https://bioconductor.org/packages/nnSVG
- **SPARK-X** (Baseline method whose rankings are compared against nnSVG; expected to show lower top-n overlap due to its fixed covariance functions and length scales)
- **Bioconductor** (Framework hosting nnSVG and SpatialExperiment objects in which ranking results are stored and inspected) — https://bioconductor.org/packages/nnSVG
- **R** (Language used to compute Spearman rank correlations and top-n set overlaps between method rankings)

## Examples

```
spe <- nnSVG(spe, n_threads = 1); rowData(spe)[order(rowData(spe)$rank)[1:10], ]
```

## Evaluation signals

- The summary table reports all method-pair comparisons per dataset (Visium DLPFC, ST mouse OB, seqFISH) with no missing pairs
- Top-n overlap between nnSVG and HVGs/Moran's I exceeds the overlap between nnSVG and SPARK-X, particularly for the seqFISH mouse embryo dataset
- Largest nnSVG vs Moran's I rank mismatches occur for genes with small length scale (phi) values, consistent with the fixed-length-scale limitation of non-GP baselines
- Genes ordered by rank should show monotonically decreasing LR_stat values, and some small negative LR statistics are expected mainly among lower-ranked genes
- Number of significant SVGs computed via table(rowData(spe)$padj <= 0.05) matches the selected gene set size when using the adjusted p-value cutoff

## Limitations

- Small negative values in estimated LR statistics occur mainly for lower-ranked genes and are difficult to interpret
- P-value calibration for low-expressed genes could be improved; adaptive filtering thresholds have not yet been implemented
- prop_sv effect size does not distinguish technical from biological variance
- For multiple-sample datasets, nnSVG uses practical rank averaging across samples rather than a principled statistical approach

## Evidence

- [discussion] rank by LR statistics with p-value cutoffs: "either select an arbitrary set of top-ranked SVGs (e.g., top 100 genes) or select a set of statistically significant SVGs with adjusted p-values from the LR test"
- [discussion] SPARK-X reduced flexibility: [evidence span withheld — anchor status: not_found]
- [readme] results stored in rowData: "In this case, the outputs are stored in the `rowData` of the `SpatialExperiment` object."
- [discussion] negative LR statistics: "we observe some small negative values in the estimated LR statistics, which are difficult to interpret"
- [readme] effect size definition: "`prop_sv`: effect size defined as proportion of spatial variance"
