---
name: model-fit-metrics-interpretation
description: Use when running kimma/kmFit with multiple candidate model types and
  you need AIC or other fit metrics to decide which model (e.g. unpaired lm vs. kinship-aware
  lmerel) fits per-gene expression.
when_to_use_negative:
- When only a single prespecified model is being fit and no model comparison is intended
  — fit metrics add no decision value
- When the software does not report fit metrics (e.g. limma unpaired or dream paired
  pipelines, which kimma was benchmarked against without AIC support)
- When the goal is solely DEG calling from existing p-values/FDR rather than evaluating
  whether kinship or random effects improve model fit
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: R package whose kmFit function, run with the metrics argument, returns per-gene
    fit metrics (AIC) for lm, lme, and lmerel model fits
  repo: https://github.com/BIGslu/kimma
- name: R
  role: Language in which kimma is implemented and where model fit comparison is performed
    on the returned data frame
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:26d7416d7165bd9ecf84471acb90909d48d77ed133544a3c682498c229fb0b8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - kimma
  - stats
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# model-fit-metrics-interpretation

## Summary

Interprets per-gene model fit metrics (e.g. AIC) returned by kimma's kmFit when the metrics argument is enabled, enabling comparison of alternative linear model specifications (unpaired lm, paired lme, kinship lmerel) for RNA-seq DEG analysis.

## When to use

Use when running kimma/kmFit with multiple candidate model types and you need AIC or other fit metrics to decide which model (e.g. unpaired lm vs. kinship-aware lmerel) best fits per-gene expression.

## When NOT to use

- When only a single prespecified model is being fit and no model comparison is intended — fit metrics add no decision value
- When the software does not report fit metrics (e.g. limma unpaired or dream paired pipelines, which kimma was benchmarked against without AIC support)
- When the goal is solely DEG calling from existing p-values/FDR rather than evaluating whether kinship or random effects improve model fit

## Inputs

- kmFit results table with fit metrics enabled (per-gene AIC and other fit metrics)
- Candidate model specifications: run_lm (unpaired), lme (random effects), lmerel (kinship covariance, via example.kin)
- TMM normalized log2 CPM gene expression (e.g. from example.count)

## Outputs

- Per-gene comparison of fit metrics (AIC) across lm/lme/lmerel model types
- Justified selection of the best-fitting model per gene or per analysis
- Final per-gene results table with coefficient estimates, fit metrics, and p-values/FDR

## How to apply

Call kmFit with metrics=TRUE so that each per-gene fit is accompanied by fit metrics such as AIC; kimma is distinctive in supporting covariance matrices alongside fit metrics like Akaike information criterion, unlike other DEG software. Compare AIC across model families — the unpaired stats::lm fit (kimma_lm), the random-effect lme fit (kimma_lme), and the kinship-covariance lmerel fit (kimma_lmerel) — using the returned results table's fit-metric fields. In the paper's related cohort, the kinship model showed that kinship impacts model fit and DEG detection, so a lower AIC for lmerel versus lme on the same genes justifies retaining the genetic kinship covariance (supplied via the kin argument, e.g. example.kin). Use these metrics in conjunction with, not instead of, FDR/p-value fields from summarise_kmFit when deciding the final DEG set.

## Related tools

- **kimma** (R package whose kmFit function, run with the metrics argument, returns per-gene fit metrics (AIC) for lm, lme, and lmerel model fits) — https://github.com/BIGslu/kimma
- **R** (Language in which kimma is implemented and where model fit comparison is performed on the returned data frame)

## Examples

```
kmFit(dat = example.voom, kin = example.kin, patientID = "donor", libraryID = "libraryid", meta = example.dat, genes = example.count, run_lm = TRUE, run_lmerel = TRUE, metrics = TRUE)
```

## Evaluation signals

- Run kmFit twice (metrics on/off) and confirm the metrics-enabled run returns a per-gene AIC column absent from the default run
- For a gene with no relatedness signal, AIC of the lmerel fit should be approximately equal to or worse than the lme fit; kinship-informed genes may show lower AIC for lmerel
- Verify each modeled gene has non-missing fit-metric values whenever metrics=TRUE, alongside estimate and p-value/FDR fields
- Cross-check that the chosen model by lowest AIC also yields a coherent DEG set at the FDR threshold used by summarise_kmFit

## Limitations

- Fit metrics add computation and are only meaningful when alternative model structures are actually fit (lm vs. lme vs. lmerel); a single unweighted stats::lm run gives limited comparison value
- AIC support distinguishes kimma from other software, so fit-metric comparisons cannot be reproduced directly in limma or dream pipelines
- Fit metrics inform model adequacy but do not replace FDR-based DEG significance thresholds

## Evidence

- [abstract] Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC): "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [methods] kmFit metrics argument parsed from methods: "kimma_lm(model_lm, to_model_gene, gene, use_weights, metrics)"
- [abstract] kinship impacts model fit and DEG detection: "Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort"
- [readme] kimma provides covariates, weights, random effects, covariance matrices, and fit metrics: "an open-source R package for flexible linear mixed effects modeling of RNA-seq including covariates, weights, random effects, covariance matrices, and fit metrics"
