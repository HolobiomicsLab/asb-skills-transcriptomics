---
name: multi-omic-batch-correction-benchmarking
description: Use when comparing batch-correction methods on paired single-cell multi-omic
  data (e.g., 10X Genomics PBMC scRNA-seq + scATAC-seq peak matrices) and need reproducible
  integration metrics like ARI, NMI, Batch ASW, and CellType ASW.
when_to_use_negative:
- The task is single-modality scRNA-seq integration only, where the multi-omic gene-activity
  conversion step is unnecessary and scATAC-seq peak data is absent
- The modalities are CITE-seq protein panels with only a few hundred features, where
  scCobra's shared-encoder approach struggles to identify shared features
- The goal is a tuned performance comparison — benchmarks here were run with default
  parameters, which may not reflect each method's optimized performance
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: scCobra
  role: Deep generative model (contrastive learning + domain adaptation) under benchmark;
    three-phase training optimization for multi-omic integration
  repo: https://github.com/mcgilldinglab/scCobra
- name: Seurat v3.2.3
  role: Benchmark integration method; data normalization and selection of 2000 highly
    variable genes via 'vst'
- name: Harmony
  role: Benchmark integration method run with default parameters
- name: Scanorama
  role: Benchmark integration method run with default parameters
- name: scVI (scvi-tools v0.11)
  role: Benchmark VAE-based integration method adjusting the 'n_batch' parameter
- name: scDML
  role: Benchmark integration method run with default parameters
- name: scDREAMER
  role: Benchmark integration method run with default parameters
- name: MAESTRO
  role: Converts scATAC-seq open chromatin peak profiles into a gene activity matrix
    for multimodal integration
- name: scIB
  role: Pipeline for Leiden clustering across resolutions 0.1–2.0 and computing integration
    metrics
- name: Scanpy
  role: Normalization of counts per cell to 10,000 with log1p transformation and preprocessing
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:2d28d85396148035d743d0eacf7f3233441501a552eb6efa381f8088c2c44701
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - MAESTRO
  - PyTorch
  - Scanorama
  - Scanpy
  - Seurat
  - Seurat v3.2.3
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  - scVI (scvi-tools v0.11)
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# multi-omic-batch-correction-benchmarking

## Summary

Benchmark single-cell multi-omic integration methods (scCobra, scVI, Seurat, Harmony, Scanorama, scDML, scDREAMER) on paired scRNA-seq + scATAC-seq data, scoring batch mixing versus biological conservation with the scIB metric suite.

## When to use

Use when comparing batch-correction methods on paired single-cell multi-omic data (e.g., 10X Genomics PBMC scRNA-seq + scATAC-seq peak matrices) and need reproducible integration metrics like ARI, NMI, Batch ASW, and CellType ASW.

## When NOT to use

- The task is single-modality scRNA-seq integration only, where the multi-omic gene-activity conversion step is unnecessary and scATAC-seq peak data is absent
- The modalities are CITE-seq protein panels with only a few hundred features, where scCobra's shared-encoder approach struggles to identify shared features
- The goal is a tuned performance comparison — benchmarks here were run with default parameters, which may not reflect each method's optimized performance

## Inputs

- Paired 10X Genomics PBMC scRNA-seq count matrix and scATAC-seq peak matrix (Cell Ranger example data)
- Preprocessed gene activity matrix derived from scATAC-seq peaks via MAESTRO

## Outputs

- Integrated embeddings from each of the seven benchmark methods
- batch_integration_metrics.csv with Batch mixing score, Batch ASW, ARI, NMI, CellType ASW and summed ARI+NMI+CellType ASW score per method
- UMAP embeddings of integrated data for visual comparison (e.g., reproducing Fig. 4b/4c ranking)

## How to apply

Preprocess the paired 10X Genomics PBMC counts by filtering cells with fewer than 200 genes, genes detected in fewer than 3 cells, and cells with mitochondrial fraction above 5%; normalize counts per cell to 10,000 with a log1p transformation in Scanpy. Convert the scATAC-seq open chromatin peak matrix into a gene activity matrix using the MAESTRO package, then treat RNA and ATAC as two batches and select 2000 highly variable genes. Run all benchmark methods with default parameters — scCobra (three-phase training optimization), Seurat v3.2.3 (2000 HVGs via 'vst'), Harmony, Scanorama, scVI (scvi-tools v0.11, 'n_batch' parameter), scDML, and scDREAMER. Cluster each method's integrated embedding with Leiden across resolutions 0.1 to 2.0 in increments of 0.1 via the scIB pipeline, compute Batch mixing score, Batch ASW, ARI, NMI, and CellType ASW from two-dimensional UMAP embeddings following the SCALEX methodology, and aggregate into a summed ARI+NMI+CellType ASW score.

## Related tools

- **scCobra** (Deep generative model (contrastive learning + domain adaptation) under benchmark; three-phase training optimization for multi-omic integration) — https://github.com/mcgilldinglab/scCobra
- **Seurat v3.2.3** (Benchmark integration method; data normalization and selection of 2000 highly variable genes via 'vst')
- **Harmony** (Benchmark integration method run with default parameters)
- **Scanorama** (Benchmark integration method run with default parameters)
- **scVI (scvi-tools v0.11)** (Benchmark VAE-based integration method adjusting the 'n_batch' parameter)
- **scDML** (Benchmark integration method run with default parameters)
- **scDREAMER** (Benchmark integration method run with default parameters)
- **MAESTRO** (Converts scATAC-seq open chromatin peak profiles into a gene activity matrix for multimodal integration)
- **scIB** (Pipeline for Leiden clustering across resolutions 0.1–2.0 and computing integration metrics)
- **Scanpy** (Normalization of counts per cell to 10,000 with log1p transformation and preprocessing)

## Examples

```
conda create -n scCobra conda-forge::python=3.10; pip install scanpy scib; git clone https://github.com/mcgilldinglab/scCobra.git
```

## Evaluation signals

- Reproduced ranking: Harmony, Scanorama, and scDREAMER struggle with modality differences, whereas scCobra, scVI, and Seurat integrate multi-omic data effectively (Fig. 4b)
- Aggregated summed ARI+NMI+CellType ASW score computed for all seven methods and saved as batch_integration_metrics.csv
- Leiden clustering sweep covers resolutions 0.1 to 2.0 in increments of 0.1 for every method's embedding
- Metrics computed from two-dimensional UMAP embeddings following the SCALEX methodology, matching Fig. 4b/4c values
- Preprocessing invariants hold: cells <200 genes removed, genes <3 cells removed, mitochondrial fraction >5% cells removed, counts normalized to 10,000 with log1p

## Limitations

- Converting scATAC-seq peak matrices into gene activity matrices is contingent on the conversion algorithm and may lead to information loss
- Benchmarks were run with default parameters, which may not reflect tuned performance of the compared methods
- Over-correction scores derived from two-dimensional UMAP embeddings introduce dependence on UMAP
- Harmony, Scanorama, and scDREAMER struggle with significant modality differences on multi-omic data

## Evidence

- [results] On the 10X Genomics PBMC multi-omic dataset, Harmony, Scanorama, and scDREAMER struggle with significant modality differences, whereas scCobra, scVI, and Seurat are all capable of effectively integrating multi-omic data (Fig. 4b).: [evidence span withheld — anchor status: not_found]
- [methods] conversion of scATAC-seq to gene activity matrix: "we converted the open chromatin profiles from scATAC-seq into a gene activity matrix utilizing the MAESTRO package"
- [discussion] scIB Leiden clustering sweep: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [readme] README install instructions: "pip install scanpy scib
# You can install addtional packages: https://scib.readthedocs.io/en/latest/index.html"
- [discussion] UMAP-derived scores per SCALEX: "The scores were derived from two-dimensional UMAP embeddings, following the methodology described in the original SCALEX26 paper"
