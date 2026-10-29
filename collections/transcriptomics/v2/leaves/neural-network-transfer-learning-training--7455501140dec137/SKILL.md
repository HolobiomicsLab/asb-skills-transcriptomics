---
name: neural-network-transfer-learning-training
description: 'Use when two species'' scRNA-seq datasets must be matched at the cell
  type level: gene identities differ across species, so a cross-species gene similarity
  distance matrix (e.g. from ProtBERT protein embeddings) is available and a confusion-matrix-style
  label transfer is the goal.'
when_to_use_negative:
- The analysis is single-species cell type classification only — TACTiCS explicitly
  requires two scRNA-seq datasets and a cross-species gene similarity distance matrix
  as input.
- A gene-similarity matrix cannot be computed (no protein sequences or embeddings
  available for the genes in the two datasets), since the alignment loss depends on
  it.
- The user expects one-to-one ortholog gene mapping rather than embedding-based matching
  — the paper's finding is that ProtBERT-based gene matching, not BLAST, drives TACTiCS's
  cell type matches.
tools:
- name: TACTiCS
  role: 'Model being trained: TACTiCS class in tactics.py encapsulates the embedding
    layers, softmax classifiers, combined loss, and classifier-swap label transfer'
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Required backend for training the TACTiCS neural network
- name: transformers
  role: Used only to generate the protein sequence embeddings from which the gene
    similarity distance matrix is built
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:fcb78562004778928f38ca093f15ecb4bd2b20ecddf2620da71d4e0eb4c72cc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  - transformers
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# neural-network-transfer-learning-training

## Summary

Trains TACTiCS, a PyTorch model with two linear embedding layers and species-specific softmax classifiers, to match cell types across species from scRNA-seq data using protein-embedding-derived gene similarity as an alignment constraint. The trained classifiers are swapped to propagate cell type labels between species via transfer learning.

## When to use

Use when two species' scRNA-seq datasets must be matched at the cell type level: gene identities differ across species, so a cross-species gene similarity distance matrix (e.g. from ProtBERT protein embeddings) is available and a confusion-matrix-style label transfer is the goal.

## When NOT to use

- The analysis is single-species cell type classification only — TACTiCS explicitly requires two scRNA-seq datasets and a cross-species gene similarity distance matrix as input.
- A gene-similarity matrix cannot be computed (no protein sequences or embeddings available for the genes in the two datasets), since the alignment loss depends on it.
- The user expects one-to-one ortholog gene mapping rather than embedding-based matching — the paper's finding is that ProtBERT-based gene matching, not BLAST, drives TACTiCS's cell type matches.

## Inputs

- Two scRNA-seq datasets, one per species (e.g. preprocessed 50k-subset count data of human, mouse, or marmoset primary motor cortex)
- Gene similarity distance matrix computed by calc_dist from ProtBERT protein sequence embeddings (embed_proteins in genes.py)

## Outputs

- Trained TACTiCS model (linear embedding layers + species-specific softmax classifiers)
- Cell embeddings for both species
- Combined normalized confusion matrix of cross-species cell type matches (averaged over both transfer directions)
- Saved artifacts via TACTiCS.save (preprocessed data, model, embeddings)

## How to apply

Instantiate the TACTiCS class in tactics.py with the two scRNA-seq datasets and a gene similarity distance matrix; the distance matrix is computed by calc_dist from protein embeddings produced by embed_proteins in genes.py (transformers package). Train the model — two linear embedding layers plus a species-specific softmax classifier per species — with the combined classification + alignment + L2 loss under the Adam optimizer (learning rate 0.001, 200 epochs), using the PyTorch backend. After training, swap the species-specific classifiers so each species' cells are predicted under the other species' classifier, then aggregate single-cell predictions into normalized confusion matrices for both transfer directions and average them into a combined matrix in which high values indicate reciprocal cross-species cell type matches. Save the preprocessed data, trained model, and resulting cell embeddings with TACTiCS.save; the tutorial.ipynb notebook walks through the full two-species comparison.

## Related tools

- **TACTiCS** (Model being trained: TACTiCS class in tactics.py encapsulates the embedding layers, softmax classifiers, combined loss, and classifier-swap label transfer) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Required backend for training the TACTiCS neural network)
- **transformers** (Used only to generate the protein sequence embeddings from which the gene similarity distance matrix is built)

## Examples

```
from tactics import TACTiCS; model = TACTiCS(scrna_speciesA, scrna_speciesB, dist_matrix)  # then train and model.save(...)
```

## Evaluation signals

- TACTiCS.save completes without error, confirming preprocessed data, trained model, and cell embeddings were written.
- The combined normalized confusion matrix is averaged from both transfer directions, so high entries correspond to matches that are high in both directions (reciprocal matches).
- The match-score matrix reproduces cross-species cell type matches consistent with the paper's reported performance on the human, mouse, and marmoset motor cortex datasets.
- Benchmark comparison: TACTiCS's matches should outperform Seurat and SAMap on the same datasets.
- Ablation check: replacing the ProtBERT-based gene similarity with BLAST-based gene matching should degrade cell type matches.

## Limitations

- Training and evaluation in the paper use 50k-subset datasets distributed via Zenodo; the full unprocessed datasets come from a separate Zenodo record, so scaling behavior on larger data is not covered by the documented workflow.
- The transformers package is only needed for the protein-embedding step and is not required for training itself.
- The README defers details to function descriptions and tutorial.ipynb; no changelog is available, so API drift between versions cannot be verified from the provided context.

## Evidence

- [readme] To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`.: "To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`."
- [methods] As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities.: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [readme] TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences.: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
- [methods] The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`.: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
- [methods] The `tutorial.ipynb` notebook explains step-by-step how to use TACTiCS for a comparison across two species, including how to create protein sequence embeddings.: "The `tutorial.ipynb` notebook explains step-by-step how to use TACTiCS for a comparison across two species, including how to create protein sequence embeddings."
