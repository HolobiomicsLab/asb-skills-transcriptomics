---
name: most-probable-path-tree-lineage-inference
description: Use when after MuTrans dynamical_analysis has identified attractor basins
  in a single-cell dataset and you need to infer the most probable transition trajectories
  between a starting state si and a final state sf for lineage reconstruction.
when_to_use_negative:
- When dynamical_analysis has not yet been run — InferLineage requires the attractor
  basins, coarse-grained transition probabilities and membership assignment as input
- When the goal is pseudo-time ordering alone (e.g., DPT, Slingshot, Monocle) rather
  than stochastic transition-path lineage on an attractor landscape
- When the dataset is too small for reliable multi-scale dynamics — e.g., the myelopoiesis
  dataset, where the paper notes 'special care is needed' to confirm transition-cell
  analyses
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuTrans (pyMuTrans)
  role: Provides pm.infer_lineage to infer cell lineage via MPPT or MPFT on the multi-scale
    dynamical analysis results
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Dependency package required for installing/running the Python MuTrans workflow
  repo: http://emma-project.org/latest/INSTALL.html
- name: Scanpy
  role: Used to load the processed gene expression matrix (e.g., Herman et al. example_mpp
    Cel-Seq2 dataset) into the AnnData object passed to infer_lineage
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:b96168ced6d70994b7d32840f29ad5c3a327b0c06d8bdb0d0d44b2056f4f0272
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - MuTrans (pyMuTrans)
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# most-probable-path-tree-lineage-inference

## Summary

Infer and overlay cell lineage on the MuTrans cell-fate dynamical manifold using the Most Probable Path Tree (MPPT) or Maximum Probability Flow Tree (MPFT) approach, tracing most-likely transition trajectories from a root attractor (e.g., lymphoid progenitor) to target attractors (e.g., Pro-B/Pre-B states).

## When to use

Use when after MuTrans dynamical_analysis has identified attractor basins in a single-cell dataset and you need to infer the most probable transition trajectories between a starting state si and a final state sf for lineage reconstruction.

## When NOT to use

- When dynamical_analysis has not yet been run — InferLineage requires the attractor basins, coarse-grained transition probabilities and membership assignment as input
- When the goal is pseudo-time ordering alone (e.g., DPT, Slingshot, Monocle) rather than stochastic transition-path lineage on an attractor landscape
- When the dataset is too small for reliable multi-scale dynamics — e.g., the myelopoiesis dataset, where the paper notes 'special care is needed' to confirm transition-cell analyses

## Inputs

- AnnData object with MuTrans dynamical_analysis results (attractor basins, coarse-grained transition probabilities, cell-cluster membership matrix)
- Starting attractor index si (e.g., lymphoid progenitor root state)
- Final attractor index sf (e.g., Pro-B or Pre-B B-cell state)
- Lineage method choice: 'MPPT' or 'MPFT'

## Outputs

- Most Probable Path Tree lineage overlay on the reconstructed cell-fate dynamical manifold
- Transition trajectory from state si to state sf visualized with attractor centers Yk and per-cell embedding xi(x)
- Figure artifacts (e.g., manifold/lineage figure saved from the example_mpp workflow)

## How to apply

Run lineage inference only after pm.dynamical_analysis has returned attractor basins, coarse-grained cluster-cluster transition probabilities, and the soft membership assignment, since InferLineage operates on those multi-scale analysis results. Choose the tree method: 'MPPT' (Most Probable Path Tree) or 'MPFT' (Maximum Probability Flow Tree), and specify the starting and final attractor indices si and sf, which for the Herman et al. Cel-Seq2 lymphoid progenitor example are the LP root and the B-cell attractors. Call pm.infer_lineage(adata, si, sf, method='MPPT', size_point=40, size_text=10, alpha_point=0.5) to plot the transition trajectory on the dynamical manifold constructed by ConstructLandscape. Rationale: MPPT/MPFT translate the coarse-grained stochastic dynamics (rwTPM-derived transition paths, consistent with transition path theory) into an interpretable lineage overlay connecting stable attractor basins through saddle regions where transition cells reside.

## Related tools

- **MuTrans (pyMuTrans)** (Provides pm.infer_lineage to infer cell lineage via MPPT or MPFT on the multi-scale dynamical analysis results) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Dependency package required for installing/running the Python MuTrans workflow) — http://emma-project.org/latest/INSTALL.html
- **Scanpy** (Used to load the processed gene expression matrix (e.g., Herman et al. example_mpp Cel-Seq2 dataset) into the AnnData object passed to infer_lineage) — https://scanpy.readthedocs.io/en/stable/installation.html

## Examples

```
import pyMuTrans as pm; pm.infer_lineage(adata, si=2, sf=0, method="MPPT", size_point=40, size_text=10, alpha_point=0.5)
```

## Evaluation signals

- infer_lineage runs only on AnnData objects containing dynamical_analysis outputs (attractor basins, coarse-grained transition probabilities, membership assignment); verify these fields exist before calling
- The lineage tree connects the specified si root attractor to sf target attractor consistent with prior annotations (e.g., LP root to Pro-B/Pre-B attractors in the Herman et al. example)
- The plotted most probable path lies along the saddle/transition region between basins on the dynamical manifold, not cutting through unrelated attractors
- For the lymphoid example, the lineage overlay is consistent with reported lymphoid lineage dynamics and the attractor assignment matches Supplementary annotations
- The method parameter is one of the two valid tree options, MPPT or MPFT, as defined in the paper

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- The number of cells in the datasets, in principle, needs to be sufficiently large in order to obtain high-resolution identification of transition cells; in small datasets such as myelopoiesis, special care is needed to confirm the analysis
- Root cell state detection could be enhanced via entropy methods or RNA velocity, which are not currently integrated

## Evidence

- [methods] we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach: "we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach"
- [readme] InferLineage operates on DynamicalAnalysis output: "%% Infer the cell lineage based on multi-scale analysis results

%% Output: the output object from DynamicalAnalysis function"
- [abstract] MuTrans quantifies likelihood of transition trajectories via transition path theory: "MuTrans quantifies the likelihood of all possible transition trajectories between cell states using coarse-grained transition path theory."
- [readme] Python invocation of infer_lineage with MPPT: "pm.infer_lineage(adata,si=2,sf=0,method = "MPPT",size_point =40, size_text = 10,alpha_point = 0.5)"
