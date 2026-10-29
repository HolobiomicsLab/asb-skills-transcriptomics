---
name: lasso-regression-velocity-prediction
description: Use when you have scRNA-seq data with scVelo-computed velocities and
  want to test whether per-gene velocity is recoverable as a linear combination of
  TF expression levels (ENCODE/ChEA candidates).
when_to_use_negative:
- Your dataset lacks spliced/unspliced counts and you need velocities directly without
  a scVelo baseline to regress against — use TFvelo's TF-based model instead
- The linear model is suspected insufficient for TF weight estimation; the article
  notes more complex models may be needed and no alternative implementation is provided
- Large-scale datasets where TFvelo-class approaches are too slow — the article notes
  time efficiency is lower than scVelo, though running with more CPUs in parallel
  can reduce running time
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TFvelo
  role: Provides the pipeline, TF selection (--TF_databases ENCODE/ChEA, --max_n_TF)
    and preprocessing that prepares the pancreas dataset for the LASSO velocity regression
  repo: https://github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Supplies the baseline RNA velocities (dynamical model, scvelo==0.2.4) that
    the LASSO regression is trained to predict from TF expressions
  repo: https://github.com/theislab/scvelo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:4b63d6a2ef4c5dec44fcf7091e136522bbfe78fce520955e8c120fa508793b8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaoyeye/TFvelo
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# lasso-regression-velocity-prediction

## Summary

Fits a per-gene LASSO regression that predicts scVelo RNA velocity as a linear combination of candidate transcription factor (TF) expressions plus the target gene's own expression, testing whether velocity can be approximated from TF abundances rather than splicing information.

## When to use

Use when you have scRNA-seq data with scVelo-computed velocities and want to test whether per-gene velocity is recoverable as a linear combination of TF expression levels (ENCODE/ChEA candidates).

## When NOT to use

- Your dataset lacks spliced/unspliced counts and you need velocities directly without a scVelo baseline to regress against — use TFvelo's TF-based model instead
- The linear model is suspected insufficient for TF weight estimation; the article notes more complex models may be needed and no alternative implementation is provided
- Large-scale datasets where TFvelo-class approaches are too slow — the article notes time efficiency is lower than scVelo, though running with more CPUs in parallel can reduce running time

## Inputs

- E15.5 mouse pancreas scRNA-seq dataset (h5ad, preprocessed with scanpy/scVelo pp.moments)
- scVelo dynamical-model RNA velocities (scvelo==0.2.4)
- Candidate TF lists from ENCODE and ChEA TF-target databases (decompress ENCODE.zip before use)
- Hyperparameters: --max_n_TF (max TFs per gene), --WX_thres (max absolute weight value)

## Outputs

- Per-gene test-set correlation scores between LASSO-predicted and true scVelo velocities
- Non-zero LASSO TF weights per gene with one-sided t-test enrichment of ENCODE TF-target sets (p = 6.66e-06)
- lasso_velocity_evaluation_results.csv
- Phase portraits showing clockwise co-expression curves for TF-target pairs

## How to apply

Load and preprocess the E15.5 mouse pancreas scRNA-seq dataset with scanpy/scVelo (pp.moments) as prepared by the TFvelo pipeline, then run scVelo's dynamical model (scvelo==0.2.4, default parameters) to obtain baseline velocities. For each gene, fit a LASSO regression predicting the scVelo velocity from candidate TF expressions selected from ENCODE and ChEA (capped by --max_n_TF) plus the target gene's own expression. Compute the correlation between predicted and true scVelo velocities on a held-out test set per gene. Assess regulatory plausibility by testing enrichment of ENCODE TF-target relationships among non-zero-weight TFs using a one-sided t-test (reported p = 6.66e-06), and qualitatively inspect clockwise co-expression curves for TF-target pairs on phase portraits. Write per-gene test-set correlations and the enrichment test to lasso_velocity_evaluation_results.csv.

## Related tools

- **TFvelo** (Provides the pipeline, TF selection (--TF_databases ENCODE/ChEA, --max_n_TF) and preprocessing that prepares the pancreas dataset for the LASSO velocity regression) — https://github.com/xiaoyeye/TFvelo
- **ScVelo** (Supplies the baseline RNA velocities (dynamical model, scvelo==0.2.4) that the LASSO regression is trained to predict from TF expressions) — https://github.com/theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- Per-gene test-set correlations between predicted and true scVelo velocities are tabulated for every gene
- One-sided t-test on ENCODE TF-target enrichment among non-zero-weight TFs yields p = 6.66e-06 on the pancreas dataset
- Results are written to lasso_velocity_evaluation_results.csv
- Phase portraits for TF-target pairs show clockwise co-expression curves, consistent with regulatory fits
- Reproduction uses scvelo==0.2.4 with default dynamical-model parameters and TF candidates capped by --max_n_TF

## Limitations

- TFvelo-class approaches are slower than scVelo; performing TFvelo with more CPUs in parallel could reduce the running time
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14); improving modeling for more genes remains challenging
- The current linear model for estimating TF weights may be insufficient; no implementation of more complex alternatives is provided
- Some datasets referenced (10x multi-omics and other scRNA-seq datasets) lack accessions or identifiers, limiting reproducibility beyond pancreas

## Evidence

- [results] On the pancreas dataset, TFs with non-zero LASSO weights for predicting scVelo velocity are significantly enriched in the ENCODE TF-target sets functionally linked to the target gene (p = 6.66e-06): [evidence span withheld — anchor status: not_found]
- [discussion] TFvelo models the dynamics of a target gene with the expression level of TFs and the target gene itself: "TFvelo models the dynamics of a target gene with the expression level of TFs and the target gene itself"
- [methods] TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default.: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [readme] decompress the ENCODE TF-target database file firstly after downloading this package: "Please decompress the ENCODE TF-target database file firstly after downloading this package."
- [discussion] TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target: "TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target"
