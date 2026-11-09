---
name: tf-target-enrichment-testing
description: Use when you need to validate that a fitted gene-wise LASSO model over
  candidate TFs recapitulates known regulatory databases, e.g. by one-sided t-test
  enrichment of non-zero-weight TFs in ENCODE/ChEA TF-target sets.
when_to_use_negative:
- When no spliced/unspliced-independent TF-based velocity model has been fitted —
  enrichment testing presumes per-gene non-zero LASSO TF weights against scVelo velocities
- When your TF database is not ENCODE/ChEA or TFs were not selected via --TF_databases
  with --max_n_TF cap, since the enrichment statistic is defined against those candidate
  sets
- 'When the dataset is very large: TFvelo is slower than scVelo and parallelization
  over CPUs is required, making the full fit-then-test workflow costly'
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Pipeline whose LASSO TF-weight fits supply the non-zero-weight TFs tested
    for ENCODE TF-target enrichment; candidate TFs are selected via --TF_databases
    (ENCODE and ChEA by default) capped by --max_n_TF
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Provides the baseline RNA velocities (dynamical model, default parameters,
    scvelo==0.2.4) that the per-gene LASSO regressions predict from TF abundances
  repo: github.com/theislab/scvelo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:2ecd2876b022761e681bde58063a5c6f56b04e0e39b80553bee3801dc53d4027
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tf-target-enrichment-testing

## Summary

Statistically tests whether transcription factors with non-zero LASSO weights, fitted to predict scVelo RNA velocity from TF abundances, are enriched in ENCODE TF-target relationships, validating TF-based velocity estimation on the pancreas dataset.

## When to use

Use when you need to validate that a fitted gene-wise LASSO model over candidate TFs recapitulates known regulatory databases, e.g. by one-sided t-test enrichment of non-zero-weight TFs in ENCODE/ChEA TF-target sets.

## When NOT to use

- When no spliced/unspliced-independent TF-based velocity model has been fitted — enrichment testing presumes per-gene non-zero LASSO TF weights against scVelo velocities
- When your TF database is not ENCODE/ChEA or TFs were not selected via --TF_databases with --max_n_TF cap, since the enrichment statistic is defined against those candidate sets
- When the dataset is very large: TFvelo is slower than scVelo and parallelization over CPUs is required, making the full fit-then-test workflow costly

## Inputs

- Preprocessed E15.5 mouse pancreas scRNA-seq AnnData (pp.moments) from the TFvelo pipeline
- scVelo (dynamical model, scvelo==0.2.4) RNA velocity estimates per gene per cell
- Candidate TF lists from ENCODE and ChEA TF-target databases (ENCODE.zip decompressed)
- Per-gene LASSO weights linking TF expressions (plus the target gene's own expression) to scVelo velocity

## Outputs

- Per-gene test-set correlation scores between LASSO-predicted and true scVelo velocities
- One-sided t-test p-value for ENCODE TF-target enrichment among non-zero-weight TFs (p = 6.66e-06 on pancreas)
- lasso_velocity_evaluation_results.csv with tabulated evaluation metrics

## How to apply

First run the TFvelo pipeline on the E15.5 mouse pancreas scRNA-seq data (scanpy/scVelo pp.moments preprocessing, scvelo==0.2.4 dynamical model) to obtain baseline RNA velocities. Per gene, fit a LASSO regression predicting the scVelo velocity from candidate TF expressions (selected from ENCODE and ChEA, capped by --max_n_TF) plus the target gene's own expression, and record the test-set correlation between predicted and true velocities. Collect the TFs receiving non-zero LASSO weights and perform a one-sided t-test for enrichment of ENCODE TF-target relationships among them; on the pancreas dataset this yields p = 6.66e-06. Qualitatively, inspect phase portraits of TF-target pairs for the clockwise co-expression curves that TFvelo expects for good regulatory fits, and tabulate all results in lasso_velocity_evaluation_results.csv.

## Related tools

- **TFvelo** (Pipeline whose LASSO TF-weight fits supply the non-zero-weight TFs tested for ENCODE TF-target enrichment; candidate TFs are selected via --TF_databases (ENCODE and ChEA by default) capped by --max_n_TF) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Provides the baseline RNA velocities (dynamical model, default parameters, scvelo==0.2.4) that the per-gene LASSO regressions predict from TF abundances) — github.com/theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- The one-sided t-test on ENCODE TF-target enrichment among non-zero-weight TFs reproduces p = 6.66e-06 on the pancreas dataset
- Per-gene test-set correlations between LASSO-predicted and true scVelo velocities are tabulated for every gene in lasso_velocity_evaluation_results.csv
- Non-zero LASSO weights are only counted for TFs drawn from the ENCODE/ChEA candidate lists (respecting --max_n_TF)
- TF-target pairs with good regulatory fits show clockwise curves on the phase portrait between TF representation and target
- The baseline scVelo velocities come from the dynamical model with scvelo==0.2.4 on pp.moments-preprocessed data

## Limitations

- The current linear model for estimating TF weights may be insufficient; the paper notes more complex models may be needed to replace the linear model in TFvelo
- TFvelo cannot fit the dynamics of some genes well (Fig. S14), so enrichment statistics may be unreliable for poorly fitted genes
- TFvelo's time efficiency is lower than scVelo, which could be a weakness when applied to large-scale datasets; parallel CPUs reduce runtime

## Evidence

- [readme] ENCODE TF candidate selection via hyperparameter: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [readme] ENCODE database prerequisite: "Please decompress the ENCODE TF-target database file firstly after downloading this package."
- [other] Enrichment test result: [evidence span withheld — anchor status: not_found]
- [discussion] Clockwise phase portrait validation: "TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target"
- [intro] Velocity modeled from TF expression: "we propose TFvelo to model the RNA velocity based on the expression levels of TFs"
