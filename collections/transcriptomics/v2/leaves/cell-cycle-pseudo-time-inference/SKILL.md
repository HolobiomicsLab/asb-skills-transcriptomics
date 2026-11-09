---
name: cell-cycle-pseudo-time-inference
description: Use when scRNA-seq gene expression is expected to follow a periodic process
  (e.g., cell cycle) and you need ab initio per-cell pseudo-time or phase ordering
  without prior marker-gene lists.
when_to_use_negative:
- The process of interest is linear/non-periodic — existing trajectory methods (e.g.,
  Monocle, Wanderlust) based on linear representations are designed for such data
- You already have trusted discrete phase labels and only need generic clustering
  — Cyclum's value is ab initio circular manifold fitting, not marker-gene scoring
- One pseudo-time value in isolation is needed for interpretation — per the paper,
  'one pseudo-time for one cell itself does not make sense'; values are meaningful
  only as an ordered trajectory
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cyclum
  role: 'Primary tool: sinusoidal autoencoder that projects high-dimensional gene
    expression data to a circular manifold and infers per-cell pseudo-time'
  repo: github.com/KChen-lab/cyclum
- name: cyclum.tuning.CyclumAutoTune
  role: Automatically selects the proper number of linear components to find the 'most
    circular' manifold before pseudo-time inference
  repo: github.com/KChen-lab/cyclum
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:7dab5d04364a0eeab7468ff4c5b82adad7a792ddd5ec7dbb31f31249b7388b8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclone
  - Cyclum
  - Oscope
  - cyclum.tuning.CyclumAutoTune
  - f-scLVM (Slalom)
  - reCAT
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cell-cycle-pseudo-time-inference

## Summary

Infers per-cell circular pseudo-time for periodic processes such as the cell cycle from scRNA-seq data using Cyclum, a sinusoidal autoencoder that projects high-dimensional gene expression onto a circular manifold instead of relying on marker genes.

## When to use

Use when scRNA-seq gene expression is expected to follow a periodic process (e.g., cell cycle) and you need ab initio per-cell pseudo-time or phase ordering without prior marker-gene lists.

## When NOT to use

- The process of interest is linear/non-periodic — existing trajectory methods (e.g., Monocle, Wanderlust) based on linear representations are designed for such data
- You already have trusted discrete phase labels and only need generic clustering — Cyclum's value is ab initio circular manifold fitting, not marker-gene scoring
- One pseudo-time value in isolation is needed for interpretation — per the paper, 'one pseudo-time for one cell itself does not make sense'; values are meaningful only as an ordered trajectory

## Inputs

- scRNA-seq gene-by-cell count matrix (raw expression data)
- Optionally: phase labels from another source (e.g., Seurat::CellCycleScoring or scanpy.tl.score_genes_cell_cycle) to refine along pseudotime
- Optionally: Hoechst-based cell-cycle phase labels for accuracy validation

## Outputs

- Per-cell circular pseudo-time (one value per cell, stage of the periodic process)
- Inferred cell-cycle phase/stage assignment per cell
- Selected number of linear dimensions for the circular manifold (from CyclumAutoTune)
- Optionally: expression matrix with cell-cycle effects removed (deconvolution)

## How to apply

Preprocess the count matrix with a log2(x+1) transformation, then center to mean = 1 and scale to variance = 1, as recommended in the Cyclum documentation. Fit the Cyclum sinusoidal autoencoder to project cells onto the circular manifold and unfold it onto a linear space via least-square embedding to obtain one pseudo-time per cell. Use cyclum.tuning.CyclumAutoTune to select the number of linear dimensions that yields the 'most circular' manifold. Optionally refine externally derived labels (e.g., from scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring) along the inferred pseudotime so out-of-order phases are smoothed. Evaluate by classifying cells into G1/S/G2M phases against Hoechst-based labels and comparing accuracy to baselines such as Cyclone, reCAT, Oscope, f-scLVM (Slalom), Cyclops, PCA, NMF, and ICA.

## Related tools

- **Cyclum** (Primary tool: sinusoidal autoencoder that projects high-dimensional gene expression data to a circular manifold and infers per-cell pseudo-time) — github.com/KChen-lab/cyclum
- **cyclum.tuning.CyclumAutoTune** (Automatically selects the proper number of linear components to find the 'most circular' manifold before pseudo-time inference) — github.com/KChen-lab/cyclum

## Examples

```
from cyclum.postproc import refine_labels; refine_labels(pseudotime=[4, 5, 1, 2, 3, 6, 7, 8], original_labels=['G1', 'G1', 'G1', 'G1', 'S', 'S', 'G2M', 'G2M'])
```

## Evaluation signals

- Preprocessing invariant: data are log2(x+1) transformed, centered to mean = 1, and scaled to variance = 1 before fitting
- Pseudo-time sanity check: exactly one pseudo-time value per cell, and labels refined along it stay contiguous (e.g., an 'S' surrounded by 'G1's becomes 'G1')
- On the four Hoechst-labeled datasets (mESC, PC3, MDA-MB-231, H9), Cyclum's classification accuracy exceeds every baseline (Cyclone, reCAT, Oscope, f-scLVM/Slalom, Cyclops, PCA/NMF/ICA) on each dataset
- All accuracy values in the comparison table lie in [0, 1] for all methods and datasets
- CyclumAutoTune output: the chosen number of linear dimensions yields the 'most circular' manifold

## Limitations

- Requires TensorFlow 1.x; tested with keras 2.2.4, tensorflow 1.14.0, numpy 1.16.5, and Python 3.7.4 — pip may otherwise install incompatible latest versions
- A single pseudo-time value per cell is not meaningful in isolation; interpretation requires the full ordered trajectory
- Accuracy advantage was demonstrated against specific baselines (including model-free reCAT) on the four Hoechst-labeled datasets; robustness claims rest on the model-based circular-manifold design

## Evidence

- [methods] scaled (variance = 1), centered (mean = 1) log-transformed (log2(x+1)) data work well: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [intro] ab initio inference method, namely Cyclum, which employs a distinct sinusoidal autoencoder to capture the circular trajectory: "To address these limitations, we develop an ab initio inference method, namely Cyclum, which employs a distinct sinusoidal autoencoder to capture the circular trajectory"
- [discussion] It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles.: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [methods] The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes.: "The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes."
- [readme] we implemented class cyclum.tuning.CyclumAutoTune, which automatically select the proper number of linear components to help find the 'most circular' manifold: [evidence span withheld — anchor status: external_doc]
- [intro] It effectively unfolds the circular manifold onto a linear space to obtain precise pseudo-time.: "It effectively unfolds the circular manifold onto a linear space to obtain precise pseudo-time."
