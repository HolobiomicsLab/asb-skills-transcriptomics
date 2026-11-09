---
name: gumbel-softmax-differentiable-marker-selection
description: Use when selecting a minimal, informative set of k gene markers from
  single-cell RNA-seq or CITE-seq data (.h5ad AnnData) such that markers are differentiably
  learned end-to-end and support both cell-type classification and whole-transcriptome
  reconstruction.
when_to_use_negative:
- You need only a simple baseline ranking of markers — RandomBaseline (k markers at
  random) or RankCorr-style rank correlation may suffice rather than training a VAE
- The goal is FISH-panel gene selection with binarized expression accounting for the
  scRNA-seq/FISH measurement shift — that is PERSIST's scope, not MarkerMap's differentiable
  sampler
- Data cannot be represented as a dense cell-by-gene matrix with categorical cell
  labels in AnnData (adata.X and adata.obs[group_by])
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MarkerMap
  role: Provides the MarkerMap VAE class with the differentiable (Gumbel-Softmax)
    k-marker selector, prepareData, getBenchmarker, and train_model used to learn
    markers end-to-end
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: scanpy
  role: Reads the .h5ad dataset (e.g. data/cite_seq/CITEseq.h5ad) into an AnnData
    object whose adata.X and adata.obs feed the selector
- name: numpy
  role: Handles the gene count matrix and concatenates train/val indices (np.concatenate)
    before dataloader construction
- name: pip
  role: Installs the markermap package (`pip install markermap`) that supplies vae_models.MarkerMap
    and train_model
  repo: https://pypi.org/project/markermap/
- name: persist
  role: Optional companion package installed separately for comparison with PERSIST's
    gene selection baselines
  repo: https://github.com/iancovert/persist/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:e49ed83cbb03aa4d080559bdf0c60c2e6653323b97fcf6304063ef66dd0f4fd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - numpy
  - persist
  - pip
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gumbel-softmax-differentiable-marker-selection

## Summary

Implements MarkerMap's differentiable marker selection: a Gumbel-Softmax selector layer samples a global set of k gene markers from scRNA-seq/CITE-seq data, feeding a VAE reconstruction branch and a supervised classification branch combined via a loss_tradeoff-weighted convex loss. It enables end-to-end joint marker selection and transcriptome reconstruction within a single variational auto-encoder model.

## When to use

Use when selecting a minimal, informative set of k gene markers from single-cell RNA-seq or CITE-seq data (.h5ad AnnData) such that markers are differentiably learned end-to-end and support both cell-type classification and whole-transcriptome reconstruction.

## When NOT to use

- You need only a simple baseline ranking of markers — RandomBaseline (k markers at random) or RankCorr-style rank correlation may suffice rather than training a VAE
- The goal is FISH-panel gene selection with binarized expression accounting for the scRNA-seq/FISH measurement shift — that is PERSIST's scope, not MarkerMap's differentiable sampler
- Data cannot be represented as a dense cell-by-gene matrix with categorical cell labels in AnnData (adata.X and adata.obs[group_by])

## Inputs

- AnnData object (.h5ad, e.g. CITEseq.h5ad) with gene expression in adata.X and cell labels in adata.obs[group_by] (e.g. 'annotation')
- Integer k, the number of markers to extract (e.g. k=50)
- loss_tradeoff in {0, 0.5, 1.0} selecting supervised, joint, or unsupervised mode
- hidden_layer_size (e.g. 64), z_size (16), batch_size (e.g. 64)
- train_dataloader and val_dataloader built by MarkerMap.prepareData from a 70/10/20 split

## Outputs

- Trained MarkerMap model whose markers() method returns exactly k marker gene indices (e.g. k=50) as a tensor usable after .clone().cpu().detach().numpy()
- Evaluation metrics from new_model_metrics: misclassification rate, weighted-avg f1-score, and a confusion matrix plottable with plot_confusion_matrix
- Latent-space reconstruction and classification behavior controlled by the loss_tradeoff mode (unsupervised/supervised/joint)

## How to apply

Build a variational auto-encoder with input_size matching the gene dimension (e.g. 500 for CITE-seq), hidden_layer_size (~10% of input, e.g. 64), z_size=16, and num_classes; add a Gumbel-Softmax selector over a d-dimensional logit vector so k markers are differentiably sampled and exposed via a markers() method. Combine the reconstruction loss and classification loss with a convex combination controlled by loss_tradeoff: 0 for supervised-only, 0.5 for mixed, 1.0 for unsupervised-only. Prepare inputs by reading the .h5ad with scanpy (converting sparse adata.X to dense via adata.X.toarray() if needed), splitting 70%/10%/20% with split_data, and building dataloaders with MarkerMap.prepareData using the group_by label column (e.g. 'annotation'). Train with train_model on the train/val dataloaders, then evaluate the returned marker indices by training a RandomForestClassifier on training data restricted to the k markers (new_model_metrics) on held-out test data.

## Related tools

- **MarkerMap** (Provides the MarkerMap VAE class with the differentiable (Gumbel-Softmax) k-marker selector, prepareData, getBenchmarker, and train_model used to learn markers end-to-end) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **scanpy** (Reads the .h5ad dataset (e.g. data/cite_seq/CITEseq.h5ad) into an AnnData object whose adata.X and adata.obs feed the selector)
- **numpy** (Handles the gene count matrix and concatenates train/val indices (np.concatenate) before dataloader construction)
- **pip** (Installs the markermap package (`pip install markermap`) that supplies vae_models.MarkerMap and train_model) — https://pypi.org/project/markermap/
- **persist** (Optional companion package installed separately for comparison with PERSIST's gene selection baselines) — https://github.com/iancovert/persist/

## Examples

```
from markermap.vae_models import MarkerMap, train_model; supervised_marker_map = MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs[group_by].unique()), 50, loss_tradeoff=0); train_model(supervised_marker_map, train_dataloader, val_dataloader)
```

## Evaluation signals

- markermap module imports MarkerMap and train_model without error
- train_model completes on train/val dataloaders built from the CITE-seq data without exceptions
- markers() returns exactly k indices (e.g. k=50) that can be consumed by a RandomForestClassifier in new_model_metrics
- Misclassification rate and weighted-avg f1-score printed for held-out test data are competitive against a RandomBaseline control across benchmarked k values
- loss_tradeoff behaves as specified: 0 (supervised only), 0.5 (mixed), 1.0 (unsupervised only) change whether the classification branch contributes to the loss

## Limitations

- hidden_layer_size is data-dependent — a rule of thumb is ~10% of input dimension (64 for the 500-column CITE-seq data), so it must be tuned per dataset
- k may require domain knowledge and is best determined by benchmarking over a range of k values
- Datasets like Mouse Brain need preprocessing (removing outliers, normalizing) before use
- Installation friction: scanpy installs a newer matplotlib (restart needed on Colab) and Smashpy pins tensorflow==2.5.0 causing conflicts
- The Gumbel-Softmax sampler is a relaxation, so selected markers are stochastic and results vary across runs (benchmark runs the model multiple times)

## Evidence

- [methods] Gumbel-Softmax selector over d-dimensional logit vector: [evidence span withheld — anchor status: not_found]
- [readme] loss_tradeoff convex combination levels: "Loss_tradeoff is also important, see the paper for a further discussion. In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only)."
- [abstract] generative model goal: "Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction."
- [readme] evaluation via RandomForest on k markers: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
- [methods] data preparation with scanpy and dense conversion: [evidence span withheld — anchor status: not_found]
