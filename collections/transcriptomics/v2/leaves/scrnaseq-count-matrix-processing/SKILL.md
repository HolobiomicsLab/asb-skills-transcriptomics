---
name: scrnaseq-count-matrix-processing
description: Use when two cross-species scRNA-seq datasets (count matrices) must be
  preprocessed and paired with a gene similarity distance matrix as input to a cell
  type matching model such as TACTiCS.
when_to_use_negative:
- When only a single-species scRNA-seq dataset is available and no cross-species gene
  matching is needed
- When the gene similarity distance matrix is unavailable and protein sequences/embeddings
  cannot be computed with embed_proteins
- When the goal is generic cell clustering or annotation within one species rather
  than cross-species cell type transfer
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TACTiCS
  role: Core class in tactics.py used to load the two scRNA-seq datasets and distance
    matrix, train/evaluate, and save preprocessed data, model, and cell embeddings
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Backend required by TACTiCS for training and inference
- name: transformers
  role: Package used only for embedding protein sequences when creating the gene similarity
    inputs (embed_proteins in genes.py)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:a77612866d222e2ea98fa12d83971af2edfe5002e5d9f109f7dcb46d7fc1783d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# scrnaseq-count-matrix-processing

## Summary

Load and preprocess scRNA-seq count matrices (e.g., 50k-subset Allen Brain M1 human/mouse data) together with ProtBERT-derived gene similarity distance matrices into the TACTiCS class, enabling cross-species cell type matching via within-species classification and transfer-learning label propagation.

## When to use

Use when two cross-species scRNA-seq datasets (count matrices) must be preprocessed and paired with a gene similarity distance matrix as input to a cell type matching model such as TACTiCS.

## When NOT to use

- When only a single-species scRNA-seq dataset is available and no cross-species gene matching is needed
- When the gene similarity distance matrix is unavailable and protein sequences/embeddings cannot be computed with embed_proteins
- When the goal is generic cell clustering or annotation within one species rather than cross-species cell type transfer

## Inputs

- Two scRNA-seq count matrices (e.g., 50k-subset Allen Brain M1 human and mouse data)
- Gene similarity distance matrix describing gene similarities
- Protein sequence embeddings (e.g., ProtBERT-derived) for computing the distance matrix
- Pretrained TACTiCS models (optional, from Zenodo 10.5281/zenodo.7582460)

## Outputs

- Preprocessed scRNA-seq datasets loaded into a TACTiCS object
- Per-cell-type matching score matrix between mouse and human subclasses (e.g., subclass_matching_results.csv)
- Saved preprocessed data, trained model, and cell embeddings via TACTiCS.save

## How to apply

Download the preprocessed 50k-subset scRNA-seq count data, protein embeddings, and pretrained TACTiCS models from Zenodo (10.5281/zenodo.7582460); the full unprocessed dataset is at 10.5281/zenodo.6786357. Load the two scRNA-seq datasets together with the gene similarity distance matrix using the TACTiCS class in tactics.py (PyTorch backend). If not precomputed, create protein sequence embeddings with embed_proteins in genes.py (requiring the transformers package) and compute the distance matrix with calc_dist. Restore pretrained models and run within-species subclass classification followed by cross-species transfer-learning label propagation on the Allen Brain M1 data, then persist preprocessed data, trained model and cell embeddings via TACTiCS.save.

## Related tools

- **TACTiCS** (Core class in tactics.py used to load the two scRNA-seq datasets and distance matrix, train/evaluate, and save preprocessed data, model, and cell embeddings) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Backend required by TACTiCS for training and inference)
- **transformers** (Package used only for embedding protein sequences when creating the gene similarity inputs (embed_proteins in genes.py))

## Examples

```
from tactics import TACTiCS; model = TACTiCS(adata_human, adata_mouse, gene_dist_matrix, backend='pytorch')
```

## Evaluation signals

- Within-species subclass classification recall reaches 1.0 on the Allen Brain M1 data
- At subclass resolution, the matching score correctly identifies the orthologous cell type for all 23 cell types shared between human and mouse
- Human VLMC–mouse VLMC matching score equals 1.0 and mouse Peri–human VLMC score is 0.5
- Species-specific (non-shared) pair scores fall between 0.04 and 0.5
- Cross-species-resolution recall of 0.96 is reproduced; TACTiCS.save round-trips preprocessed data, model, and cell embeddings

## Limitations

- Pretrained models and preprocessed inputs are provided only for 50k subsets of the data; the full unprocessed dataset must be fetched separately from Zenodo 10.5281/zenodo.6786357
- The transformers package is only necessary for embedding protein sequences, so it is an extra dependency when the distance matrix must be recomputed
- No changelog found in the repository discussion, so version-specific behavior changes cannot be traced

## Evidence

- [methods] TACTiCS class requires two scRNA-seq datasets and a distance matrix: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [readme] Save preprocessed data, trained model and cell embeddings: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
- [readme] PyTorch requirement and transformers only for protein embeddings: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
- [readme] 50k subsets data and trained models on Zenodo: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo]"
- [methods] calc_dist computes the distance matrix from embed_proteins embeddings: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
