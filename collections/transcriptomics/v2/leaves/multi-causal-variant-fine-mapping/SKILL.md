---
name: multi-causal-variant-fine-mapping
description: Use when a locus may harbor multiple causal variants (allelic heterogeneity)
  across many tissues or studies and you need joint fine-mapping and colocalization
  rather than single-causal-variant meta-analysis.
when_to_use_negative:
- Loci whose effects are mediated via coding variants with no eQTL signal — CAFEH
  cannot explore them
- Studies with large LD differences (e.g., ancestry differences) from the single LD
  matrix CAFEH relies on, or datasets with missing entries in the effect size matrix
- When distinguishing multiple signals among variants in nearly perfect LD is required
  — CAFEH struggles in very high LD
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0625
tools:
- name: CAFEH
  role: Bayesian joint fine-mapping and colocalization across multiple traits/tissues
    in the presence of allelic heterogeneity
  repo: github.com/karltayeb/cafeh
- name: conda
  role: create environment with dependencies via environment.yml
- name: pip
  role: install the cafeh package into the cafeh environment
- name: Python
  role: use CAFEH as a python package (fit_cafeh_genotype) or command line tool
  repo: github.com/karltayeb/cafeh
- name: coloc
  role: LD-aware approximate Bayes colocalization used pairwise across GTEx v8 tissues
    in the reproduced comparison workflow (priors p1=1e-4, p2=1e-4, p12=1e-6; PPH4
    >= 0.5)
- name: eCAVIAR
  role: colocalization/fine-mapping with up to 2 causal variants per locus on a random
    subset of 1,000 genes for cross-method comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:65e87cb5141936f2e2ebfadd3c7d2feb218626da7c4b0deb28fddd4d850914ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  - coloc
  - conda
  - eCAVIAR
  - pip
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# multi-causal-variant-fine-mapping

## Summary

Jointly fine-map causal regulatory variants and assess colocalization across multiple tissues/studies using CAFEH, a Bayesian method that accounts for allelic heterogeneity (multiple causal variants per locus) that confounds traditional meta-analytic tissue-sharing estimates.

## When to use

Use when a locus may harbor multiple causal variants (allelic heterogeneity) across many tissues or studies and you need joint fine-mapping and colocalization rather than single-causal-variant meta-analysis.

## When NOT to use

- Loci whose effects are mediated via coding variants with no eQTL signal — CAFEH cannot explore them
- Studies with large LD differences (e.g., ancestry differences) from the single LD matrix CAFEH relies on, or datasets with missing entries in the effect size matrix
- When distinguishing multiple signals among variants in nearly perfect LD is required — CAFEH struggles in very high LD

## Inputs

- Reference LD matrix per gene locus (tab-delimited, e.g. example/LD_example.tsv)
- cis-eQTL summary statistics per tissue/study: effect sizes and standard errors (e.g. example/beta_example.tsv, example/stderr_example.tsv) or z scores (e.g. example/z_example.tsv)
- Sample sizes per study (e.g. example/n_example.tsv)
- Alternatively, individual-level genotypes, phenotypes, and covariates (example/X_example.tsv, example/y_example.tsv, example/cov_example.tsv)

## Outputs

- cafeh.{mode}.results table with one row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, effect, and effect_var
- cafeh.{mode}.model pickle of the fitted CAFEH model (with --save-model) for downstream Python queries and plotting
- Per-gene tissue-specific vs shared classification (e.g. coloc_ecaviar_tissue_specificity.csv in the reproduced workflow)

## How to apply

Clone and install CAFEH (github.com/karltayeb/cafeh) by creating the conda environment from environment.yml and running pip install . in the cafeh environment. Choose the input mode: genotype mode for individual-level data (--genotype, --phenotype, optional --covariates), beta mode for effect sizes and standard errors (--ld, --betas, --standard-errors, --sample-sizes), or z mode for z scores (--ld, --zscores, --sample-sizes), where effect sizes and standard errors are converted to z scores automatically. Set an arbitrarily high number of causal components (20 components per locus in the paper) so the algorithm automatically prunes components beyond those needed to explain the association signals. Run CAFEH per locus across all tissues/studies jointly, using a single reference LD matrix (e.g., GTEx v8 cis-eQTL summary statistics), and include --save-model to persist the fitted model pickle. Interpret the .results table, which reports per (SNP, study) pip, top_component, alpha (credible-set level), rank, p_active, pi, and posterior effect mean/variance, to classify each gene as tissue specific or shared and compare against LD-unaware approaches.

## Related tools

- **CAFEH** (Bayesian joint fine-mapping and colocalization across multiple traits/tissues in the presence of allelic heterogeneity) — github.com/karltayeb/cafeh
- **conda** (create environment with dependencies via environment.yml)
- **pip** (install the cafeh package into the cafeh environment)
- **Python** (use CAFEH as a python package (fit_cafeh_genotype) or command line tool) — github.com/karltayeb/cafeh
- **coloc** (LD-aware approximate Bayes colocalization used pairwise across GTEx v8 tissues in the reproduced comparison workflow (priors p1=1e-4, p2=1e-4, p12=1e-6; PPH4 >= 0.5))
- **eCAVIAR** (colocalization/fine-mapping with up to 2 causal variants per locus on a random subset of 1,000 genes for cross-method comparison)

## Examples

```
python cafeh.py --mode beta -R example/LD_example.tsv -B example/beta_example.tsv -S example/stderr_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Results table contains one row for every (SNP, study) pair with defined pip, top_component, alpha, rank, p_active, pi, effect, and effect_var columns
- Number of active components per locus does not exceed the number needed to explain the tested association signals (CAFEH auto-prunes the 20 initialized components)
- Reproduced tissue-specific vs shared gene distribution qualitatively matches the paper's Figure 1C/1D (far less sharing than meta-analysis approaches report)
- Credible-set alpha values fall in (0,1]; alpha = 0.3 means the SNP is included in every credible set with a >= alpha
- In the COLOC workflow, tissue pairs are called colocalized only when PPH4 >= 0.5, and eCAVIAR is capped at 2 causal variants per locus

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH does not handle missing data in the effect size matrix; it implicitly assumes causal variants are present in the tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise without a linking causal pathway due to horizontal pleiotropic effects
- Loci whose effects are mediated via coding variants may not demonstrate eQTL effects and therefore cannot be explored with CAFEH

## Evidence

- [intro] we propose a Bayesian method, CAFEH (colocalization and fine-mapping in the presence of allelic heterogeneity), that incorporates genetic association signal and LD structure across multiple traits: "we propose a Bayesian method, CAFEH (colocalization and fine-mapping in the presence of allelic heterogeneity), that incorporates genetic association signal and LD structure across multiple"
- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [readme] To run CAFEH with effect sizes and standard errors, include `--mode beta` in the arguments. You must provide input `--ld (-R)`, `--betas (-B)`, `--standard-errors (-S)`: "To run CAFEH with effect sizes and standard errors, include `--mode beta` in the arguments. You must provide input `--ld (-R)`, `--betas (-B)`, `--standard-errors (-S)`"
- [methods] conda env create --file environment.yml  # create envrionment with dependencies: "conda env create --file environment.yml  # create envrionment with dependencies"
- [readme] The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair.: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
