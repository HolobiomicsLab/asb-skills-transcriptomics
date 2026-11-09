---
name: pseudotime-sample-ranking
description: Use when you have per-sample cell-type/structure cluster distributions
  (e.g. pathomics .h5ad with clusters_col='Cell_type', sample_col='sampleID', status='status')
  and need an unsupervised ordering of patients along disease progression.
when_to_use_negative:
- Data are not represented as per-sample cluster distributions (e.g. raw per-cell
  expression without clustering), since PILOT's Wasserstein distances are computed
  over cluster distributions
- You need per-cell pseudotime inference rather than patient/sample-level ranking
  (a cell-level tool like Monocle fits that goal instead)
- Samples come from distinct technologies (single nucleus vs. single cell) or isolation
  methods (droplet vs. well based), which the authors state is an open challenge for
  this analysis
tools:
- name: PILOT (pilotpy)
  role: Computes sample-level Wasserstein/EMD distances, diffusion-map trajectory,
    EIPLGraph principal graph, and sample ranking
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads the pathomics AnnData .h5ad objects (sc.read_h5ad) before distance computation
- name: POT library
  role: Backend optimal-transport solver (emd2 function) underlying PILOT's Wasserstein
    distance estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:359c1e106dd4fc745a98d925794cb88f44d018a6a1f5a28764a0050e0613b4bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - PILOT (pilotpy)
  - POT library
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pseudotime-sample-ranking

## Summary

Ranks patient samples along a disease-progression pseudotime by combining sample-level Wasserstein (EMD) distances from PILOT, embedding them with diffusion maps, and fitting an EIPLGraph principal graph rooted at a control sample. The resulting ordered samples serve as a disease progression score (e.g. for kidney IgAN pathomics and kidney-failure association testing).

## When to use

Use when you have per-sample cell-type/structure cluster distributions (e.g. pathomics .h5ad with clusters_col='Cell_type', sample_col='sampleID', status='status') and need an unsupervised ordering of patients along disease progression.

## When NOT to use

- Data are not represented as per-sample cluster distributions (e.g. raw per-cell expression without clustering), since PILOT's Wasserstein distances are computed over cluster distributions
- You need per-cell pseudotime inference rather than patient/sample-level ranking (a cell-level tool like Monocle fits that goal instead)
- Samples come from distinct technologies (single nucleus vs. single cell) or isolation methods (droplet vs. well based), which the authors state is an open challenge for this analysis

## Inputs

- Pathomics AnnData objects (.h5ad) with sample-level cell-type/structure cluster distributions and status labels (e.g. Kidney_IgAN_T.h5ad, Kidney_IgAN_G.h5ad)
- Cluster annotation column (clusters_col='Cell_type'), sample ID column (sample_col='sampleID'), and disease status column (status='status')

## Outputs

- Sample x sample EMD distance matrices (adata.uns['EMD']) per compartment and combined
- Diffusion-map trajectory embedding of samples
- Ranked samples along the EIPLGraph backbone = disease progression score
- Ranked cell types along the trajectory (pl.tl.cell_importance)
- Pseudotime associations of morphometric features (e.g. glom_tuft_sizes, tubule_diameters) via linear/quadratic robust fits

## How to apply

For each compartment (glomeruli Kidney_IgAN_G.h5ad, tubuli Kidney_IgAN_T.h5ad), compute sample-level Wasserstein distances with pilotpy's pl.tl.wasserstein_distance (clusters_col='Cell_type', sample_col='sampleID', status='status', data_type='Pathomics') and inspect cost matrices/EMD heatmaps with pl.pl.heatmaps. Combine compartments by summing their EMD matrices (adata_Com.uns['EMD'] = adata_G.uns['EMD'] + adata_T.uns['EMD']), then embed with pl.pl.trajectory using a diffusion map (colors ['red','blue','orange']). Fit the EIPLGraph principal graph with pl.pl.fit_pricipla_graph, choosing source_node as a visually/known normal-eGFR control sample (here source_node=2); the graph ranks samples as the progression score. Validate that normal samples fall at the trajectory start and case samples toward the end, then optionally link morphometric features (glom_sizes, tubule_diameters, etc.) to pseudotime and test the score against clinical outcomes (e.g. eGFR groups <30/30-60/>60) via likelihood-ratio test.

## Related tools

- **PILOT (pilotpy)** (Computes sample-level Wasserstein/EMD distances, diffusion-map trajectory, EIPLGraph principal graph, and sample ranking) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads the pathomics AnnData .h5ad objects (sc.read_h5ad) before distance computation)
- **POT library** (Backend optimal-transport solver (emd2 function) underlying PILOT's Wasserstein distance estimation)

## Examples

```
adata_Com.uns['EMD'] = adata_G.uns['EMD'] + adata_T.uns['EMD']; pl.pl.trajectory(adata_Com, colors=['red','blue','orange']); pl.pl.fit_pricipla_graph(adata_Com, source_node=2)
```

## Evaluation signals

- The trajectory should order normal-eGFR samples at the start and case samples toward the end of the pseudotime
- The EIPLGraph root (source_node) must correspond to a normal-eGFR control sample (source_node=2 in the IgAN case)
- The resulting progression score should outperform the multivariate morphometric model for kidney failure: LRT p=2.4e-11 vs. p=7e-05
- Significant morphological features should be recovered: ~14 significant glomerular features and 3 tubular features with p<0.05 in the robust linear/quadratic fits
- EMD cost matrices and heatmaps (pl.pl.heatmaps) should show interpretable cost structure across cell types before trajectory fitting

## Limitations

- The EIPLGraph root sample was visually selected to reflect trajectory parts containing control samples rather than by an objective rule (reproducibility gap)
- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence
- The analysis assumes samples were collected uniformly (similar tissue areas, same technology); mixing technologies or isolation methods is an open challenge

## Evidence

- [discussion] PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph: "PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph"
- [discussion] EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples.: "EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples."
- [readme] PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples. This allows us to perform unsupervised analysis at the sample level: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples. This allows us to perform unsupervised analysis at the sample level"
- [methods] You can find the pathomics data [here](https://github.com/CostaLab/PILOT/tree/main/Tutorial/Datasets).: "You can find the pathomics data [here](https://github.com/CostaLab/PILOT/tree/main/Tutorial/Datasets)."
