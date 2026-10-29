---
name: diffusion-map-eiplgraph-trajectory-inference
description: Use when sample-level distances (e.g., PILOT's Wasserstein matrix over
  single-cell or pathomics samples) should be ordered into a disease trajectory rather
  than clustered, e.g., lupus PBMC, kidney IgAN, or myocardial infarction cohorts.
when_to_use_negative:
- When the cohort has no expected progression structure and the goal is grouping rather
  than ordering — use PILOT's Leiden clustering on the distance matrix W instead (resolution
  chosen via pl.pl.select_best_sil(adata, start = 0.2))
- When samples mix distinct technologies or isolation methods (single nucleus vs.
  single cell, droplet vs. well based), which the article flags as an open challenge
  for trajectory comparison
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PILOT
  role: provides pilotpy (pip install pilotpy) with diffusion-maps + EIPLGraph trajectory
    estimation on top of the Wasserstein distance matrix
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: reads and preprocesses the .h5ad single-cell data (e.g., sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad'))
    that PILOT's trajectory step operates on
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: its emd2 solver computes the Wasserstein distances underlying the distance
    structure used for trajectory inference
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:011df95fdea30a7804214e1e9cfa04714d5e240d0c233d7b4a95358281e9b11a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# diffusion-map-eiplgraph-trajectory-inference

## Summary

Trajectory estimation step of PILOT: diffusion maps embed the sample-level Wasserstein distance structure and EIPLGraph infers a disease-progression backbone across patient samples, enabling unsupervised ordering of samples along disease progression.

## When to use

Use when sample-level distances (e.g., PILOT's Wasserstein matrix over single-cell or pathomics samples) should be ordered into a disease trajectory rather than clustered, e.g., lupus PBMC, kidney IgAN, or myocardial infarction cohorts.

## When NOT to use

- When the cohort has no expected progression structure and the goal is grouping rather than ordering — use PILOT's Leiden clustering on the distance matrix W instead (resolution chosen via pl.pl.select_best_sil(adata, start = 0.2))
- When samples mix distinct technologies or isolation methods (single nucleus vs. single cell, droplet vs. well based), which the article flags as an open challenge for trajectory comparison

## Inputs

- Sample-level Wasserstein distance matrix produced by PILOT (pilotpy) from single-cell (e.g., .h5ad) or pathomics data
- A control sample identified as the root/source node for EIPLGraph
- Sample disease-status labels (for validation of the predicted trajectory ordering)

## Outputs

- Disease-progression trajectory (ordered samples along an EIPLGraph backbone) over patients
- Sample ordering/pseudotime that can feed downstream statistical tests of cell populations and gene expression changes
- Trajectory agreement metrics (label concordance with true disease status) for benchmarking configurations

## How to apply

After PILOT computes the sample-level Wasserstein distance matrix (via emd2 from the POT library, with a cosine-distance cost matrix between cluster medians in PCA space), apply diffusion maps to the distance matrix and fit the trajectory backbone with EIPLGraph. Choose one sample as the root/source node, selecting a control sample that reflects the start of the trajectory (note: in the paper this root was visually selected). In benchmarks (e.g., Lupus PBMC GSE174188), score the resulting sample ordering by concordance with true disease-status labels. For kidney IgAN data, this step is also used in the dedicated tutorial evaluating batch effects along the trajectory.

## Related tools

- **PILOT** (provides pilotpy (pip install pilotpy) with diffusion-maps + EIPLGraph trajectory estimation on top of the Wasserstein distance matrix) — https://github.com/CostaLab/PILOT
- **scanpy** (reads and preprocesses the .h5ad single-cell data (e.g., sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad')) that PILOT's trajectory step operates on) — https://github.com/CostaLab/PILOT
- **POT library** (its emd2 solver computes the Wasserstein distances underlying the distance structure used for trajectory inference)

## Evaluation signals

- Predicted trajectory ordering concordes with true disease-status labels (e.g., control samples at the trajectory root, disease samples downstream)
- In sensitivity benchmarks (integration method × cost-metric grid), the diffusion-map + EIPLGraph trajectory reproduces the reported behavior, i.e., cosine similarity for the cost matrix performs best per Fig. S5
- Root sample choice corresponds to a control sample reflecting the trajectory start; changing the root to a non-control sample should visibly distort the backbone
- On kidney trajectory data, batch effects along the trajectory can be assessed as in the dedicated PILOT Kidney_trajectory tutorial
- Trajectory outliers (e.g., pancreas Embryos E12-E15 samples) are identifiable and can be excluded as the paper did

## Limitations

- EIPLGraph requires a root sample, which was visually selected to reflect control samples — an objective rule for root selection is a reproducibility gap
- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence on trajectory predictions
- Analysis across distinct technologies or isolation methods is still an open challenge
- Four pancreas diabetes samples (Embryos E12-E15) were excluded as trajectory outliers, showing sensitivity to outlier samples

## Evidence

- [discussion] Trajectory estimation via diffusion maps and EIPLGraph: "PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph"
- [discussion] Root sample selection limitation: "EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples."
- [abstract] Unsupervised trajectory goal: "This allows us to perform unsupervised analysis at the sample level and uncover trajectories or cellular clusters associated with disease progression."
- [readme] Kidney trajectory batch-effect tutorial: "the forth one for 
[evaluation of the presence of batch effects in Trajectory](https://pilot.readthedocs.io/en/latest/Kidney_trajectory.html)"
