---
name: variational-autoencoder-latent-embedding-learning
description: Use when raw scRNA-seq count matrices from multiple batches (e.g., pancreas
  16,382 cells, immune 33,506 cells, heart atlas 486,134 cells) need batch-corrected
  latent embeddings for atlas-level or cross-species integration.
when_to_use_negative:
- The number of batches is unknown — the model assumes prior knowledge of the number
  of batches, and unsupervised treatment of batch information is unexplored
- Input is a multiomic (e.g., RNA+ATAC) dataset — the multiomic extension of scDREAMER
  is not yet implemented (planned future work)
- Batches carry a hierarchical structure (e.g., nested batch variables) — utilization
  of hierarchical batch structure is not yet supported
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDREAMER
  role: 'Adversarial variational autoencoder framework: encoder E, ZINB decoder D,
    Bhattacharyya-loss discriminator, and adversarial batch classifier B learn the
    batch-ignorant latent embedding'
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scDREAMER-Sup
  role: Supervised extension adding VAE AY={Ey, Dy} and feed-forward cell type classifier
    C (cross-entropy loss) for semi-supervised inference of latent spaces and label
    prediction
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: 'Preprocessing: AnnData ingestion, low-quality cell removal via mitochondrial
    gene counts, normalize_total + log transform, and top 2000 HVG selection (flavor=''seurat'')'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:17b0387e2aa16e8f54efd64649a838defbba7347e077cc5766e9e2bf95a165fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - Scanpy
  - scDREAMER
  - scDREAMER-Sup
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# variational-autoencoder-latent-embedding-learning

## Summary

Learn batch-ignorant low-dimensional cellular embeddings from scRNA-seq count matrices by training the scDREAMER adversarial variational autoencoder: encoder E maps {x, s} to {μz, σz, μl, σl}, a ZINB decoder D reconstructs expression, and adversarial training of a discriminator (Bhattacharyya loss) plus a batch classifier B removes batch effects while conserving biology.

## When to use

Use when raw scRNA-seq count matrices from multiple batches (e.g., pancreas 16,382 cells, immune 33,506 cells, heart atlas 486,134 cells) need batch-corrected latent embeddings for atlas-level or cross-species integration.

## When NOT to use

- The number of batches is unknown — the model assumes prior knowledge of the number of batches, and unsupervised treatment of batch information is unexplored
- Input is a multiomic (e.g., RNA+ATAC) dataset — the multiomic extension of scDREAMER is not yet implemented (planned future work)
- Batches carry a hierarchical structure (e.g., nested batch variables) — utilization of hierarchical batch structure is not yet supported

## Inputs

- Raw count expression matrices as Scanpy AnnData objects
- Batch labels for each cell
- Cell type annotations (partial, e.g., 10/20/50% held-out, for scDREAMER-Sup semi-supervised mode)
- Preprocessed expression matrix of top 2000 highly variable genes (Scanpy-normalized, log-transformed)

## Outputs

- Low-dimensional batch-corrected latent cellular embeddings z for all cells
- Predicted cell type labels for annotation-free cells (scDREAMER-Sup)
- Single h5ad file containing latent embeddings and predicted labels

## How to apply

Import raw count matrices as a Scanpy AnnData object, remove low-quality cells based on mitochondrial gene counts, normalize with scanpy.pp.normalize_total and log-transform, then select the top 2000 highly variable genes (scanpy.pp.highly_variable_genes, flavor='seurat') as input to scDREAMER. Train the adversarial VAE with the ADAM optimizer at batch size 128, maximizing the ELBO (KL terms for z and l) while the discriminator (Bhattacharyya loss) and adversarially trained batch classifier B remove batch effects from the latent space. For supervised/semi-supervised settings, extend to scDREAMER-Sup by adding a second variational autoencoder AY={Ey, Dy} and a feed-forward cell type classifier C trained with cross-entropy loss only on cells with available annotations, then use C to predict labels for unlabeled cells. Save the informative latent embeddings z for all cells, and optionally the predicted cell type labels, to a single h5ad file.

## Related tools

- **scDREAMER** (Adversarial variational autoencoder framework: encoder E, ZINB decoder D, Bhattacharyya-loss discriminator, and adversarial batch classifier B learn the batch-ignorant latent embedding) — https://github.com/Zafar-Lab/scDREAMER
- **scDREAMER-Sup** (Supervised extension adding VAE AY={Ey, Dy} and feed-forward cell type classifier C (cross-entropy loss) for semi-supervised inference of latent spaces and label prediction) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing: AnnData ingestion, low-quality cell removal via mitochondrial gene counts, normalize_total + log transform, and top 2000 HVG selection (flavor='seurat'))

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- Latent embeddings must support batch mixing and biological conservation simultaneously (quantitative integration metrics)
- Louvain clustering on the embedding at resolution maximizing NMI should recover biological cell identities
- For scDREAMER-Sup, predicted labels on held-out cells must score well by macro F1-score against the true annotations
- Held-out cell type experiments (e.g., alpha or delta cells removed from training) should still capture novel/rare cell types in the embedding
- Benchmark against state-of-the-art integration methods (e.g., BBKNN, Scanorama, Seurat v3, Harmony, scVI, DESC, INSCT) on batch-correction and bio-conservation metrics

## Limitations

- Model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- May require fine-tuning of some parameters (deep learning method) for best performance
- Hierarchical structure between different batch information is not yet utilized
- Multiomic dataset extension not yet implemented (planned future work)

## Evidence

- [abstract] encoder maps expression and batch covariates to latent embedding: "we present scDREAMER ( https://github.com/Zafar-Lab/scDREAMER ), a data-integration framework that employs deep generative models and adversarial training"
- [methods] preprocessing threshold: top 2000 HVGs with seurat flavor: "we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”) as input to scDREAMER"
- [methods] training hyperparameters: ADAM optimizer, batch size 128: [evidence span withheld — anchor status: not_found]
- [intro] scDREAMER-Sup architecture: additional VAE and cell type classifier: "The supervised version, scDREAMER-Sup, employs an additional variational autoencoder and a cell type classifier (another feed-forward neural network) to utilize available cell type annotations"
- [readme] installation via conda environment from the scDREAMER repo: "For now, we recommend users to directly clone our stable `main` branch and set `scDREAMER` as the working directory. Creating conda environment using ./ENVIRONMENTS/scDREAMER.yml"
