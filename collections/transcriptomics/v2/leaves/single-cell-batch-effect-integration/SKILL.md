---
name: single-cell-batch-effect-integration
description: Use when raw scRNA-seq count matrices from multiple batches (e.g., pancreas,
  human immune, lung, heart atlas) must be integrated to remove batch effects while
  conserving cell types, trajectories, or rare populations.
when_to_use_negative:
- When the model cannot assume prior knowledge of the number of batches — scDREAMER's
  current model assumes prior knowledge of the number of batches and does not treat
  batch information unsupervised
- When kBET must be reported for every method on very large, many-batch data — on
  the healthy heart atlas kBET could not be computed for some methods due to computational
  limitations in handling large numbers of cells and batches
- When batch labels or cell count annotations for low-quality filtering are unavailable,
  since the pipeline starts from raw counts requiring mitochondrial-gene-based quality
  control
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDREAMER
  role: 'Primary unsupervised integration method: adversarial VAE with batch classifier
    (ADAM, batch size 128) producing batch-corrected embeddings'
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scDREAMER-Sup
  role: Supervised/semi-supervised variant that adds a VAE and cell type classifier
    for label-prediction on cells with missing annotations
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: Preprocessing (AnnData import, normalize_total, log-transform, highly_variable_genes
    flavor='seurat', top 2000 HVGs) and Louvain clustering
- name: INSCT
  role: 'Competing baseline: batch-aware triplet network integration model run alongside
    scDREAMER'
  repo: https://github.com/lkmklsmn/insct
- name: scVI
  role: 'Competing baseline: deep variational autoencoder integration method'
- name: Scanorama
  role: 'Competing baseline: global mutual nearest neighbor (MNN) integration method'
- name: Harmony
  role: Competing baseline applying local correction for better batch mixing
- name: Seurat
  role: Competing baseline (Seurat v3) using global MNNs for integration
- name: BBKNN
  role: Competing baseline using global MNNs for batch integration
- name: LIGER
  role: Competing baseline integration method
- name: iMAP
  role: Competing baseline combining GANs with autoencoder for batch-ignorant representations
- name: scANVI
  role: Competing supervised baseline for semi-supervised cell-type label prediction
- name: scGEN
  role: Competing baseline integration method
- name: scDML
  role: Competing baseline integration method
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:ff41b902c20c1ec98be44d9254d38027c782265389d38c704d42e66fe2928e46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM
  - ADAM optimizer
  - BBKNN
  - Harmony
  - INSCT
  - LIGER
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scDREAMER-Sup
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# single-cell-batch-effect-integration

## Summary

Integrates multi-batch scRNA-seq datasets by learning batch-corrected cellular embeddings with scDREAMER, an adversarial variational autoencoder paired with a batch classifier, and benchmarking against eleven competing integration methods using standardized bio-conservation and batch-correction metrics.

## When to use

Use when raw scRNA-seq count matrices from multiple batches (e.g., pancreas, human immune, lung, heart atlas) must be integrated to remove batch effects while conserving cell types, trajectories, or rare populations.

## When NOT to use

- When the model cannot assume prior knowledge of the number of batches — scDREAMER's current model assumes prior knowledge of the number of batches and does not treat batch information unsupervised
- When kBET must be reported for every method on very large, many-batch data — on the healthy heart atlas kBET could not be computed for some methods due to computational limitations in handling large numbers of cells and batches
- When batch labels or cell count annotations for low-quality filtering are unavailable, since the pipeline starts from raw counts requiring mitochondrial-gene-based quality control

## Inputs

- Raw count expression matrix from multiple scRNA-seq batches as a Scanpy AnnData object (.h5ad)
- Batch label vector annotating each cell's batch of origin
- Optional ground-truth cell type annotations for supervised (scDREAMER-Sup) integration and label-prediction benchmarks
- Processed benchmark datasets from the Figshare deposit (human pancreas, human immune, lung atlas) or heartcellatlas.org

## Outputs

- Integrated low-dimensional cellular embedding for each method
- UMAP visualization of the integrated embedding
- Per-method metric table (CSV) with NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, kBET, iLISI/cLISI, isolated-label scores, and composite scores
- Predicted cell type labels with macro F1-scores for semi-supervised settings

## How to apply

Import raw count matrices as a Scanpy AnnData object, remove low-quality cells based on mitochondrial gene counts (and annotated doublets for the heart atlas), normalize with scanpy.pp.normalize_total, log-transform, and select the top 2000 highly variable genes with scanpy.pp.highly_variable_genes (flavor='seurat'). Train scDREAMER — an adversarial VAE with a batch classifier using the ADAM optimizer and batch size 128 — and run competing methods (scVI, Scanorama, Harmony, Seurat v3, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, scDML) with their published configurations. For supervised or semi-supervised settings (10/20/50% missing labels), use scDREAMER-Sup, which adds a variational autoencoder and a cell type classifier trained with cross-entropy loss on labeled cells. Louvain-cluster each integrated embedding at the resolution that maximizes NMI, then compute NMI, ARI, cell-type ASW scaled as (ASW+1)/2, batch ASW as 1−abs(ASW), PCR batch, graph connectivity, kBET with k=50, graph iLISI/cLISI, and isolated-label F1/ASW. Min-max normalize each metric across methods and average into composite bio-conservation, composite batch-correction, and combined composite scores; visualize embeddings with UMAP and write per-method metric tables to CSV.

## Related tools

- **scDREAMER** (Primary unsupervised integration method: adversarial VAE with batch classifier (ADAM, batch size 128) producing batch-corrected embeddings) — https://github.com/Zafar-Lab/scDREAMER
- **scDREAMER-Sup** (Supervised/semi-supervised variant that adds a VAE and cell type classifier for label-prediction on cells with missing annotations) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing (AnnData import, normalize_total, log-transform, highly_variable_genes flavor='seurat', top 2000 HVGs) and Louvain clustering)
- **INSCT** (Competing baseline: batch-aware triplet network integration model run alongside scDREAMER) — https://github.com/lkmklsmn/insct
- **scVI** (Competing baseline: deep variational autoencoder integration method)
- **Scanorama** (Competing baseline: global mutual nearest neighbor (MNN) integration method)
- **Harmony** (Competing baseline applying local correction for better batch mixing)
- **Seurat** (Competing baseline (Seurat v3) using global MNNs for integration)
- **BBKNN** (Competing baseline using global MNNs for batch integration)
- **LIGER** (Competing baseline integration method)
- **iMAP** (Competing baseline combining GANs with autoencoder for batch-ignorant representations)
- **scANVI** (Competing supervised baseline for semi-supervised cell-type label prediction)
- **scGEN** (Competing baseline integration method)
- **scDML** (Competing baseline integration method)

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git && cd scDREAMER/Environments && conda env create -f scDREAMER.yml && conda activate scdreamer
```

## Evaluation signals

- Metric bounds: NMI and ARI in [0,1]; scaled cell-type ASW as (ASW+1)/2 in [0,1]; scaled kBET and batch ASW as 1−abs(ASW) in [0,1] with 1 indicating perfect mixing
- Composite scores must be computable for all twelve methods, with kBET explicitly excluded/marked for methods where it failed on the heart atlas
- scDREAMER should attain the highest combined composite score (e.g., ~12.9% above second-best on pancreas; 25% improvement on heart atlas)
- Semi-supervised label prediction should yield macro F1 in [0,1] (expected 0.86–0.9) with scDREAMER-Sup ≥ scANVI at 10/20/50% missing labels
- Biological validation: integrated embedding should preserve known trajectories (e.g., HSPC-to-erythrocyte continuum in the human immune dataset) and separate tissue-specific cell types

## Limitations

- The current model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- Hierarchical structure between different batch information is not yet utilized when it exists
- Being a deep learning-based method, scDREAMER may require fine-tuning of some parameters for extracting the best performance
- kBET could not be computed for some methods on the healthy heart atlas due to computational limitations in handling large numbers of cells and batches
- Extension to multiomic datasets is planned but not yet implemented

## Evidence

- [intro] scDREAMER (single-cell Deep geneRativE integrAtion Model with advErsarial classifieR): "here we present a deep learning-based data integration framework, called scDREAMER (single-cell Deep geneRativE integrAtion Model with advErsarial classifieR)"
- [methods] Preprocessing and HVG selection: "we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”) as input to scDREAMER"
- [methods] Training configuration: [evidence span withheld — anchor status: not_found]
- [methods] Louvain clustering for metrics: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [intro] Dataset scale: "Using multiple real datasets consisting of up to 1 million cells and 147 batches (Supplementary Table 1), we demonstrate that scDREAMER is able to overcome a variety of integration challenges"
- [readme] Semi-supervised capability: "scDREAMER-SUP can also be when cell type annotations are missing in the datasets i.e., 10%, 20%, 50%"
- [methods] kBET limitation: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
- [discussion] Batch-count assumption: "Our current model assumes prior knowledge of the number of batches. An important future direction would be to explore the unsupervised treatment of batch information"
- [readme] Environment setup: "conda env create -f scDREAMER.yml
conda activate scdreamer"
