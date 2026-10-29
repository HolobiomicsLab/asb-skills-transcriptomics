---
name: single-cell-rna-seq-emt-lineage-inference
description: Use when a single-cell RNA-seq dataset (e.g., Smart-Seq2 SCC EMT) needs
  attractor decomposition into multiple stable/hybrid states and quantification of
  transition flux through intermediate cell states between epithelial and mesenchymal
  attractors.
when_to_use_negative:
- Datasets with too few cells for high-resolution transition-cell identification —
  the article notes small datasets like the myelopoiesis Fluidigm C1 data require
  special care to confirm transition-cell analysis
- Analyses that require modeling non-stationary effects such as cell cycle or cell
  proliferation dynamics, which MuTrans does not currently consider
- Systems violating the steady-state/detailed-balance assumptions where coarse-grained
  transition probabilities may not follow Kramers' reaction rate theory
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MuTrans
  role: 'Core method: multiscale reduction of the rwTPM via pyMuTrans dynamical_analysis
    (K_cluster=5) to obtain attractor basins, coarse-grained transition probabilities,
    and soft membership assignment'
  repo: github.com/cliffzhou92/MuTrans-release
- name: DECLARE
  role: Optional pre-processing aggregation module (reduce_num_meta_cell=1500) to
    speed up computation on large datasets by aggregating cells into micro-states
  repo: github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Implements the Most Probable Path Tree (MPPT) lineage inference used to quantify
    E->ICS->M transition trajectory likelihoods
- name: Scanpy
  role: Loads the processed EMT gene expression matrix into an AnnData object and
    handles downstream plotting of the dynamical manifold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:87c11a964d16508c405543b6fa8c6de6811049d0ae4caca99943f71804b158c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DECLARE
  - MuTrans
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-rna-seq-emt-lineage-inference

## Summary

Applies MuTrans multiscale stochastic dynamics to reconstruct attractor basins and infer E→ICS→M cell lineage transition paths on the Smart-Seq2 squamous cell carcinoma EMT dataset, quantifying the fraction of E-to-M transition flux routed through intermediate cell states (ICS).

## When to use

Use when a single-cell RNA-seq dataset (e.g., Smart-Seq2 SCC EMT) needs attractor decomposition into multiple stable/hybrid states and quantification of transition flux through intermediate cell states between epithelial and mesenchymal attractors.

## When NOT to use

- Datasets with too few cells for high-resolution transition-cell identification — the article notes small datasets like the myelopoiesis Fluidigm C1 data require special care to confirm transition-cell analysis
- Analyses that require modeling non-stationary effects such as cell cycle or cell proliferation dynamics, which MuTrans does not currently consider
- Systems violating the steady-state/detailed-balance assumptions where coarse-grained transition probabilities may not follow Kramers' reaction rate theory

## Inputs

- Processed EMT single-cell RNA-seq gene expression matrix (N_cells x N_genes) from the Smart-Seq2 SCC dataset (Pastushenko et al.), stored in the MuTrans-release Data folder and loaded into a Scanpy AnnData object

## Outputs

- Attractor basin assignments (5 basins) and soft membership matrix per cell
- Coarse-grained cluster-cluster transition probabilities between E, M, and hybrid attractors
- Transition-trajectory likelihood table (with likelihood field) for E->ICS->M paths via MPPT
- ICS-mediated transition flux fraction (>0.5 expected) written to emt_mutrans_results.csv

## How to apply

Load the processed EMT gene expression matrix from the MuTrans-release Data folder into a Scanpy AnnData object. Run pyMuTrans dynamical_analysis with K_cluster=5 on the cell-cell scale random-walk transition probability matrix (rwTPM), optionally enabling the DECLARE aggregation module with reduce_num_meta_cell=1500 for speed; this returns attractor basins, coarse-grained transition probabilities, and the soft cell-cluster membership matrix. Construct the dynamical manifold (ConstructLandscape) so epithelial, mesenchymal, and hybrid attractor basins with transition cells along connecting paths are recovered. Infer the lineage using the Most Probable Path Tree (MPPT) method implemented with PyEMMA (pm.infer_lineage with method='MPPT'), producing transition-trajectory likelihoods for E→ICS→M routes. Finally compute the fraction of E-to-M transition flux passing through ICS attractors and write attractor assignments, path likelihoods, and the flux fraction to emt_mutrans_results.csv.

## Related tools

- **MuTrans** (Core method: multiscale reduction of the rwTPM via pyMuTrans dynamical_analysis (K_cluster=5) to obtain attractor basins, coarse-grained transition probabilities, and soft membership assignment) — github.com/cliffzhou92/MuTrans-release
- **DECLARE** (Optional pre-processing aggregation module (reduce_num_meta_cell=1500) to speed up computation on large datasets by aggregating cells into micro-states) — github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Implements the Most Probable Path Tree (MPPT) lineage inference used to quantify E->ICS->M transition trajectory likelihoods)
- **Scanpy** (Loads the processed EMT gene expression matrix into an AnnData object and handles downstream plotting of the dynamical manifold)

## Examples

```
par = {"K_cluster": 5.0, "reduce_large_scale": True, "reduce_num_meta_cell": 1500}; adata = pm.dynamical_analysis(adata, par); pm.infer_lineage(adata, si=2, sf=0, method="MPPT")
```

## Evaluation signals

- Recovered attractor count equals 5 (K_cluster=5) with E, M, and hybrid basins identifiable on the ConstructLandscape manifold
- ICS-mediated E-to-M transition flux fraction exceeds 0.5
- MPPT path-likelihood table contains the 'likelihood' field for transition trajectories
- Trajectory likelihoods reproduce the reported Fig. 3f-g values of the paper
- Transition cell score (TCS) and transition entropy correctly separate stable attractor cells from transition cells along paths

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- The number of cells needs to be sufficiently large for high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) need special care
- Detailed balance may be violated at the microscopic scale, relying on coarse-grained dynamics to recover transitions
- The correct number of attractors (K_cluster) must be chosen via the Eigen-Peak Index (EPI), Louvain/Leiden clustering, or biological prior

## Evidence

- [abstract] MuTrans quantifies the likelihood of all possible transition trajectories between cell states using coarse-grained transition path theory.: "MuTrans quantifies the likelihood of all possible transition trajectories between cell states using coarse-grained transition path theory."
- [intro] MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [methods] we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach: "we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach"
- [methods] mouse cancer EMT data (Smart-Seq2): "The single-cell datasets analyzed were from different systems and platforms, namely mouse cancer EMT data (Smart-Seq2)"
- [readme] par = {"K_cluster": 5.0} # number of attractors, can plot EPI to give the rough estimate of clusters: "par = {"K_cluster": 5.0} # number of attractors, can plot EPI to give the rough estimate of clusters or refer to other clustering results such as Louvain/Leiden results"
- [readme] pm.infer_lineage(adata,si=2,sf=0,method = "MPPT",size_point =40, size_text = 10,alpha_point = 0.5): "pm.infer_lineage(adata,si=2,sf=0,method = "MPPT",size_point =40, size_text = 10,alpha_point = 0.5) # plot the transition trajectory on dynamical manifold"
