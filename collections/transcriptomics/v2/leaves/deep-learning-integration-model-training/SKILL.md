---
name: deep-learning-integration-model-training
description: Use when multiple scRNA-seq batches (e.g., 16,382 pancreas cells across
  nine sequencing protocols) must be integrated unsupervised to remove batch effects
  while conserving biological variation such as skewed or rare cell types.
when_to_use_negative:
- Cell type annotations are available for all cells and supervised integration is
  desired — use scDREAMER-Sup instead of the unsupervised model
- The number of batches is unknown and cannot be supplied, since the model assumes
  prior knowledge of the number of batches and unsupervised treatment of batch information
  is unexplored
- A multiomic (e.g., RNA + ATAC) dataset must be integrated, as the multiomic extension
  is planned but not yet implemented
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3170
tools:
- name: scDREAMER
  role: Adversarial variational autoencoder trained to produce batch-corrected cellular
    embeddings for unsupervised integration
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: 'Preprocessing: AnnData import, mitochondrial QC filtering, normalize_total,
    log-transform, top-2000 HVG selection (flavor=''seurat''), and Louvain clustering'
- name: scVI
  role: Deep variational autoencoder baseline run with published configuration for
    comparison
- name: Scanorama
  role: MNN-based baseline run with published configuration for comparison
- name: Harmony
  role: Baseline run with published configuration for comparison
- name: Seurat
  role: MNN-based baseline run with published configuration for comparison
- name: BBKNN
  role: MNN-based baseline run with published configuration for comparison
- name: INSCT
  role: Batch-aware triplet network baseline run with published configuration for
    comparison
  repo: https://github.com/lkmklsmn/insct
- name: LIGER
  role: Baseline run with published configuration for comparison
- name: iMAP
  role: GAN-plus-autoencoder baseline run with published configuration for comparison
- name: scANVI
  role: Baseline run with published configuration for comparison
- name: scGEN
  role: Baseline run with published configuration for comparison
- name: scDML
  role: Baseline run with published configuration for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:871ad7b52475ef91f9ef1169807d2951003646dd24d50951302c65ac5220408c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
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
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# deep-learning-integration-model-training

## Summary

Train the scDREAMER adversarial variational autoencoder to produce batch-corrected low-dimensional cellular embeddings for single-cell RNA-seq integration, and benchmark it against state-of-the-art unsupervised integration baselines.

## When to use

Use when multiple scRNA-seq batches (e.g., 16,382 pancreas cells across nine sequencing protocols) must be integrated unsupervised to remove batch effects while conserving biological variation such as skewed or rare cell types.

## When NOT to use

- Cell type annotations are available for all cells and supervised integration is desired — use scDREAMER-Sup instead of the unsupervised model
- The number of batches is unknown and cannot be supplied, since the model assumes prior knowledge of the number of batches and unsupervised treatment of batch information is unexplored
- A multiomic (e.g., RNA + ATAC) dataset must be integrated, as the multiomic extension is planned but not yet implemented

## Inputs

- Raw scRNA-seq count matrices as Scanpy AnnData objects (e.g., the processed human pancreas integration dataset from the Figshare deposit)
- Batch label vector for each cell (scDREAMER assumes prior knowledge of the number of batches)
- Ground-truth cell type annotations for evaluation metrics

## Outputs

- Integrated low-dimensional cellular embedding per cell (batch-corrected latent representation)
- Louvain clusters on the integrated embedding
- Per-method metric table (e.g., pancreas_integration_metrics.csv) with bio-conservation, batch-correction, isolated-label, and combined composite scores

## How to apply

Import raw count matrices as a Scanpy AnnData object, remove low-quality cells based on mitochondrial gene counts, normalize with scanpy.pp.normalize_total, log-transform, and select the top 2000 highly variable genes using scanpy.pp.highly_variable_genes with flavor='seurat'. Train scDREAMER on the resulting matrix using the ADAM optimizer with batch size 128, a beta-scaled KL term, and an 80% UMI down-sampling penalty; the model couples a variational autoencoder for latent embeddings with a batch classifier for adversarial batch-effect removal. Clone the scDREAMER repo and set up the conda environment from ./ENVIRONMENTS/scDREAMER.yml (conda env create -f scDREAMER.yml; conda activate scdreamer). For benchmarking, run eleven baselines (scVI, Scanorama, Harmony, Seurat, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, scDML) with published configurations, then run Louvain clustering at the resolution maximizing NMI and compute bio-conservation (NMI, ARI, scaled cell-type ASW), batch-correction (batch ASW, PCR batch, graph connectivity, kBET with k=50, graph iLISI/cLISI), and isolated-label metrics before min-max normalizing into composite scores.

## Related tools

- **scDREAMER** (Adversarial variational autoencoder trained to produce batch-corrected cellular embeddings for unsupervised integration) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing: AnnData import, mitochondrial QC filtering, normalize_total, log-transform, top-2000 HVG selection (flavor='seurat'), and Louvain clustering)
- **scVI** (Deep variational autoencoder baseline run with published configuration for comparison)
- **Scanorama** (MNN-based baseline run with published configuration for comparison)
- **Harmony** (Baseline run with published configuration for comparison)
- **Seurat** (MNN-based baseline run with published configuration for comparison)
- **BBKNN** (MNN-based baseline run with published configuration for comparison)
- **INSCT** (Batch-aware triplet network baseline run with published configuration for comparison) — https://github.com/lkmklsmn/insct
- **LIGER** (Baseline run with published configuration for comparison)
- **iMAP** (GAN-plus-autoencoder baseline run with published configuration for comparison)
- **scANVI** (Baseline run with published configuration for comparison)
- **scGEN** (Baseline run with published configuration for comparison)
- **scDML** (Baseline run with published configuration for comparison)

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- Every metric is bounded as defined: NMI/ARI in [0,1], scaled cell-type ASW = (ASW+1)/2 in [0,1], and scaled kBET and batch ASW = 1-abs(ASW) in [0,1] with 1 indicating perfect mixing
- Composite scores (bio-conservation, batch-correction, isolated-label, combined) must be computable for all twelve methods, confirming the results table is complete
- On the pancreas task, scDREAMER should achieve the highest combined composite score, ~12.9% above the second-best method
- Louvain clustering resolution is chosen as the one that maximizes NMI against ground-truth cell labels
- The integrated embedding should separate the held-out/rare cell types (e.g., alpha and delta cells, DC subtypes) rather than mixing by batch

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- Hierarchical structure between different batch information is not yet utilized
- As a deep learning-based method, scDREAMER may require fine-tuning of some parameters for extracting the best performance
- Extension to multiomic datasets is planned but not yet implemented
- For large datasets like the healthy heart atlas, kBET could not be computed for some methods due to computational limitations in handling large numbers of cells and batches

## Evidence

- [methods] ADAM optimizer, batch size 128: [evidence span withheld — anchor status: not_found]
- [methods] Top 2000 HVG selection: [evidence span withheld — anchor status: not_found]
- [methods] UMI down-sampling ratio: [evidence span withheld — anchor status: not_found]
- [discussion] Prior knowledge of number of batches: "Our current model assumes prior knowledge of the number of batches. An important future direction would be to explore the unsupervised treatment of batch information"
- [readme] Reproducibility environment setup: "Creating conda environment using ./ENVIRONMENTS/scDREAMER.yml will install all the dependent packages and libraries."
- [methods] Louvain clustering for metrics: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
