---
name: single-cell-rna-velocity-baseline-evaluation
description: Use when you need to benchmark TF-based velocity estimation against splicing-based
  scVelo velocities, or test whether TF expression linearly explains RNA velocity
  on a scRNA-seq dataset with spliced/unspliced counts.
when_to_use_negative:
- The dataset lacks unspliced/spliced counts, so scVelo baseline velocities cannot
  be computed and the regression has no ground-truth target (use TFvelo directly instead)
- You only need velocity estimates or trajectory inference, not a benchmark of TF-based
  versus splicing-based velocity (run TFvelo or scVelo alone)
- The candidate TF set does not cover the genes of interest (e.g. neither ENCODE nor
  ChEA contains regulators for the tissue being studied), making LASSO weights uninterpretable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ScVelo
  role: Provides the baseline dynamical-model RNA velocities that the LASSO regression
    predicts from TF expressions
  repo: https://github.com/theislab/scvelo
- name: TFvelo
  role: Supplies the data loading/preprocessing pipeline (pp.moments), TF selection
    via ENCODE/ChEA databases, and hyperparameters (max_n_TF) for the regression workflow
  repo: https://github.com/xiaoyeye/TFvelo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:ebc0bf1d74e910cf02c03c0446b01612332764877365c7e26c51f98563fa74a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scvelo
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# single-cell-rna-velocity-baseline-evaluation

## Summary

Evaluates whether RNA velocity can be approximated as a linear combination of transcription factor (TF) expression levels by fitting per-gene LASSO regressions that predict scVelo dynamical-model velocities from candidate TF abundances (ENCODE/ChEA) plus the target gene's own expression.

## When to use

Use when you need to benchmark TF-based velocity estimation against splicing-based scVelo velocities, or test whether TF expression linearly explains RNA velocity on a scRNA-seq dataset with spliced/unspliced counts.

## When NOT to use

- The dataset lacks unspliced/spliced counts, so scVelo baseline velocities cannot be computed and the regression has no ground-truth target (use TFvelo directly instead)
- You only need velocity estimates or trajectory inference, not a benchmark of TF-based versus splicing-based velocity (run TFvelo or scVelo alone)
- The candidate TF set does not cover the genes of interest (e.g. neither ENCODE nor ChEA contains regulators for the tissue being studied), making LASSO weights uninterpretable

## Inputs

- Preprocessed scRNA-seq AnnData (.h5ad) with spliced/unspliced counts, e.g. E15.5 mouse pancreas (TFvelo_pancreas/rc.h5ad)
- scVelo dynamical-model velocities (scvelo==0.2.4, default parameters)
- Candidate TF lists from ENCODE and ChEA TF-target databases (decompress ENCODE.zip before use)

## Outputs

- Per-gene held-out correlation scores between predicted and true scVelo velocities, tabulated in lasso_velocity_evaluation_results.csv
- Per-gene LASSO TF weight vectors with non-zero weights identified
- One-sided t-test p-value for ENCODE TF-target enrichment among non-zero-weight TFs (p = 6.66e-06 on pancreas)
- Phase portraits showing clockwise co-expression curves for TF-target pairs (qualitative validation)

## How to apply

Load and preprocess the scRNA-seq data with scanpy/scVelo (pp.moments) as prepared by the TFvelo pipeline (e.g. --dataset_name pancreas, E15.5 mouse pancreas). Run scVelo's dynamical model (scvelo==0.2.4, default parameters) to obtain baseline RNA velocities. For each gene, fit a LASSO regression predicting the scVelo velocity from candidate TF expressions — selected from ENCODE and ChEA TF-target databases and capped by --max_n_TF — plus the target gene's own expression. Compute the correlation between predicted and true scVelo velocities on a held-out test set per gene. Assess regulatory plausibility by testing enrichment of ENCODE TF-target relationships among non-zero-weight TFs with a one-sided t-test (expected p = 6.66e-06 on pancreas), and qualitatively check clockwise co-expression curves for TF-target pairs on phase portraits.

## Related tools

- **ScVelo** (Provides the baseline dynamical-model RNA velocities that the LASSO regression predicts from TF expressions) — https://github.com/theislab/scvelo
- **TFvelo** (Supplies the data loading/preprocessing pipeline (pp.moments), TF selection via ENCODE/ChEA databases, and hyperparameters (max_n_TF) for the regression workflow) — https://github.com/xiaoyeye/TFvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- Test-set correlation between LASSO-predicted and true scVelo velocities is tabulated per gene in lasso_velocity_evaluation_results.csv
- One-sided t-test on ENCODE TF-target enrichment among non-zero-weight TFs yields p = 6.66e-06 on the pancreas dataset
- Phase portraits of TF representation vs. target gene show the desired clockwise curve for top-fit genes
- scVelo runs with scvelo==0.2.4 under default dynamical-model parameters; preprocessing matches the TFvelo pipeline for --dataset_name pancreas

## Limitations

- TFvelo is slower than the scVelo baseline, which is a weakness when applied to large-scale datasets (parallelizing over CPUs can reduce running time)
- The current linear model for estimating TF weights may be insufficient; no implementation for alternative (e.g. nonlinear) models is provided
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so per-gene correlation scores will vary

## Evidence

- [results] TFs with non-zero LASSO weights for predicting scVelo velocity are significantly enriched in the ENCODE TF-target sets functionally linked to the target gene (p = 6.66e-06): [evidence span withheld — anchor status: not_found]
- [readme] --TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default.: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [readme] pip install scvelo==0.2.4: "pip install scvelo==0.2.4"
- [readme] This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'.: "This will automatically download, preprocess and run TF model on pancrease dataset."
- [discussion] TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target: "TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait"
