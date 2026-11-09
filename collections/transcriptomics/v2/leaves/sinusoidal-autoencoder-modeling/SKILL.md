---
name: sinusoidal-autoencoder-modeling
description: Use when scRNA-seq expression should be modeled as a circular/periodic
  trajectory (e.g., cell cycle) to infer per-cell pseudo-time wrapped to [0, 2π],
  or to nominate genes by their circular-pattern magnitude.
when_to_use_negative:
- The process of interest is a non-periodic, linear developmental trajectory — linear
  pseudo-time methods (e.g., Monocle, Wanderlust) are more appropriate since Cyclum
  fits predefined circular manifolds
- Only gene-set enrichment is needed — Cyclum identifies periodic genes by degree
  of match to inferred periodicity, not by differential expression or enrichment testing
- The matrix is already reduced to a feature table with no per-cell expression values,
  so cells cannot be embedded in the circular manifold
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: Cyclum
  role: 'Primary package: provides cyclum.models.ae (sinusoidal autoencoder), cyclum.tuning.CyclumAutoTune
    (linear-component selection), cyclum.evaluation (gene circular-pattern magnitude),
    and cyclum.postproc (label refinement)'
  repo: https://github.com/KChen-lab/Cyclum
- name: Keras
  role: 'Implements the autoencoder architecture: tanh-activation MLP encoder, cos/sin
    decoder first layer, compiled with MSE loss and L2 regularization'
- name: TensorFlow
  role: Backend runtime for the Keras autoencoder; Cyclum requires TensorFlow 1.x
    (tested at 1.14.0, latest tested 1.15.2)
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:75b94899a8e9a0e8edb677915a86288596b10b594c39c5e6c0f58e1204d0ae91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - Keras
  - TensorFlow
  - cyclum.tuning.CyclumAutoTune
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# sinusoidal-autoencoder-modeling

## Summary

Fit the Cyclum sinusoidal asymmetric autoencoder (tanh encoder, cos/sin decoder) to scRNA-seq data to embed cells on a circular manifold, infer periodic pseudo-time, and rank genes by circular-pattern magnitude. This captures nonlinear periodic processes like the cell cycle that linear trajectory methods cannot represent.

## When to use

Use when scRNA-seq expression should be modeled as a circular/periodic trajectory (e.g., cell cycle) to infer per-cell pseudo-time wrapped to [0, 2π], or to nominate genes by their circular-pattern magnitude.

## When NOT to use

- The process of interest is a non-periodic, linear developmental trajectory — linear pseudo-time methods (e.g., Monocle, Wanderlust) are more appropriate since Cyclum fits predefined circular manifolds
- Only gene-set enrichment is needed — Cyclum identifies periodic genes by degree of match to inferred periodicity, not by differential expression or enrichment testing
- The matrix is already reduced to a feature table with no per-cell expression values, so cells cannot be embedded in the circular manifold

## Inputs

- scRNA-seq gene-expression count matrix (e.g., 10X Chromium hESC: 12,280 cells x 33,694 genes; or mESC matrix)
- Optional pre-computed cell-cycle labels (e.g., from scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring) for label refinement

## Outputs

- Trained Keras model file (e.g., cyclum_ae_model.h5)
- Per-cell circular pseudo-time table wrapped to [0, 2π] (e.g., pseudotime_2pi.csv)
- Ranked table of gene circular-pattern magnitudes or treated-vs-control differences (e.g., gene_circular_magnitude_ranking.csv)
- Refined cell-cycle phase labels via cyclum.postproc.refine_labels

## How to apply

Preprocess the count matrix with log2(x+1) transformation, scaled to variance 1 and centered to mean 1, as recommended in the Cyclum FAQ. Optionally use cyclum.tuning.CyclumAutoTune to select the proper number of linear components so the manifold is 'most circular'. Build the encoder as a multi-layer perceptron with tanh activations (cyclum.models.ae) and a decoder whose first layer computes cosine and sine activations followed by a linear layer back to gene-expression space; compile with mean-squared-error loss plus L2 weight regularization in Keras with TensorFlow. After training converges, extract circular latent coordinates and wrap each cell's pseudo-time into [0, 2π] via the modulus operation. Use cyclum.evaluation to compute per-gene circular-pattern magnitude and rank genes (e.g., treated-minus-control differences to nominate targets like LDHA and ENO1).

## Related tools

- **Cyclum** (Primary package: provides cyclum.models.ae (sinusoidal autoencoder), cyclum.tuning.CyclumAutoTune (linear-component selection), cyclum.evaluation (gene circular-pattern magnitude), and cyclum.postproc (label refinement)) — https://github.com/KChen-lab/Cyclum
- **Keras** (Implements the autoencoder architecture: tanh-activation MLP encoder, cos/sin decoder first layer, compiled with MSE loss and L2 regularization)
- **TensorFlow** (Backend runtime for the Keras autoencoder; Cyclum requires TensorFlow 1.x (tested at 1.14.0, latest tested 1.15.2))

## Examples

```
git clone https://github.com/KChen-lab/Cyclum.git && cd Cyclum && jupyter notebook  # run tests/notebooks for sinusoidal-AE pseudo-time demos
```

## Evaluation signals

- Every value in the saved pseudo-time table falls within [0, 2π] after modulus wrapping
- cyclum_ae_model.h5 exists and is loadable by Keras
- Reconstruction loss (mean squared error plus L2 regularization) converges during training
- CyclumAutoTune identifies a number of linear components giving the 'most circular' manifold
- Gene ranking validation: LDHA and ENO1 appear among the top-ranked genes by treated-vs-control circular-pattern magnitude difference (reproducing Supplementary Fig. 8b)

## Limitations

- Cyclum requires TensorFlow 1.x; pip will try to install the latest dependencies, which differ from the tested versions (python 3.7.4, keras 2.2.4, tensorflow 1.14.0, numpy 1.16.5) — pin versions in setup.py if needed
- Runtime and hardware scale with dataset size: 12,280 cells x 33,694 genes takes ~40 min on a laptop with a GTX 960M GPU and 2 GB graphic memory
- The original article omitted Python/Keras/TensorFlow version numbers in the text, so exact provenance of the trained model depends on the environment pins used

## Evidence

- [abstract] Cyclum autoencoder identifies circular trajectories: "we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space"
- [methods] Preprocessing recipe from the methods: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [readme] AutoTune selects linear components for the most circular manifold: [evidence span withheld — anchor status: external_doc]
- [discussion] Scalability of Cyclum analysis: "Cyclum can analyze an scRNA-seq dataset consisting of 12,280 cells 33,694 genes in 40 min"
