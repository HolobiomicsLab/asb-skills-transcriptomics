---
name: adversarial-component-ablation-analysis
description: Use when you need to determine whether each adversarial component of
  an adversarial-VAE integration method (discriminator, batch classifier) contributes
  to integration, and what each contributes.
when_to_use_negative:
- The goal is to compare scDREAMER against other integration methods (BBKNN, Scanorama,
  Seurat v3, Harmony, scVI, DESC, iMAP) rather than to dissect scDREAMER's internal
  components
- Only a single trained model is available — ablation requires training discriminator-removed
  and batch-classifier-removed variants from scratch
- Cell-type labels are unavailable, since bio-conservation attribution to the discriminator
  relies on NMI, ARI and cell-type ASW
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scDREAMER
  role: Adversarial VAE framework whose discriminator and batch classifier components
    are ablated and retrained to measure each one's contribution
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: Preprocessing (AnnData import, mitochondrial QC, normalize_total, log-transformation,
    top-2000 HVG selection with flavor 'seurat') and metric computation context
- name: Louvain
  role: Clustering of latent embeddings at the resolution that maximizes NMI to compute
    bio-conservation metrics
- name: ADAM optimizer
  role: Optimizer used to train the full and ablated scDREAMER variants with batch
    size 128
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:c1bf95df1c8f514034d6fe7b92e9bdc14be57b8eea5d3f6ecd484d133b772e67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - Louvain
  - Scanpy
  - scDREAMER
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# adversarial-component-ablation-analysis

## Summary

Ablation-analysis skill for scDREAMER that trains the full adversarial VAE plus variants with the adversarial discriminator or batch classifier removed, then compares bio-conservation and batch-correction metrics to attribute integration performance to each component.

## When to use

Use when you need to determine whether each adversarial component of an adversarial-VAE integration method (discriminator, batch classifier) contributes to integration, and what each contributes.

## When NOT to use

- The goal is to compare scDREAMER against other integration methods (BBKNN, Scanorama, Seurat v3, Harmony, scVI, DESC, iMAP) rather than to dissect scDREAMER's internal components
- Only a single trained model is available — ablation requires training discriminator-removed and batch-classifier-removed variants from scratch
- Cell-type labels are unavailable, since bio-conservation attribution to the discriminator relies on NMI, ARI and cell-type ASW

## Inputs

- Processed lung atlas dataset (32,472 cells, 10X and drop-seq, 16 donors) from the Figshare benchmarking deposit, as a Scanpy AnnData object
- scDREAMER model variants: full adversarial VAE, discriminator-ablated, and batch-classifier-ablated
- Cell-type annotations for computing NMI, ARI and ASW
- Batch labels for batch-correction metrics

## Outputs

- ablation_metrics.csv aggregating bio-conservation (NMI, ARI, cell-type ASW) and batch-correction (ASW batch, PCR batch, graph connectivity, kBET) metrics across the full and ablated variants
- Latent embeddings per variant with Louvain cluster assignments at NMI-maximizing resolution

## How to apply

Load the processed lung atlas dataset (32,472 cells from lung transplant and biopsy samples of 16 donors sequenced with 10X and drop-seq) from the Figshare benchmarking deposit as a Scanpy AnnData object. Preprocess by removing low-quality cells based on mitochondrial gene counts, normalizing with scanly.pp.normalize_total and log-transforming, then selecting the top 2000 highly variable genes via scanpy.pp.highly_variable_genes with flavor 'seurat'. Train three variants with the ADAM optimizer and batch size 128: the full model, one without the adversarial discriminator, and one without the batch classifier. For each variant, extract latent embeddings, run Louvain clustering at the resolution that maximizes NMI, and compute bio-conservation metrics (NMI, ARI, cell-type ASW scaled as (ASW+1)/2) and batch-correction metrics (ASW batch as 1-abs(ASW), PCR batch, graph connectivity, kBET with k=50 on the kNN graph). Aggregate into ablation_metrics.csv; the discriminator should support bio-conservation and rare cell-type identification while the batch classifier drives batch-effect removal, with the full model achieving the best composite score.

## Related tools

- **scDREAMER** (Adversarial VAE framework whose discriminator and batch classifier components are ablated and retrained to measure each one's contribution) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing (AnnData import, mitochondrial QC, normalize_total, log-transformation, top-2000 HVG selection with flavor 'seurat') and metric computation context)
- **Louvain** (Clustering of latent embeddings at the resolution that maximizes NMI to compute bio-conservation metrics)
- **ADAM optimizer** (Optimizer used to train the full and ablated scDREAMER variants with batch size 128)

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- ablation_metrics.csv contains rows for all three variants (full, no-discriminator, no-batch-classifier) with NMI, ARI, cell-type ASW ((ASW+1)/2), ASW batch (1-abs(ASW)), PCR batch, graph connectivity and kBET (k=50) columns
- Full model achieves the best combined composite score versus both ablated variants
- Removing the batch classifier degrades batch-correction metrics (ASW batch, PCR batch, kBET) more than bio-conservation metrics
- Removing the discriminator degrades bio-conservation (NMI, ARI, cell-type ASW) and rare cell-type identification more than batch-correction metrics
- Louvain clustering was run at the resolution that maximizes NMI for each variant's embedding

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- scDREAMER, being a deep learning-based method, may require fine-tuning of some parameters for extracting the best performance
- Ablation conclusions are tied to the benchmark datasets (e.g., lung atlas with nested batch effects) and may not transfer to all data regimes

## Evidence

- [results] Ablation results show both components play a significant role: "We performed ablation studies to examine the contribution of the two adversarial components of scDREAMER—adversarial discriminator and batch classifier"
- [methods] Preprocessing parameters: "we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”) as input to scDREAMER"
- [methods] Training configuration: [evidence span withheld — anchor status: not_found]
- [readme] Framework composition: "scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings and a batch classifier neural network"
- [supplementary] Dataset source: "The processed human pancreas, human immune, and lung atlas datasets are available at https://figshare.com/articles/dataset/Benchmarking_atlas-level_data_integration_in_single-cell_genomics_-_integrati"
