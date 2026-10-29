---
name: scrnaseq-batch-integration-mixing-evaluation
description: Use when two scRNA-seq datasets from different species (e.g., human and
  mouse primary motor cortex raw counts) must be integrated into one latent space
  and you need a mixing metric plus cross-species cell-type matching recall to judge
  embedding quality.
when_to_use_negative:
- Input is a single-species dataset or already-integrated feature table — this skill
  targets cross-species (human–mouse) count-matrix integration
- You need de novo gene matching across species — this skill deliberately retains
  the ProtBERT-based gene matching (calc_dist/embed_proteins) and only swaps the cell-embedding
  component
- The goal is to benchmark TACTiCS against Seurat or SAMap rather than to evaluate
  embedding mixing/scVI substitution
tools:
- name: TACTiCS
  role: 'Reference model and evaluation harness: trains/evaluates cell-type matching
    with the swapped scVI embeddings via the TACTiCS class in tactics.py; baseline
    matches all 23 shared human–mouse cell types at subclass resolution'
  repo: https://github.com/kbiharie/TACTiCS
- name: scvi-tools (scVI)
  role: Trains the latent cell embeddings on the combined human–mouse count matrix
    that replace TACTiCS' protein-sequence-derived cell embedding input
- name: PyTorch
  role: Required backend for both TACTiCS training/evaluation and the scVI model
- name: calc_dist / embed_proteins
  role: Generates the ProtBERT-based gene-similarity distance matrix kept unchanged
    while the cell-embedding component is replaced
  repo: https://github.com/kbiharie/TACTiCS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:6fe30bc82df7b8f9a4a8cb2519443be28b9b5c4a81b915783a57317903a60a04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  - calc_dist / embed_proteins
  - scvi-tools (scVI)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scrnaseq-batch-integration-mixing-evaluation

## Summary

Evaluates how well two species' (e.g., human–mouse motor cortex) scRNA-seq cells mix in a shared latent embedding, by training a comparison embedding (scVI via scvi-tools/PyTorch) against a reference model (TACTiCS protein-embedding cells) and reporting species-mixing scores alongside per-cell-type matching scores and label-transfer recall.

## When to use

Use when two scRNA-seq datasets from different species (e.g., human and mouse primary motor cortex raw counts) must be integrated into one latent space and you need a mixing metric plus cross-species cell-type matching recall to judge embedding quality.

## When NOT to use

- Input is a single-species dataset or already-integrated feature table — this skill targets cross-species (human–mouse) count-matrix integration
- You need de novo gene matching across species — this skill deliberately retains the ProtBERT-based gene matching (calc_dist/embed_proteins) and only swaps the cell-embedding component
- The goal is to benchmark TACTiCS against Seurat or SAMap rather than to evaluate embedding mixing/scVI substitution

## Inputs

- Preprocessed 50k-subset raw count data for human and mouse primary motor cortex (from Zenodo 10.5281/zenodo.7582460)
- Trained TACTiCS model with ProtBERT/gene-match-derived cell embeddings (baseline)
- Combined human–mouse count matrix for scVI training (scvi-tools, PyTorch)
- Gene-similarity distance matrix computed with calc_dist on ProtBERT embeddings from embed_proteins in genes.py

## Outputs

- Latent cell embeddings from scVI on the shared human–mouse space
- Per-cell-type matching scores and human→mouse label-transfer recall for the scVI-embedding TACTiCS run and the original protein-embedding TACTiCS
- Species-mixing score on the shared latent space for both embedding variants
- scvi_vs_tactiCS_metrics.csv assembling matching scores, recall and mixing values
- Saved preprocessed data, trained model and cell embeddings via TACTiCS.save

## How to apply

Download the preprocessed 50k-subset raw count data and trained TACTiCS model from Zenodo (10.5281/zenodo.7582460). Train a scVI model (scvi-tools, PyTorch backend) on the combined human–mouse count matrix to obtain latent cell embeddings that replace the protein-sequence embedding input of TACTiCS, while retaining the original ProtBERT-based gene matching via calc_dist in tactics.py/genes.py. Train and evaluate TACTiCS (TACTiCS class in tactics.py) with the scVI embeddings, propagating cell-type labels human→mouse via transfer learning, and compute per-cell-type matching scores and label-transfer recall for both the scVI-embedding run and the original protein-embedding TACTiCS. Compute a species-mixing score on the shared latent space for both models and assemble all metrics into scvi_vs_tactiCS_metrics.csv; save preprocessed data, trained model and cell embeddings with TACTiCS.save.

## Related tools

- **TACTiCS** (Reference model and evaluation harness: trains/evaluates cell-type matching with the swapped scVI embeddings via the TACTiCS class in tactics.py; baseline matches all 23 shared human–mouse cell types at subclass resolution) — https://github.com/kbiharie/TACTiCS
- **scvi-tools (scVI)** (Trains the latent cell embeddings on the combined human–mouse count matrix that replace TACTiCS' protein-sequence-derived cell embedding input)
- **PyTorch** (Required backend for both TACTiCS training/evaluation and the scVI model)
- **calc_dist / embed_proteins** (Generates the ProtBERT-based gene-similarity distance matrix kept unchanged while the cell-embedding component is replaced) — https://github.com/kbiharie/TACTiCS

## Examples

```
import scvi; scvi.model.SCVI.setup_anndata(adata_human_mouse_counts); vae = scvi.model.SCVI(adata_human_mouse_counts); vae.train(); latent = vae.get_latent_representation()
```

## Evaluation signals

- Baseline check: original TACTiCS (protein embeddings) must match all 23 cell types present in both human and mouse at subclass resolution before swapping embeddings
- scvi_vs_tactiCS_metrics.csv contains matching scores, recall and species-mixing values for BOTH the scVI-embedding and protein-embedding runs, enabling a direct head-to-head diff
- Gene-similarity distance matrix is unchanged (computed with calc_dist on ProtBERT embeddings), so any matching-score change is attributable to the cell-embedding swap, not gene matching
- Preprocessed data, trained model and cell embeddings round-trip successfully through TACTiCS.save
- Human→mouse label-transfer recall computed per cell type for the scVI run is comparable in schema to the original TACTiCS recall

## Limitations

- Mixing improvement from scVI embeddings does not automatically imply better cell-type matching: TACTiCS' ProtBERT-based gene matching outperformed BLAST, and the original model already matched all 23 shared subclass cell types, leaving little headroom
- Evaluation uses 50k subsets of the motor cortex data (full unprocessed data is Zenodo 10.5281/zenodo.6786357); conclusions may not transfer to full datasets or other tissues/species pairs
- TACTiCS requires PyTorch; the transformers package is only needed for protein-sequence embedding steps retained in the gene-matching branch

## Evidence

- [readme] TACTiCS class trains and evaluates with two datasets plus a distance matrix: "To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`. As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [methods] calc_dist computes gene-similarity matrix from ProtBERT embeddings: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [abstract] TACTiCS aligns cell types cross-species: "we present a model to transfer and align cell types in cross-species analysis (TACTiCS)"
- [readme] Results can be saved for reproducibility: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
- [readme] PyTorch is required for training/evaluation: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
