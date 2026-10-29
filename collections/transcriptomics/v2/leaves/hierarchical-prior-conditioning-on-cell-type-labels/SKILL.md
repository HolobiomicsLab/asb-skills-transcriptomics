---
name: hierarchical-prior-conditioning-on-cell-type-labels
description: Use when integrating multi-batch scRNA-seq data (e.g. AnnData of normalized,
  log-transformed cells) where some or all cell type annotations are available and
  a supervised/semi-supervised latent space with label prediction is desired.
when_to_use_negative:
- No cell type annotations exist at all and a purely unsupervised integration is required
  — use plain scDREAMER instead, since the classifier C and label-conditioned y-prior
  need annotations to train
- Annotations are used only for downstream evaluation and should not influence the
  latent space — conditioning the prior on labels would leak supervision into the
  embedding
- The task requires handling unknown batch structure, since the model assumes prior
  knowledge of the number of batches
tools:
- name: scDREAMER-Sup
  role: 'Supervised variant of the backbone: adds VAE AY and feed-forward classifier
    C with cross-entropy loss on annotated cells to condition the latent prior on
    cell type labels'
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scDREAMER
  role: Unsupervised adversarial VAE backbone (encoder E, ZINB decoder D, discriminator,
    adversarial batch classifier B) onto which the supervised components are bolted
  repo: https://github.com/Zafar-Lab/scDREAMER
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:fe9f1c883504522eda861f8880db6af0d21a05f8a695777d7ae8ffa6d01497b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - scDREAMER
  - scDREAMER-Sup
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hierarchical-prior-conditioning-on-cell-type-labels

## Summary

scDREAMER-Sup extends the adversarial VAE backbone of scDREAMER with an additional variational autoencoder AY and a feed-forward cell type classifier C, conditioning the latent prior on available cell type annotations so that supervised or semi-supervised (10/20/50% labeled) integration yields annotation-guided latent embeddings and predicted labels.

## When to use

Use when integrating multi-batch scRNA-seq data (e.g. AnnData of normalized, log-transformed cells) where some or all cell type annotations are available and a supervised/semi-supervised latent space with label prediction is desired.

## When NOT to use

- No cell type annotations exist at all and a purely unsupervised integration is required — use plain scDREAMER instead, since the classifier C and label-conditioned y-prior need annotations to train
- Annotations are used only for downstream evaluation and should not influence the latent space — conditioning the prior on labels would leak supervision into the embedding
- The task requires handling unknown batch structure, since the model assumes prior knowledge of the number of batches

## Inputs

- Scanpy AnnData object of raw counts with batch labels and (possibly partially masked) cell type annotations
- Preprocessed expression matrix of top 2000 highly variable genes (Scanpy-normalized and log-transformed, e.g. the 16,382-cell human pancreas dataset)
- Batch-covariate vector s per cell
- Cell type annotation vector with a masking fraction (10/20/50%) marking unlabeled cells

## Outputs

- Informative latent embeddings z for all cells, saved in a single h5ad file
- Predicted cell type labels for the unlabeled cells produced by classifier C
- Trained scDREAMER-Sup model (adversarial VAE + AY + classifier C)

## How to apply

Preprocess the raw counts as a Scanpy AnnData object (remove low-quality cells by mitochondrial gene counts, normalize with scanly.pp.normalize_total, log-transform, and select the top 2000 highly variable genes via scanly.pp.highly_variable_genes with flavor 'seurat'). Instantiate the full scDREAMER-Sup architecture: the adversarial VAE backbone (encoder E mapping {x, s} to {mu_z, sigma_z, mu_l, sigma_l}, ZINB decoder D, Bhattacharyya-loss discriminator, adversarial batch classifier B) plus the additional VAE AY = {Ey, Dy} and the feed-forward classifier C, all multi-layer dense networks with linear relu activation. Train with the ADAM optimizer at batch size 128, maximizing the extended ELBO (KL terms for z, l, and a y-prior conditioned on the cell type label c) plus a cross-entropy loss for C computed only on cells with available annotations; for the semi-supervised setting, mask labels on a held-out fraction (10%, 20% or 50%) of cells. Use the trained classifier C to predict labels for unlabeled cells, then save the informative latent embeddings z and predicted labels to a single h5ad file.

## Related tools

- **scDREAMER-Sup** (Supervised variant of the backbone: adds VAE AY and feed-forward classifier C with cross-entropy loss on annotated cells to condition the latent prior on cell type labels) — https://github.com/Zafar-Lab/scDREAMER
- **scDREAMER** (Unsupervised adversarial VAE backbone (encoder E, ZINB decoder D, discriminator, adversarial batch classifier B) onto which the supervised components are bolted) — https://github.com/Zafar-Lab/scDREAMER

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- Latent embeddings z must support batch mixing (batch-correction metrics) while conserving biological variation, comparable to the article's benchmarking
- Predicted cell type labels for masked cells must be scored with macro F1-score against the held-out annotations
- Louvain clusters in the embedding (at resolution maximizing NMI) must align with annotated cell types, including rare cell types
- Semi-supervised runs at 10%, 20% and 50% missing labels should degrade gracefully, with scDREAMER-Sup reported to outperform state-of-the-art supervised integration methods
- Training converges under ADAM at batch size 128 with the extended ELBO plus classifier cross-entropy loss applied only to annotated cells

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- As a deep learning-based method, scDREAMER may require fine-tuning of some parameters for extracting the best performance
- A pip installation release was not yet available; users must clone the main branch and build the conda environment from scDREAMER.yml

## Evidence

- [intro] Supervised variant architecture: "The supervised version, scDREAMER-Sup, employs an additional variational autoencoder and a cell type classifier (another feed-forward neural network)"
- [other] Classifier training signal: [evidence span withheld — anchor status: not_found]
- [readme] Semi-supervised masking fractions: "scDREAMER-SUP can also be when cell type annotations are missing in the datasets i.e., 10%, 20%, 50%"
- [methods] Training hyperparameters: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark performance: "scDREAMER and scDREAMER-Sup outperform state-of-the-art unsupervised and supervised integration methods respectively"
