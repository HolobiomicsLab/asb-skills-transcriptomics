---
name: mait-nkt-cell-population-visualization
description: Use when visualizing consensus clustering results on a CITE-seq embedding
  to highlight a specific minor cell population (e.g. MAIT-NKT cells) and reproduce
  the paper's Figure 3 / Supplementary Fig. S3.
when_to_use_negative:
- The input is single-modality scRNA-seq with no second data modality, since SnapCCESS
  integrates multiple modalities of the same cells (e.g. RNA + ADT from CITE-seq)
- You need ready-made best hyperparameters for a new dataset — the tutorial only explains
  package usage and states it 'doesn't recommend the best parameters for your datasets';
  those are in the paper
- The goal is supervised cell-type classification rather than unsupervised consensus
  clustering and embedding visualization
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SnapCCESS
  role: Unsupervised ensemble deep learning framework that creates snapshots of multimodality
    variational-autoencoder embeddings used to generate the consensus clustering visualized
    with the MAIT-NKT population highlighted
  repo: https://github.com/PYangLab/SnapCCESS
- name: snapccess
  role: Python package (pip-installable) that generates the multimodal embeddings
    underlying the clustering; parameters for the published datasets are listed in
    the paper
  repo: https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- name: Python
  role: Implementation language for the snapccess package and the example scripts
    for generating embeddings
- name: R
  role: Optional R wrapper installation via remotes::install_github; the R wrapper
    calls the Python package and expects modalities arranged as features by cells
  repo: https://github.com/yulijia/SnapCCESS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:69f38949f46d77449de9879d62ad57a48908011f409996c01fc38cb889ec9abb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - R
  - SnapCCESS
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mait-nkt-cell-population-visualization

## Summary

Reproduce the Ramaswamy CITE-seq consensus-clustering embedding visualization from SnapCCESS (Figure 3 and Supplementary Fig. S3) by highlighting the MAIT-NKT cell population identified via ensemble clustering of multimodal RNA/ADT embeddings.

## When to use

Use when visualizing consensus clustering results on a CITE-seq embedding to highlight a specific minor cell population (e.g. MAIT-NKT cells) and reproduce the paper's Figure 3 / Supplementary Fig. S3.

## When NOT to use

- The input is single-modality scRNA-seq with no second data modality, since SnapCCESS integrates multiple modalities of the same cells (e.g. RNA + ADT from CITE-seq)
- You need ready-made best hyperparameters for a new dataset — the tutorial only explains package usage and states it 'doesn't recommend the best parameters for your datasets'; those are in the paper
- The goal is supervised cell-type classification rather than unsupervised consensus clustering and embedding visualization

## Inputs

- Ramaswamy CITE-seq multimodal single-cell omics data (RNA and ADT modalities for the same cells)
- Cell-type annotation labels split into major and minor cell-type label sets
- SnapCCESS consensus clustering of cells (epoch=1, ensemble size 50)

## Outputs

- figure3.png — Ramaswamy CITE-seq embedding with the MAIT-NKT cell population highlighted (reproduction of Figure 3)
- figure_S3.png — supplementary visualization of the same embedding/clustering (reproduction of Supplementary Fig. S3)
- ari_ensemble_comparison.csv — ensemble-size-50 vs base (size 1) ARI against major and minor cell-type labels per dataset

## How to apply

Run SnapCCESS (epoch=1, ensemble size 50) on the Ramaswamy CITE-seq data to generate snapshot variational-autoencoder embeddings of the RNA and ADT modalities and derive the consensus clustering of cells. Project the consensus clustering onto a 2-D embedding of the cells, then color/highlight cells belonging to the MAIT-NKT cell-type label set to reproduce Figure 3 and Supplementary Fig. S3 as figure3.png and figure_S3.png. The MAIT-NKT population is a minor cell type, so verify it is resolved only under the ensemble-of-50 configuration, where minor-partition ARI improves over the base (ensemble size 1) clustering.

## Related tools

- **SnapCCESS** (Unsupervised ensemble deep learning framework that creates snapshots of multimodality variational-autoencoder embeddings used to generate the consensus clustering visualized with the MAIT-NKT population highlighted) — https://github.com/PYangLab/SnapCCESS
- **snapccess** (Python package (pip-installable) that generates the multimodal embeddings underlying the clustering; parameters for the published datasets are listed in the paper) — https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- **Python** (Implementation language for the snapccess package and the example scripts for generating embeddings)
- **R** (Optional R wrapper installation via remotes::install_github; the R wrapper calls the Python package and expects modalities arranged as features by cells) — https://github.com/yulijia/SnapCCESS

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- figure3.png and figure_S3.png render the MAIT-NKT cells highlighted in the Ramaswamy CITE-seq embedding, matching Figure 3 and Supplementary Fig. S3
- The ensemble-size-50 ARI against the minor cell-type labels matches or improves on the base (ensemble size 1) ARI
- Reported ARI values are consistent with those in the paper (Bioinformatics 39(6), btad382, 2023)
- Consensus clustering is computed with epoch=1 and ensemble size 50, not the base configuration of 1 snapshot
- The comparison is tabulated per dataset in ari_ensemble_comparison.csv covering both major and minor cell-type partitions

## Limitations

- The tutorial only explains how to use the package; it does not recommend the best parameters for your datasets — for the datasets used in the published paper, the parameters are listed in the paper
- The Python package depends on PyTorch; GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation
- The published article discussion content was not available (no changelog or body text), so failure modes for the visualization itself must be inferred from the figures' validation criteria
- Each modality must be measured across the same cells and arranged as features by cells before preprocessing in the R wrapper

## Evidence

- [results] Visualize the Ramaswamy CITE-seq consensus clustering, highlighting the MAIT-NKT cell population, to reproduce Figure 3 and Supplementary Fig. S3: [evidence span withheld — anchor status: not_found]
- [results] Snapshot ensemble improves minor cell-type identification: [evidence span withheld — anchor status: not_found]
- [abstract] SnapCCESS method description: "By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells"
- [readme] Tutorial parameter caveat: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
- [readme] Input modality format: [evidence span withheld — anchor status: external_doc]
