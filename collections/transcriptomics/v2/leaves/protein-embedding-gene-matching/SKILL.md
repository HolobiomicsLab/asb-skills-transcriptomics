---
name: protein-embedding-gene-matching
description: Use when preparing cross-species single-cell comparisons that need a
  gene similarity distance matrix derived from protein sequence embeddings rather
  than sequence alignment tools like BLAST.
when_to_use_negative:
- The analysis is a single-species scRNA-seq analysis with no need for cross-species
  gene orthology or a gene similarity distance matrix
- A downstream method other than TACTiCS requires BLAST-style alignment scores instead
  of embedding-based distances, and the comparison target explicitly benchmarks BLAST-based
  gene matching
- Precomputed protein embeddings for the species pair are already available in the
  Zenodo deposit, making regeneration redundant (though reuse is preferred over recompute)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: TACTiCS
  role: Downstream consumer of the gene similarity distance matrix; its TACTiCS class
    requires two scRNA-seq datasets and the distance matrix as input
  repo: https://github.com/kbiharie/TACTiCS
- name: transformers
  role: Used only for embedding protein sequences via embed_proteins in genes.py
- name: BLAST
  role: Alternative sequence-alignment-based gene matching baseline; the embedding
    approach outperforms BLAST in cell type matching within TACTiCS
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:5825e52daf73e52a40bfb59817b18a883c66ec68caf9193f6f2a0a66246ed0ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - PyTorch
  - SAMap
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# protein-embedding-gene-matching

## Summary

Match genes across two species by embedding their protein sequences with a natural language processing model (transformers-based, e.g. ProtBERT) and computing a gene similarity distance matrix, which TACTiCS uses to align cross-species cell types. This NLP-based gene matching yields better cell type matches than BLAST within TACTiCS.

## When to use

Use when preparing cross-species single-cell comparisons that need a gene similarity distance matrix derived from protein sequence embeddings rather than sequence alignment tools like BLAST.

## When NOT to use

- The analysis is a single-species scRNA-seq analysis with no need for cross-species gene orthology or a gene similarity distance matrix
- A downstream method other than TACTiCS requires BLAST-style alignment scores instead of embedding-based distances, and the comparison target explicitly benchmarks BLAST-based gene matching
- Precomputed protein embeddings for the species pair are already available in the Zenodo deposit, making regeneration redundant (though reuse is preferred over recompute)

## Inputs

- Protein sequences for the genes of each species (FASTA-like protein sequence records)
- Two scRNA-seq datasets (one per species) that the resulting distance matrix will support, e.g. preprocessed 50k-subset count data from Zenodo doi:10.5281/zenodo.7582460

## Outputs

- Protein sequence embeddings (from embed_proteins in genes.py)
- Gene similarity distance matrix across the two species (from calc_dist), consumed by the TACTiCS class in tactics.py

## How to apply

Use embed_proteins in genes.py (transformers package) to generate protein sequence embeddings for the genes of each species, e.g. ProtBERT-derived embeddings as used by TACTiCS. Then call calc_dist to compute the gene similarity distance matrix from these embeddings; this distance matrix is a required TACTiCS input alongside the two scRNA-seq datasets. The tutorial.ipynb notebook explains step-by-step how to create the protein sequence embeddings and run a two-species comparison. Protein sequences, precomputed protein embeddings, and trained models for 50k subsets of the human, mouse, and marmoset motor cortex data are available on Zenodo (doi:10.5281/zenodo.7582460), so embeddings can be downloaded instead of regenerated. Rationale: the paper shows that this NLP-based gene matching produces better cell type matches in TACTiCS than BLAST-based gene matching.

## Related tools

- **TACTiCS** (Downstream consumer of the gene similarity distance matrix; its TACTiCS class requires two scRNA-seq datasets and the distance matrix as input) — https://github.com/kbiharie/TACTiCS
- **transformers** (Used only for embedding protein sequences via embed_proteins in genes.py)
- **BLAST** (Alternative sequence-alignment-based gene matching baseline; the embedding approach outperforms BLAST in cell type matching within TACTiCS)

## Examples

```
from genes import embed_proteins; from tactics import calc_dist; embeddings = embed_proteins(protein_sequences); dist_matrix = calc_dist(embeddings)
```

## Evaluation signals

- calc_dist runs without error on embeddings produced by embed_proteins and returns a gene-by-gene distance matrix acceptable to the TACTiCS class
- Using the resulting distance matrix, TACTiCS reproduces cross-species subclass matches consistent with the paper (e.g. correct match for all 23 shared human–mouse cell types; human VLMC–mouse VLMC score 1.0)
- Cell type matching quality with embedding-based gene distances exceeds the BLAST-based gene matching baseline within TACTiCS
- Embeddings regenerated locally match the deposited protein embeddings on Zenodo (doi:10.5281/zenodo.7582460) for the same protein sequences
- The transformers package is installed only when embeddings must be created; if embeddings are downloaded, the pipeline still yields a valid distance matrix

## Limitations

- The transformers package is only necessary for embedding protein sequences; if it is unavailable, embeddings must be obtained from the Zenodo deposit instead
- Performance drops when TACTiCS is trained on equal-sized 50k-cells-per-species subsets relative to the full datasets, indicating embedding/gene-matching quality alone does not guarantee full performance at reduced dataset sizes
- Validation of gene matching quality is indirect: it is judged via downstream cross-species cell type match scores rather than a direct gene orthology metric

## Evidence

- [abstract] First, TACTiCS uses a natural language processing model to match genes using their protein sequences: "First, TACTiCS uses a natural language processing model to match genes using their protein sequences"
- [abstract] ProtBERT-based gene matching outperforms BLAST: "our gene matching method results in better cell type matches than BLAST in our model"
- [methods] calc_dist computes distance matrix from embed_proteins embeddings: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [methods] TACTiCS input requirements: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [readme] transformers only needed for protein embedding: "The transformers package is only necessary for embedding protein sequences."
- [readme] Protein sequences and embeddings available on Zenodo: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo](https://zenodo.org/doi/10.5281/zenodo.7582460)"
