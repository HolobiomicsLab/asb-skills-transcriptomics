---
name: circular-manifold-pseudotime-inference
description: Use when scRNA-seq count matrices must be assigned circular/periodic
  pseudo-time (e.g., cell-cycle stages) or when periodic-process-associated genes
  must be identified without marker genes.
when_to_use_negative:
- The biological process of interest is non-periodic (e.g., linear differentiation
  trajectories better served by Monocle or Wanderlust).
- Only marker-gene-based cell-cycle scoring is required and no pseudo-time or periodic-gene
  ranking is needed.
- The expression matrix is a model-ready feature table already deconvolved of cell-cycle
  effects — applying Cyclum again would remove no additional periodic signal.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that fits the data to predefined circular manifolds,
    infers circular pseudo-time, ranks periodic genes, and removes cell-cycle effects
  repo: github.com/KChen-lab/cyclum
- name: cyclum.tuning.CyclumAutoTune
  role: Automatically selects the proper number of linear components to find the 'most
    circular' manifold before pseudo-time inference
  repo: github.com/KChen-lab/Cyclum
- name: reCAT
  role: Model-free Gaussian mixture model comparator for benchmarking Cyclum's pseudo-time/cell-cycle
    classification accuracy
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:66b17207aa556b7ac62fee520ada93c2ac0f02ac46b2050599e0ede8eae7d6ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - cyclum.tuning.CyclumAutoTune
  - reCAT
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# circular-manifold-pseudotime-inference

## Summary

Infers latent periodic (e.g., cell-cycle) pseudo-time by projecting scRNA-seq gene expression onto a circular manifold with a sinusoidal autoencoder (Cyclum), instead of relying on marker genes or linear trajectory models.

## When to use

Use when scRNA-seq count matrices must be assigned circular/periodic pseudo-time (e.g., cell-cycle stages) or when periodic-process-associated genes must be identified without marker genes.

## When NOT to use

- The biological process of interest is non-periodic (e.g., linear differentiation trajectories better served by Monocle or Wanderlust).
- Only marker-gene-based cell-cycle scoring is required and no pseudo-time or periodic-gene ranking is needed.
- The expression matrix is a model-ready feature table already deconvolved of cell-cycle effects — applying Cyclum again would remove no additional periodic signal.

## Inputs

- scRNA-seq count/expression matrix (e.g., 10X Chromium hESC matrix with 12,280 cells; mESC matrix from the cyclum2 repository notebook data)
- Cell metadata with annotated cell-cycle stages (for accuracy evaluation)
- Labels from other sources (e.g., scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring) for refinement

## Outputs

- Per-cell circular pseudo-time / inferred cell-cycle stage
- Ranked table of genes by circular-pattern magnitude (e.g., gene_circular_magnitude_ranking.csv)
- Refined stage labels and cell-cycle-effect-removed expression matrix
- Accuracy tables (e.g., accuracy_comparison.csv) and robustness curves vs. number of cells or genes

## How to apply

Preprocess the count matrix with a log2(x+1) transform, scaling to variance = 1 and centering (mean = 1), as recommended in the Cyclum FAQ. Use cyclum.tuning.CyclumAutoTune to determine the optimal number of linear dimensions, then fit the Cyclum sinusoidal autoencoder (cyclum.models) so expression is projected to a circular manifold and unfolded back onto a linear space via optimal (least square) embedding to obtain one pseudo-time per cell. Genes associated with the periodic process can then be ranked by the degree of match between their expression kinetics and the inferred periodicity; differential circular-pattern magnitude (e.g., treated minus control) nominates condition-specific periodic genes. As a robustness control, compare against the model-free GMM-based reCAT on the same matrix.

## Related tools

- **Cyclum** (Sinusoidal autoencoder that fits the data to predefined circular manifolds, infers circular pseudo-time, ranks periodic genes, and removes cell-cycle effects) — github.com/KChen-lab/cyclum
- **cyclum.tuning.CyclumAutoTune** (Automatically selects the proper number of linear components to find the 'most circular' manifold before pseudo-time inference) — github.com/KChen-lab/Cyclum
- **reCAT** (Model-free Gaussian mixture model comparator for benchmarking Cyclum's pseudo-time/cell-cycle classification accuracy)

## Examples

```
import cyclum.tuning; tuner = cyclum.tuning.CyclumAutoTune(molecule_matrix, n_circular_unit=1); tuner.show_total_loss(); model = tuner.model.fit(x, epochs=300); pseudotime = model.predict(x)
```

## Evaluation signals

- Inferred pseudo-time assigns exactly one stage per cell and folds onto a coherent circular trajectory (e.g., G1 -> S -> G2M order).
- Down-sampling robustness: median cell-cycle classification accuracy stays within the 0.70–0.75 band while reCAT degrades and fails below 5000 genes.
- On the nicotine-treated vs. control hESC comparison, LDHA and ENO1 appear among the top-ranked genes by circular-pattern magnitude difference (Supplementary Fig. 8b).
- Preprocessed matrix is scaled to variance = 1 and centered with the log2(x+1) transform before model fitting.
- Scaling check: a 12,280-cell, 33,694-gene dataset should complete in ~40 min on a GTX 960M GPU with 2 GB graphic memory.

## Limitations

- Requires TensorFlow 1.x; pip will install latest dependencies that differ from tested versions (e.g., keras 2.2.4, numpy 1.16.5, tensorflow 1.14.0), so pin versions in setup.py if failures occur.
- Model-based fitting to predefined circular manifolds assumes the periodic structure exists ab initio; it cannot invent periodicity absent from the data.
- Performance/hardware details (40 min for 12,280 cells on GTX 960M) are only described informally with detail deferred to Supplementary Note 2; no version numbers for Python/Keras/TensorFlow are stated in the paper itself.
- An autoencoder implementation based on Gaussian Process Latent Variable Models was planned but not yet available ('we are adding Gaussian Process Latent Variable Model soon').

## Evidence

- [abstract] we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space: "we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space"
- [methods] scaled (variance = 1), centered (mean = 1) log-transformed (log2(x+1)) data work well: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [methods] The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes.: "The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes."
- [intro] It effectively unfolds the circular manifold onto a linear space to obtain precise pseudo-time.: "It effectively unfolds the circular manifold onto a linear space to obtain precise pseudo-time."
- [readme] We also implemented class `cyclum.tuning.CyclumAutoTune`, which automatically select the proper number of linear components: [evidence span withheld — anchor status: external_doc]
- [readme] Please make sure that you have TensorFlow 1.x.: [evidence span withheld — anchor status: external_doc]
