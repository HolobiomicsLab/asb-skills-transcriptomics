---
name: summary-statistic-fine-mapping-with-allelic-heterogeneity
description: Use when you have summary statistics (z-scores or betas+SEs) plus a reference
  LD matrix for multiple traits/tissues over the same variants, and need fine-mapping
  with colocalization where multiple causal alleles may mask tissue specificity.
when_to_use_negative:
- Loci whose disease effects are mediated by coding variants that may not show eQTL
  effects — CAFEH cannot explore these
- Studies with large LD differences (e.g., different ancestries) — CAFEH relies on
  a single LD matrix and inference may be biased
- Datasets with missing variants in the effect-size matrix — CAFEH implicitly assumes
  causal variants are present in all tested datasets
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Bayesian fine-mapping/colocalization engine run in summary-statistic (z-score)
    mode with K=20 causal components per locus; outputs pip, top_component, p_active,
    pi per (SNP, tissue) pair
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml
  repo: https://github.com/karltayeb/cafeh
- name: pip
  role: Installs the CAFEH package into the activated cafeh environment
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: Used to parse output/cafeh.z.results tables, count tissue sharing per active
    component, and fit the linear mixed model of p_active vs. tissues shared
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:d0886cd6ac8a3c44f1b8fff3572f19b7a2e8535e4840040236c7ed1eec635da9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# summary-statistic-fine-mapping-with-allelic-heterogeneity

## Summary

Jointly fine-map and colocalize eQTL and GWAS signals across multiple traits/tissues using CAFEH from summary statistics, accounting for allelic heterogeneity by fitting multiple causal components per locus. Used here to quantify how tissue-specific eQTL components relate to GWAS causality across UK Biobank traits and GTEx v8 tissues.

## When to use

Use when you have summary statistics (z-scores or betas+SEs) plus a reference LD matrix for multiple traits/tissues over the same variants, and need fine-mapping with colocalization where multiple causal alleles may mask tissue specificity.

## When NOT to use

- Loci whose disease effects are mediated by coding variants that may not show eQTL effects — CAFEH cannot explore these
- Studies with large LD differences (e.g., different ancestries) — CAFEH relies on a single LD matrix and inference may be biased
- Datasets with missing variants in the effect-size matrix — CAFEH implicitly assumes causal variants are present in all tested datasets

## Inputs

- Tab-delimited summary statistics per locus: z-scores (or effect sizes + standard errors) for the GWAS trait and the 49 GTEx v8 tissue eQTLs over the same SNPs
- Reference LD matrix for the locus (e.g., UK Biobank-matched LD)
- Per-study sample sizes for each trait/tissue

## Outputs

- cafeh.z.results table: one row per (SNP, tissue) with pip, top_component, alpha, rank, p_active, pi, effect, and effect_var columns
- cafeh.z.model pickle of the fitted CAFEH model (with --save-model) for downstream Python queries
- Per-component tissue-sharing counts, tissue-sharing quartile assignments, and linear mixed model test of p_active vs. number of tissues shared

## How to apply

Install CAFEH via `conda env create --file environment.yml`, `conda activate cafeh`, and `pip install .`. For each genome-wide significant locus, run CAFEH in z-score mode (`--mode z`) providing the reference LD matrix with `-R`, cis-eQTL z-scores with `-z`, and sample sizes with `-n`, plus `--save-model`; set an arbitrarily high number of causal components (K = 20 per locus in the study) and let the algorithm prune unneeded components. Parse the output `cafeh.z.results` table, which contains one row per (SNP, study) pair with `pip`, `top_component`, `rank`, `pi`, and `p_active` columns, to identify active components (high `p_active`) in each tissue. Then count the number of tissues in which each active component is active, assign components to tissue-sharing quartiles, and fit a linear mixed model of mean `p_active` versus number of tissues shared to test the inverse association. Rationale: LD can mislead traditional meta-analytic colocalization, and modeling allelic heterogeneity with multiple components recovers tissue specificity that single-signal methods miss.

## Related tools

- **CAFEH** (Bayesian fine-mapping/colocalization engine run in summary-statistic (z-score) mode with K=20 causal components per locus; outputs pip, top_component, p_active, pi per (SNP, tissue) pair) — https://github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml) — https://github.com/karltayeb/cafeh
- **pip** (Installs the CAFEH package into the activated cafeh environment) — https://github.com/karltayeb/cafeh
- **Python** (Used to parse output/cafeh.z.results tables, count tissue sharing per active component, and fit the linear mixed model of p_active vs. tissues shared)

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- The results table must contain pip, p_active, top_component, rank, and pi fields for every (SNP, tissue) pair
- The linear mixed model of mean p_active versus number of tissues shared reproduces the reported significant inverse association (p = 3.3 × 10−160)
- The CAD artery-vs-liver 2×2 contingency comparison matches Figure 4D/4E qualitatively
- Components shared by fewer GTEx tissues show higher probability of being active in GWAS loci (inverse p_active–tissue-sharing relationship)
- CAFEH prunes unneeded components when initialized with K = 20, converging on the number needed to explain the association signals

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH includes a simple prior on variant causality assuming equal prior probability across all evaluated variants (no informed priors)
- CAFEH does not handle missing data in the effect-size matrix and implicitly assumes causal variants are present in the tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; horizontal pleiotropy can produce colocalizing signals without a causal pathway
- Loci whose effects are mediated via coding variants may not show eQTL effects and cannot be explored with CAFEH

## Evidence

- [readme] To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`.: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [discussion] The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals.: "The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals."
- [abstract] We find evidence of pervasive tissue specificity of eQTLs, often masked by linkage disequilibrium that misleads traditional meta-analytic approaches.: "We find evidence of pervasive tissue specificity of eQTLs, often masked by linkage disequilibrium that misleads traditional meta-analytic approaches."
- [methods] git clone https://github.com/karltayeb/cafeh.git ... pip install .  # install package in cafeh environment: "pip install .  # install package in cafeh environment"
- [discussion] the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
