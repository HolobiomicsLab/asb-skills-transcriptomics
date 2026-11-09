---
name: clustering-benchmark-evaluation
description: Use when a new single-cell clustering or batch-correction method must
  be compared quantitatively against published baselines on multiple real scRNA-seq
  datasets with known cell-type labels.
when_to_use_negative:
- Datasets lack ground-truth cell-type labels, so NMI/ARI cannot be computed and only
  embedding-quality metrics are possible
- The goal is a single-dataset application rather than a multi-dataset method comparison
  against published baselines
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scMGCA
  role: Method under test; graph-embedding autoencoder clustering run on all 20 datasets
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: 'Preprocessing: filter unexpressed genes, normalize, select top d=500 highly
    variable genes'
- name: scikit-learn
  role: Compute NMI and ARI against ground-truth labels; silhouette-based ASW on embeddings
- name: scDeepCluster
  role: Baseline clustering method run with default settings
  repo: https://github.com/ttgump/scDeepCluster
- name: DESC
  role: Baseline deep embedding clustering method run with default settings
  repo: https://github.com/eleozzr/desc
- name: scziDesk
  role: Baseline deep self-training K-means clustering method
  repo: https://github.com/xuebaliang/scziDesk
- name: DCA
  role: Baseline denoising autoencoder method
  repo: https://github.com/theislab/dca
- name: DEC
  role: Baseline unsupervised deep embedding clustering (Keras)
  repo: https://github.com/XifengGuo/DEC-keras
- name: scVI
  role: Baseline single-cell variational inference method
  repo: https://github.com/YosefLab/scVI
- name: scCAEs
  role: Baseline convolutional autoencoder clustering method
  repo: https://github.com/gushenweiz/scCAEs
- name: scGNN
  role: Baseline single-cell graph neural network method
  repo: https://github.com/juexinwang/scGNN
- name: Seurat
  role: Baseline R-based clustering method
- name: SHARP
  role: Baseline random-projection ensemble clustering method
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:3f7ea4737534b1488066e747edca311a5664f0b1c4d8353105f32c3d3d7fd899
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - DEC
  - DESC
  - SCANPY
  - SHARP
  - Seurat
  - anndata
  - numpy
  - pandas
  - scCAEs
  - scDeepCluster
  - scGNN
  - scMGCA
  - scVI
  - scikit-learn
  - scipy
  - scziDesk
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 22
---

# clustering-benchmark-evaluation

## Summary

Benchmark a single-cell clustering method (e.g., scMGCA) against state-of-the-art baselines across multiple scRNA-seq datasets, scoring NMI/ARI against ground-truth cell-type labels and ASW on learned embeddings, then assemble a per-dataset comparison table.

## When to use

Use when a new single-cell clustering or batch-correction method must be compared quantitatively against published baselines on multiple real scRNA-seq datasets with known cell-type labels.

## When NOT to use

- Datasets lack ground-truth cell-type labels, so NMI/ARI cannot be computed and only embedding-quality metrics are possible
- The goal is a single-dataset application rather than a multi-dataset method comparison against published baselines

## Inputs

- Preprocessed scRNA-seq count matrices (AnnData) for ~20 real datasets with ground-truth cell-type labels
- Published implementations of baseline methods (scziDesk, scDeepCluster, DCA, DEC, DESC, scVI, scCAEs, scGNN, scGAE, GraphSCC, Seurat, SHARP)
- Learned embedding matrices from each method for ASW computation

## Outputs

- clustering_benchmark_results.csv with per-dataset NMI, ARI, ASW for all 13 methods plus row-wise means
- Per-method mean NMI, ARI, ASW across all datasets

## How to apply

Preprocess each dataset in SCANPY: filter genes not expressed in cells, normalize the expression matrix, and retain the top d=500 highly variable genes. Run the method under test with the paper's settings (e.g., scMGCA with transfer step s=2, two GCN layers, KL-divergence self-optimized clustering; split datasets with >25,000 cells into batches). Run the 12 comparison methods (scziDesk, scDeepCluster, DCA, DEC, DESC, scVI, scCAEs, scGNN, scGAE, GraphSCC, Seurat, SHARP) from their published implementations with default settings. Compute NMI and ARI with scikit-learn against ground-truth cell-type labels and ASW on each method's learned embeddings, then assemble per-dataset scores plus row-wise means (e.g., scMGCA reached mean NMI 0.8304, ARI 0.8278, ASW 0.5827) into a CSV table.

## Related tools

- **scMGCA** (Method under test; graph-embedding autoencoder clustering run on all 20 datasets) — https://pypi.org/project/scMGCA
- **SCANPY** (Preprocessing: filter unexpressed genes, normalize, select top d=500 highly variable genes)
- **scikit-learn** (Compute NMI and ARI against ground-truth labels; silhouette-based ASW on embeddings)
- **scDeepCluster** (Baseline clustering method run with default settings) — https://github.com/ttgump/scDeepCluster
- **DESC** (Baseline deep embedding clustering method run with default settings) — https://github.com/eleozzr/desc
- **scziDesk** (Baseline deep self-training K-means clustering method) — https://github.com/xuebaliang/scziDesk
- **DCA** (Baseline denoising autoencoder method) — https://github.com/theislab/dca
- **DEC** (Baseline unsupervised deep embedding clustering (Keras)) — https://github.com/XifengGuo/DEC-keras
- **scVI** (Baseline single-cell variational inference method) — https://github.com/YosefLab/scVI
- **scCAEs** (Baseline convolutional autoencoder clustering method) — https://github.com/gushenweiz/scCAEs
- **scGNN** (Baseline single-cell graph neural network method) — https://github.com/juexinwang/scGNN
- **Seurat** (Baseline R-based clustering method)
- **SHARP** (Baseline random-projection ensemble clustering method)

## Examples

```
import scanpy as sc; from sklearn.metrics import normalized_mutual_info_score as nmi, adjusted_rand_score as ari, silhouette_score; adata = sc.read_h5ad('dataset.h5ad'); sc.pp.filter_genes(adata, min_cells=1); sc.pp.normalize_total(adata); sc.pp.highly_variable_genes(adata, n_top_genes=500)
```

## Evaluation signals

- Table reports NMI, ARI, and ASW for the method under test and all 12 baselines on all 20 datasets
- Method under test achieves the highest mean NMI/ARI/ASW across datasets (scMGCA reference: 0.8304 NMI, 0.8278 ARI, 0.5827 ASW)
- Method under test ranks highest on the majority of datasets (scMGCA: 15 of 20 datasets)
- Validation explicitly checks rare cell types and small-cluster datasets (e.g., 'QS heart', 'Muraro', 'Qx limb Muscle', 'Adam') for correct cell segregation
- Row-wise means computed across the 20 datasets match the published Source Data values

## Limitations

- Benchmark data only described as coming from public repositories with no specific dataset accessions provided
- No software version numbers reported (Python, TensorFlow, SCANPY, Seurat all unstated), so environment reproduction may vary
- Sample size not predetermined; the maximum number of samples available per dataset was used, with no power analysis

## Evidence

- [results] Across the 20 datasets, scMGCA achieved the highest mean NMI, ARI and ASW of all 13 methods, reaching 0.8304, 0.8278, and 0.5827: [evidence span withheld — anchor status: not_found]
- [discussion] The first step is to filter out genes that are not expressed in cells: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [results] scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells: "scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells"
- [results] four datasets ('QS heart', 'Muraro', 'Qx limb Muscle', and 'Adam') that contain rare cell types and small clusters: [evidence span withheld — anchor status: not_found]
- [discussion] the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA: "the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
