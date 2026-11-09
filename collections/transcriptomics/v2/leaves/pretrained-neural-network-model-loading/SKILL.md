---
name: pretrained-neural-network-model-loading
description: Use when a trained TACTiCS PyTorch model must be loaded from the Zenodo
  deposit (10.5281/zenodo.7582460) to extract cell embeddings for motor cortex scRNA-seq
  cells without retraining.
when_to_use_negative:
- You need to train TACTiCS from scratch on a new species pair or dataset — follow
  the tutorial.ipynb training workflow (embed_proteins, calc_dist, `TACTiCS` class)
  instead of loading a deposited checkpoint.
- The task is creating protein-sequence embeddings or computing the gene similarity
  distance matrix (embed_proteins / calc_dist in genes.py) — model loading presupposes
  that matrix already exists.
- The input scRNA-seq data is not one of the deposited 50k motor cortex subsets the
  released models were trained on; checkpoints will not directly transfer to unrelated
  preprocessed datasets.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: TACTiCS
  role: Provides the `TACTiCS` class in `tactics.py` used to load the deposited trained
    models and extract cell embeddings; models and embeddings are saved with `TACTiCS.save`.
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Deep-learning framework required to instantiate the `TACTiCS` class and run/load
    the pretrained model weights.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:75c633c0441d90488b2ce77b32f0a567a025ec17a66fdcd57fbb75e010e285d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# pretrained-neural-network-model-loading

## Summary

Loads the pretrained TACTiCS neural-network checkpoints (the PyTorch `TACTiCS` class in `tactics.py`) from the Zenodo deposit to extract per-cell embeddings for human/mouse/marmoset primary motor cortex scRNA-seq data. This reproduces the paper's embedding space for downstream cell-type cross-species alignment without retraining the network.

## When to use

Use when a trained TACTiCS PyTorch model must be loaded from the Zenodo deposit (10.5281/zenodo.7582460) to extract cell embeddings for motor cortex scRNA-seq cells without retraining.

## When NOT to use

- You need to train TACTiCS from scratch on a new species pair or dataset — follow the tutorial.ipynb training workflow (embed_proteins, calc_dist, `TACTiCS` class) instead of loading a deposited checkpoint.
- The task is creating protein-sequence embeddings or computing the gene similarity distance matrix (embed_proteins / calc_dist in genes.py) — model loading presupposes that matrix already exists.
- The input scRNA-seq data is not one of the deposited 50k motor cortex subsets the released models were trained on; checkpoints will not directly transfer to unrelated preprocessed datasets.

## Inputs

- Trained TACTiCS model files from Zenodo deposit doi:10.5281/zenodo.7582460
- Preprocessed 50k-subset scRNA-seq count data (primary motor cortex of human, mouse, and marmoset)
- Gene similarity distance matrix (from calc_dist over protein-sequence embeddings)

## Outputs

- Per-cell embedding vectors in the TACTiCS embedding space, per species
- Persisted bundle of preprocessed data, trained model, and cell embeddings via TACTiCS.save
- Downstream 2D UMAP coordinates/scatter plots colored by cell type and species

## How to apply

Download the trained TACTiCS models and the preprocessed 50k-subset count data from Zenodo (doi:10.5281/zenodo.7582460) and set up a PyTorch environment; the transformers package is not needed unless you also embed protein sequences. Instantiate the `TACTiCS` class from `tactics.py`, whose inputs are two scRNA-seq datasets plus a gene similarity distance matrix (the matrix is computed by `calc_dist` from embeddings made with `embed_proteins` in `genes.py`), then load the deposited trained model instead of re-fitting it. Run the loaded model on each species' cells to extract cell embeddings, and call `TACTiCS.save` when you want the preprocessed data, trained model, and resulting cell embeddings persisted together. Confirm correct loading by reducing the extracted embeddings to 2D with UMAP: homologous cell types (e.g. Vip, Sncg) should partially overlap or sit close across human and mouse, while mouse-specific Meis2, pericytes, and SMC remain separated and individual cells of the two species do not fully mix. The `tutorial.ipynb` notebook documents step-by-step usage for a two-species comparison.

## Related tools

- **TACTiCS** (Provides the `TACTiCS` class in `tactics.py` used to load the deposited trained models and extract cell embeddings; models and embeddings are saved with `TACTiCS.save`.) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Deep-learning framework required to instantiate the `TACTiCS` class and run/load the pretrained model weights.)

## Examples

```
from tactics import TACTiCS  # PyTorch; inputs: two scRNA-seq datasets + gene distance matrix; persist with TACTiCS.save
```

## Evaluation signals

- The `TACTiCS` object instantiates from `tactics.py` under PyTorch and loads the deposited Zenodo checkpoint without re-fitting weights.
- Extracted embeddings cover the cells of the 50k subsets, and their UMAP shows homologous cell types (e.g. Vip, Sncg) partially overlapping or close across human and mouse.
- Mouse-specific Meis2, pericyte, and SMC clusters stay separated from human cells in the UMAP, while individual human and mouse cells do not fully mix.
- A `TACTiCS.save` round-trip reloads the preprocessed data, trained model, and cell embeddings and reproduces the same embeddings.
- The transformers dependency can be skipped entirely — loading trained models should not require the transformers package.

## Limitations

- Released trained models cover 50k subsets of the motor cortex datasets; the full unprocessed dataset (Zenodo doi:10.5281/zenodo.6786357) has no deposited trained model, so new embeddings for it require training.
- In the TACTiCS embedding space individual human and mouse cells do not mix well — only cell-type-level alignment holds, so single-cell mixing should not be expected as a success signal.
- Checkpoints are tied to their paired scRNA-seq datasets and gene similarity distance matrices; substituting different inputs requires retraining rather than reloading.

## Evidence

- [abstract] The preprocessed datasets and trained models can be downloaded from Zenodo (https://doi.org/10.5281/zenodo.7582460): "The preprocessed datasets and trained models can be downloaded from Zenodo (https://doi.org/10.5281/zenodo.7582460)"
- [methods] TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences.: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
- [methods] The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`.: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
- [readme] To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`. As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities.: "To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`. As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
