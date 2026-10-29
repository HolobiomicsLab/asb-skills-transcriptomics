---
name: transition-path-likelihood-comparison
description: Use when a large single-cell dataset makes full MuTrans dynamical_analysis
  slow, and you must verify that DECLARE-aggregated micro-states reproduce coarse-grained
  transition probabilities and transition path likelihoods computed on all cells,
  plus measure the speedup.
when_to_use_negative:
- Dataset is small (e.g., the Fluidigm C1 myelopoiesis data), where the article notes
  special care is needed and full-data analysis without aggregation is feasible
- Goal is de novo clustering or pseudotime ordering (SC3/SIMLR, DPT/Slingshot/Monocle)
  rather than validating dynamics-preserving aggregation of a MuTrans stochastic dynamical
  analysis
- The analysis does not involve transition probabilities or transition path likelihoods,
  so there is nothing to compare against a DECLARE-accelerated run
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuTrans
  role: Runs the multi-scale stochastic dynamical analysis (pyMuTrans dynamical_analysis)
    producing attractor basins, coarse-grained transition probabilities, and transition
    path likelihoods on both complete and aggregated data
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: DECLARE
  role: MuTrans pre-processing module (dynamics-preserving cells aggregation) that
    aggregates cells into micro-states to speed up computation while preserving the
    dynamical landscape
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Dependency used for transition path theory computation of transition trajectory
    likelihoods
- name: Scanpy
  role: Loads the processed gene expression matrix into the AnnData object passed
    to pyMuTrans
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:ae4bc5d27282533c28ef88501a96638ea9131315b7d906fb2da1141a4985d20e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - MuTrans
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# transition-path-likelihood-comparison

## Summary

Compares the cell-fate transition path likelihoods and dynamical manifold structure produced by full MuTrans runs versus runs accelerated with the DECLARE dynamics-preserving aggregation module, and quantifies the wall-clock speedup on large-scale datasets such as the human bone-marrow hematopoiesis data (10X Chromium).

## When to use

Use when a large single-cell dataset makes full MuTrans dynamical_analysis slow, and you must verify that DECLARE-aggregated micro-states reproduce coarse-grained transition probabilities and transition path likelihoods computed on all cells, plus measure the speedup.

## When NOT to use

- Dataset is small (e.g., the Fluidigm C1 myelopoiesis data), where the article notes special care is needed and full-data analysis without aggregation is feasible
- Goal is de novo clustering or pseudotime ordering (SC3/SIMLR, DPT/Slingshot/Monocle) rather than validating dynamics-preserving aggregation of a MuTrans stochastic dynamical analysis
- The analysis does not involve transition probabilities or transition path likelihoods, so there is nothing to compare against a DECLARE-accelerated run

## Inputs

- Processed single-cell gene expression matrix (N_cells x N_genes) as an AnnData object, e.g. human bone-marrow hematopoietic progenitors (10X Chromium) from MuTrans-release/Data
- MuTrans parameter dictionary (par) including K_cluster, reduce_large_scale, reduce_num_meta_cell

## Outputs

- Transition path likelihoods (coarse-grained transition path theory, PyEMMA) for complete-data, DECLARE, and expression-averaging runs
- Attractor basins, coarse-grained transition probabilities, and cell-cluster membership assignments per run
- declare_speedup_comparison.csv containing transition-likelihood comparisons and runtime/speedup table

## How to apply

Load the processed gene expression matrix (e.g., human hematopoietic progenitors, 10X Chromium, from MuTrans-release/Data) into an AnnData object with Scanpy. First run the full pyMuTrans dynamical_analysis (K_cluster chosen via Eigen-Peak Index/EPI or biological prior) on all cells to obtain attractor basins, coarse-grained transition probabilities, and transition path likelihoods via PyEMMA-based coarse-grained transition path theory. Then re-run with the DECLARE module enabled by setting par['reduce_large_scale']=True and par['reduce_num_meta_cell']=1500 so cells are aggregated into hundreds/thousands of micro-states via K-means or kNN partition before multi-scale reduction. As a control, aggregate cells by simple expression averaging rather than dynamics-preserving aggregation, and compare the dynamical landscape structure and transition path likelihoods (as in Fig. 6d-e) between complete-data, DECLARE, and averaging runs. Record wall-clock runtimes of the complete-data and DECLARE runs and compute the speedup factor, writing the comparison and runtime table to declare_speedup_comparison.csv. Rationale: DECLARE should approximate no-DECLARE transition probabilities well and preserve the landscape structure better than expression averaging, because it aggregates while preserving the random-walk (rwTPM) dynamics.

## Related tools

- **MuTrans** (Runs the multi-scale stochastic dynamical analysis (pyMuTrans dynamical_analysis) producing attractor basins, coarse-grained transition probabilities, and transition path likelihoods on both complete and aggregated data) — https://github.com/cliffzhou92/MuTrans-release
- **DECLARE** (MuTrans pre-processing module (dynamics-preserving cells aggregation) that aggregates cells into micro-states to speed up computation while preserving the dynamical landscape) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Dependency used for transition path theory computation of transition trajectory likelihoods)
- **Scanpy** (Loads the processed gene expression matrix into the AnnData object passed to pyMuTrans)

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 5.0, "reduce_large_scale": True, "reduce_num_meta_cell": 1500}; adata = pm.dynamical_analysis(adata,par)
```

## Evaluation signals

- Transition path likelihoods from the DECLARE run match those of the complete-data MuTrans run (consistency reported in Fig. 6d-e)
- DECLARE preserves the dynamical manifold/landscape structure better than simple expression-averaging aggregation of cell groups
- Wall-clock speedup of the DECLARE run versus the complete-data run is at least 5x (paper reports roughly one order of magnitude)
- Coarse-grained transition probabilities computed among DECLARE micro-states approximate those computed without DECLARE
- declare_speedup_comparison.csv contains both the transition-likelihood comparison and the runtime table

## Limitations

- When the number of cells is relatively small (e.g., the myelopoiesis dataset), special care is needed and aggregation may be inappropriate
- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the method
- A sufficiently large number of cells is required for high-resolution identification of transition cells

## Evidence

- [discussion] DECLARE aggregation speeds up MuTrans: "MuTrans introduces a pre-processing method (DECLARE) to aggregate the cells and speed up computation."
- [intro] TPT quantifies transition likelihoods: "we use the transition path theory to quantify the likelihood of all possible transition trajectories between cell states"
- [readme] DECLARE usage in pyMuTrans: "par["reduce_large_scale"] = True # optional, to use DECLARE module speeding-up the calculation"
- [methods] Lineage inference via path trees: "we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach"
- [intro] Coarse-graining of rwTPM: "the method performs coarse-graining on the cell–cell scale rwTPM to learn the dynamics on the cluster-cluster scale"
