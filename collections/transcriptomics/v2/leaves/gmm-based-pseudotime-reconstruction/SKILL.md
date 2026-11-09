---
name: gmm-based-pseudotime-reconstruction
description: 'Decision support for comparing model-free pseudo-time reconstruction
  against model-based methods: use when you have a scRNA-seq expression matrix (e.g.,
  mESC, log2(x+1), scaled variance=1, centered mean=1) and need GMM-based cell-cycle
  pseudo-time or a robustness baseline.'
when_to_use_negative:
- When the goal is robust pseudo-time under very small cell counts or very few genes
  (<5000), where the article reports reCAT degrades or fails and a model-based circular-manifold
  method is preferred
- When marker-gene-based phase scoring (e.g., scanpy.tl.score_genes_cell_cycle or
  Seurat::CellCycleScoring) labels are already available and only need refinement
  rather than de novo pseudo-time reconstruction
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: reCAT
  role: GMM-based pseudo-time reconstruction method run as the model-free comparator
    on the same down-sampled mESC matrices as Cyclum
- name: Cyclum
  role: Sinusoidal autoencoder benchmarked head-to-head against the GMM-based reCAT
    under cell and gene down-sampling
  repo: github.com/KChen-lab/cyclum
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:503250925eca7c33bbc1d3dc564284ca1a59e57256b91f540870fbd5cdd86ffe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - reCAT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gmm-based-pseudotime-reconstruction

## Summary

Runs reCAT, a Gaussian mixture model (GMM)-based method that reconstructs cell-cycle pseudo-time from scRNA-seq expression data, and serves as the model-free comparator when benchmarking model-based circular-manifold approaches such as Cyclum.

## When to use

Decision support for comparing model-free pseudo-time reconstruction against model-based methods: use when you have a scRNA-seq expression matrix (e.g., mESC, log2(x+1), scaled variance=1, centered mean=1) and need GMM-based cell-cycle pseudo-time or a robustness baseline.

## When NOT to use

- When the goal is robust pseudo-time under very small cell counts or very few genes (<5000), where the article reports reCAT degrades or fails and a model-based circular-manifold method is preferred
- When marker-gene-based phase scoring (e.g., scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring) labels are already available and only need refinement rather than de novo pseudo-time reconstruction

## Inputs

- scRNA-seq gene expression matrix (genes x cells), log2(x+1)-transformed, scaled to variance = 1 and centered to mean = 1
- Down-sampled replicate matrices (e.g., 10- or 30-cell stratified by cell-cycle stage; decreasing gene counts)
- Annotated cell-cycle stage labels for accuracy evaluation

## Outputs

- Cell-cycle pseudo-time per cell inferred by the GMM
- Per-condition cell-cycle stage prediction accuracy values
- accuracy_comparison.csv table and accuracy-vs-cells / accuracy-vs-genes curves (e.g., fig2cd_accuracy_robustness.png)

## How to apply

Preprocess the mESC scRNA-seq expression matrix with a log2(x+1) transform, scaling to variance = 1 and centering to mean = 1, as recommended in the Cyclum FAQ. Generate down-sampled replicate matrices (e.g., stratified sampling of cells by annotated stage to n=10 and n=30, or uniform gene sampling at a decreasing series of gene counts). Run reCAT — which reconstructs cell-cycle pseudo-time using a Gaussian mixture model — on the identical down-sampled matrices used for the Cyclum sinusoidal autoencoder, so the two methods are compared under matched inputs. Compute cell-cycle stage prediction accuracy per condition and plot accuracy versus cell count and versus gene count. Accept the benchmark if reCAT's accuracy degrades with fewer cells and reCAT fails below 5000 genes, while the model-based comparator's median accuracy stays within a stable band (0.70-0.75).

## Related tools

- **reCAT** (GMM-based pseudo-time reconstruction method run as the model-free comparator on the same down-sampled mESC matrices as Cyclum)
- **Cyclum** (Sinusoidal autoencoder benchmarked head-to-head against the GMM-based reCAT under cell and gene down-sampling) — github.com/KChen-lab/cyclum

## Examples

```
from cyclum.tuning import CyclumAutoTune  # comparator reCAT run on same down-sampled matrix; accuracy saved to accuracy_comparison.csv
```

## Evaluation signals

- reCAT's median cell-cycle classification accuracy on mESC becomes substantially worse as the number of cells is down-sampled
- reCAT fails on matrices sampled below 5000 genes
- Accuracy curves for reCAT degrade monotonically with fewer cells/genes while the model-based comparator's median accuracy stays within the 0.70-0.75 band
- Accuracy table saved as accuracy_comparison.csv matches the plotted curves in fig2cd_accuracy_robustness.png
- Both methods are run on the identical down-sampled matrices so the comparison is matched-input

## Limitations

- As a model-free approach, reCAT is less robust to random noise and small sample sizes than model-based circular-manifold methods like Cyclum
- reCAT fails below approximately 5000 genes, limiting use on heavily down-sampled or low-depth data

## Evidence

- [intro] A recent method reCAT25 reconstructs cell-cycle pseudo-time using a Gaussian mixture model (GMM): "A recent method reCAT25 reconstructs cell-cycle pseudo-time using a Gaussian mixture model (GMM)"
- [discussion] Cyclum robust to noise and small samples: "Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes."
- [discussion] Advantage over reCAT: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [methods] Preprocessing recommendation: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [methods] mESC notebook source: "`Cyclum analysis of the mESC dataset <https://github.com/lshh125/cyclum2/blob/master/tests/notebooks/mESC.ipynb>`_"
