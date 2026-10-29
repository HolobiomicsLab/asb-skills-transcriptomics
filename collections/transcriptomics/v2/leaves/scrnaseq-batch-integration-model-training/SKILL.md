---
name: scrnaseq-batch-integration-model-training
description: Use when you have raw count matrices from multiple scRNA-seq batches
  (e.g. .h5ad AnnData, ≥2 batches, up to ~1M cells) and need an unsupervised integrated
  embedding plus an ablation-based attribution of which adversarial components drive
  batch correction vs bio-conservation.
when_to_use_negative:
- Data has no batch structure (a single batch) — there is nothing for the batch classifier
  to remove and the adversarial setup is unnecessary.
- Cell type labels are available and you want supervised integration — that is scDREAMER-Sup's
  role (additional VAE plus cell type classifier), not the unsupervised training skill.
- Input is multiomic data — the article notes the multiomic extension is planned future
  work and not yet implemented.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDREAMER
  role: The adversarial VAE + batch classifier integration model being trained and
    ablated; installed by cloning the repo and creating the conda environment from
    scDREAMER.yml.
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: 'Preprocessing and metric computation: AnnData import, mitochondrial-gene
    QC filtering, scanly.pp.normalize_total, log-transformation, highly_variable_genes
    with flavor ''seurat''.'
- name: Louvain
  role: Clustering of the learned latent embedding at the resolution that maximizes
    NMI for bio-conservation scoring.
- name: ADAM optimizer
  role: Optimizer used to train the scDREAMER model with batch size 128.
- name: scDREAMER-reproducibility
  role: Notebooks reproducing the ablation analyses and results presented in the manuscript.
  repo: https://github.com/Zafar-Lab/scDREAMER-reproducibility
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:5bffaa314e2d774a1d296d2f4bfa138db28015ad343a0a411175607536e948b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - Louvain
  - Scanpy
  - scDREAMER
  - scDREAMER-reproducibility
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# scrnaseq-batch-integration-model-training

## Summary

Train a single-cell RNA-seq batch-integration model — here the scDREAMER adversarial variational autoencoder paired with a batch classifier — on preprocessed AnnData (top 2000 HVGs, flavor 'seurat') to produce batch-corrected latent embeddings, and evaluate variants (full model, discriminator-removed, batch-classifier-removed) with bio-conservation and batch-correction metrics.

## When to use

Use when you have raw count matrices from multiple scRNA-seq batches (e.g. .h5ad AnnData, ≥2 batches, up to ~1M cells) and need an unsupervised integrated embedding plus an ablation-based attribution of which adversarial components drive batch correction vs bio-conservation.

## When NOT to use

- Data has no batch structure (a single batch) — there is nothing for the batch classifier to remove and the adversarial setup is unnecessary.
- Cell type labels are available and you want supervised integration — that is scDREAMER-Sup's role (additional VAE plus cell type classifier), not the unsupervised training skill.
- Input is multiomic data — the article notes the multiomic extension is planned future work and not yet implemented.

## Inputs

- Raw scRNA-seq count expression matrices (Scanpy AnnData .h5ad) from multiple batches, e.g. the lung atlas dataset (32,472 cells, 16 donors, 10X and drop-seq) from the Figshare benchmarking deposit
- Cell batch labels vector for each batch
- Cell-type annotations (for computing NMI, ARI, and cell-type ASW)

## Outputs

- Latent cell embeddings for each trained variant (full, discriminator-removed, batch-classifier-removed)
- Louvain cluster assignments per variant at the NMI-maximizing resolution
- Ablation metrics table (e.g. ablation_metrics.csv) with NMI, ARI, ASW cell-type (ASW+1)/2, ASW batch 1-abs(ASW), PCR batch, graph connectivity, and kBET (k=50)

## How to apply

Import raw count matrices as a Scanpy AnnData object, remove low-quality cells based on mitochondrial gene counts, normalize with scanpy.pp.normalize_total, log-transform, and select the top 2000 highly variable genes using scanpy.pp.highly_variable_genes with flavor 'seurat'. Train three variants of the adversarial VAE with the ADAM optimizer and batch size 128: (a) the full model with adversarial discriminator plus batch classifier, (b) with the discriminator removed, and (c) with the batch classifier removed. For each variant, extract the latent embedding and run Louvain clustering at the resolution that maximizes NMI. Compute bio-conservation metrics (NMI, ARI, cell-type ASW scaled as (ASW+1)/2) and batch-correction metrics (ASW batch scaled as 1-abs(ASW), PCR batch, graph connectivity, kBET with k=50 on the kNN graph), then aggregate into a single table to attribute each adversarial component's contribution. The expectation is that the batch classifier drives batch-effect removal while the discriminator supports faithful reconstruction and bio-conservation.

## Related tools

- **scDREAMER** (The adversarial VAE + batch classifier integration model being trained and ablated; installed by cloning the repo and creating the conda environment from scDREAMER.yml.) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing and metric computation: AnnData import, mitochondrial-gene QC filtering, scanly.pp.normalize_total, log-transformation, highly_variable_genes with flavor 'seurat'.)
- **Louvain** (Clustering of the learned latent embedding at the resolution that maximizes NMI for bio-conservation scoring.)
- **ADAM optimizer** (Optimizer used to train the scDREAMER model with batch size 128.)
- **scDREAMER-reproducibility** (Notebooks reproducing the ablation analyses and results presented in the manuscript.) — https://github.com/Zafar-Lab/scDREAMER-reproducibility

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- Removing the batch classifier should degrade batch-correction metrics (ASW batch, PCR batch, graph connectivity, kBET k=50) while removing the discriminator should degrade bio-conservation (NMI, ARI, cell-type ASW) — the ablation table should reproduce this pattern.
- The full adversarial model should achieve the best combined composite score relative to both ablated variants.
- Louvain clustering on the embedding is run at the resolution that maximizes NMI; confirm the resolution sweep was performed rather than a fixed resolution.
- Preprocessing invariants: cells filtered on mitochondrial gene counts, normalized with scanly.pp.normalize_total, log-transformed, and exactly the top 2000 HVGs selected with flavor 'seurat' are used as model input.
- Ablation metrics for all variants aggregate into one table (ablation_metrics.csv) with the same metric set per variant.

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored.
- scDREAMER is a deep learning-based method and may require fine-tuning of some parameters for extracting the best performance.
- The hierarchical structure between different batch information is not yet utilized.
- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations with large numbers of cells and batches.

## Evidence

- [results] ablation studies of adversarial components: "We performed ablation studies to examine the contribution of the two adversarial components of scDREAMER—adversarial discriminator and batch classifier"
- [methods] preprocessing parameters: "we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”) as input to scDREAMER"
- [methods] training hyperparameters: [evidence span withheld — anchor status: not_found]
- [methods] clustering for metrics: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [readme] model architecture rationale: "scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings and a batch classifier neural network"
- [readme] installation route: "Creating conda environment using ./ENVIRONMENTS/scDREAMER.yml will install all the dependent packages and libraries."
- [discussion] model limitation: "Our current model assumes prior knowledge of the number of batches. An important future direction would be to explore the unsupervised treatment of batch information"
