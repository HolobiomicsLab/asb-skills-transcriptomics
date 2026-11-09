---
name: baseline-imputation-method-reproduction
description: 'Use when benchmarking scRNA-seq imputation methods on simulated dropout:
  non-zero expression entries are masked at 10/30/50% rates and imputed values must
  be compared against retained ground truth.'
when_to_use_negative:
- When the goal is evaluating imputation on real, unmasked scRNA-seq data (e.g. downstream
  clustering on 14 real datasets or gene-expression enhancement in Alzheimer's disease
  datasets) rather than synthetic-dropout recovery against known ground truth
- When the downstream task is similarity search or node classification — the scGCL
  CLI supports task=node/clustering/similarity, and this skill covers only the gene-imputation
  comparison workflow
- When you need to choose among scRNA-seq imputation methods conceptually rather than
  reproduce the specific AutoClass/GraphSCI/MAGIC baseline set on Zeisel and Klein
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGCL
  role: Target imputation method under test; graph contrastive learning with a ZINB
    autoencoder used to estimate dropout values on the masked data
  repo: https://github.com/zehaoxiong123/scGCL
- name: AutoClass
  role: Baseline imputation method run on identically masked data for comparison
- name: GraphSCI
  role: Baseline imputation method run on identically masked data for comparison
- name: MAGIC
  role: Baseline imputation method run on identically masked data for comparison
- name: Python
  role: Runtime environment (Python 3.8.8) for running scGCL and assembling imputation_results.csv
- name: PyTorch
  role: Deep-learning framework (version 1.10.0) underpinning scGCL training
- name: torch-geometric
  role: Graph-learning library (version 2.2.0) used by scGCL for graph contrastive
    learning
- name: torch-cluster
  role: Supporting dependency (version 1.6.0) for scGCL's graph construction
- name: torch-scatter
  role: Supporting dependency (version 2.0.9) for scGCL's graph message passing
- name: torch-sparse
  role: Supporting dependency (version 0.6.13) for scGCL's sparse graph operations
- name: faiss-cpu
  role: Nearest-neighbour library (version 1.7.3) used in scGCL's environment for
    positive-sample selection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:85565285016b048b9dca840d3964bb6659425ed1016f34b97560d4ebc585a411
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - AutoClass
  - GraphSCI
  - MAGIC
  - PyTorch
  - Python
  - faiss-cpu
  - scGCL
  - torch-cluster
  - torch-geometric
  - torch-scatter
  - torch-sparse
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# baseline-imputation-method-reproduction

## Summary

Reproduce the dropout-imputation baseline comparison protocol from the scGCL paper: apply the scVI-style synthetic dropout strategy to scRNA-seq h5 data, run scGCL and the baseline imputers (AutoClass, GraphSCI, MAGIC) on identically masked data, and score them with median L1 distance, RMSE and cosine similarity.

## When to use

Use when benchmarking scRNA-seq imputation methods on simulated dropout: non-zero expression entries are masked at 10/30/50% rates and imputed values must be compared against retained ground truth.

## When NOT to use

- When the goal is evaluating imputation on real, unmasked scRNA-seq data (e.g. downstream clustering on 14 real datasets or gene-expression enhancement in Alzheimer's disease datasets) rather than synthetic-dropout recovery against known ground truth
- When the downstream task is similarity search or node classification — the scGCL CLI supports task=node/clustering/similarity, and this skill covers only the gene-imputation comparison workflow
- When you need to choose among scRNA-seq imputation methods conceptually rather than reproduce the specific AutoClass/GraphSCI/MAGIC baseline set on Zeisel and Klein

## Inputs

- scRNA-seq dataset stored as an h5 file containing the expression matrix, true labels and other information
- Masked (dropout) expression matrices generated at 10%, 30% and 50% dropout rates
- Ground-truth expression values retained for masked entries

## Outputs

- Imputed expression matrices from scGCL, AutoClass, GraphSCI and MAGIC at each dropout rate
- imputation_results.csv table of median L1 distance, RMSE and cosine similarity per method per dropout rate

## How to apply

Load each scRNA-seq dataset from its h5 file containing the expression matrix and true labels. Apply the scVI-style dropout strategy, randomly masking non-zero expression entries at 10%, 30% and 50% rates while retaining the true values as ground truth. Impute the masked data once with scGCL (PyTorch/torch-geometric graph contrastive learning with a ZINB autoencoder) and once with each baseline method — AutoClass, GraphSCI and MAGIC — on the identically masked matrices. For every method at every dropout rate, compute the median L1 distance, RMSE and cosine similarity between imputed and true values, then assemble the metrics into a results table (imputation_results.csv). The comparison is judged correct when scGCL achieves the best (lowest) median L1 distance and RMSE at all three dropout rates on Zeisel and Klein, with cosine similarity competitive with AutoClass and MAGIC.

## Related tools

- **scGCL** (Target imputation method under test; graph contrastive learning with a ZINB autoencoder used to estimate dropout values on the masked data) — https://github.com/zehaoxiong123/scGCL
- **AutoClass** (Baseline imputation method run on identically masked data for comparison)
- **GraphSCI** (Baseline imputation method run on identically masked data for comparison)
- **MAGIC** (Baseline imputation method run on identically masked data for comparison)
- **Python** (Runtime environment (Python 3.8.8) for running scGCL and assembling imputation_results.csv)
- **PyTorch** (Deep-learning framework (version 1.10.0) underpinning scGCL training)
- **torch-geometric** (Graph-learning library (version 2.2.0) used by scGCL for graph contrastive learning)
- **torch-cluster** (Supporting dependency (version 1.6.0) for scGCL's graph construction)
- **torch-scatter** (Supporting dependency (version 2.0.9) for scGCL's graph message passing)
- **torch-sparse** (Supporting dependency (version 0.6.13) for scGCL's sparse graph operations)
- **faiss-cpu** (Nearest-neighbour library (version 1.7.3) used in scGCL's environment for positive-sample selection)

## Examples

```
python main.py --dataset klein --epochs 300
```

## Evaluation signals

- scGCL achieves the best (lowest) median L1 distance at all three tested synthetic dropout rates (10%, 30% and 50%) on both Zeisel and Klein
- scGCL achieves the best (lowest) RMSE at all three dropout rates on both datasets
- Cosine similarity for scGCL is competitive with AutoClass and MAGIC rather than markedly worse
- All methods are scored on the identically masked matrices — the same masked entries at each dropout rate — so per-entry comparisons are valid
- Results table imputation_results.csv contains one row per method per dropout rate for all three metrics

## Limitations

- Dropout is synthetic (scVI-style masking of non-zero values), so recovery performance reflects simulated dropout rather than naturally occurring technical dropouts
- Results are grounded in the Zeisel and Klein datasets; the paper's broader claims (clustering and imputation on 14 real datasets) are separate from this dropout-recovery evaluation
- Cosine similarity is only competitive, not dominant, for scGCL — the strongest claims are limited to median L1 distance and RMSE

## Evidence

- [readme] scVI-style dropout strategy with rates: [evidence span withheld — anchor status: not_found]
- [results] baseline set: [evidence span withheld — anchor status: not_found]
- [readme] h5 input format: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [methods] scGCL CLI: "$ python main.py --dataset adam --epochs 300"
- [abstract] ZINB method description: "scGCL, which integrates graph contrastive learning and Zero-inflated Negative Binomial (ZINB) distribution to estimate dropout values"
