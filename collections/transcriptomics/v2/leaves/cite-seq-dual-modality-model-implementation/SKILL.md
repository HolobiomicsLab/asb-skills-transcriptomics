---
name: cite-seq-dual-modality-model-implementation
description: Use when you have paired CITE-seq RNA gene counts and raw protein ADT
  counts (CSV, rows=cells) and need a joint cell embedding that retains modality-specific
  complementary information, not just shared structure.
when_to_use_negative:
- Data are unpaired single-cell multiomics measurements — scME requires paired RNA
  and protein (CITE-seq) counts for the same cells
- Only one modality is available (RNA-only or protein-only); the dual-modality factor
  model needs both matrices
- Input is already an embedded/normalized feature table rather than raw gene and ADT
  counts in CSV format
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scME
  role: 'Dual-modality factor model package: build_scme, train_model, and scme.inference
    produce the shared cell embedding zm'
  repo: https://github.com/bucky527/scME
- name: PyTorch
  role: Backend for the variational factor model training and inference
  repo: https://pytorch.org/get-started/locally/
- name: pyro-ppl
  role: Provides negative-binomial distributions for the probabilistic model's likelihoods
- name: pandas
  role: Loads the paired RNA gene counts and raw protein ADT counts CSV matrices
- name: scanpy
  role: Leiden clustering per modality to seed the categorical complementary latents
    YR and YP
- name: anndata
  role: Single-cell data structures in the scME environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad337
    title: 'scME: a dual-modality factor model for single-cell multiomics embedding'
  source_content_hash: sha256:f9ad3abcb6c283937edb6856a1704d0554ce3b9a7fe6dc7898d71d43b7ac1446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bucky527/scME
  tools:
  - PyTorch
  - anndata
  - numpy
  - pandas
  - pyro-ppl
  - scME
  - scanpy
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 20
---

# cite-seq-dual-modality-model-implementation

## Summary

Implements scME, a deep dual-modality factor model for CITE-seq single-cell multiomics that learns a shared latent embedding ZM alongside modality-specific complementary latents YR (RNA) and YP (protein), producing joint cell embeddings that improve clustering and cell-type classification.

## When to use

Use when you have paired CITE-seq RNA gene counts and raw protein ADT counts (CSV, rows=cells) and need a joint cell embedding that retains modality-specific complementary information, not just shared structure.

## When NOT to use

- Data are unpaired single-cell multiomics measurements — scME requires paired RNA and protein (CITE-seq) counts for the same cells
- Only one modality is available (RNA-only or protein-only); the dual-modality factor model needs both matrices
- Input is already an embedded/normalized feature table rather than raw gene and ADT counts in CSV format

## Inputs

- RNA gene counts matrix (.csv, rows=cells, columns=genes)
- Raw protein ADT counts matrix (.csv, rows=cells, columns=ADTs, column 0 = cell IDs)

## Outputs

- Shared cell embedding zm (one latent vector of dimension latent_dim=32 per input cell) saved to the output directory

## How to apply

Load the RNA gene counts matrix and raw protein ADT counts matrix from CSV (rows are cells, columns are genes/ADTs, column 0 holds cell IDs) with pandas, then optionally cluster each modality with scanpy Leiden to seed the categorical complementary latents YR and YP. Build the model with build_scme specifying rna_latent_dim=24, protein_latent_dim=20, latent_dim=32, and protein_dist='NB' (or 'MNB' for mixture negative binomial protein data). Train via stochastic variational inference in PyTorch with pyro-ppl using train_model with max_epochs=200, batchsize=256, lr=1e-4, lr_cla=1e-4, and lr decay milestones=[80]; the total loss is L = ELBO + BCE(YR) + BCE(YP), so complementary information is preserved rather than discarded. Finally call scme.inference(rna_data, protein_data) to obtain the shared cell embedding zm, one latent vector per input cell. The key design decision is using explicit auxiliary BCE losses on the modality-specific latents so the joint representation captures nuanced differences among cells that shared-only methods lose.

## Related tools

- **scME** (Dual-modality factor model package: build_scme, train_model, and scme.inference produce the shared cell embedding zm) — https://github.com/bucky527/scME
- **PyTorch** (Backend for the variational factor model training and inference) — https://pytorch.org/get-started/locally/
- **pyro-ppl** (Provides negative-binomial distributions for the probabilistic model's likelihoods)
- **pandas** (Loads the paired RNA gene counts and raw protein ADT counts CSV matrices)
- **scanpy** (Leiden clustering per modality to seed the categorical complementary latents YR and YP)
- **anndata** (Single-cell data structures in the scME environment)

## Examples

```
python scme.py --rna rna_counts.csv --protein protein_adt.csv --output-dir ./embeddings --batch-size 256
```

## Evaluation signals

- Training schedule runs to completion for max_epochs=200 with lr decay firing at milestone epoch 80
- The saved embedding zm contains exactly one latent vector per input cell (cell count preserved between input CSVs and output)
- Embedding dimensionality equals latent_dim=32 as configured in build_scme
- Joint representation improves downstream single-cell clustering and cell-type classification relative to single-modality baselines
- The 'NB' vs 'MNB' protein_dist choice is documented and reproducible for the given protein ADT data

## Limitations

- Most existing integration methods discard modality-specific complementary information — scME addresses this, but requires paired data; unpaired modalities are out of scope
- protein_dist must be chosen ('NB' or 'MNB') depending on whether protein ADT data require a mixture negative binomial likelihood
- Results depend on the specified hyperparameters (rna_latent_dim=24, protein_latent_dim=20, latent_dim=32, batchsize=256, milestones=[80]); default settings from the paper's tutorial may need retuning for other datasets

## Evidence

- [results] scME's dual-modality factor model defines a shared latent variable ZM, and two distinct latent variables YR and YP that represent the complementary information of the RNA modality and the protein modality: [evidence span withheld — anchor status: not_found]
- [readme] scME accepts as input the RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format: "scME accepts as input the  RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format usually end in ".csv", where rows are cells and columns are genes"
- [readme] Building model and training code from README showing build_scme parameters: "scme=build_scme(rnadata=rna
                ,proteindata=protein
                ,protein_dist="NB",#'NB' or 'MNB'
                rna_latent_dim=24,
                protein_latent_dim=20,"
- [methods] Inference call producing cell embedding: "#Inference cell embedding

zm=scme.inference(rna_data,protein_data)"
- [abstract] Complementary information often discarded by existing methods: "Most single-cell multiomics integration methods focus on shared information among modalities while complementary information specific to each modality is often discarded"
