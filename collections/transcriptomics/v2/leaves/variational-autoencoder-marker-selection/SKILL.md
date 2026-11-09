---
name: variational-autoencoder-marker-selection
description: 'Use when scRNA-seq AnnData needs a small informative gene panel: supervised
  or unsupervised marker selection, cell-type classification from k markers, or full-transcriptome
  reconstruction quality comparison against baselines like scVI.'
when_to_use_negative:
- Input data is not gene-expression counts from single-cell assays (the model is designed
  for scRNA-seq/scATAC-seq-like AnnData matrices, not feature tables or bulk metadata)
- You only need fast statistical marker ranking rather than a learned generative model
  — simpler methods like COSG or rank-correlation-based RankCorr may suffice
- The dataset is too small or labels are unreliable such that training a VAE would
  overfit; the paper benchmarks on curated datasets (CITE-seq, Mouse Brain, Paul,
  Zeisel) with outlier removal and normalization already applied
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MarkerMap
  role: Core VAE-based generative model providing marker selection (supervised, mixed,
    unsupervised via loss_tradeoff), data preparation (prepareData), training (train_model),
    and benchmarking utilities (getBenchmarker, benchmark)
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: scanpy
  role: Reads h5ad data into an AnnData object used as the input to MarkerMap
- name: numpy
  role: Handles dense gene-count arrays (adata.X) and index concatenation for train/validation
    splits
- name: anndata
  role: Underlying data structure (AnnData) storing gene counts and cell labels
- name: pip
  role: 'Installation mechanism: `pip install markermap`; editable install via `pip
    install -e .` for development'
  repo: https://pypi.org/project/markermap/
- name: persist
  role: Comparator marker-selection method for spatial transcriptomics; requires separate
    installation when benchmarked against MarkerMap
  repo: https://github.com/iancovert/persist/
- name: scVI
  role: Comparator generative model whose top k=50 markers are extracted to benchmark
    MarkerMap reconstruction quality (Jaccard, Spearman rho, l2) on Mouse Brain
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:a90d988480e906a3b6fbc9afc39b6b2fabadb3d545c7deb4cdd0e9fd66871947
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - anndata
  - numpy
  - persist
  - pip
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# variational-autoencoder-marker-selection

## Summary

Trains MarkerMap, a VAE-based generative model, to select a minimal set of k gene markers (e.g., k=50) whose latent representation supports cell-type identification and whole-transcriptome reconstruction from single-cell RNA-seq data.

## When to use

Use when scRNA-seq AnnData needs a small informative gene panel: supervised or unsupervised marker selection, cell-type classification from k markers, or full-transcriptome reconstruction quality comparison against baselines like scVI.

## When NOT to use

- Input data is not gene-expression counts from single-cell assays (the model is designed for scRNA-seq/scATAC-seq-like AnnData matrices, not feature tables or bulk metadata)
- You only need fast statistical marker ranking rather than a learned generative model — simpler methods like COSG or rank-correlation-based RankCorr may suffice
- The dataset is too small or labels are unreliable such that training a VAE would overfit; the paper benchmarks on curated datasets (CITE-seq, Mouse Brain, Paul, Zeisel) with outlier removal and normalization already applied

## Inputs

- scanpy AnnData object with gene counts in adata.X (dense numpy array; apply .toarray() if sparse) and cell labels in adata.obs['annotation'] (or adata.obs['names'], controlled by group_by)
- h5ad file (e.g., data/cite_seq/CITEseq.h5ad) or a dataset loader (get_citeseq, get_mouse_brain, get_paul, get_zeisel)
- Hyperparameters: z_size=16, hidden_layer_size, k (number of markers), batch_size=64, loss_tradeoff (0, 0.5, or 1)

## Outputs

- Selected k marker genes from the trained MarkerMap model (model.markers())
- Misclassification rate, weighted f1-score, and confusion matrix from new_model_metrics
- Reconstruction metrics (Jaccard Index, Spearman rho, l2 distance) per cell type and overall, e.g., written to reconstruction_metrics_mouse_brain.csv
- Benchmark results over a range of k values for plotting with plot_benchmarks

## How to apply

Load the dataset with a markermap loader (e.g., get_citeseq, get_mouse_brain, get_paul, get_zeisel) into a scanpy AnnData object, converting sparse adata.X to dense via adata.X.toarray(). Split cells into train/validation/test sets (e.g., 70%/10%/20%) with split_data, then build train_dataloader and val_dataloader with MarkerMap.prepareData. Instantiate MarkerMap with input_size, hidden_layer_size (~10% of input dimension, e.g., 64 for the 500-column CITE-seq data), z_size=16, num_classes, and k (e.g., 50); set loss_tradeoff to 0 for supervised-only, 0.5 for mixed, or 1 for unsupervised marker selection. Train with train_model on the dataloaders, then extract markers from the model and evaluate: classification via new_model_metrics (RandomForestClassifier on the k markers, reporting misclassification rate and weighted f1-score) or reconstruction quality via Jaccard Index, Spearman rho, and l2 distance per cell type. For benchmarking, use MarkerMap.getBenchmarker and the benchmark/plot_benchmarks functions over a range of k values, including a RandomBaseline control.

## Related tools

- **MarkerMap** (Core VAE-based generative model providing marker selection (supervised, mixed, unsupervised via loss_tradeoff), data preparation (prepareData), training (train_model), and benchmarking utilities (getBenchmarker, benchmark)) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **scanpy** (Reads h5ad data into an AnnData object used as the input to MarkerMap)
- **numpy** (Handles dense gene-count arrays (adata.X) and index concatenation for train/validation splits)
- **anndata** (Underlying data structure (AnnData) storing gene counts and cell labels)
- **pip** (Installation mechanism: `pip install markermap`; editable install via `pip install -e .` for development) — https://pypi.org/project/markermap/
- **persist** (Comparator marker-selection method for spatial transcriptomics; requires separate installation when benchmarked against MarkerMap) — https://github.com/iancovert/persist/
- **scVI** (Comparator generative model whose top k=50 markers are extracted to benchmark MarkerMap reconstruction quality (Jaccard, Spearman rho, l2) on Mouse Brain)

## Examples

```
from markermap.vae_models import MarkerMap, train_model; model = MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs['annotation'].unique()), 50, loss_tradeoff=1); train_model(model, train_dataloader, val_dataloader)
```

## Evaluation signals

- For unsupervised MarkerMap with k=50 on Mouse Brain, reconstruction metrics should show better l2 distance than scVI while scVI scores better on variance-based metrics (Jaccard Index, Spearman rho), matching Table 4 trends
- new_model_metrics should return a misclassification rate and weighted f1-score for a RandomForestClassifier trained on the k selected markers; low misclassification rate validates marker informativeness
- Selected markers should be exactly k in number and result from a converged train_model run (train/val dataloaders built from split_data indices)
- Benchmark over a range of k should include the RandomBaseline control; MarkerMap curves should dominate random marker selection
- Sanity check that adata.X is dense: if sparse, .toarray() conversion must precede model construction or MarkerMap will fail on the input shape

## Limitations

- On Google Colab, runtime restart may be required after install because scanpy installs a newer version of matplotlib than the default one
- Smashpy requires an older tensorflow (==2.5.0) causing installation conflicts; it is optional and only needed for that comparison
- Windows installation is listed as 'Coming soon'
- hidden_layer_size depends on the data — rule of thumb ~10% of input dimension (e.g., 64 for 500-gene CITE-seq), so parameters do not transfer blindly across datasets
- loss_tradeoff meaning changes the model class entirely (0=supervised, 0.5=mixed, 1=unsupervised); unsupervised MarkerMap may score worse on variance-based reconstruction metrics (Jaccard, Spearman) than scVI even when it wins on l2 distance

## Evidence

- [abstract] Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction.: "Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction."
- [readme] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [readme] hidden_layer_size is the dimension of the hidden layers in the auto-encoder... a good rule of thumb is ~10% of the dimension of the input data.: "This is dependent on the data, a good rule of thumb is ~10% of the dimension of the input data. For the CITE-seq data which has 500 columns, we will use 64"
- [readme] In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only).: "In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only)."
- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
- [methods] The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data.: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
