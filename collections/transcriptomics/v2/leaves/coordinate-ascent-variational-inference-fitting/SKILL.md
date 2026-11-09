---
name: coordinate-ascent-variational-inference-fitting
description: Use when you need joint fine-mapping and colocalization across multiple
  studies/traits from either individual-level genotype/phenotype data or summary statistics
  (LD matrix plus effect sizes/SEs or z-scores and sample sizes), especially under
  suspected allelic heterogeneity.
when_to_use_negative:
- The locus's effects are mediated via coding variants that may not show eQTL effects
  — such loci cannot be explored with CAFEH.
- Studies have large differences in LD (e.g. ancestry differences), since the current
  version of CAFEH relies on a single LD matrix.
- The effect size matrix has missing data, because CAFEH implicitly assumes causal
  variants are present in the tested datasets.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Reference implementation providing fit_cafeh_genotype / CAFEHSummary / CAFEHGenotype
    classes and the cafeh.py command-line script used as the verification baseline
    for the variational fit
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: Language used both for the command-line tool and the Python package API for
    initializing and fitting the model
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml before
    pip-installing the package
- name: pip
  role: Installs the cafeh package into the activated conda environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:4de0250fa34a458dd6d62dc70cd9b8e133040121dd9a7eaf1f44700f35bd1f79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# coordinate-ascent-variational-inference-fitting

## Summary

Fits the CAFEH spike-and-slab Bayesian model (CAFEH-G for individual-level genotypes/phenotypes, CAFEH-S for summary statistics) via coordinate-ascent variational inference, jointly fine-mapping causal variants and colocalizing signals across multiple traits while automatically pruning superfluous components.

## When to use

Use when you need joint fine-mapping and colocalization across multiple studies/traits from either individual-level genotype/phenotype data or summary statistics (LD matrix plus effect sizes/SEs or z-scores and sample sizes), especially under suspected allelic heterogeneity.

## When NOT to use

- The locus's effects are mediated via coding variants that may not show eQTL effects — such loci cannot be explored with CAFEH.
- Studies have large differences in LD (e.g. ancestry differences), since the current version of CAFEH relies on a single LD matrix.
- The effect size matrix has missing data, because CAFEH implicitly assumes causal variants are present in the tested datasets.

## Inputs

- Individual-level genotypes X (tab-delimited text file, e.g. example/X_example.tsv)
- Phenotypes y (tab-delimited text file, e.g. example/y_example.tsv)
- Optional covariates (tab-delimited text file, e.g. example/cov_example.tsv)
- Reference LD matrix R (tab-delimited, e.g. example/LD_example.tsv) for summary-statistic modes
- Effect sizes B and standard errors S, or z-scores z (tab-delimited), plus sample sizes n, for summary-statistic modes

## Outputs

- Fitted CAFEH model object saved as a pickle (output/cafeh.{mode}.model) when --save-model is set
- Summary table with one row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, effect, effect_var (output/cafeh.{mode}.results)
- Per-component colocalization calls: traits t1 and t2 colocalize in component k when st1k = st2k = 1

## How to apply

Initialize the spike-and-slab multi-component model with an arbitrarily high number of causal components (K=10 in the Python demo, 20 components per locus in the paper) and fit it by coordinate-ascent variational inference; the algorithm automatically removes any additional components beyond the number needed to explain the tested association signals. For CAFEH-G, supply individual-level genotypes X, phenotypes y, and optional covariates (e.g. via fit_cafeh_genotype(X, y, K=10)); for CAFEH-S, construct the RSS-based likelihood from a reference LD matrix R together with effect sizes and standard errors (beta mode) or z-scores (z mode, computable from beta/SE), plus sample sizes. Two traits t1 and t2 colocalize in component k when both are active in that component (st1k = st2k = 1). After fitting, compute per-(SNP, study) posteriors: pip, top_component, alpha, rank, p_active, pi, effect, and effect_var. Validate by running the reference cafeh.py script on the same locus (e.g. --mode genotype with --save-model) and comparing PIP, rank, and alpha values against the reimplemented module.

## Related tools

- **CAFEH** (Reference implementation providing fit_cafeh_genotype / CAFEHSummary / CAFEHGenotype classes and the cafeh.py command-line script used as the verification baseline for the variational fit) — https://github.com/karltayeb/cafeh
- **Python** (Language used both for the command-line tool and the Python package API for initializing and fitting the model)
- **conda** (Creates the cafeh environment with dependencies from environment.yml before pip-installing the package)
- **pip** (Installs the cafeh package into the activated conda environment)

## Examples

```
python cafeh.py --mode genotype -X example/X_example.tsv -Y example/y_example.tsv -c example/cov_example.tsv --out output --save-model
```

## Evaluation signals

- Model fit automatically prunes to the number of components needed to explain the association signals (no persistently empty/superfluous active components).
- PIP, rank, and alpha values from the reimplemented variational fit match the reference script run (python cafeh.py --mode genotype ... --save-model) on the same locus.
- Output directory contains both cafeh.{mode}.model (pickle) and cafeh.{mode}.results (summary table), with the results table having one row per (SNP, study) pair and the documented columns pip, top_component, alpha, rank, p_active, pi, effect, effect_var.
- Colocalization calls are consistent with the spike-and-slab definition: two traits colocalize in component k only when both are active (st1k = st2k = 1).
- alpha values are interpretable credible-set levels: alpha = 0.3 means the SNP is included in every credible set with a >= 0.3, and rank n means n SNPs have larger posterior probability in top_component.

## Limitations

- CAFEH relies on a single LD matrix; large differences in LD between studies, due to ancestry differences or other factors, could impact colocalization and fine-mapping inference.
- CAFEH includes a simple prior on variant causality, assuming equal prior probability across all evaluated variants for a locus (no informed priors).
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets.
- It is difficult to distinguish the presence of multiple signals among variants in nearly perfect or very high LD with each other.
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise without a linking causal pathway due to horizontal pleiotropic effects.

## Evidence

- [discussion] Set an arbitrarily high number of causal components for each CAFEH run (20 components per locus in this study): [evidence span withheld — anchor status: not_found]
- [discussion] CAFEH automatically prunes superfluous components: "The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals."
- [readme] Python API initialization and fit with K=10 components: "# initialize and fit model
cafehg = fit_cafeh_genotype(X, y, K=10)"
- [readme] Summary-statistic modes require LD matrix plus effect sizes/SEs or z-scores and sample sizes: "If you are running CAFEH with summary stats you will need to provide a reference LD matrix AND (effect sizes + standard errors OR z scores) AND sample sizes."
- [readme] Output columns of the results table per (SNP, study) pair: "It contains a row for each (SNP, study) pair. It reports
- `pip` The posterior inclusion probability for that SNP in that study"
- [readme] Command-line genotype mode invocation with --save-model: "python cafeh.py --mode genotype -X example/X_example.tsv -Y example/y_example.tsv -c example/cov_example.tsv --out output --save-model"
- [methods] Conda environment and pip installation of the reference package: "conda env create --file environment.yml  # create envrionment with dependencies
conda activate cafeh  # activate environment
pip install .  # install package in cafeh environment"
- [discussion] Single LD matrix limitation: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
