---
name: variational-autoencoder-latent-sampling
description: Use when a generative model must select a discrete subset of k genes
  (or features) during training and the selection must remain differentiable — e.g.,
  scRNA-seq AnnData in adata.X with cell-type labels in adata.obs['annotation'].
when_to_use_negative:
- When a deterministic post-hoc ranking of markers suffices (e.g. rank-correlation
  methods like RankCorr or cosine-similarity methods like COSG) rather than a selector
  jointly trained inside a generative model.
- When the goal is whole-transcriptome reconstruction with all genes retained rather
  than selection of a minimal k-marker set.
- When the selection step does not need to be differentiable or trained with gradient-based
  optimization together with a downstream objective.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: MarkerMap
  role: VAE-based generative model whose selector layer hosts the temperature-annealed
    discrete γ(k) sampling; installed via pip and trained with train_model
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: numpy
  role: Handles the AnnData-backed data arrays and records γ(k) samples and sharpness
    (entropy, max-probability) diagnostics per τ
- name: scanpy
  role: Reads the CITE-seq .h5ad file into the AnnData object whose adata.X and adata.obs['annotation']
    feed the selector
- name: matplotlib
  role: Plots γ(k) sample distributions / sharpness metrics against τ into temperature_annealing_diagnostics.png
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:0cf244bb0647fbd1ea767e9a2d1b0dfc5bade8e19ffe98a29287b582be5385b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - matplotlib
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# variational-autoencoder-latent-sampling

## Summary

Implements MarkerMap's differentiable discrete marker selector: a temperature-annealed Gumbel-Softmax-style sampling module draws γ(k) gene-subset samples inside a VAE so the selection of k markers is optimized end-to-end with the downstream predictive/reconstruction objectives.

## When to use

Use when a generative model must select a discrete subset of k genes (or features) during training and the selection must remain differentiable — e.g., scRNA-seq AnnData in adata.X with cell-type labels in adata.obs['annotation'].

## When NOT to use

- When a deterministic post-hoc ranking of markers suffices (e.g. rank-correlation methods like RankCorr or cosine-similarity methods like COSG) rather than a selector jointly trained inside a generative model.
- When the goal is whole-transcriptome reconstruction with all genes retained rather than selection of a minimal k-marker set.
- When the selection step does not need to be differentiable or trained with gradient-based optimization together with a downstream objective.

## Inputs

- scRNA-seq AnnData object (e.g. data/cite_seq/CITEseq.h5ad) read with scanpy; gene counts in adata.X (converted to dense via adata.X.toarray() if sparse)
- Cell-type labels in adata.obs['names'] copied into adata.obs['annotation'] (or another group_by key)
- Hyperparameters: number of markers k, z_size (16), hidden_layer_size (~10% of input dim, e.g. 64 for 500-gene CITE-seq), batch_size, loss_tradeoff (0 supervised / 0.5 mixed / 1 unsupervised)
- Temperature schedule parameters: τ_initial ≥ 2, geometric decay, τ_final ∈ (0.001, 0.1)

## Outputs

- γ(k) marker-subset samples drawn at each τ checkpoint
- Structured annealing diagnostics per τ: normalized entropy and maximum selection probability of the γ(k) samples
- Trained MarkerMap markers tensor (markers().clone().cpu().detach().numpy()) used by new_model_metrics
- Diagnostics figure saved as temperature_annealing_diagnostics.png

## How to apply

Implement the discrete sampling in the selector layer of the VAE: parameterize a relaxed categorical (Gumbel-Softmax) distribution over genes and expose the temperature τ, initialized at τ_initial ≥ 2 and annealed via a geometric decay schedule down to τ_final in the open interval (0.001, 0.1). At each training step draw γ(k) samples from the distribution so that gradients propagate through the discrete selection into the encoder and the downstream predictive or reconstruction loss, enabling joint end-to-end optimization of selection and prediction. During a diagnostic sweep, checkpoint the τ value, draw γ(k) samples at each τ, record them with numpy, and quantify sharpness via normalized entropy and maximum selection probability per τ. Plot the γ(k) sample distributions (or their sharpness metrics) against τ with matplotlib. Validate that samples are diffuse (high entropy) at τ_initial ≥ 2 and near one-hot (entropy approaching 0, max probability approaching 1) as τ reaches τ_final.

## Related tools

- **MarkerMap** (VAE-based generative model whose selector layer hosts the temperature-annealed discrete γ(k) sampling; installed via pip and trained with train_model) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **numpy** (Handles the AnnData-backed data arrays and records γ(k) samples and sharpness (entropy, max-probability) diagnostics per τ)
- **scanpy** (Reads the CITE-seq .h5ad file into the AnnData object whose adata.X and adata.obs['annotation'] feed the selector)
- **matplotlib** (Plots γ(k) sample distributions / sharpness metrics against τ into temperature_annealing_diagnostics.png)

## Examples

```
supervised_marker_map = MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs['annotation'].unique()), k=50, loss_tradeoff=0); train_model(supervised_marker_map, train_dataloader, val_dataloader)
```

## Evaluation signals

- Annealing invariant: γ(k) samples are diffuse (high normalized entropy) at τ_initial ≥ 2 and near one-hot (entropy approaching 0, max selection probability approaching 1) as τ reaches τ_final.
- τ_final stays inside the open interval (0.001, 0.1) and τ_initial ≥ 2 throughout the geometric decay schedule.
- Each γ(k) sample contains exactly k genes, and gradients flow end-to-end through the sampling step into the downstream loss (no autograd break at the discrete selection).
- Downstream sanity check: new_model_metrics trains a RandomForestClassifier on the k selected markers and reports misclassification rate and weighted f1-score; RandomBaseline on the same k should be outperformed.
- Trained markers tensor (markers().clone().cpu().detach().numpy()) has length k and matches the selected genes in adata.X.

## Limitations

- Sparse array data must first be converted to dense (adata.X = adata.X.toarray()) before being fed to the model.
- On Google Colab, the runtime may need restarting after installation because scanpy installs a newer matplotlib than the default; Windows support is listed as 'Coming soon!'.
- τ values outside the validated annealing range (τ_final below 0.001 or above 0.1) are not covered by the diagnostic workflow.
- The number of markers k may require domain knowledge and typically needs benchmarking over a range of k values.

## Evidence

- [other] MarkerMap's selector performs marker selection probabilistically by sampling from a discrete distribution, which makes the discrete selection of k genes compatible with end-to-end gradient-based optimization of the selection and predictive steps.: [evidence span withheld — anchor status: not_found]
- [readme] MarkerMap is a generative model for selecting the most informative gene markers by projecting cells into a shared, interpretable embedding without sacrificing accuracy.: "MarkerMap is a generative model for selecting the most informative gene markers by projecting cells into a shared, interpretable embedding without sacrificing accuracy."
- [readme] Data handling and dataloader construction used to feed the selector.: "We can use MarkerMap.prepareData to construct the train_dataloader and val_dataloader that will be used during training."
- [methods] Tooling context for data handling during the sampling sweep.: "Data is handled with numpy and with scanpy which is used for many computational biology datasets."
