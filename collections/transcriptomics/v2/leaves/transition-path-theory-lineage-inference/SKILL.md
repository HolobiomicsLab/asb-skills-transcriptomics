---
name: transition-path-theory-lineage-inference
description: Use when single-cell transcriptome data has been reduced to attractor
  basins via MuTrans multi-scale rwTPM analysis and you need to reconstruct transition
  trajectories/lineage between a defined starting and target attractor state, e.g.
  bifurcating lineages.
when_to_use_negative:
- When attractor basins and the coarse-grained rwTPM have not yet been computed —
  lineage inference requires the multi-scale analysis output
- When the number of cells is too small for high-resolution identification of transition
  cells (e.g. the myelopoiesis dataset), where transition-cell calls need special
  confirmation
- When cell-cycle or proliferation-driven non-stationary dynamics dominate, which
  the current method does not model
edam_operation: http://edamontology.org/operation_3930
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MuTrans
  role: Framework performing coarse-grained transition path theory and lineage inference
    on the dynamical manifold
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: pyMuTrans
  role: Python API exposing pm.infer_lineage with MPPT/MPFT methods on Anndata objects
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Dependency package required for the Python MuTrans implementation
- name: Scanpy
  role: Dependency for handling AnnData objects passed through pyMuTrans
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:4b43990f8dde51b49092e34cf949f0a83f5ed37df81a86c506415ebfb919c5e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - PyEMMA
  - Scanpy
  - pyMuTrans
  edam_operation: http://edamontology.org/operation_3930
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# transition-path-theory-lineage-inference

## Summary

Infer cell-fate lineages by applying coarse-grained transition path theory on the MuTrans dynamical manifold, using Most Probable Path Tree (MPPT) or Maximum Probability Flow Tree (MPFT) to recover bifurcating trajectories between attractor basins.

## When to use

Use when single-cell transcriptome data has been reduced to attractor basins via MuTrans multi-scale rwTPM analysis and you need to reconstruct transition trajectories/lineage between a defined starting and target attractor state, e.g. bifurcating lineages.

## When NOT to use

- When attractor basins and the coarse-grained rwTPM have not yet been computed — lineage inference requires the multi-scale analysis output
- When the number of cells is too small for high-resolution identification of transition cells (e.g. the myelopoiesis dataset), where transition-cell calls need special confirmation
- When cell-cycle or proliferation-driven non-stationary dynamics dominate, which the current method does not model

## Inputs

- AnnData object (or Matlab Output object) post dynamical analysis: attractor basins, coarse-grained transition probabilities, cell-cluster membership matrix
- Pre-processed single-cell gene expression matrix (N_cells x N_genes), e.g. processed iPSC induction RT-qPCR matrix from MuTrans-release/Data
- Indices of starting (si) and targeting (sf) attractor states

## Outputs

- Inferred lineage tree (MPPT/MPFT) between the chosen attractor states
- Most probable transition paths quantified via coarse-grained transition path theory
- Visualization of the transition trajectory plotted on the dynamical manifold

## How to apply

After running dynamical analysis (DynamicalAnalysis or pm.dynamical_analysis) that yields attractor basins, coarse-grained transition probabilities, and the optimized membership assignment, run lineage inference with starting (si) and targeting (sf) attractor indices and method='MPPT' (Most Probable Path Tree) or 'MPFT' (Maximum Probability Flow Tree). Transition path theory quantifies the likelihood of all possible transition trajectories between cell states, so the inferred tree ranks most-probable paths across the cell-cell rwTPM rather than relying on static pseudo-time ordering. Visualize the recovered transition trajectory on the dynamical manifold (e.g. pm.infer_lineage with plotting parameters like size_point, size_text, alpha_point). For the iPSC induction case, infer_lineage recovers the bifurcating lineage from the Pre-M state into the Pre-En branch. Choose MPPT vs MPFT based on whether most-probable paths (path likelihood) or maximum probability flow better matches the expected branching topology.

## Related tools

- **MuTrans** (Framework performing coarse-grained transition path theory and lineage inference on the dynamical manifold) — https://github.com/cliffzhou92/MuTrans-release
- **pyMuTrans** (Python API exposing pm.infer_lineage with MPPT/MPFT methods on Anndata objects) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Dependency package required for the Python MuTrans implementation)
- **Scanpy** (Dependency for handling AnnData objects passed through pyMuTrans)

## Examples

```
import pyMuTrans as pm; adata = pm.dynamical_analysis(adata, par); pm.infer_lineage(adata, si=2, sf=0, method='MPPT', size_point=40, alpha_point=0.5)
```

## Evaluation signals

- Inferred lineage connects the specified starting (si) and targeting (sf) attractor basins on the dynamical manifold
- Recovered lineage shows expected bifurcation topology, e.g. Pre-M to Pre-En branch in the iPSC induction dataset
- Lineage tree path likelihoods derive from the coarse-grained transition probabilities consistent with Kramers' reaction rate theory under steady-state/detailed-balance conditions
- Inferred lineage tree is reproducible against the published example notebooks (e.g. example-ipsc.ipynb) in MuTrans-release/Example

## Limitations

- Root cell state detection is not integrated with entropy methods or RNA velocity, which the authors note could enhance robustness
- Detailed balance may be violated at the microscopic scale; only the coarse-grained dynamics may recover transitions
- Requires sufficiently large cell numbers for high-resolution identification of transition cells; small datasets like myelopoiesis need special care

## Evidence

- [intro] TPT lineage inference method: "we use the transition path theory to quantify the likelihood of all possible transition trajectories between cell states"
- [methods] MPPT/MPFT lineage tree choices: "we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach"
- [readme] Python API call for lineage plotting: "pm.infer_lineage(adata,si=2,sf=0,method = "MPPT",size_point =40, size_text = 10,alpha_point = 0.5) # plot the transition trajectory on dynamical manifold"
- [readme] Prerequisites from multi-scale analysis: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [discussion] Kramers consistency of transition probabilities: "The transition probability is shown to be consistent with Kramers’ reaction rate theory for over-damped Langevin dynamics"
