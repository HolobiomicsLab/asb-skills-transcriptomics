---
name: scrna-seq-cell-embedding-extraction
description: Use when trained cross-species cell-type classification models are available
  (e.g. TACTiCS from Zenodo) and you need per-cell embeddings from scRNA-seq data
  to assess cell-type alignment across species.
when_to_use_negative:
- When no trained model is available and cell types must be matched de novo — first
  train TACTiCS via the `TACTiCS` class on two scRNA-seq datasets plus the gene-similarity
  distance matrix.
- When raw unprocessed data is the only input — the full unprocessed dataset (Zenodo
  doi:10.5281/zenodo.6786357) must be preprocessed to the 50k-subset format before
  embeddings can be extracted.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: TACTiCS
  role: Provides the `TACTiCS` class in `tactics.py` used to load trained models and
    extract cell embeddings
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Deep-learning framework required to instantiate and run the TACTiCS model
    for embedding extraction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:2f00a1d3c1aeca9ab4a09e6783af591f9d2722c357d5e68e40b717b153658c2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scRNA-seq-cell-embedding-extraction

## Summary

Extract per-cell embeddings from trained TACTiCS neural network models (PyTorch) for cross-species scRNA-seq comparison, enabling downstream visualization of cell-type alignment, e.g. UMAP of human and mouse motor cortex cells.

## When to use

Use when trained cross-species cell-type classification models are available (e.g. TACTiCS from Zenodo) and you need per-cell embeddings from scRNA-seq data to assess cell-type alignment across species.

## When NOT to use

- When no trained model is available and cell types must be matched de novo — first train TACTiCS via the `TACTiCS` class on two scRNA-seq datasets plus the gene-similarity distance matrix.
- When raw unprocessed data is the only input — the full unprocessed dataset (Zenodo doi:10.5281/zenodo.6786357) must be preprocessed to the 50k-subset format before embeddings can be extracted.

## Inputs

- Trained TACTiCS model checkpoints from Zenodo doi:10.5281/zenodo.7582460
- Preprocessed 50k-subset scRNA-seq count matrices of primary motor cortex cells (human, mouse, marmoset)
- Gene similarity distance matrix computed with `calc_dist` from ProtBERT protein embeddings (`embed_proteins` in `genes.py`)

## Outputs

- Per-cell embedding vectors for each species in the TACTiCS latent space
- Saved bundle (preprocessed data + trained model + cell embeddings) via `TACTiCS.save`
- Downstream UMAP scatter plots colored by cell type and species (PNG) and per-cell-type coordinate tables (TSV)

## How to apply

Download the preprocessed 50k-subset scRNA-seq data and trained models from the Zenodo deposit (doi:10.5281/zenodo.7582460). Load each deposited trained model using the `TACTiCS` class in `tactics.py` (PyTorch is required). Run the loaded model over the preprocessed count matrices of each species to extract the cell embeddings, which the model produces after its transfer-learning label propagation between species. Save intermediate artifacts with `TACTiCS.save`, which stores the preprocessed data, trained model and resulting cell embeddings together. Feed the embeddings to dimensionality reduction (e.g. UMAP) and color by cell type and species to verify alignment.

## Related tools

- **TACTiCS** (Provides the `TACTiCS` class in `tactics.py` used to load trained models and extract cell embeddings) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Deep-learning framework required to instantiate and run the TACTiCS model for embedding extraction)

## Examples

```
from tactics import TACTiCS; model = TACTiCS.load('zenodo_7582460_model'); emb = model.transform(preprocessed_counts)
```

## Evaluation signals

- UMAP of extracted embeddings shows corresponding cell types (e.g. Vip, Sncg) forming partially overlapping or close clusters across human and mouse.
- Mouse-specific cell types Meis2, pericytes (Peri), and smooth muscle cells (SMC) cluster separately from human cells in the embedding space.
- Individual human and mouse cells do not fully mix in the UMAP — imperfect per-cell mixing is expected.
- `TACTiCS.save` round-trip: reloaded model reproduces the same cell embeddings for the same preprocessed input.

## Limitations

- Embeddings are only meaningful for the species/datasets the model was trained on (human, mouse, marmoset primary motor cortex 50k subsets).
- Individual cells of different species do not mix well in the embedding space; alignment is at the cell-type level, not single-cell level.
- The `transformers` package is only needed for embedding protein sequences — extraction from a saved model does not require re-embedding unless recomputing the distance matrix.

## Evidence

- [readme] TACTiCS class usage: "To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`."
- [readme] Save capability: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
- [readme] PyTorch requirement: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
- [abstract] Transfer learning step: "Afterward, TACTiCS uses transfer learning to propagate cell type labels between species"
- [methods] Zenodo deposit: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo](https://zenodo.org/doi/10.5281/zenodo.7582460)"
