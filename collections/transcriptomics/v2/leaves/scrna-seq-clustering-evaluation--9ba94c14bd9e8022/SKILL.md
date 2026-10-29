---
name: scrna-seq-clustering-evaluation
description: Use when you need to evaluate scRNA-seq imputation/clustering models
  by clustering their learned cell embeddings and scoring agreement against true cell
  labels supplied in a h5 expression matrix (e.g., Adam, Romanov).
when_to_use_negative:
- The h5 input lacks true cell labels, so no clustering agreement metrics against
  ground truth can be computed
- The goal is gene imputation quality assessment or gene expression pattern enhancement
  (e.g., in Alzheimer's disease datasets) rather than clustering-based performance
  evaluation
- The data are not scRNA-seq dropout-affected count matrices; scGCL and its ZINB-based
  evaluation target single-cell RNA-seq data
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Main pipeline that produces the cell embeddings (and ablation variants no-ZINB
    / Only_ZINB) that are clustered and evaluated against true labels
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python 3.8.8
  role: Runtime environment in which main.py is invoked to train variants and compute
    clustering agreement metrics
- name: faiss-cpu 1.7.3
  role: Listed project dependency supporting the similarity/clustering downstream
    tasks of the scGCL pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:8fe6f18d2b0b5db6954f7ea951682a626481ff19c7782043543c621393241756
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - Python 3.8.8
  - faiss-cpu 1.7.3
  - scGCL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrna-seq-clustering-evaluation

## Summary

Cluster the learned cell embeddings produced by scGCL (or its ablation variants) and compute clustering agreement metrics against the true cell labels stored in the h5 expression file, to quantify imputation/clustering performance and assemble ablation comparisons such as Table 2.

## When to use

Use when you need to evaluate scRNA-seq imputation/clustering models by clustering their learned cell embeddings and scoring agreement against true cell labels supplied in a h5 expression matrix (e.g., Adam, Romanov).

## When NOT to use

- The h5 input lacks true cell labels, so no clustering agreement metrics against ground truth can be computed
- The goal is gene imputation quality assessment or gene expression pattern enhancement (e.g., in Alzheimer's disease datasets) rather than clustering-based performance evaluation
- The data are not scRNA-seq dropout-affected count matrices; scGCL and its ZINB-based evaluation target single-cell RNA-seq data

## Inputs

- h5 scRNA-seq expression files (e.g., Adam and Romanov datasets) containing the expression matrix, true labels, and other information
- Learned cell embeddings from scGCL and its ablation variants (no-ZINB, Only_ZINB)
- scGCL source code (main.py) configured for the clustering downstream task

## Outputs

- Cluster assignments of the learned cell embeddings per variant and dataset
- Clustering agreement scores of each variant against the true labels
- Variant-by-dataset clustering comparison table (Table 2 ablation comparison)

## How to apply

Load the h5 scRNA-seq expression file, which contains the expression matrix, true labels, and other information, for each dataset to be evaluated (e.g., Adam and Romanov). Generate cell embeddings by running the scGCL pipeline or a modified ablation variant (no-ZINB, or Only_ZINB with graph convolution and AFGRL removed) via main.py with --dataset <name> --epochs 300 and the clustering downstream task. Cluster the learned cell embeddings of each variant using the same settings across variants, then compute clustering agreement metrics against the true labels. Assemble the variant-by-dataset clustering scores into the ablation comparison table (Table 2). Validate the reproduction by checking that the ZINB distribution is necessary and that Only_ZINB can outperform full scGCL on the Romanov dataset, since graph representation learning sometimes disrupts the probabilistic features of the data.

## Related tools

- **scGCL** (Main pipeline that produces the cell embeddings (and ablation variants no-ZINB / Only_ZINB) that are clustered and evaluated against true labels) — https://github.com/zehaoxiong123/scGCL
- **Python 3.8.8** (Runtime environment in which main.py is invoked to train variants and compute clustering agreement metrics)
- **faiss-cpu 1.7.3** (Listed project dependency supporting the similarity/clustering downstream tasks of the scGCL pipeline)

## Examples

```
python main.py --dataset adam --epochs 300 --task clustering
```

## Evaluation signals

- The h5 dataset used for evaluation contains true labels alongside the expression matrix
- All variants (full scGCL, no-ZINB, Only_ZINB) are trained under identical command-line settings (--epochs 300, clustering task) so scores are comparable
- Reproduced Table 2 matches the article's reported outcome: the ZINB distribution is necessary, and Only_ZINB clustering outperforms full scGCL on the Romanov dataset
- Clustering agreement metrics are computed against the true labels for both the Adam and Romanov datasets
- Same embedding-clustering and metric computation procedure is applied consistently across all variant-by-dataset cells of the comparison table

## Limitations

- Graph representation learning sometimes disrupts the probabilistic features of the data, so Only_ZINB can achieve better clustering than full scGCL on some datasets (Romanov)
- Evaluation depends on true labels being present in the h5 expression file; datasets without ground-truth labels cannot be scored this way
- Results are tied to fixed training settings reported in the article (epochs=300, clustering task); other settings may not reproduce the reported outcomes

## Evidence

- [readme] dataset: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [methods] cli: "$ python main.py --dataset adam --epochs 300"
- [abstract] clustering_performance_claim: "scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 scRNA-seq datasets"
- [readme] task_argument: "task | Downstream task. Supported tasks are: node, clustering, similarity"
