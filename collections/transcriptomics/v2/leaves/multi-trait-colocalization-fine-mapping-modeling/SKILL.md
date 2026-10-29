---
name: multi-trait-colocalization-fine-mapping-modeling
description: Use when you have association data (individual-level genotypes/phenotypes,
  or summary statistics with LD + betas/SEs or z-scores and sample sizes) for multiple
  traits and need to fine-map causal variants while testing whether signals colocalize
  across traits.
when_to_use_negative:
- Studies with large LD differences (e.g. different ancestries), since CAFEH relies
  on a single LD matrix
- Effect-size matrices with missing data — CAFEH implicitly assumes causal variants
  are present in all tested datasets
- Loci whose effects are mediated via coding variants with no eQTL signal, which cannot
  be explored with CAFEH
edam_operation: http://edamontology.org/operation_3452
edam_topics:
- http://edamontology.org/topic_3263
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Bayesian spike-and-slab method performing joint colocalization and fine-mapping
    over multiple phenotypes via CAFEH genotype (individual-level) and CAFEH summary
    (CAFEHSummary) classes
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: Package and notebook interface for initializing and fitting the model (fit_cafeh_genotype)
    and querying fitted objects
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml
  repo: https://github.com/karltayeb/cafeh
- name: pip
  role: Installs the cafeh package into the conda environment
  repo: https://github.com/karltayeb/cafeh
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:899e44cc5b7c7a863d86b46201ab0e926930afc408b5e61f8a672b4a7d753591
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3452
  edam_topics:
  - http://edamontology.org/topic_3263
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-trait-colocalization-fine-mapping-modeling

## Summary

Jointly perform colocalization and fine-mapping across multiple traits/studies using CAFEH's spike-and-slab Bayesian model, which accommodates allelic heterogeneity (multiple causal variants per locus) while estimating tissue/eQTL sharing of genetic effects.

## When to use

Use when you have association data (individual-level genotypes/phenotypes, or summary statistics with LD + betas/SEs or z-scores and sample sizes) for multiple traits and need to fine-map causal variants while testing whether signals colocalize across traits.

## When NOT to use

- Studies with large LD differences (e.g. different ancestries), since CAFEH relies on a single LD matrix
- Effect-size matrices with missing data — CAFEH implicitly assumes causal variants are present in all tested datasets
- Loci whose effects are mediated via coding variants with no eQTL signal, which cannot be explored with CAFEH

## Inputs

- Individual-level data: tab-delimited genotypes (X), phenotypes (Y), covariates (e.g. example/X_example.tsv, example/y_example.tsv, example/cov_example.tsv)
- Summary statistics: reference LD matrix (R), effect sizes + standard errors (B, S) or z-scores, and sample sizes (n), as tab-delimited text (e.g. example/LD_example.tsv)

## Outputs

- cafeh.{mode}.results summary table with one row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, effect, effect_var
- cafeh.{mode}.model: pickled CAFEH model object (with --save-model flag) for downstream Python analysis

## How to apply

Set an arbitrarily high number of causal components per locus (20 per locus in the paper; K=10 in the demo) and fit the spike-and-slab multi-component model by coordinate-ascent variational inference; CAFEH automatically prunes components beyond those needed to explain the signals. For individual-level data use CAFEH genotype mode with genotypes (X), phenotypes (Y), and covariates; for summary statistics use CAFEH-S (CAFEHSummary) in beta mode (LD matrix R, betas B, standard errors S, sample sizes n) or z mode (R and z-scores), with z-scores computed from effect sizes and standard errors when needed. Two traits t1 and t2 colocalize in component k when both are active in that component (st1k = st2k = 1). Extract the per-(SNP, study) posterior summary: pip, top_component, alpha (smallest credible set level containing the SNP), rank, p_active, pi (component-specific causality probability, distinct from pip), effect, and effect_var. Validate by running the reference cafeh.py command-line script on the same locus and comparing PIP, rank, and alpha values against the reimplemented module.

## Related tools

- **CAFEH** (Bayesian spike-and-slab method performing joint colocalization and fine-mapping over multiple phenotypes via CAFEH genotype (individual-level) and CAFEH summary (CAFEHSummary) classes) — https://github.com/karltayeb/cafeh
- **Python** (Package and notebook interface for initializing and fitting the model (fit_cafeh_genotype) and querying fitted objects) — https://github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml) — https://github.com/karltayeb/cafeh
- **pip** (Installs the cafeh package into the conda environment) — https://github.com/karltayeb/cafeh

## Examples

```
python cafeh.py --mode genotype -X example/X_example.tsv -Y example/y_example.tsv -c example/cov_example.tsv --out output --save-model
```

## Evaluation signals

- PIP, rank, and alpha values from the reimplemented module match the reference cafeh.py script run on the same locus
- Fitted model prunes superfluous components: number of active components does not exceed those needed to explain the tested association signals
- Results table contains one row per (SNP, study) pair with the full field set (pip, top_component, alpha, rank, p_active, pi, effect, effect_var)
- pi and pip behave as expected: pip is the probability of the SNP being causal in any component, pi only for top_component
- Model pickle loads in Python and supports downstream queries via cafeh_model_queries

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g. ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in all tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD
- Colocalization does not necessarily suggest mediation; colocalizing signals may arise without a linking causal pathway due to horizontal pleiotropy
- CAFEH uses a simple prior on variant causality, assuming equal prior probability across all evaluated variants (no informed priors)

## Evidence

- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [discussion] The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals.: "The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals."
- [methods] cafehg = fit_cafeh_genotype(X, y, K=10): "cafehg = fit_cafeh_genotype(X, y, K=10)"
- [readme] The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair.: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
- [readme] To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`.: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [discussion] the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
