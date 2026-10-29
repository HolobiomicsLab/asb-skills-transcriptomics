---
name: discrete-selector-layer-implementation
description: Use when building or re-implementing a neural gene/marker selector on
  scRNA-seq data (e.g. CITE-seq .h5ad) where selecting a discrete subset of k genes
  must remain differentiable for joint gradient-based optimization with a VAE-style
  reconstruction or classification objective.
when_to_use_negative:
- When only a final, non-differentiable discrete marker ranking is needed (e.g. RankCorr-
  or COSG-style score-and-rank selection) with no joint end-to-end training of the
  selector
- When the input is not per-cell gene expression compatible with the MarkerMap VAE
  pipeline (e.g. already-aggregated feature tables or non-AnnData formats without
  conversion such as adata.X = adata.X.toarray() for sparse matrices)
- When the goal is unsupervised-only embedding without marker selection supervision
  — that is a different loss_tradeoff regime of the same model rather than this selector-layer
  skill
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MarkerMap
  role: Generative VAE model whose selector layer performs probabilistic discrete
    k-gene marker selection jointly with reconstruction/classification; provides prepareData,
    train_model and markers()
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: scanpy
  role: Reads the CITE-seq .h5ad file into the AnnData object that feeds the selector
- name: numpy
  role: Handles the γ(k) samples and diagnostics arrays recorded across the τ annealing
    sweep
- name: matplotlib
  role: Plots the annealing diagnostics (sample sharpness vs τ) to temperature_annealing_diagnostics.png
- name: pip
  role: Installs the pip-installable markermap package used for the selector implementation
  repo: https://pypi.org/project/markermap/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:f6210b5c904f13db4760f7b8d679f5f705bef872ea18b23ef6f5c8d6282d3745
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - matplotlib
  - numpy
  - pip
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# discrete-selector-layer-implementation

## Summary

Implements MarkerMap's differentiable discrete marker-selection layer, where the selection of k genes is performed by sampling from a probabilistic discrete distribution so it can be trained end-to-end with the downstream predictive/reconstruction loss, typically controlled by a temperature parameter τ annealed from a diffuse (τ_init ≥ 2) to a near one-hot (τ_final in (0.001, 0.1)) regime.

## When to use

Use when building or re-implementing a neural gene/marker selector on scRNA-seq data (e.g. CITE-seq .h5ad) where selecting a discrete subset of k genes must remain differentiable for joint gradient-based optimization with a VAE-style reconstruction or classification objective.

## When NOT to use

- When only a final, non-differentiable discrete marker ranking is needed (e.g. RankCorr- or COSG-style score-and-rank selection) with no joint end-to-end training of the selector
- When the input is not per-cell gene expression compatible with the MarkerMap VAE pipeline (e.g. already-aggregated feature tables or non-AnnData formats without conversion such as adata.X = adata.X.toarray() for sparse matrices)
- When the goal is unsupervised-only embedding without marker selection supervision — that is a different loss_tradeoff regime of the same model rather than this selector-layer skill

## Inputs

- Single-cell RNA-seq AnnData object (adata.X gene counts, adata.obs['annotation'] labels) read from a .h5ad file such as data/cite_seq/CITEseq.h5ad
- Train/validation indices from a 70%/10%/20% split_data call
- Selector hyperparameters: number of markers k (e.g. k=50), z_size=16, hidden_layer_size (≈10% of input dims, e.g. 64 for the 500-column CITE-seq data), batch_size=64
- Temperature schedule parameters: τ_initial ≥ 2 and τ_final ∈ (0.001, 0.1)

## Outputs

- Trained selector producing a discrete set of k marker gene indices (supervised_marker_map.markers() as numpy indices)
- Recorded γ(k) samples per temperature checkpoint (numpy arrays)
- Structured diffuseness diagnostics per τ (normalized entropy, maximum selection probability)
- Annealing diagnostic figure, e.g. temperature_annealing_diagnostics.png

## How to apply

Load the single-cell AnnData object with scanpy (e.g. data/cite_seq/CITEseq.h5ad), split it 70%/10%/20% and build train/val dataloaders with MarkerMap.prepareData so the selector is exercised inside the model. Implement the selector layer to sample marker subsets from a discrete distribution rather than making hard index choices, keeping the k-gene selection compatible with backpropagation through both the selection and predictive/reconstruction steps. Expose the sampling temperature τ as a parameter initialized at τ_initial ≥ 2 and annealed down geometrically to a τ_final in the open interval (0.001, 0.1), drawing γ(k) samples at each annealing checkpoint and recording them with numpy. At each τ, quantify diffuseness vs. one-hotness (e.g. normalized entropy and maximum selection probability) so the anneal can be validated, and plot the γ(k) sample sharpness against τ with matplotlib (e.g. temperature_annealing_diagnostics.png). Correctly applied, the selector trains jointly via train_model with the supervised MarkerMap (loss_tradeoff=0 for cell-type selection) and the resulting supervised_marker_map.markers() are discrete k-gene indices usable by a downstream classifier.

## Related tools

- **MarkerMap** (Generative VAE model whose selector layer performs probabilistic discrete k-gene marker selection jointly with reconstruction/classification; provides prepareData, train_model and markers()) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **scanpy** (Reads the CITE-seq .h5ad file into the AnnData object that feeds the selector)
- **numpy** (Handles the γ(k) samples and diagnostics arrays recorded across the τ annealing sweep)
- **matplotlib** (Plots the annealing diagnostics (sample sharpness vs τ) to temperature_annealing_diagnostics.png)
- **pip** (Installs the pip-installable markermap package used for the selector implementation) — https://pypi.org/project/markermap/

## Examples

```
from markermap.vae_models import MarkerMap, train_model; train_dataloader, val_dataloader = MarkerMap.prepareData(adata, train_indices, val_indices, 'annotation', None, batch_size=64); supervised_marker_map = MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs['annotation'].unique()), 50, loss_tradeoff=0); train_model(supervised_marker_map, train_dataloader, val_dataloader)
```

## Evaluation signals

- γ(k) samples are diffuse (high normalized entropy) at τ_initial ≥ 2 and near one-hot (entropy approaching 0, max selection probability approaching 1) at τ_final in (0.001, 0.1)
- The selector's markers() output is a valid discrete set of exactly k gene indices within adata.X.shape[1]
- Gradients flow end-to-end: loss decreases during train_model(supervised_marker_map, train_dataloader, val_dataloader) rather than stopping at the discrete selection
- Downstream sanity check: new_model_metrics on the k selected markers yields a finite misclassification rate and weighted f1-score on the held-out 20% test split, above the RandomBaseline control
- Sparse input was converted to dense before the selector (adata.X = adata.X.toarray()) so sampling is numerically stable

## Limitations

- The k parameter and loss_tradeoff (0 supervised, 0.5 mixed, 1 unsupervised) require tuning; k 'may require some domain knowledge'
- The temperature schedule and its (τ_init ≥ 2 → τ_final ∈ (0.001, 0.1)) envelope are diagnostic conventions from the surfaced task; the MarkerMap paper/README do not prescribe exact annealing parameters, so they must be validated empirically
- Comparing against Smashpy creates installation problems (tensorflow ==2.5.0), and on Google Colab a runtime restart may be needed after install because scanpy installs a newer matplotlib

## Evidence

- [abstract] abstract: "Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction."
- [readme] readme: "MarkerMap is a generative model for selecting the most informative gene markers by projecting cells into a shared, interpretable embedding without sacrificing accuracy."
- [methods] methods: "We can use MarkerMap.prepareData to construct the train_dataloader and val_dataloader that will be used during training."
