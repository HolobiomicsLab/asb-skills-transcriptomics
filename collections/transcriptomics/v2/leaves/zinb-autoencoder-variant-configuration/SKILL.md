---
name: zinb-autoencoder-variant-configuration
description: Use when dissecting which scGCL component (ZINB encoder, graph convolution,
  AFGRL contrastive learning) drives clustering/imputation results, i.e. an internal
  component-ablation on Adam/Romanov h5 scRNA-seq data rather than a cross-method
  benchmark.
when_to_use_negative:
- The goal is comparing scGCL against external imputation methods across the 14 scRNA-seq
  datasets rather than attributing performance to scGCL's internal components.
- The input h5 file lacks true labels, so clustering agreement metrics needed for
  the ablation validation cannot be computed.
- You need the full scGCL pipeline for production imputation rather than component-isolation
  experiments (the ablation variants deliberately drop graph convolution or AFGRL).
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Modified main.py used to train the no-ZINB and Only_ZINB ablation variants
    with task=clustering, epochs=300 on Adam and Romanov
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python
  role: Runtime for executing main.py from the command line to train each ablation
    variant
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:8a2ff16b27617a9a961f281448cbac78c5635c010845d12373a25dd2382eed56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - scGCL
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# zinb-autoencoder-variant-configuration

## Summary

Configure and train scGCL ablation variants that isolate the ZINB autoencoder from the graph-convolution/AFGRL contrastive components, evaluating each variant's clustering on h5 scRNA-seq datasets such as Adam and Romanov. This tests whether the Zero-inflated Negative Binomial distribution itself, rather than graph representation learning, drives imputation/clustering gains.

## When to use

Use when dissecting which scGCL component (ZINB encoder, graph convolution, AFGRL contrastive learning) drives clustering/imputation results, i.e. an internal component-ablation on Adam/Romanov h5 scRNA-seq data rather than a cross-method benchmark.

## When NOT to use

- The goal is comparing scGCL against external imputation methods across the 14 scRNA-seq datasets rather than attributing performance to scGCL's internal components.
- The input h5 file lacks true labels, so clustering agreement metrics needed for the ablation validation cannot be computed.
- You need the full scGCL pipeline for production imputation rather than component-isolation experiments (the ablation variants deliberately drop graph convolution or AFGRL).

## Inputs

- Adam dataset h5 file (scRNA-seq expression matrix with true labels)
- Romanov dataset h5 file (scRNA-seq expression matrix with true labels)
- Modified scGCL source supporting ablation variants (ZINB-only and no-ZINB)

## Outputs

- Cell embeddings for each ablation variant on each dataset
- Clustering agreement metrics computed against true labels
- Table 2 ablation comparison (variant x dataset clustering scores)

## How to apply

Load the Adam or Romanov h5 file, which the README defines as containing a matrix of scRNA-seq expression values plus true labels. Train ablation variant (i) by removing the ZINB-based encoder while retaining graph convolution and AFGRL, and variant (ii) Only_ZINB by removing graph convolution and AFGRL while keeping the ZINB autoencoder, using the modified scGCL main.py with task=clustering and epochs=300 (e.g. --dataset adam --epochs 300). Repeat both variants on the other dataset so each variant-by-dataset cell is filled. Cluster the learned cell embeddings and compute clustering agreement metrics against the true labels in the h5 file, then assemble the scores into the Table 2 ablation comparison. Validate by checking that the reproduced Table 2 shows the ZINB distribution is necessary and that Only_ZINB outperforms full scGCL on Romanov, implying graph representation learning sometimes disrupts the data's probabilistic features.

## Related tools

- **scGCL** (Modified main.py used to train the no-ZINB and Only_ZINB ablation variants with task=clustering, epochs=300 on Adam and Romanov) — https://github.com/zehaoxiong123/scGCL
- **Python** (Runtime for executing main.py from the command line to train each ablation variant)

## Examples

```
python main.py --dataset adam --epochs 300 --task clustering
```

## Evaluation signals

- Reproduced Table 2 matches the article's reported ablation outcome.
- Only_ZINB outperforms full scGCL on the Romanov dataset, confirming graph representation learning sometimes disrupts probabilistic features.
- Removing the ZINB encoder degrades clustering relative to ZINB-retaining variants, showing the ZINB distribution is necessary.
- Both variants trained under identical settings: epochs=300, task=clustering, same h5 inputs per dataset.
- Clustering agreement metrics are computed against the true labels embedded in the h5 file, not inferred labels.

## Limitations

- Graph representation learning can disrupt the probabilistic (ZINB) features of the data, so the full model may underperform the Only_ZINB variant on some datasets (observed on Romanov).
- The ablation conclusion is grounded in the Adam and Romanov datasets; extrapolation to the other scRNA-seq datasets is not established by this card.
- The dataset argument expects a h5 file containing expression values, true labels, and other information; other input formats are not supported by the documented interface.

## Evidence

- [readme] README task argument: "Downstream task. Supported tasks are: node, clustering, similarity"
- [readme] README dataset argument: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [methods] CLI usage: "$ python main.py --dataset adam --epochs 300"
- [intro] ZINB autoencoder rationale: "To capture the global probability distribution, scGCL introduces an autoencoder based on the ZINB distribution, which reconstructs the scRNA-seq data based on the prior distribution"
