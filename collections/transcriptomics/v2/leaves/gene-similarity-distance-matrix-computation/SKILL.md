---
name: gene-similarity-distance-matrix-computation
description: 'Use when preparing the gene-similarity input required by the TACTiCS
  class in tactics.py: you have protein sequences for the genes of two species'' scRNA-seq
  datasets and need the distance matrix describing gene similarities.'
when_to_use_negative:
- You only need within-species cell type classification with no cross-species gene
  matching — the TACTiCS neural network alone suffices without the gene similarity
  matrix.
- A gene distance matrix has already been computed and saved — pass it directly to
  the TACTiCS class instead of recomputing embeddings with embed_proteins.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3308
tools:
- name: transformers
  role: Used to embed protein sequences (ProtBERT-based) before distance calculation
  repo: https://github.com/kbiharie/TACTiCS
- name: TACTiCS
  role: Consumes the gene similarity distance matrix as input alongside two scRNA-seq
    datasets (calc_dist and embed_proteins live in genes.py of this repo)
  repo: https://github.com/kbiharie/TACTiCS
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:c5a85669410bd35def3df6c1a52731693f2754adc35a2d6b61a78474bbe00c84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-similarity-distance-matrix-computation

## Summary

Compute a cross-species gene similarity distance matrix from protein sequence embeddings (ProtBERT via the transformers package) using `calc_dist` on outputs of `embed_proteins` in `genes.py`; this matrix is a required input to the TACTiCS class for cross-species cell type matching.

## When to use

Use when preparing the gene-similarity input required by the TACTiCS class in tactics.py: you have protein sequences for the genes of two species' scRNA-seq datasets and need the distance matrix describing gene similarities.

## When NOT to use

- You only need within-species cell type classification with no cross-species gene matching — the TACTiCS neural network alone suffices without the gene similarity matrix.
- A gene distance matrix has already been computed and saved — pass it directly to the TACTiCS class instead of recomputing embeddings with embed_proteins.

## Inputs

- Protein sequences of genes from the two species' scRNA-seq datasets (e.g., human and mouse motor cortex)
- Protein sequence embeddings created with embed_proteins in genes.py (or downloaded precomputed embeddings from Zenodo doi:10.5281/zenodo.7582460)

## Outputs

- Distance matrix describing gene similarities (required input for the TACTiCS class in tactics.py)

## How to apply

First generate protein sequence embeddings with `embed_proteins` in `genes.py`, which requires the transformers package (ProtBERT-based NLP model); this gene matching approach yields better cell type matches than BLAST within TACTiCS. Then call `calc_dist`, also in `genes.py`, on these embeddings to obtain the gene similarity distance matrix. Supply this matrix, together with the two scRNA-seq datasets, as input to the `TACTiCS` class in `tactics.py`. Protein sequences and precomputed embeddings for 50k subsets are available on Zenodo (doi:10.5281/zenodo.7582460) if you want to skip the embedding step.

## Related tools

- **transformers** (Used to embed protein sequences (ProtBERT-based) before distance calculation) — https://github.com/kbiharie/TACTiCS
- **TACTiCS** (Consumes the gene similarity distance matrix as input alongside two scRNA-seq datasets (calc_dist and embed_proteins live in genes.py of this repo)) — https://github.com/kbiharie/TACTiCS

## Examples

```
from genes import embed_proteins, calc_dist; embeddings = embed_proteins(protein_sequences); gene_dist_matrix = calc_dist(embeddings)
```

## Evaluation signals

- calc_dist runs without error on embeddings produced by embed_proteins and returns a symmetric square matrix of gene similarity distances.
- The resulting distance matrix has the correct dimensions to pair with the gene sets of the two scRNA-seq datasets supplied to the TACTiCS class.
- Downstream TACTiCS training with this matrix reproduces accurate cross-species cell type matches (e.g., all 23 human–mouse motor cortex subclass matches), consistent with the ProtBERT-based gene matching outperforming BLAST within TACTiCS.

## Limitations

- Protein embeddings must be computed separately from model training: the transformers package is only needed for embedding protein sequences, not for running TACTiCS itself.
- Only two species can be compared per TACTiCS run, since it requires exactly two scRNA-seq datasets and one distance matrix.

## Evidence

- [methods] calc_dist computes gene similarity distance matrix: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [readme] transformers needed for protein embeddings: "The transformers package is only necessary for embedding protein sequences."
- [abstract] ProtBERT gene matching beats BLAST: "our gene matching method results in better cell type matches than BLAST in our model"
- [readme] distance matrix is required TACTiCS input: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
