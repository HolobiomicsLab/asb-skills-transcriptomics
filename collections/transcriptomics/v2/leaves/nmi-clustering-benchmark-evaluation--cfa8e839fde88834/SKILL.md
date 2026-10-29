---
name: nmi-clustering-benchmark-evaluation
description: 'Use when comparing whether gene modules or regulons from competing GRN
  methods better capture known cell states: you have an activity matrix per cell,
  reference cell-type labels, and need an NMI clustering benchmark.'
when_to_use_negative:
- Input modules lack an associated per-cell activity representation and no expression
  matrix is available to score module activity (AUCell cannot be computed)
- The dataset has no reference cell-type labels, making NMI against ground truth undefined
- The goal is evaluating GRN edge prediction accuracy rather than gene-module/cell-state
  capture (use the article's GRN-construction or driver-regulator benchmarks instead)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: Produces the regulon-like gene modules (RGMs) and their per-cell activities
    via cefcon_results.RGM_activity() that are clustered and NMI-scored in the benchmark
  repo: https://github.com/WPZgithub/CEFCON
- name: AUCell
  role: Scores per-cell activity of each RGM/regulon to build the activity matrix
    on which cells are hierarchically clustered
  repo: https://github.com/aertslab/AUCell
- name: pyscenic
  role: Baseline method whose regulons are run through the same AUCell + clustering
    + NMI evaluation for comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:2ad3678149adb46bfa35666eb4a07a7e9cd067dc48b0a163aca39a9fcdd7ce23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - AUCell
  - CEFCON
  - pyscenic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nmi-clustering-benchmark-evaluation

## Summary

Benchmark the biological informativeness of gene modules (CEFCON regulon-like gene modules vs. pySCENIC regulons) by scoring per-cell module activity with AUCell, hierarchically clustering cells on that activity matrix, and evaluating cluster quality via normalized mutual information (NMI) against reference cell-type labels.

## When to use

Use when comparing whether gene modules or regulons from competing GRN methods better capture known cell states: you have an activity matrix per cell, reference cell-type labels, and need an NMI clustering benchmark.

## When NOT to use

- Input modules lack an associated per-cell activity representation and no expression matrix is available to score module activity (AUCell cannot be computed)
- The dataset has no reference cell-type labels, making NMI against ground truth undefined
- The goal is evaluating GRN edge prediction accuracy rather than gene-module/cell-state capture (use the article's GRN-construction or driver-regulator benchmarks instead)

## Inputs

- Regulon-like gene modules per lineage (RGMs.csv) from CEFCON (edge_threshold_avgDegree=8 GRNs) and regulons from pySCENIC
- Single-cell expression matrix (scanpy adata) for the five lineages (hESC, mESC, mHSC lineages) used for AUCell scoring
- Reference cell-type labels for each cell in each dataset

## Outputs

- Per-cell AUCell RGM-activity matrix per lineage (AUCell_mtx.csv)
- Hierarchical cluster assignments of cells from the RGM-activity matrix
- Per-dataset NMI scores comparing CEFCON RGMs vs. SCENIC regulons (compiled benchmark table for Fig. 5b-d)

## How to apply

For each of the five lineages (hESC, mESC, and the three mHSC lineages), take the regulon-like gene modules produced by cefcon_results.RGM_activity() (RGMs.csv) and compute a per-cell AUCell activity matrix (AUCell_mtx.csv) using the aertslab AUCell scoring scheme. Hierarchically cluster the cells directly on the AUCell RGM-activity matrix rather than on raw expression, then compute NMI between the resulting cluster assignments and the reference cell-type labels. Repeat the identical AUCell, clustering, and NMI steps for modules inferred by the pySCENIC (>= 0.12.0) baseline run on the same datasets. Compile the per-dataset NMI side-by-side with the GO_BP/KEGG enrichment-precision metric to confirm CEFCON's superiority as reported in Fig. 5b-d. Rationale: NMI between RGM-activity-derived clusters and reference labels directly tests whether the modules are informative enough to separate known cell states.

## Related tools

- **CEFCON** (Produces the regulon-like gene modules (RGMs) and their per-cell activities via cefcon_results.RGM_activity() that are clustered and NMI-scored in the benchmark) — https://github.com/WPZgithub/CEFCON
- **AUCell** (Scores per-cell activity of each RGM/regulon to build the activity matrix on which cells are hierarchically clustered) — https://github.com/aertslab/AUCell
- **pyscenic** (Baseline method whose regulons are run through the same AUCell + clustering + NMI evaluation for comparison)

## Examples

```
from sklearn.metrics import normalized_mutual_info_score; from scipy.cluster.hierarchy import linkage, fcluster; labels = fcluster(linkage(AUCell_mtx, method='ward'), k); nmi = normalized_mutual_info_score(cell_types, labels)
```

## Evaluation signals

- The compiled comparison table should show CEFCON attaining higher NMI than SCENIC across hESC, mESC, and mHSC lineage datasets, consistent with Fig. 5b-d
- Each lineage produces a non-empty RGMs.csv and a corresponding AUCell_mtx.csv whose rows are cells and columns are modules
- Hierarchical clustering is performed on the AUCell RGM-activity matrix (not raw expression), and NMI is computed against the reference cell-type labels for the same cells
- CEFCON should also show higher GO_BP/KEGG enrichment precision (fraction of modules significant at BH-adjusted p <= 0.05) than SCENIC on all five datasets, corroborating the NMI result
- The same AUCell scoring scheme and clustering procedure are applied identically to CEFCON and pySCENIC modules so the NMI comparison is fair

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously, so the benchmark must be run per lineage
- A finding of high NMI does not identify the causal regulators; supervised frameworks like GENELink may yield superior results but ground-truth label datasets are often lacking
- Results depend on upstream GRN edge selection (edge_threshold_avgDegree=8) and on AUCell scoring choices

## Evidence

- [abstract] Benchmarking superiority of CEFCON gene modules: "Extensive benchmarking tests consistently demonstrated the superiority of CEFCON in GRN construction, driver regulator identification, and gene module identification over baseline methods"
- [readme] AUCell activity of RGMs: "detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using [AUCell]"
- [methods] RGM identification API step: "# Identify regulon-like gene modules
cefcon_results.RGM_activity()"
- [discussion] Lineage-by-lineage evaluation scope: "First, CEFCON can only handle one cell lineage at a time."
