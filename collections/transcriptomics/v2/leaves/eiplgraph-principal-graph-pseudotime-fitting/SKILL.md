---
name: eiplgraph-principal-graph-pseudotime-fitting
description: Use when a PILOT sample-level distance matrix (Wasserstein distances
  between patient samples, e.g. myocardial_infarction.h5ad) must be converted into
  an ordered disease-progression pseudotime so that cell-population and gene changes
  can be regressed along disease progression.
when_to_use_negative:
- The analysis goal is clustering patients into sub-groups rather than ordering them
  along a trajectory (use PILOT's Leiden clustering on W instead)
- Samples come from mixed technologies or isolation methods (e.g. single nucleus vs.
  single cell, droplet vs. well based), which the authors state is still an open challenge
  and can confound the distance structure underlying the graph
- Input is raw cell-level data rather than an already computed sample-level Wasserstein
  distance matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PILOT
  role: Provides diffusion-map trajectory embedding and EIPLGraph principal graph
    fitting (pl.pl.trajectory, pl.pl.fit_pricipla_graph) over patient-level Wasserstein
    distances
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Underlies the emd2-based Wasserstein distances between samples that the trajectory
    graph is built from
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads the AnnData (h5ad) input containing sample/cell metadata used for the
    trajectory analysis
  repo: https://github.com/CostaLab/PILOT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:cfb76256209389c29b3f21021aeb08e07c38d01b3ef80d3ff2f226fc8629d09f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# eiplgraph-principal-graph-pseudotime-fitting

## Summary

Fits a disease-progression pseudotime by embedding PILOT's patient-by-patient Wasserstein distance matrix with diffusion maps and then inferring an EIPLGraph principal graph backbone rooted at a control sample, as done for the myocardial infarction scRNA-seq cohort.

## When to use

Use when a PILOT sample-level distance matrix (Wasserstein distances between patient samples, e.g. myocardial_infarction.h5ad) must be converted into an ordered disease-progression pseudotime so that cell-population and gene changes can be regressed along disease progression.

## When NOT to use

- The analysis goal is clustering patients into sub-groups rather than ordering them along a trajectory (use PILOT's Leiden clustering on W instead)
- Samples come from mixed technologies or isolation methods (e.g. single nucleus vs. single cell, droplet vs. well based), which the authors state is still an open challenge and can confound the distance structure underlying the graph
- Input is raw cell-level data rather than an already computed sample-level Wasserstein distance matrix

## Inputs

- AnnData object (e.g. myocardial_infarction.h5ad) with sample-level Wasserstein distance matrix computed via pl.tl.wasserstein_distance (emb_matrix='PCA', clusters_col='cell_subtype', sample_col='sampleID', status='Status')
- Sample-level embedding/distance matrix W from PILOT
- Root sample identifier for the EIPLGraph backbone (e.g. a control sample such as source_node=7)

## Outputs

- Diffusion-map embedding of patient samples along the inferred trajectory
- EIPLGraph principal graph backbone defining disease-progression pseudotime per sample
- Pseudotime-ordered sample metadata used downstream by pl.tl.cell_importance and pl.tl.genes_importance

## How to apply

First run pl.pl.trajectory to embed the sample distance matrix with diffusion maps (Coifman et al, 2005), which organizes patient samples along a nonlinear continuum. Then call pl.pl.fit_pricipla_graph with a source_node set to a control sample (in the MI tutorial, source_node=7, a control sample) to fit the EIPLGraph principal graph; the graph backbone defines the pseudotime ordering of samples. The root/sample choice is a decision point: PILOT requires a root sample, and in the paper it was visually selected to reflect the part of the trajectory containing control samples rather than chosen by an objective rule, so document and validate the choice. Downstream, the fitted pseudotime feeds pl.tl.cell_importance and pl.tl.genes_importance, which fit robust Huber regression models with linear, quadratic, and linear-quadratic terms per feature along the trajectory.

## Related tools

- **PILOT** (Provides diffusion-map trajectory embedding and EIPLGraph principal graph fitting (pl.pl.trajectory, pl.pl.fit_pricipla_graph) over patient-level Wasserstein distances) — https://github.com/CostaLab/PILOT
- **POT library** (Underlies the emd2-based Wasserstein distances between samples that the trajectory graph is built from) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads the AnnData (h5ad) input containing sample/cell metadata used for the trajectory analysis) — https://github.com/CostaLab/PILOT

## Examples

```
pl.pl.trajectory(adata); pl.pl.fit_pricipla_graph(adata, source_node=7)
```

## Evaluation signals

- Diffusion-map embedding of the sample distance matrix forms a coherent trajectory in which control samples occupy the region near the chosen root
- The fitted EIPLGraph with source_node set to a control sample (e.g. 7) assigns pseudotime that increases from control toward disease samples
- Cell populations regressed on the pseudotime reproduce expected biology: quadratic increase of SPP1+ macrophages and myofibroblasts, quadratic decrease of healthy cardiomyocytes
- Root-sample choice is documented, since it was visually selected in the paper (reproducibility gap) and can change the inferred ordering

## Limitations

- EIPLGraph requires a root sample, which in the paper was visually selected rather than chosen by an objective rule — a reproducibility gap
- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence on predictions
- Analysis across distinct technologies (single nucleus vs. single cell) or isolation methods (droplet vs. well based) remains an open challenge
- One myocardial infarction sample was excluded due to low quality (less than 1000 genes on average per cell) before trajectory analysis

## Evidence

- [discussion] Trajectory estimation via diffusion maps followed by EIPLGraph backbone inference: "PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph"
- [discussion] Root sample selection: "EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples."
- [readme] Trajectory purpose: "This allows us to perform unsupervised analysis at the sample level and uncover trajectories or cellular clusters associated with disease progression."
