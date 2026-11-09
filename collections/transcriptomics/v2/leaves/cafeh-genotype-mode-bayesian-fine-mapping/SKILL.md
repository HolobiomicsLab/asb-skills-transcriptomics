---
name: cafeh-genotype-mode-bayesian-fine-mapping
description: Use when individual-level genotype and phenotype matrices are available
  for multiple eQTL studies or tissues and you need joint fine-mapping and colocalization
  that can resolve multiple causal variants (allelic heterogeneity) that mislead standard
  colocalization.
when_to_use_negative:
- Only summary statistics are available without individual-level genotypes — use CAFEH
  beta or z mode instead (beta mode needs LD, betas, standard errors, sample sizes;
  z mode needs LD and z scores).
- The causal variants may be absent from the tested datasets — CAFEH does not handle
  missing data in the effect size matrix and implicitly assumes causal variants are
  present.
- Large LD differences between studies (e.g., ancestry differences) are expected,
  since CAFEH relies on a single LD matrix and inference could be impacted.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0625
tools:
- name: CAFEH
  role: Bayesian method that performs joint fine-mapping and colocalization across
    multiple traits, run here in genotype mode on individual-level data
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: creates the cafeh environment from environment.yml with dependencies
- name: pip
  role: installs the cafeh package inside the activated cafeh conda environment
- name: Python
  role: runtime for the cafeh.py command line tool and for loading/querying the pickled
    model via the Python package
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:3abba65a6d72fd95258a1d0c3bfc760bfcbc896d2d632eb22159164206f3c60e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cafeh-genotype-mode-bayesian-fine-mapping

## Summary

Fit the CAFEH Bayesian model in genotype (individual-level data) mode to jointly fine-map and colocalize genetic association signals across multiple studies/traits, accounting for allelic heterogeneity (multiple causal components per locus).

## When to use

Use when individual-level genotype and phenotype matrices are available for multiple eQTL studies or tissues and you need joint fine-mapping and colocalization that can resolve multiple causal variants (allelic heterogeneity) that mislead standard colocalization.

## When NOT to use

- Only summary statistics are available without individual-level genotypes — use CAFEH beta or z mode instead (beta mode needs LD, betas, standard errors, sample sizes; z mode needs LD and z scores).
- The causal variants may be absent from the tested datasets — CAFEH does not handle missing data in the effect size matrix and implicitly assumes causal variants are present.
- Large LD differences between studies (e.g., ancestry differences) are expected, since CAFEH relies on a single LD matrix and inference could be impacted.

## Inputs

- Tab-delimited genotype matrix (individual-level, passed via -X/--genotype)
- Tab-delimited phenotype matrix (-Y/--phenotype)
- Optional tab-delimited covariates matrix (-c/--covariates)

## Outputs

- output/cafeh.genotype.results — table with one row per (SNP, study) pair containing pip, top_component, alpha, rank, p_active, pi, effect, effect_var
- output/cafeh.genotype.model — pickled CAFEHGenotype model (requires --save-model)

## How to apply

Create the environment from environment.yml with conda, activate the 'cafeh' environment, and pip install the package cloned from github.com/karltayeb/cafeh. Prepare tab-delimited inputs: genotype matrix (-X), phenotype matrix (-Y), and optional covariates (-c). Run 'python cafeh.py --mode genotype ...' with --save-model so the pickled model is written alongside the results table; confirm the run prints 'fitting CAFEH genotype...'. Set a high number of causal components per locus (the study used 20) — CAFEH automatically removes components beyond those needed to explain the association signals. For downstream analysis, load the pickled model (cafeh.genotype.model) or use the Python API (fit_cafeh_genotype(X, y, K=10)) to query components.

## Related tools

- **CAFEH** (Bayesian method that performs joint fine-mapping and colocalization across multiple traits, run here in genotype mode on individual-level data) — https://github.com/karltayeb/cafeh
- **conda** (creates the cafeh environment from environment.yml with dependencies)
- **pip** (installs the cafeh package inside the activated cafeh conda environment)
- **Python** (runtime for the cafeh.py command line tool and for loading/querying the pickled model via the Python package)

## Examples

```
python cafeh.py --mode genotype -X example/X_example.tsv -Y example/y_example.tsv -c example/cov_example.tsv --out output --save-model
```

## Evaluation signals

- The run prints 'fitting CAFEH genotype...' followed by 'saving output...' with results table and model paths.
- output/cafeh.genotype.results exists and contains one row per (SNP, study) pair with fields pip, top_component, alpha, rank, p_active, pi, effect, and effect_var.
- output/cafeh.genotype.model exists (a pickle of the CAFEH model) because --save-model was included.
- The number of fitted components does not exceed what is needed: with an arbitrarily high K (e.g., 20), the algorithm automatically removes unnecessary causal components.
- The model pickle loads in Python and supports component queries (e.g., via cafeh_model_queries.py / fit_cafeh_genotype API).

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies (e.g., due to ancestry differences) could impact colocalization and fine-mapping inference.
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets.
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other.
- CAFEH uses a simple, uniform prior on variant causality (no informed priors); colocalization does not necessarily imply mediation due to possible horizontal pleiotropy; loci mediated by coding variants without eQTL effects cannot be explored.

## Evidence

- [readme] You can use CAFEH as a command line tool or a python package.: "You can use CAFEH as a command line tool or a python package."
- [methods] To run CAFEH genotype, include `--mode genotype` in the arguments. You must specify `--genotype (-X)`, `--phenotype (-Y)`: "To run CAFEH genotype, include `--mode genotype` in the arguments. You must specify `--genotype (-X)`, `--phenotype (-Y)`"
- [methods] git clone https://github.com/karltayeb/cafeh.git: "git clone https://github.com/karltayeb/cafeh.git"
- [readme] The `.model` file is a pickle of the CAFEH model. You can load this pickle and interact with the CAFEH model in python. To generate this file you must include the `--save-model` flag.: "The `.model` file is a pickle of the CAFEH model. You can load this pickle and interact with the CAFEH model in python. To generate this file you must include the `--save-model` flag."
- [discussion] The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals.: "The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals."
- [readme] cafehg = fit_cafeh_genotype(X, y, K=10): "cafehg = fit_cafeh_genotype(X, y, K=10)"
