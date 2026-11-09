---
name: imputation-accuracy-metric-computation
description: Use when you have an imputed scRNA-seq expression matrix plus the original
  values and a known synthetic-dropout mask, and need per-method accuracy scores (median
  L1, RMSE, cosine similarity) at each dropout rate.
when_to_use_negative:
- Evaluating imputation of naturally missing values, where no ground-truth true values
  exist — this skill requires known true values retained from synthetic masking
- Benchmarking clustering performance of imputation methods instead of gene-imputation
  accuracy — scGCL also supports clustering tasks (node, clustering, similarity) that
  need different metrics
- The input matrix has no non-zero entries to mask, or the data is not scRNA-seq expression
  in the h5 format scGCL expects
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Imputation method under evaluation; run from the command line to impute the
    masked entries whose accuracy is then scored
  repo: https://github.com/zehaoxiong123/scGCL
- name: AutoClass
  role: Baseline imputation method run on the identically masked data to provide comparison
    L1/RMSE/cosine scores
- name: GraphSCI
  role: Baseline imputation method run on the identically masked data to provide comparison
    L1/RMSE/cosine scores
- name: MAGIC
  role: Baseline imputation method run on the identically masked data to provide comparison
    L1/RMSE/cosine scores
- name: Python
  role: Environment in which scGCL is invoked and the metric computation and results
    table assembly are performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:977d504dd1e766fdc3aec632a56ddded0c4cd9fd40b4dab57f70a661ab828817
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# imputation-accuracy-metric-computation

## Summary

Computes median L1 distance, RMSE and cosine similarity between imputed and true values on synthetically masked scRNA-seq entries (10%, 30% and 50% scVI-style dropout) to benchmark imputation methods such as scGCL against AutoClass, GraphSCI and MAGIC.

## When to use

Use when you have an imputed scRNA-seq expression matrix plus the original values and a known synthetic-dropout mask, and need per-method accuracy scores (median L1, RMSE, cosine similarity) at each dropout rate.

## When NOT to use

- Evaluating imputation of naturally missing values, where no ground-truth true values exist — this skill requires known true values retained from synthetic masking
- Benchmarking clustering performance of imputation methods instead of gene-imputation accuracy — scGCL also supports clustering tasks (node, clustering, similarity) that need different metrics
- The input matrix has no non-zero entries to mask, or the data is not scRNA-seq expression in the h5 format scGCL expects

## Inputs

- h5 file containing a matrix of scRNA-seq expression values, true labels and other information
- Imputed expression matrices from scGCL and the baselines (AutoClass, GraphSCI, MAGIC) run on identically masked data
- Synthetic dropout mask (10%/30%/50% of non-zero entries) with the retained true values as ground truth

## Outputs

- imputation_results.csv with median L1 distance, RMSE and cosine similarity per method at each dropout rate (10%, 30%, 50%)
- Comparison verdict: whether scGCL attains the lowest median L1 and RMSE at all three dropout rates

## How to apply

Load the scRNA-seq dataset from its h5 file, which contains the expression matrix, true labels and other information. Apply the scVI-style dropout strategy: randomly mask 10%, 30% and 50% of non-zero expression entries to zero while retaining the true values as ground truth, so only entries that originally had expression are evaluated. Run scGCL from the command line (python main.py --dataset <dataset> --epochs 300) and run AutoClass, GraphSCI and MAGIC on the identically masked data so all methods are scored on the same masked entries. Compute the median L1 distance, RMSE and cosine similarity between imputed and true values for each method at each dropout rate, restricting comparisons to the masked positions. Assemble the metrics into imputation_results.csv and verify the expected pattern — scGCL achieves the best (lowest) median L1 and RMSE at all three rates, with cosine similarity competitive with AutoClass and MAGIC.

## Related tools

- **scGCL** (Imputation method under evaluation; run from the command line to impute the masked entries whose accuracy is then scored) — https://github.com/zehaoxiong123/scGCL
- **AutoClass** (Baseline imputation method run on the identically masked data to provide comparison L1/RMSE/cosine scores)
- **GraphSCI** (Baseline imputation method run on the identically masked data to provide comparison L1/RMSE/cosine scores)
- **MAGIC** (Baseline imputation method run on the identically masked data to provide comparison L1/RMSE/cosine scores)
- **Python** (Environment in which scGCL is invoked and the metric computation and results table assembly are performed)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Metrics are computed only on masked (synthetic dropout) entries, using the retained true values as ground truth — not on the full matrix
- imputation_results.csv contains all three metrics (median L1 distance, RMSE, cosine similarity) for each of the four methods at all three dropout rates (10%, 30%, 50%)
- Expected outcome: scGCL achieves the best (lowest) median L1 distance and RMSE at all tested dropout rates, with cosine similarity competitive with AutoClass and MAGIC
- All baseline methods (AutoClass, GraphSCI, MAGIC) were run on the identically masked matrix, ensuring a fair like-for-like comparison
- Dropout masking followed the scVI strategy and touched only non-zero expression values

## Limitations

- Ground truth is synthetic: true values are known only because dropout was injected via the scVI-style strategy, so scores reflect recovery of artificial zeros rather than natural dropout
- Cosine similarity alone does not discriminate scGCL from AutoClass and MAGIC; ranking relies on median L1 distance and RMSE
- Results are tied to the scVI-style dropout scheme and the tested rates (10%, 30%, 50%); other masking strategies or rates may yield different rankings

## Evidence

- [other] Apply the scVI-style dropout strategy, randomly masking non-zero expression entries at 10%, 30% and 50% rates while retaining the true values as ground truth.: [evidence span withheld — anchor status: not_found]
- [other] Compute the median L1 distance, RMSE and cosine similarity between imputed and true values for each method at each dropout rate.: [evidence span withheld — anchor status: not_found]
- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [methods] $ python main.py --dataset adam --epochs 300: "$ python main.py --dataset adam --epochs 300"
