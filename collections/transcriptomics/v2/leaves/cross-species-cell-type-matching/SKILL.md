---
name: cross-species-cell-type-matching
description: Use when matching/aligning cell types between two species from scRNA-seq
  count data (e.g., human–mouse, human–marmoset, mouse–marmoset cortex), especially
  when homologous gene lists are incomplete and protein-sequence-based gene similarity
  is needed.
when_to_use_negative:
- The goal is single-cell (per-cell) species integration rather than cell-type-level
  matching — TACTiCS embeddings do not mix individual cells of the two species well;
  alignment is at the cell-type level
- Only a standard ortholog table (BLAST one-to-one matches) is desired as the gene-matching
  basis — the paper shows ProtBERT-based gene matching yields better cell type matches
  than BLAST within TACTiCS
- Datasets are small or heavily unbalanced — on equal-sized 50k subsets TACTiCS performance
  drops relative to the full datasets
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TACTiCS
  role: 'Core model: TACTiCS class in tactics.py trains species-specific classifiers
    and performs cross-species transfer-learning label propagation and matching'
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Backend required to train and run the TACTiCS neural network
- name: transformers
  role: Used only for embedding protein sequences (embed_proteins in genes.py) to
    build the ProtBERT-based gene similarity matrix
- name: SAMap
  role: Baseline method for benchmarking cross-species cell type matching (ADS and
    recall comparisons)
- name: Seurat
  role: Baseline method using BLAST one-to-one gene matches for benchmarking ADS and
    recall
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  - task_006
  - task_008
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:e369cafaa1226776436bf25b8dcec2cd8a225426366f76374ca66cabfe623aa5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - SAMap
  - Seurat
  - TACTiCS
  - transformers
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-species-cell-type-matching

## Summary

TACTiCS matches cell types across species (human, mouse, marmoset primary motor cortex) by combining ProtBERT-based protein-sequence gene matching, neural-network cell-type classification within a species, and transfer-learning label propagation between species. It outperforms Seurat and SAMap on these datasets and avoids reliance on one-to-one ortholog tables such as BLAST.

## When to use

Use when matching/aligning cell types between two species from scRNA-seq count data (e.g., human–mouse, human–marmoset, mouse–marmoset cortex), especially when homologous gene lists are incomplete and protein-sequence-based gene similarity is needed.

## When NOT to use

- The goal is single-cell (per-cell) species integration rather than cell-type-level matching — TACTiCS embeddings do not mix individual cells of the two species well; alignment is at the cell-type level
- Only a standard ortholog table (BLAST one-to-one matches) is desired as the gene-matching basis — the paper shows ProtBERT-based gene matching yields better cell type matches than BLAST within TACTiCS
- Datasets are small or heavily unbalanced — on equal-sized 50k subsets TACTiCS performance drops relative to the full datasets

## Inputs

- Two species-specific scRNA-seq count datasets (raw counts; 50k-cell subsets for human, mouse, marmoset primary motor cortex)
- Gene similarity distance matrix computed by calc_dist from ProtBERT protein-sequence embeddings (embed_proteins in genes.py)
- Optional: pretrained TACTiCS models and deposited protein embeddings from Zenodo (10.5281/zenodo.7582460)

## Outputs

- Combined normalized confusion matrix of cross-species cell-type match scores (high values = reciprocal matches)
- Per-cell-type matching scores between species (e.g., subclass_matching_results.csv)
- Trained TACTiCS model and cell embeddings persisted via TACTiCS.save
- ADS and recall metrics for benchmarking against SAMap/Seurat

## How to apply

Download the preprocessed 50k-subset raw count data, protein embeddings and pretrained models from Zenodo (10.5281/zenodo.7582460) or compute protein embeddings with embed_proteins in genes.py (transformers package) and derive the gene-similarity distance matrix with calc_dist. Instantiate the TACTiCS class in tactics.py (PyTorch backend) with the two scRNA-seq datasets and the distance matrix; train the two linear embedding layers and species-specific softmax classifiers using the combined classification + alignment + L2 loss with Adam (lr 0.001, 200 epochs). Propagate cell-type labels between species via transfer learning, swap the species-specific classifiers to form normalized confusion matrices for both transfer directions, and average them into a combined matrix where high values indicate reciprocal cross-species matches. Save preprocessed data, trained model and cell embeddings with TACTiCS.save.

## Related tools

- **TACTiCS** (Core model: TACTiCS class in tactics.py trains species-specific classifiers and performs cross-species transfer-learning label propagation and matching) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Backend required to train and run the TACTiCS neural network)
- **transformers** (Used only for embedding protein sequences (embed_proteins in genes.py) to build the ProtBERT-based gene similarity matrix)
- **SAMap** (Baseline method for benchmarking cross-species cell type matching (ADS and recall comparisons))
- **Seurat** (Baseline method using BLAST one-to-one gene matches for benchmarking ADS and recall)

## Examples

```
from tactics import TACTiCS; model = TACTiCS(scrna_species1, scrna_species2, gene_dist_matrix)
```

## Evaluation signals

- At subclass resolution, TACTiCS correctly matches all 23 cell types shared between human and mouse (recall 1.0)
- Within-species classification recall reaches 1.0; cross-species-resolution recall ≈ 0.96
- Species-specific pairs (e.g., mouse Peri–human VLMC 0.5) score ≤ 0.5 while reciprocal pairs (human VLMC–mouse VLMC 1.0) score 1.0; off-target pair scores fall between 0.04 and 0.5
- Reported ADS and recall exceed SAMap and Seurat at subclass resolution across all three species pairs (human–mouse, human–marmoset, mouse–marmoset)
- TACTiCS.save completes successfully, persisting preprocessed data, trained model and cell embeddings

## Limitations

- On equal-sized 50k cells-per-species subsets, TACTiCS performance drops relative to the full datasets (though it still outperforms SAMap at the higher cross-species resolution in all three comparisons)
- Replacing ProtBERT gene matching with BLAST matches substantially decreases TACTiCS ADS and recall at the cross-species resolution
- Individual human and mouse cells do not mix well in the TACTiCS embedding space; alignment holds at the cell-type level, and mouse-specific types (Meis2, Peri, SMC) remain separated
- The transformers package is only necessary for protein embedding and can be skipped when using deposited embeddings

## Evidence

- [abstract] TACTiCS overview: NLP gene matching then neural-network classification then transfer learning: "First, TACTiCS uses a natural language processing model to match genes using their protein sequences"
- [methods] TACTiCS input requirements: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [methods] Saving outputs: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
- [abstract] ProtBERT gene matching beats BLAST: "our gene matching method results in better cell type matches than BLAST in our model"
- [readme] Dependencies for TACTiCS: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
- [readme] Zenodo deposit with preprocessed 50k subsets and trained models: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo](https://zenodo.org/doi/10.5281/zenodo.7582460)"
