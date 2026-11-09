---
name: aucell-module-activity-scoring
description: Use when you have regulon-like gene modules (RGMs.csv) detected on a
  cell-lineage-specific GRN and need per-cell module activity scores over the scRNA-seq
  expression matrix for clustering or benchmarking against SCENIC regulons.
when_to_use_negative:
- You do not yet have gene modules/regulons — AUCell scores existing gene sets; run
  CEFCON's RGM_activity() or pySCENIC regulon detection first
- The goal is GRN inference or driver regulator identification — those are upstream
  CEFCON stages (NetModel training, driver_regulators()), not activity scoring
- Data are not single-cell expression profiles — AUCell is designed for single-cell
  RNA-seq gene-set activity analysis
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: AUCell
  role: Computes per-cell activity scores of RGMs / regulons via its ranking-based
    scoring scheme
  repo: https://github.com/aertslab/AUCell
- name: CEFCON
  role: Detects the regulon-like gene modules (RGM_activity()) whose activity is scored;
    supplies AUCell activity in the RGM identification stage
  repo: https://github.com/WPZgithub/CEFCON
- name: pyscenic
  role: Baseline regulon detection; its regulons are scored with the same AUCell scheme
    for CEFCON-vs-SCENIC comparison
- name: scanpy
  role: Handles the scRNA-seq expression matrix (cells x genes) on which AUCell scores
    are computed
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:211692c5d1e3a9b162355be9078c29f16c9d5cd38b6b61fa5c227d5c56759a12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aertslab/AUCell
  tools:
  - AUCell
  - CEFCON
  - PyTorch
  - Python
  - cvxpy
  - gurobipy
  - networkx
  - pyscenic
  - scanpy
  - torch-geometric
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# aucell-module-activity-scoring

## Summary

Scores per-cell activity of regulon-like gene modules (RGMs) detected by CEFCON using the AUCell ranking-based scheme, producing an activity matrix (AUCell_mtx.csv) used downstream for enrichment-precision and NMI clustering evaluations.

## When to use

Use when you have regulon-like gene modules (RGMs.csv) detected on a cell-lineage-specific GRN and need per-cell module activity scores over the scRNA-seq expression matrix for clustering or benchmarking against SCENIC regulons.

## When NOT to use

- You do not yet have gene modules/regulons — AUCell scores existing gene sets; run CEFCON's RGM_activity() or pySCENIC regulon detection first
- The goal is GRN inference or driver regulator identification — those are upstream CEFCON stages (NetModel training, driver_regulators()), not activity scoring
- Data are not single-cell expression profiles — AUCell is designed for single-cell RNA-seq gene-set activity analysis

## Inputs

- RGMs.csv — detected regulon-like gene modules (passed DE filter, >= 10 genes per module)
- scRNA-seq expression matrix (rows = cells, columns = genes) for the lineage (e.g., hESC, mESC, mHSC lineages)
- cell_lineage_GRN.csv and driver_regulators.csv from the upstream CEFCON run (edge_threshold_avgDegree=8)

## Outputs

- AUCell_mtx.csv — per-cell AUCell activity matrix with an activity score for every RGM across all cells
- Hierarchical clustering of cells on the AUCell matrix with NMI against reference cell-type labels
- Per-module GO_BP/KEGG enrichment precision (fraction of modules significant at BH-adjusted p <= 0.05)

## How to apply

After CEFCON detects RGMs involving the identified driver regulators (via cefcon_results.RGM_activity(), retaining modules passing the differential-expression criterion with at least 10 genes), compute the per-cell AUCell activity matrix of each module using the aertslab AUCell scoring scheme, which ranks genes per cell and quantifies enrichment of module genes in the top of the ranking. The expression matrix has rows = cells and columns = genes; the result is written to AUCell_mtx.csv with a score for every RGM across all cells. The same AUCell scoring is applied to pySCENIC (>=0.12.0) regulons when benchmarking, so activity matrices are computed identically for both methods. Downstream, hierarchically cluster cells from the AUCell RGM-activity matrix and compute normalized mutual information (NMI) against reference cell-type labels, and/or perform GO_BP and KEGG enrichment on each module with BH-adjusted p <= 0.05 to compute enrichment precision.

## Related tools

- **AUCell** (Computes per-cell activity scores of RGMs / regulons via its ranking-based scoring scheme) — https://github.com/aertslab/AUCell
- **CEFCON** (Detects the regulon-like gene modules (RGM_activity()) whose activity is scored; supplies AUCell activity in the RGM identification stage) — https://github.com/WPZgithub/CEFCON
- **pyscenic** (Baseline regulon detection; its regulons are scored with the same AUCell scheme for CEFCON-vs-SCENIC comparison)
- **scanpy** (Handles the scRNA-seq expression matrix (cells x genes) on which AUCell scores are computed) — https://scanpy.readthedocs.io/en/stable/installation.html

## Examples

```
cefcon_results.RGM_activity()  # writes RGMs.csv and computes per-cell AUCell activity matrix AUCell_mtx.csv
```

## Evaluation signals

- AUCell_mtx.csv contains an activity score for every RGM across all cells (no missing rows/columns)
- RGMs.csv contains only modules that pass the differential-expression filter and the >= 10 gene size threshold
- In the CEFCON-vs-SCENIC benchmark, CEFCON attains higher GO_BP/KEGG enrichment precision and higher NMI than SCENIC across hESC, mESC, and mHSC lineage datasets, consistent with Fig. 5b-d
- Hierarchical clustering of cells on the AUCell RGM-activity matrix yields NMI values comparable to the published results when computed against reference cell-type labels
- Repeating steps 3-5 for pySCENIC regulons on the same five datasets uses an identically computed AUCell activity matrix, ensuring a fair comparison

## Limitations

- AUCell activity is only as informative as the upstream RGM detection; modules must pass the DE criterion and >= 10 gene threshold to be scored
- The comparison is tied to CEFCON's edge_threshold_avgDegree=8 GRNs; changing the threshold changes the modules and thus the activity scores
- For the pySCENIC baseline, results depend on pyscenic version (>= 0.12.0) and the same five datasets must be used for a valid comparison

## Evidence

- [methods] Compute the per-cell AUCell activity matrix of the obtained RGMs (AUCell_mtx.csv) with the aertslab AUCell scoring scheme.: [evidence span withheld — anchor status: not_found]
- [readme] Moreover, CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using AUCell: "Moreover, CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using [AUCell]"
- [results] Hierarchically cluster the cells from the AUCell RGM-activity matrix and compute the normalized mutual information (NMI) against the reference cell-type labels.: [evidence span withheld — anchor status: not_found]
- [methods] outputs are accepted when RGMs.csv contains only modules that pass the differential-expression filter and the >= 10 gene size threshold: [evidence span withheld — anchor status: not_found]
- [results] the compiled comparison table should show CEFCON attaining higher GO_BP/KEGG enrichment precision and higher NMI than SCENIC: [evidence span withheld — anchor status: not_found]
