---
name: spike-and-slab-bayesian-regression-implementation
description: Use when you have genetic association data across multiple traits (individual-level
  genotypes/phenotypes, or summary statistics with an LD matrix plus effect sizes/standard
  errors or z-scores and sample sizes) and need joint fine-mapping and colocalization
  in the presence of allelic heterogeneity.
when_to_use_negative:
- The effect size matrix contains missing data — CAFEH implicitly assumes causal variants
  are present in the tested datasets
- Large LD differences between studies (e.g. due to ancestry differences) — the current
  version of CAFEH relies on a single LD matrix
- Locus effects are mediated via coding variants with no demonstrable eQTL effects
  — such loci cannot be explored with CAFEH
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: CAFEH
  role: Reference implementation of the spike-and-slab model used as the verification
    baseline for the reimplementation; usable as a command line tool or Python package
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: Language for the reimplementation and for interacting with the fitted model
    via the Python API (fit_cafeh_genotype)
- name: conda
  role: Creates the environment with dependencies from environment.yml before installing
    CAFEH
- name: pip
  role: Installs the reference cafeh package in the cafeh conda environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:4fa18b488588f02e9fabf215918b96ca95ec1e1beb4891c52fb2ff793d3d30ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spike-and-slab-bayesian-regression-implementation

## Summary

Implements the CAFEH (CAFEH-G and CAFEH-S) spike-and-slab Bayesian model for joint colocalization and fine-mapping across multiple traits, fitted by coordinate-ascent variational inference with automatic pruning of superfluous components.

## When to use

Use when you have genetic association data across multiple traits (individual-level genotypes/phenotypes, or summary statistics with an LD matrix plus effect sizes/standard errors or z-scores and sample sizes) and need joint fine-mapping and colocalization in the presence of allelic heterogeneity.

## When NOT to use

- The effect size matrix contains missing data — CAFEH implicitly assumes causal variants are present in the tested datasets
- Large LD differences between studies (e.g. due to ancestry differences) — the current version of CAFEH relies on a single LD matrix
- Locus effects are mediated via coding variants with no demonstrable eQTL effects — such loci cannot be explored with CAFEH

## Inputs

- Tab-delimited individual-level data: genotype matrix (X), phenotype matrix (Y), optional covariates (e.g. example/X_example.tsv, example/y_example.tsv, example/cov_example.tsv)
- Summary statistics: reference LD matrix (R), effect sizes and standard errors (B, S) or z-scores (z), and sample sizes (n)
- Number of causal components K (e.g. K=10 in the demo, 20 per locus in the paper)

## Outputs

- Pickle of the fitted CAFEH model (cafeh.{mode}.model) loadable in Python
- Results table with one row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, effect, and effect_var (cafeh.{mode}.results)

## How to apply

Set an arbitrarily high number of causal components per locus (K=10 in the Python demo; 20 components per locus in the paper) and fit the multi-component spike-and-slab model via coordinate-ascent variational inference, letting the algorithm automatically remove components beyond those needed to explain the association signals. For CAFEH-G with individual-level data, initialize and fit with fit_cafeh_genotype(X, y, K=10) using genotypes, phenotypes, and optional covariates; for CAFEH-S with summary statistics, use the reference LD matrix (R) together with effect sizes and standard errors (mode beta) or z-scores computed from them (mode z) plus sample sizes to construct the RSS-based likelihood. Two traits t1 and t2 colocalize in component k when both are active in that component (st1k = st2k = 1). Compute posterior quantities for each (SNP, study) pair: pip, top_component, alpha, rank, p_active, pi, effect, and effect_var, and compare against the reference implementation's output table to verify the reimplementation.

## Related tools

- **CAFEH** (Reference implementation of the spike-and-slab model used as the verification baseline for the reimplementation; usable as a command line tool or Python package) — https://github.com/karltayeb/cafeh
- **Python** (Language for the reimplementation and for interacting with the fitted model via the Python API (fit_cafeh_genotype))
- **conda** (Creates the environment with dependencies from environment.yml before installing CAFEH)
- **pip** (Installs the reference cafeh package in the cafeh conda environment)

## Examples

```
python cafeh.py --mode genotype -X example/X_example.tsv -Y example/y_example.tsv -c example/cov_example.tsv --out output --save-model
```

## Evaluation signals

- PIP, rank, and alpha values from the reimplemented module match those from the reference script run (python cafeh.py --mode genotype ... --save-model) on the same locus
- Superfluous components are automatically pruned: the number of active components after fitting does not exceed those needed to explain the association signals
- Results table schema check: one row per (SNP, study) pair with columns pip, top_component, alpha, rank, p_active, pi, effect, and effect_var
- Colocalization in component k is only declared when both traits are active in that component (st1k = st2k = 1)
- Credible-set semantics hold: alpha = 0.3 means the SNP is included in every credible set with a >= 0.3, and rank of n means n SNPs have larger posterior probability in top_component

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g. ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise without a linking causal pathway due to horizontal pleiotropy
- CAFEH uses a simple prior on variant causality, assuming equal prior probability across all evaluated variants at a locus (no informed priors)

## Evidence

- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [discussion] The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals.: "The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals."
- [readme] cafehg = fit_cafeh_genotype(X, y, K=10): "# initialize and fit model
cafehg = fit_cafeh_genotype(X, y, K=10)"
- [readme] To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`.: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [discussion] the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [readme] pip` The posterior inclusion probability for that SNP in that study: "- `pip` The posterior inclusion probability for that SNP in that study"
