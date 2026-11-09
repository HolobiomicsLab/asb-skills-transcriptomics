---
name: scrnaseq-dropout-masking
description: Use when benchmarking gene-imputation methods on scRNA-seq count matrices
  (h5 format) under controlled synthetic dropout, e.g. to verify a method recovers
  masked non-zero entries better than baselines.
when_to_use_negative:
- Data is bulk RNA-seq or single-cell data without excessive dropouts, where masking
  zero-valued (biological) entries would create artificial non-zero targets the methods
  are not designed to impute
- Goal is downstream clustering or cell-type labeling rather than quantitative imputation
  benchmarking (scGCL supports a separate 'task' argument for node/clustering/similarity)
- Raw counts have not passed quality-control filtering (>1% non-zero expression for
  genes and cells) and size-factor log normalization plus top-2048 HVG selection via
  scanpy, so masking would be applied to an unprocessed matrix inconsistent with the
  published protocol
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: scGCL
  role: Candidate imputation method whose recovery of masked entries is benchmarked
    under synthetic dropout
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python
  role: Implementation language for dropout masking, running imputation and computing
    metrics
- name: PyTorch
  role: Deep-learning backend (version 1.10.0) for scGCL training on masked data
- name: torch-geometric
  role: Graph-learning library (version 2.2.0) supporting scGCL's graph contrastive
    learning during imputation
- name: faiss-cpu
  role: Nearest-neighbor library (version 1.7.3) used in the scGCL dependency stack
- name: AutoClass
  role: Baseline imputation method run on identically masked data for comparison
- name: GraphSCI
  role: Baseline imputation method run on identically masked data for comparison
- name: MAGIC
  role: Baseline imputation method run on identically masked data for comparison
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:7fa912b5d6cbf7753745d56ab8ed7e282af4455c9c0eb50003d5ca40db5be76c
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
  - scanpy
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# scrnaseq-dropout-masking

## Summary

Simulates scRNA-seq dropout by randomly masking non-zero expression entries at 10%, 30% and 50% rates (scVI-style strategy), retaining true values as ground truth so that imputation methods like scGCL, AutoClass, GraphSCI and MAGIC can be quantitatively compared on masked-entry recovery.

## When to use

Use when benchmarking gene-imputation methods on scRNA-seq count matrices (h5 format) under controlled synthetic dropout, e.g. to verify a method recovers masked non-zero entries better than baselines.

## When NOT to use

- Data is bulk RNA-seq or single-cell data without excessive dropouts, where masking zero-valued (biological) entries would create artificial non-zero targets the methods are not designed to impute
- Goal is downstream clustering or cell-type labeling rather than quantitative imputation benchmarking (scGCL supports a separate 'task' argument for node/clustering/similarity)
- Raw counts have not passed quality-control filtering (>1% non-zero expression for genes and cells) and size-factor log normalization plus top-2048 HVG selection via scanpy, so masking would be applied to an unprocessed matrix inconsistent with the published protocol

## Inputs

- scRNA-seq dataset in h5 format (expression matrix, true labels, other information)
- Dropout rates: 10%, 30% and 50% of non-zero expression values
- Baseline imputation outputs on the same masked matrices (AutoClass, GraphSCI, MAGIC)

## Outputs

- Three dropout-masked copies of the expression matrix (10%, 30%, 50%) with held-out ground-truth values
- Median L1 distance, RMSE and cosine similarity per method per dropout rate
- Aggregated results table (imputation_results.csv)

## How to apply

Load the scRNA-seq dataset from its h5 file (containing the expression matrix and true labels), then apply the scVI-style dropout strategy: randomly mask non-zero expression entries at 10%, 30% and 50% rates while retaining the true values as ground truth. Run the candidate method (e.g. scGCL via `python main.py --dataset <dataset> --epochs 300`) and baselines (AutoClass, GraphSCI, MAGIC) on the identically masked matrices. Compute median L1 distance, RMSE and cosine similarity between imputed and true values at each dropout rate for every method, and assemble the metrics into a results table (imputation_results.csv). The rationale is that masking only previously non-zero entries creates a controlled recovery task where ground truth is known exactly, avoiding confounds from true biological zeros. Acceptance follows the paper's check: the best methods achieve the lowest median L1 distance and RMSE at all three dropout rates.

## Related tools

- **scGCL** (Candidate imputation method whose recovery of masked entries is benchmarked under synthetic dropout) — https://github.com/zehaoxiong123/scGCL
- **Python** (Implementation language for dropout masking, running imputation and computing metrics)
- **PyTorch** (Deep-learning backend (version 1.10.0) for scGCL training on masked data)
- **torch-geometric** (Graph-learning library (version 2.2.0) supporting scGCL's graph contrastive learning during imputation)
- **faiss-cpu** (Nearest-neighbor library (version 1.7.3) used in the scGCL dependency stack)
- **AutoClass** (Baseline imputation method run on identically masked data for comparison)
- **GraphSCI** (Baseline imputation method run on identically masked data for comparison)
- **MAGIC** (Baseline imputation method run on identically masked data for comparison)

## Examples

```
python main.py --dataset zeisel --epochs 300
```

## Evaluation signals

- Masked entries are drawn only from previously non-zero expression values (scVI strategy), so ground truth for every masked entry is a known non-zero value
- All methods (scGCL, AutoClass, GraphSCI, MAGIC) are evaluated on the identically masked matrices at the same three rates (10%, 30%, 50%)
- Median L1 distance and RMSE should be lowest for the best method (scGCL) at all three dropout rates on Zeisel and Klein datasets
- Cosine similarity should be competitive with the strongest baselines (AutoClass and MAGIC) even where it is not the absolute best
- Metrics assemble into a single results file (imputation_results.csv) covering all method x dropout-rate combinations

## Limitations

- Results are specific to simulated dropout on Zeisel and Klein datasets; cosine similarity of scGCL is only 'competitive with' AutoClass and MAGIC, not uniformly best
- Synthetic dropout assumes masked non-zero entries mimic real scRNA-seq dropout; true biological zeros are not distinguished in this benchmark
- The pipeline depends on prior pre-processing (QC filter of >1% non-zero expression for genes and cells, size-factor log normalization, top t=2048 highly variable genes via scanpy), so masking a raw unfiltered matrix is out of protocol

## Evidence

- [full_text] Apply the scVI-style dropout strategy, randomly masking non-zero expression entries at 10%, 30% and 50% rates while retaining the true values as ground truth.: [evidence span withheld — anchor status: not_found]
- [results] In simulated dropout recovery on the Zeisel and Klein datasets, scGCL achieves the best median L1 distance and RMSE scores at all tested synthetic dropout rates: [evidence span withheld — anchor status: not_found]
- [methods] $ python main.py --dataset adam --epochs 300: "$ python main.py --dataset adam --epochs 300"
- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
