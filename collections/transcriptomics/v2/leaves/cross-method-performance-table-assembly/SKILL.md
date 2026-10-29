---
name: cross-method-performance-table-assembly
description: Use when multiple scRNA-seq clustering methods (e.g., scDCC, CIDR, DCA+k-means,
  DEC, MPSSC, PCA+k-means, SIMLR, SC3, Seurat) have been run on the same labeled datasets
  and their per-method metric values must be tabulated for comparison.
when_to_use_negative:
- The datasets have no ground-truth cell labels — NMI, CA, and ARI all require reference
  labels and cannot be computed for purely unlabeled data
- Only one method was run — a cross-method comparison table requires cluster assignments
  from all competing methods on identical cell sets
- Metrics were computed on different cell subsets per method (e.g., different random
  2100-cell samples per baseline) — values are then not comparable in a single table
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDCC
  role: 'Method under test: run in unsupervised mode (0 constraints) and with generated
    soft pairwise constraints before metric computation'
  repo: https://github.com/ttgump/scDCC
- name: CIDR
  role: Competing baseline whose clustering output is scored on the same 2100-cell
    samples
  repo: https://github.com/VCCRI/CIDR
- name: DCA
  role: Competing baseline used as DCA+k-means on the same samples
  repo: https://github.com/theislab/dca
- name: DEC
  role: Competing deep-embedding baseline scored alongside scDCC
  repo: https://github.com/XifengGuo/DEC-keras
- name: MPSSC
  role: Competing spectral-clustering baseline included in the comparison table
  repo: https://github.com/ishspsy/project
- name: scvi-tools
  role: Competing baseline used as SCVI+k-means on the same samples
  repo: https://github.com/scverse/scvi-tools
- name: scvis
  role: Competing baseline used as SCVIS+k-means on the same samples
  repo: https://github.com/shahcompbio/scvis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:ab75c88061795b5aca6c432e3f67ae2cef8b35c300198c985087609c3a3cb44b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - CIDR
  - DCA
  - DEC
  - MPSSC
  - Nvidia Tesla P100
  - Python
  - Scanpy
  - scDCC
  - scVI
  - scvi-tools
  - scvis
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-method-performance-table-assembly

## Summary

Assembles a single benchmark table of clustering metrics (NMI, CA, ARI) computed per method against ground-truth cell labels, as done when scDCC was compared to twelve competing scRNA-seq clustering baselines on down-sampled 2100-cell datasets.

## When to use

Use when multiple scRNA-seq clustering methods (e.g., scDCC, CIDR, DCA+k-means, DEC, MPSSC, PCA+k-means, SIMLR, SC3, Seurat) have been run on the same labeled datasets and their per-method metric values must be tabulated for comparison.

## When NOT to use

- The datasets have no ground-truth cell labels — NMI, CA, and ARI all require reference labels and cannot be computed for purely unlabeled data
- Only one method was run — a cross-method comparison table requires cluster assignments from all competing methods on identical cell sets
- Metrics were computed on different cell subsets per method (e.g., different random 2100-cell samples per baseline) — values are then not comparable in a single table

## Inputs

- 2100-cell down-sampled scRNA-seq count datasets from the four small datasets (figshare deposit https://figshare.com/articles/dataset/scDCC_data/21563517)
- Ground-truth cell labels for each dataset
- Cluster assignments produced by each method (scDCC with 0 and with pairwise constraints, plus the twelve competing baselines) on the same cells

## Outputs

- benchmark_comparison_table.csv containing per-method NMI, CA, and ARI values for each of the four small datasets

## How to apply

Run every method — scDCC in unsupervised mode (0 constraints, equivalent to scDeepCluster) and with soft pairwise constraints (setting --n_pairwise, --gamma, --ml_weight, --cl_weight) plus the twelve competing baselines (CIDR, DCA+k-means, DEC, MPSSC, PCA+k-means, SCVI+k-means, SCVIS+k-means, SIMLR, SC3, Seurat, COP K-means, MPC K-means) — on identical 2100-cell down-sampled samples from each of the four small datasets. Compute NMI, CA, and ARI for every method against the ground-truth cell labels on each dataset, so all methods are scored on the same cells and the metrics are directly comparable. Assemble the per-method metric values into a single comparison table and save it as benchmark_comparison_table.csv. Judge correctness by verifying from the table that scDCC with pairwise constraints attains higher NMI, CA, and ARI than all competing methods, consistent with the paper's reported result that scDCC significantly improves clustering performance.

## Related tools

- **scDCC** (Method under test: run in unsupervised mode (0 constraints) and with generated soft pairwise constraints before metric computation) — https://github.com/ttgump/scDCC
- **CIDR** (Competing baseline whose clustering output is scored on the same 2100-cell samples) — https://github.com/VCCRI/CIDR
- **DCA** (Competing baseline used as DCA+k-means on the same samples) — https://github.com/theislab/dca
- **DEC** (Competing deep-embedding baseline scored alongside scDCC) — https://github.com/XifengGuo/DEC-keras
- **MPSSC** (Competing spectral-clustering baseline included in the comparison table) — https://github.com/ishspsy/project
- **scvi-tools** (Competing baseline used as SCVI+k-means on the same samples) — https://github.com/scverse/scvi-tools
- **scvis** (Competing baseline used as SCVIS+k-means on the same samples) — https://github.com/shahcompbio/scvis

## Examples

```
import pandas as pd; pd.DataFrame(rows, columns=['method','dataset','NMI','CA','ARI']).to_csv('benchmark_comparison_table.csv', index=False)
```

## Evaluation signals

- benchmark_comparison_table.csv contains rows for scDCC (0 constraints), scDCC (pairwise constraints), and all twelve baselines across the four 2100-cell datasets
- Every NMI, CA, and ARI value falls in the valid [0, 1] range for clustering metrics against ground truth
- scDCC with pairwise constraints attains strictly higher NMI, CA, and ARI than every competing method on each of the four datasets, matching the paper's reported result
- All methods were scored against the same ground-truth labels on the same 2100-cell samples, so per-dataset metric comparisons are apples-to-apples
- Saved table reproduces the relative ordering seen in the paper's figures S1–S7 (scDCC 0 constraints outperforms most baselines; constrained scDCC outperforms all)

## Limitations

- Results are tied to randomly selected 2100-cell down-sampled samples of the four small datasets; sampling variability means a different random draw can shift metric values
- Reproducing the scDCC runs requires the specific stack reported in the paper — Python 3.6.8, PyTorch 1.5.1+cu101, Scanpy 1.0.4, and an Nvidia Tesla P100
- The paper's discussion section content is absent from the provided material, so caveats stated only in the discussion cannot be grounded here

## Evidence

- [other] Compute NMI, CA, and ARI for every method against the ground-truth cell labels on each dataset.: [evidence span withheld — anchor status: not_found]
- [other] Assemble the per-method metric values into a single comparison table and save it as benchmark_comparison_table.csv.: [evidence span withheld — anchor status: not_found]
- [methods] Datasets used in the study is available in the figshare deposit.: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [abstract] scDCC significantly improves clustering performance across dataset sizes.: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
