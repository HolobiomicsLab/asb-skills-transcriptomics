---
name: clustering-metric-computation-vs-true-labels
description: Use when scRNA-seq cell embeddings have been produced by scGCL or one
  of its ablation variants (task=clustering, epochs=300) and you need agreement metrics
  against ground-truth cell labels to compare variants across datasets like Adam and
  Romanov.
when_to_use_negative:
- The input h5 file lacks true cell labels, so agreement metrics against ground truth
  cannot be computed
- The evaluation goal is gene imputation accuracy rather than clustering performance
  of cell embeddings
- The data are not scRNA-seq dropout-affected expression matrices of the kind scGCL's
  ZINB autoencoder is designed for
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Produces the cell embeddings (task=clustering) whose cluster assignments are
    scored against true labels; ablation variants isolate the ZINB encoder and the
    graph convolution/AFGRL components
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python
  role: Runtime for invoking main.py and computing the clustering agreement metrics
    over the embeddings
  repo: https://github.com/zehaoxiong123/scGCL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:c5907a22c4529d29f2374fda66d65438a03085416b0744a1539607e3a498f833
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - scGCL
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clustering-metric-computation-vs-true-labels

## Summary

Clusters the cell embeddings learned by scGCL (or its ablation variants such as Only_ZINB and the no-ZINB encoder) and computes clustering agreement metrics against the true cell labels stored in the h5 file, to quantify clustering performance as reported in the paper's Table 2.

## When to use

Use when scRNA-seq cell embeddings have been produced by scGCL or one of its ablation variants (task=clustering, epochs=300) and you need agreement metrics against ground-truth cell labels to compare variants across datasets like Adam and Romanov.

## When NOT to use

- The input h5 file lacks true cell labels, so agreement metrics against ground truth cannot be computed
- The evaluation goal is gene imputation accuracy rather than clustering performance of cell embeddings
- The data are not scRNA-seq dropout-affected expression matrices of the kind scGCL's ZINB autoencoder is designed for

## Inputs

- h5 file containing a matrix of scRNA-seq expression values, true labels, and other information (e.g. adam, romanov)
- Learned cell embeddings from scGCL or an ablation variant (ZINB encoder removed, or Only_ZINB with graph convolution and AFGRL removed)
- Command-line settings: task=clustering, epochs=300

## Outputs

- Cluster assignments over the learned cell embeddings
- Per-variant, per-dataset clustering agreement scores against true labels, assembled into a Table 2-style ablation comparison

## How to apply

Run scGCL's main.py with --task clustering and --epochs 300 on the target h5 dataset (e.g. --dataset adam or --dataset romanov), which contains the scRNA-seq expression matrix, true labels, and other information. Cluster the learned cell embeddings of each variant (full scGCL, no-ZINB encoder, Only_ZINB) using the same settings. Compute clustering agreement metrics between the resulting cluster assignments and the true labels from the h5 file for each variant-by-dataset combination. Assemble the scores into the Table 2 ablation comparison, mirroring how the paper evaluates 'clustering performance ... on 14 real scRNA-seq datasets'. Validation: the reproduced Table 2 should match the article's reported outcome, including Only_ZINB outperforming full scGCL on the Romanov dataset, which implies graph representation learning can sometimes disrupt the ZINB-derived probabilistic features.

## Related tools

- **scGCL** (Produces the cell embeddings (task=clustering) whose cluster assignments are scored against true labels; ablation variants isolate the ZINB encoder and the graph convolution/AFGRL components) — https://github.com/zehaoxiong123/scGCL
- **Python** (Runtime for invoking main.py and computing the clustering agreement metrics over the embeddings) — https://github.com/zehaoxiong123/scGCL

## Examples

```
python main.py --dataset romanov --epochs 300 --task clustering
```

## Evaluation signals

- The reproduced Table 2 variant-by-dataset clustering scores match the article's reported outcome
- Only_ZINB outperforms full scGCL on the Romanov dataset, as the paper reports
- The ZINB distribution is found to be necessary in the ablation on both Adam and Romanov
- All variants are clustered and scored under identical settings (--epochs 300, task=clustering) on the same h5 inputs
- The agreement metrics are computed against the true labels contained in the h5 file, not against labels inferred de novo

## Limitations

- On the Romanov dataset, Only_ZINB clustering results are better than full scGCL, implying the graph representation learning component can sometimes disrupt the probabilistic (ZINB) features of the data
- Results depend on the fixed training configuration used in the ablation (epochs=300, task=clustering); changing these settings is not validated by the reported Table 2
- The comparison is limited to the datasets used in the ablation (Adam and Romanov); scores on other datasets are not covered by the reported outcome

## Evidence

- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [readme] Downstream task. Supported tasks are: node, clustering, similarity: "Downstream task. Supported tasks are: node, clustering, similarity"
- [intro] we verify that scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets: "we verify that scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets"
