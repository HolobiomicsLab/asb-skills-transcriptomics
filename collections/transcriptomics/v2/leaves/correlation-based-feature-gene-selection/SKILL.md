---
name: correlation-based-feature-gene-selection
description: Use when building cell-line kernel features for drug response prediction
  (e.g., PRISM models) and you need to reduce the expression matrix to genes informative
  of drug sensitivity rather than using all ~19k genes.
when_to_use_negative:
- Training the GDSC IC50 model, where the full set of 17,419 common genes across 1,018
  cell lines is used without AUC-correlation filtering
- Input expression data are single-cell profiles being scored directly with a pre-trained
  model rather than building a new training kernel
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0154
tools:
- name: scDrug
  role: Pipeline in which correlation-based feature gene selection feeds the PRISM
    kernel feature construction for drug response model training
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Recommender-system framework whose kernel features consume the selected feature
    genes for model training
  repo: https://github.com/CSB5/CaDRReS-Sc
- name: Python
  role: Environment for computing per-gene Pearson correlations and building kernel
    features
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:a5657270043bebd9cc0037359e6fa5a44058efc290e7efba6a5bcbcf1a0c243d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - scDrug
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Correlation-based feature gene selection

## Summary

Selects feature genes for drug response modeling by retaining only those genes whose CCLE expression is correlated with drug response (PRISM AUC) with at least 0.2 absolute Pearson correlation, before building kernel-based cell similarity features for CaDRReS-Sc/scDrug model training.

## When to use

Use when building cell-line kernel features for drug response prediction (e.g., PRISM models) and you need to reduce the expression matrix to genes informative of drug sensitivity rather than using all ~19k genes.

## When NOT to use

- Training the GDSC IC50 model, where the full set of 17,419 common genes across 1,018 cell lines is used without AUC-correlation filtering
- Input expression data are single-cell profiles being scored directly with a pre-trained model rather than building a new training kernel

## Inputs

- CCLE (version 21Q3) expression matrix from DepMap Portal (1,379 cell lines, 19,177 genes)
- PRISM Repurposing dataset version 19Q4 response matrix (1,448 drugs × 480 cell lines), transformed as D′ = 2.40D − 1.20 (scaled 1 − AUC)

## Outputs

- List of feature genes with |Pearson correlation| ≥ 0.2 between expression and PRISM AUC (8,087 genes in scDrug)
- Kernel feature matrix (per-gene log2 expression fold-changes and Pearson-correlation cell–cell similarities) computed over the selected genes

## How to apply

Compute, per gene, the Pearson correlation between its log2 expression (CCLE 21Q3, log2 expression fold-change per gene) across cell lines and the transformed drug response values (PRISM AUC scaled as D′ = 2.40D − 1.20, or equivalently 1 − AUC). Retain genes with absolute Pearson correlation coefficient ≥ 0.2; in the scDrug PRISM pipeline this yielded 8,087 feature genes. Use only these genes to compute the Pearson-correlation cell–cell similarity kernel (in contrast to the GDSC model, which uses the 17,419 genes common across 1,018 cell lines without correlation filtering). The rationale is that restricting kernel features to response-correlated genes improves the recommender-model's ability to learn drug–cell latent relationships.

## Related tools

- **scDrug** (Pipeline in which correlation-based feature gene selection feeds the PRISM kernel feature construction for drug response model training) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Recommender-system framework whose kernel features consume the selected feature genes for model training) — https://github.com/CSB5/CaDRReS-Sc
- **Python** (Environment for computing per-gene Pearson correlations and building kernel features)

## Examples

```
corr = expr.apply(lambda g: g.corr(auc)); feature_genes = corr[corr.abs() >= 0.2].index  # 8,087 PRISM feature genes
```

## Evaluation signals

- The selected feature-gene set should contain 8,087 genes when reproducing the scDrug PRISM pipeline (|Pearson r| ≥ 0.2 threshold)
- Downstream PRISM model, after training (140-dim latent space, lr 0.01, ≤100,000 epochs), should achieve an overall Pearson correlation of 0.615 between PRISM (1 − AUC) and GDSC (−log(IC50)) across shared drugs
- Drug-wise Spearman correlations between actual and predicted response should be computed and drugs below the first-quartile coefficient dropped
- Kernel feature matrix dimensions should match the number of cell lines in the training response matrix (480 for PRISM)

## Limitations

- Feature selection is limited to compounds experimentally screened in PRISM/GDSC; de novo prediction of drug sensitivity is not supported
- Threshold of 0.2 absolute Pearson correlation is specific to the PRISM AUC response scale (D′ = 2.40D − 1.20); re-tune when using other response datasets
- Drug response prediction in scDrug is limited by available scRNA-seq data with drug treatments for evaluation

## Evidence

- [methods] Feature gene selection by AUC correlation: "We used 8,087 genes whose expressions were correlated with PRISM AUC with at least 0.2 absolute Pearson correlation coefficient as features genes"
- [other] Response scale transformation: [evidence span withheld — anchor status: not_found]
- [other] Kernel feature construction: [evidence span withheld — anchor status: not_found]
- [methods] CCLE expression source: "the CCLE (Cancer Cell Line Encyclopedia) expression data (version 21Q3) was downloaded from DepMap Portal"
