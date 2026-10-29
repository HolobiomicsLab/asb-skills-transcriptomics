---
name: single-cell-multimodal-embedding-analysis
description: Use when you have paired CITE-seq data (RNA gene counts CSV + raw protein
  ADT counts CSV, rows = cells, columns = genes/proteins) and need joint cell embeddings
  or want to quantify complementary, modality-specific information across cells.
when_to_use_negative:
- Input is a single-modality dataset (RNA-only or protein-only) with no paired ADT
  counts — scME requires both modalities
- The goal is a fused/consensus clustering only and modality-specific complementary
  information should be discarded — the article notes most integration methods do
  this instead
- Data is not paired CITE-seq RNA/protein counts in the CSV layout scME expects (rows
  = cells, columns = genes/proteins, cell ids in column 0)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scME
  role: Dual-modality factor model that builds and infers joint cell embeddings from
    RNA and protein count matrices
  repo: https://github.com/bucky527/scME
- name: scanpy
  role: Leiden clustering independently on RNA-modality and protein-modality representations
  repo: https://github.com/bucky527/scME
- name: pandas
  role: Loading RNA and protein CSV count matrices and enumerating cell pairs
- name: numpy
  role: Comparing each cell pair's cluster relationship across modalities to label
    shared vs complementary
- name: anndata
  role: Dependency supporting single-cell data handling with scanpy clustering
- name: PyTorch
  role: Deep factor modeling backend for training the scME model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad337
    title: 'scME: a dual-modality factor model for single-cell multiomics embedding'
  source_content_hash: sha256:63188677f32ac6be5c67280f9252cff3fd7ae01835f7256f320ff8e70e8fd817
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bucky527/scME
  tools:
  - PyTorch
  - anndata
  - numpy
  - pandas
  - scME
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-multimodal-embedding-analysis

## Summary

Generate joint single-cell multiomics embeddings with scME, a dual-modality factor model that combines RNA gene counts and raw protein ADT counts from CITE-seq data, then use the embeddings to quantify shared vs complementary (modality-inconsistent) information across cell pairs.

## When to use

Use when you have paired CITE-seq data (RNA gene counts CSV + raw protein ADT counts CSV, rows = cells, columns = genes/proteins) and need joint cell embeddings or want to quantify complementary, modality-specific information across cells.

## When NOT to use

- Input is a single-modality dataset (RNA-only or protein-only) with no paired ADT counts — scME requires both modalities
- The goal is a fused/consensus clustering only and modality-specific complementary information should be discarded — the article notes most integration methods do this instead
- Data is not paired CITE-seq RNA/protein counts in the CSV layout scME expects (rows = cells, columns = genes/proteins, cell ids in column 0)

## Inputs

- RNA gene counts matrix in CSV format (rows = cells, columns = genes, column 0 = cell ids)
- Raw protein ADT counts matrix in CSV format (rows = cells, columns = proteins, column 0 = cell ids)

## Outputs

- Joint cell embedding matrix zm from scme.inference (plus modality-specific embeddings)
- Trained scME dual-modality factor model
- Per-cell-pair shared/complementary labels and complementary-pair fraction (e.g., complementary_pairs.csv with counts and the ~10.5% fraction for BMNC)

## How to apply

Load the RNA gene counts and raw protein ADT counts CSV matrices with pandas, ensuring column 0 holds the cell ids as scME requires. Build and train the scME model via build_scme (e.g., rna_latent_dim=24, protein_latent_dim=20, latent_dim=32, protein_dist='NB' or 'MNB'), then train with train_model (max_epochs=200, batchsize=256, lr=1e-4, lr_cla=1e-4, milestones=[80]) and infer the joint embedding with zm=scme.inference(rna_data, protein_data). For the shared-vs-complementary quantification (task_004), run Leiden clustering independently on the RNA-modality and protein-modality representations with scanpy, enumerate all cell pairs (470,370,456 pairs in the BMNC dataset), and label each pair as shared (consistent cluster relationship) or complementary (inconsistent relationship); the BMNC analysis found ~10.5% complementary pairs. Rationale: most integration methods discard modality-specific complementary information, while scME's dual-modality factor model preserves it for clustering and classification.

## Related tools

- **scME** (Dual-modality factor model that builds and infers joint cell embeddings from RNA and protein count matrices) — https://github.com/bucky527/scME
- **scanpy** (Leiden clustering independently on RNA-modality and protein-modality representations) — https://github.com/bucky527/scME
- **pandas** (Loading RNA and protein CSV count matrices and enumerating cell pairs)
- **numpy** (Comparing each cell pair's cluster relationship across modalities to label shared vs complementary)
- **anndata** (Dependency supporting single-cell data handling with scanpy clustering)
- **PyTorch** (Deep factor modeling backend for training the scME model)

## Examples

```
zm = scme.inference(rna_data, protein_data)
```

## Evaluation signals

- Total enumerated cell pairs equal N*(N-1)/2 for N cells (470,370,456 pairs for the BMNC dataset)
- Complementary-pair fraction reproduces ~10.5% on BMNC when Leiden clustering is run independently on RNA and protein embeddings
- Embedding inference returns zm from scme.inference(rna_data, protein_data) with the configured latent_dim=32 (rna_latent_dim=24, protein_latent_dim=20)
- Output CSV contains shared/complementary pair counts plus the complementary fraction, with cell ids correctly read from column 0 of the input CSVs
- Reproduced joint embedding yields clustering/classification consistent with scME's reported improvement over other single-cell multiomics integration algorithms

## Limitations

- Input format is constrained: CSV files where rows are cells and columns are genes/proteins, with column 0 the cell ids
- Protein data distribution must be chosen ('NB' or 'MNB') via the use-mnb flag / protein_dist argument; the wrong choice can misfit ADT counts
- Most integration methods (and naive joint analyses) discard modality-specific complementary information, so fused-only embeddings cannot capture the ~10.5% complementary pairs
- Training hyperparameters (max_epochs, batch size, learning rate, lr step milestone) affect the embedding and downstream pair quantification

## Evidence

- [methods] scME accepts RNA and protein CSV inputs: "scME accepts as input the  RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format"
- [methods] Inference call signature: "#Inference cell embedding

zm=scme.inference(rna_data,protein_data)"
- [readme] CSV layout requirement: "rows are cells and columns are genes, and the columns 0 in csv file should be the cells ids."
- [abstract] Complementary information discarded by other methods: "Most single-cell multiomics integration methods focus on shared information among modalities while complementary information specific to each modality is often discarded"
- [abstract] Joint representation improves clustering: "the joint representation of multiple modalities yielded by scME can provide salient information to improve both single-cell clustering and cell-type classification"
- [readme] CLI usage for CITE-seq embeddings: "python scme.py --rna [your rna gene counts csv file path] --protein [your protein ADTs counts csv file path] --output-dir [result save path] --batch-size 256"
