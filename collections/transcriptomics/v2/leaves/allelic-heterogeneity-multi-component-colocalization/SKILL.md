---
name: allelic-heterogeneity-multi-component-colocalization
description: Use when multiple studies or tissues (e.g., eQTLGen and GTEx whole blood)
  have summary statistics or individual-level genotypes/phenotypes for the same locus
  and you need to test whether signals colocalize despite possible multiple causal
  alleles and LD.
when_to_use_negative:
- The locus's effects are likely mediated by coding variants that may not show eQTL
  effects, so cannot be explored with CAFEH
- Studies have large LD differences (e.g., ancestry differences), since CAFEH relies
  on a single LD matrix
- The effect-size matrix has missing data, because CAFEH implicitly assumes causal
  variants are present in all tested datasets
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
tools:
- name: CAFEH
  role: Bayesian method performing joint colocalization and fine-mapping across multiple
    traits; run as CLI (cafeh.py) or Python package (fit_cafeh_genotype)
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml
- name: pip
  role: Installs the cafeh package into the activated cafeh conda environment
- name: Python
  role: Used to load the pickled CAFEH model and query components for downstream analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:5097fbb34a67740164026ca40712145931e7f82bf154bfd385e8edda5b1392fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# allelic-heterogeneity-multi-component-colocalization

## Summary

Jointly fine-map and colocalize eQTL signals across multiple tissues/studies with CAFEH, a Bayesian method that models allelic heterogeneity (multiple causal variants per locus) instead of assuming a single shared causal SNP, revealing tissue specificity masked by LD in meta-analytic approaches.

## When to use

Use when multiple studies or tissues (e.g., eQTLGen and GTEx whole blood) have summary statistics or individual-level genotypes/phenotypes for the same locus and you need to test whether signals colocalize despite possible multiple causal alleles and LD.

## When NOT to use

- The locus's effects are likely mediated by coding variants that may not show eQTL effects, so cannot be explored with CAFEH
- Studies have large LD differences (e.g., ancestry differences), since CAFEH relies on a single LD matrix
- The effect-size matrix has missing data, because CAFEH implicitly assumes causal variants are present in all tested datasets

## Inputs

- Tab-delimited individual-level genotype matrix (-X), phenotype matrix (-Y), and covariates (-c) for genotype mode
- Reference LD matrix (-R) with effect sizes (-B), standard errors (-S), and sample sizes (-n) for beta mode
- Reference LD matrix (-R) with z-scores (-z) and sample sizes (-n) for z mode
- eQTL data from multiple studies/tissues, e.g., GTEx v7 and eQTLGen whole blood

## Outputs

- cafeh.{mode}.results table with one row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, effect, effect_var
- cafeh.{mode}.model — a pickled CAFEH model object for downstream component querying
- Colocalization and fine-mapping classifications (e.g., PPH3 > 0.9 separate signals vs PPH4 > 0.9 colocalization)

## How to apply

Install CAFEH from github.com/karltayeb/cafeh via 'conda env create --file environment.yml', 'conda activate cafeh', then 'pip install .'. Run the high-level script in one of three modes: genotype (individual-level data, supplying -X genotype, -Y phenotype, and optional -c covariates), beta (LD matrix -R, effect sizes -B, standard errors -S, sample sizes -n), or z (LD matrix -R and z-scores -z); include --save-model to pickle the fitted model. Set an arbitrarily high number of causal components per locus (20 in this study, or K=10 via the Python API fit_cafeh_genotype) — the algorithm automatically prunes components beyond those needed to explain the tested association signals, which is what allows allelic heterogeneity to be captured rather than forcing a single shared signal. Query the pickled model (cafeh.genotype.model) for downstream component-level inference, and validate that the results table has one row per (SNP, study) pair with pip, top_component, alpha, rank, p_active, pi, effect, and effect_var fields.

## Related tools

- **CAFEH** (Bayesian method performing joint colocalization and fine-mapping across multiple traits; run as CLI (cafeh.py) or Python package (fit_cafeh_genotype)) — https://github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml)
- **pip** (Installs the cafeh package into the activated cafeh conda environment)
- **Python** (Used to load the pickled CAFEH model and query components for downstream analysis)

## Examples

```
python cafeh.py --mode genotype -X example/X_example.tsv -Y example/y_example.tsv -c example/cov_example.tsv --out output --save-model
```

## Evaluation signals

- The run prints 'fitting CAFEH genotype...' (or the beta/z equivalent) followed by 'saving output...' and writes both cafeh.{mode}.results and cafeh.{mode}.model to the --out directory
- The results table contains one row per (SNP, study) pair with fields pip, top_component, alpha, rank, p_active, pi, effect, and effect_var present
- The .model file is a valid pickle loadable in Python and exposes component queries; it is only generated when --save-model was included
- Setting a high K (e.g., 20 components per locus) does not inflate the active component count — CAFEH automatically removes unneeded components
- For the GTEx/eQTLGen confounding analysis, genes with PPH3 > 0.9 (separate signals) should be enriched for cell-type interaction QTLs relative to PPH4 > 0.9 colocalizing genes

## Limitations

- CAFEH relies on a single LD matrix, so large differences in LD between studies (e.g., ancestry differences) could impact inference
- CAFEH does not handle missing data in the effect-size matrix and implicitly assumes causal variants are present in the tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- The prior on variant causality is simple and uniform across evaluated variants at a locus; no informed priors
- Colocalization does not necessarily imply mediation — horizontal pleiotropy can produce colocalizing signals without a causal pathway

## Evidence

- [abstract] We propose CAFEH (colocalization and fine-mapping in the presence of allelic heterogeneity), a Bayesian method that integrates genetic association data across multiple traits: "We propose CAFEH (colocalization and fine-mapping in the presence of allelic heterogeneity), a Bayesian method that integrates genetic association data across multiple traits"
- [abstract] We find evidence of pervasive tissue specificity of eQTLs, often masked by linkage disequilibrium that misleads traditional meta-analytic approaches.: "We find evidence of pervasive tissue specificity of eQTLs, often masked by linkage disequilibrium that misleads traditional meta-analytic approaches."
- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [discussion] the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [readme] To run CAFEH genotype, include `--mode genotype` in the arguments.: "To run CAFEH genotype, include `--mode genotype` in the arguments. You must specify `--genotype (-X)`, `--phenotype (-Y)`"
- [readme] It contains a row for each (SNP, study) pair. It reports - `pip` The posterior inclusion probability for that SNP in that study: "It contains a row for each (SNP, study) pair. It reports
- `pip` The posterior inclusion probability for that SNP in that study"
- [methods] conda env create --file environment.yml: "conda env create --file environment.yml  # create envrionment with dependencies"
