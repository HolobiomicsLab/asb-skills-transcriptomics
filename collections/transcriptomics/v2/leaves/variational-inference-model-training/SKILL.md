---
name: variational-inference-model-training
description: Use when you have paired single-cell multiomics data as an RNA gene counts
  CSV and a raw protein ADT counts CSV (rows = cells, columns = genes/ADTs) and need
  a joint cell embedding that preserves both shared and modality-specific information.
when_to_use_negative:
- The input is already a joint embedding or feature table rather than raw paired RNA
  and protein count matrices in CSV format
- Only a single modality is available (no paired protein ADT counts), since the model
  requires both rna and protein inputs
- The goal is integration of modalities where discarding modality-specific complementary
  information is acceptable and a simpler shared-only method suffices
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scME
  role: Dual-modality factor model being trained; provides build_scme, train_model,
    and inference for the shared embedding zm
  repo: https://github.com/bucky527/scME
- name: PyTorch
  role: Deep-learning backend in which the factorized variational posterior and total
    loss (ELBO + BCE terms) are implemented
- name: pyro-ppl
  role: Provides the negative-binomial (NB/MNB) distributions used in the model's
    probabilistic layer
- name: pandas
  role: Loads the paired RNA and protein ADT counts CSV matrices with cells as rows
- name: scanpy
  role: Leiden clustering per modality to seed the categorical complementary latents
    YR and YP
- name: anndata
  role: Installed dependency for single-cell data handling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad337
    title: 'scME: a dual-modality factor model for single-cell multiomics embedding'
  source_content_hash: sha256:f1f8380e4a9d9603ea45d0d75850cec55bf6a166f43dfb7d2c0a77be8bfb8a8b
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# variational-inference-model-training

## Summary

Train the scME dual-modality factor model via Adam-based stochastic variational inference to embed paired CITE-seq RNA and protein ADT counts into a shared latent zm plus modality-specific complementary latents. Rationale: most single-cell multiomics integration methods discard modality-specific complementary information, while scME's joint representation improves clustering and cell-type classification.

## When to use

Use when you have paired single-cell multiomics data as an RNA gene counts CSV and a raw protein ADT counts CSV (rows = cells, columns = genes/ADTs) and need a joint cell embedding that preserves both shared and modality-specific information.

## When NOT to use

- The input is already a joint embedding or feature table rather than raw paired RNA and protein count matrices in CSV format
- Only a single modality is available (no paired protein ADT counts), since the model requires both rna and protein inputs
- The goal is integration of modalities where discarding modality-specific complementary information is acceptable and a simpler shared-only method suffices

## Inputs

- RNA gene counts matrix in CSV format (rows are cells, columns are genes; column 0 must be cell IDs)
- Raw protein ADT counts matrix in CSV format (rows are cells, columns are ADTs)

## Outputs

- Shared cell embedding zm (one latent vector per input cell), produced by scme.inference(rna_data, protein_data) and saved to the output directory
- Trained scME model (optionally saved when save_model=True with a save_dir)

## How to apply

Load the paired RNA gene counts matrix and raw protein ADT counts matrix from CSV with pandas, ensuring column 0 holds cell IDs. Optionally cluster each modality separately with scanpy Leiden to seed the categorical complementary latents YR and YP. Build the model with build_scme (rna_latent_dim=24, protein_latent_dim=20, latent_dim=32, protein_dist='NB', or 'MNB' for a mixture negative binomial on protein data) and define the factorized variational posterior with total loss L = ELBO + BCE(YR) + BCE(YP) using PyTorch with pyro-ppl for the negative-binomial distributions. Train with train_model (max_epochs=200, batchsize=256, lr=1e-4, lr_cla=1e-4, lr decay milestones=[80]). The decision point on protein_dist is whether protein ADT counts warrant a mixture negative binomial; batch size can be set via the CLI flag --batch-size 256.

## Related tools

- **scME** (Dual-modality factor model being trained; provides build_scme, train_model, and inference for the shared embedding zm) — https://github.com/bucky527/scME
- **PyTorch** (Deep-learning backend in which the factorized variational posterior and total loss (ELBO + BCE terms) are implemented)
- **pyro-ppl** (Provides the negative-binomial (NB/MNB) distributions used in the model's probabilistic layer)
- **pandas** (Loads the paired RNA and protein ADT counts CSV matrices with cells as rows)
- **scanpy** (Leiden clustering per modality to seed the categorical complementary latents YR and YP)
- **anndata** (Installed dependency for single-cell data handling)

## Examples

```
python scme.py --rna rna_counts.csv --protein adt_counts.csv --output-dir ./embeddings --batch-size 256
```

## Evaluation signals

- The training schedule runs to completion for max_epochs=200 with lr decay at milestones=[80] without error
- The saved embedding contains one latent vector (zm) per input cell, matching the row count of the input CSVs
- The output zm is reproducibly obtained from the call zm=scme.inference(rna_data, protein_data)
- The model converges under Adam-based stochastic variational inference with the specified learning rates (lr=1e-4, lr_cla=1e-4) and batchsize=256
- Downstream, the joint representation yields salient information for single-cell clustering and cell-type classification, per the paper's reported benefit

## Limitations

- scME accepts only RNA gene counts and raw protein ADT counts matrices in CSV format; other formats or unpaired data require conversion or are unsupported
- Column 0 of the input CSV must be the cell IDs; deviations break data loading
- protein_dist must be chosen between 'NB' and 'MNB'; the wrong choice for protein ADT count distributions may degrade the embedding

## Evidence

- [readme] build_scme: "scme=build_scme(rnadata=rna
                ,proteindata=protein
                ,protein_dist="NB",#'NB' or 'MNB'
                rna_latent_dim=24,"
- [methods] inference: "#Inference cell embedding

zm=scme.inference(rna_data,protein_data)"
- [methods] install: "pip3 install numpy scipy pandas scikit-learn pyro-ppl matplotlib scanpy anndata scvi-tools"
- [abstract] complementary info: "Most single-cell multiomics integration methods focus on shared information among modalities while complementary information specific to each modality is often discarded"
- [readme] input format: "scME accepts as input the  RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format usually end in ".csv", where rows are cells and columns are genes, and the columns 0"
