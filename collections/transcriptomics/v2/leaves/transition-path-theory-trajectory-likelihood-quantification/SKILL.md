---
name: transition-path-theory-trajectory-likelihood-quantification
description: Use when single-cell transcriptome data has been multi-scale analyzed
  (attractor basins and coarse-grained transition probabilities from MuTrans dynamical_analysis)
  and you must infer lineage/transition path likelihoods between states, e.g. routing
  E->M flux through intermediate cell states.
when_to_use_negative:
- Dataset is small (e.g. the Olsson myelopoiesis dataset), where transition-cell analysis
  needs special care and confirmation
- The goal is static clustering only (SC3/SIMLR-style distinct states) rather than
  transition paths and trajectory likelihoods
- Non-stationary effects from cell cycle or proliferation dynamics dominate the system,
  which the current method does not model
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MuTrans (pyMuTrans)
  role: Provides the multiscale dynamical analysis (rwTPM coarse-graining, attractor
    basins) and the infer_lineage function implementing MPPT/MPFT transition path
    quantification
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Backend functions used to implement the Most Probable Path Tree (MPPT) transition
    path computations
  repo: http://emma-project.org/latest/INSTALL.html
- name: DECLARE
  role: Optional pre-processing cell-aggregation module (reduce_num_meta_cell=1500)
    that speeds up the dynamical analysis feeding the lineage inference
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Scanpy
  role: Used to load the processed EMT expression matrix into an AnnData object for
    pyMuTrans
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:4fd0f7f229933bdfa06ddd36759003c6c5d2d37aa85b2de07b919ab562a71b58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - MuTrans
  - MuTrans (pyMuTrans)
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# transition-path-theory-trajectory-likelihood-quantification

## Summary

Quantify the likelihood of all possible transition trajectories between cell states by applying coarse-grained transition path theory (Most Probable Path Tree, MPPT, or Maximum Probability Flow Tree, MPFT) on the MuTrans multiscale stochastic dynamics output, as done for the E->ICS->M routes in the SCC Smart-Seq2 EMT dataset.

## When to use

Use when single-cell transcriptome data has been multi-scale analyzed (attractor basins and coarse-grained transition probabilities from MuTrans dynamical_analysis) and you must infer lineage/transition path likelihoods between states, e.g. routing E->M flux through intermediate cell states.

## When NOT to use

- Dataset is small (e.g. the Olsson myelopoiesis dataset), where transition-cell analysis needs special care and confirmation
- The goal is static clustering only (SC3/SIMLR-style distinct states) rather than transition paths and trajectory likelihoods
- Non-stationary effects from cell cycle or proliferation dynamics dominate the system, which the current method does not model

## Inputs

- AnnData object with processed single-cell gene expression matrix (e.g. EMT Smart-Seq2 matrix from MuTrans-release/Data)
- MuTrans dynamical_analysis output (attractor basins, coarse-grained transition probabilities, cell-cluster membership matrix)
- Parameter dictionary (par) with K_cluster and lineage options

## Outputs

- Cell lineage / transition-trajectory likelihood table (E->ICS->M routes with likelihood fields)
- Most Probable Path Tree visualization on the dynamical manifold
- ICS-mediated transition flux fraction (e.g. written to emt_mutrans_results.csv)

## How to apply

First run pyMuTrans dynamical_analysis (e.g. K_cluster=5, optionally with DECLARE via reduce_large_scale=True and reduce_num_meta_cell=1500) so the rwTPM is coarse-grained into attractor basins with soft membership assignments. Then call pm.infer_lineage with the source state index (si) and final state index (sf) and method='MPPT' (or 'MPFT'), which uses PyEMMA functions to build the Most Probable Path Tree and produce transition-trajectory likelihoods. Construct the dynamical manifold with ConstructLandscape to visualize attractor basins and transition cells along connecting paths. Finally aggregate path likelihoods, e.g. compute the fraction of E->M flux routed through ICS attractors, and validate that the fraction exceeds 0.5 for the SCC EMT data as reported in Fig. 3f-g.

## Related tools

- **MuTrans (pyMuTrans)** (Provides the multiscale dynamical analysis (rwTPM coarse-graining, attractor basins) and the infer_lineage function implementing MPPT/MPFT transition path quantification) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Backend functions used to implement the Most Probable Path Tree (MPPT) transition path computations) — http://emma-project.org/latest/INSTALL.html
- **DECLARE** (Optional pre-processing cell-aggregation module (reduce_num_meta_cell=1500) that speeds up the dynamical analysis feeding the lineage inference) — https://github.com/cliffzhou92/MuTrans-release
- **Scanpy** (Used to load the processed EMT expression matrix into an AnnData object for pyMuTrans) — https://scanpy.readthedocs.io/en/stable/installation.html

## Examples

```
import pyMuTrans as pm; adata = pm.dynamical_analysis(adata, {"K_cluster": 5.0}); pm.infer_lineage(adata, si=2, sf=0, method="MPPT", size_point=40, size_text=10, alpha_point=0.5)
```

## Evaluation signals

- Recovered attractor count equals K_cluster (e.g. 5 attractors on the SCC EMT dataset)
- Fraction of E-to-M transition flux routed through ICS attractors exceeds 0.5
- The transition-trajectory likelihood table contains the 'likelihood' field for E->ICS->M routes
- Trajectory likelihoods reproduce the reported Fig. 3f-g values for the SCC Smart-Seq2 EMT data
- Coarse-grained transition probabilities and membership assignment returned by dynamical_analysis are consistent before lineage inference

## Limitations

- A sufficiently large number of cells is required for high-resolution identification of transition cells
- Non-stationary effects due to cell cycle or proliferation dynamics were not considered in the current method
- Detailed balance may be violated at the microscopic scale (Eq. 1), relying on coarse-grained dynamics to recover transitions; results depend on the steady-state assumption

## Evidence

- [abstract] MuTrans quantifies the likelihood of all possible transition trajectories between cell states using coarse-grained transition path theory.: "MuTrans quantifies the likelihood of all possible transition trajectories between cell states using coarse-grained transition path theory."
- [intro] we use the transition path theory to quantify the likelihood of all possible transition trajectories between cell states: "we use the transition path theory to quantify the likelihood of all possible transition trajectories between cell states"
- [methods] we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach: "we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach"
- [methods] InferLineage (cell lineage inference via MPPT or MPFT): "%% Infer the cell lineage based on multi-scale analysis results"
- [readme] README Python usage of pm.infer_lineage with MPPT method: "pm.infer_lineage(adata,si=2,sf=0,method = "MPPT",size_point =40, size_text = 10,alpha_point = 0.5) # plot the transition trajectory on dynamical manifold"
