---
name: rss-likelihood-summary-statistics-modeling
description: Use when only GWAS/eQTL summary statistics are available (no individual-level
  genotypes/phenotypes) and joint fine-mapping plus colocalization across multiple
  traits or tissues is needed, provided a reference LD matrix and sample sizes can
  be supplied.
when_to_use_negative:
- Individual-level genotypes and phenotypes are available — use CAFEH-G genotype mode
  (--mode genotype) instead, which does not require an LD matrix reference
- The effect size matrix contains missing data — CAFEH implicitly assumes causal variants
  are present in all tested datasets and does not handle missing entries
- LD differs substantially between studies (e.g., due to ancestry differences) — CAFEH
  relies on a single LD matrix, so such heterogeneity can bias colocalization and
  fine-mapping inference
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: CAFEH
  role: Provides CAFEHSummary (cafeh/cafeh_summary.py) and the cafeh.py command-line
    driver for fitting the spike-and-slab variational model from summary statistics
    in z or beta mode
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: Runtime for the CAFEH package, CLI entry point (python cafeh.py), and Python
    API for downstream analysis of the pickled model
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml before
    pip-installing the package
- name: pip
  role: Installs the cafeh package into the conda environment (pip install .)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:b7eaf9f996bded128de484371471032ccc10a756a56206dc45db9c2489177caa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# RSS-likelihood summary-statistics modeling

## Summary

Constructs an RSS-based likelihood from a reference LD matrix together with z-scores (or effect sizes and standard errors) plus sample sizes, so that CAFEH's spike-and-slab variational model (CAFEH-S) can jointly fine-map and colocalize signals across traits using summary statistics instead of individual-level genotypes and phenotypes.

## When to use

Use when only GWAS/eQTL summary statistics are available (no individual-level genotypes/phenotypes) and joint fine-mapping plus colocalization across multiple traits or tissues is needed, provided a reference LD matrix and sample sizes can be supplied.

## When NOT to use

- Individual-level genotypes and phenotypes are available — use CAFEH-G genotype mode (--mode genotype) instead, which does not require an LD matrix reference
- The effect size matrix contains missing data — CAFEH implicitly assumes causal variants are present in all tested datasets and does not handle missing entries
- LD differs substantially between studies (e.g., due to ancestry differences) — CAFEH relies on a single LD matrix, so such heterogeneity can bias colocalization and fine-mapping inference

## Inputs

- Reference LD matrix (tab-delimited, --ld/-R)
- Z-scores per SNP per study (--zscores/-z), OR effect sizes (--betas/-B) with standard errors plus sample sizes (-n/--sample-sizes)
- Sample sizes per study (tab-delimited, --sample-sizes/-n)

## Outputs

- Summary results table (cafeh.z.results or cafeh.beta.results) with one row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, effect, effect_var
- Pickled fitted CAFEHSummary model object (cafeh.z.model or cafeh.beta.model) when --save-model is set

## How to apply

Format inputs as tab-delimited text files: a reference LD matrix (-R/--ld) and either z-scores (-z/--zscores) or effect sizes (-B/--betas) with standard errors, plus sample sizes; if effect sizes and standard errors are given, the script computes z-scores from them. Fit CAFEH-S (the CAFEHSummary model) by coordinate-ascent variational inference, initializing a high number of spike-and-slab causal components (K=10 in the Python demo; 20 components per locus in the paper) and letting the algorithm automatically prune superfluous components. Run the command-line script in z mode (--mode z) or beta mode (--mode beta), and pass --save-model to pickle the fitted model object. Extract per-(SNP, study) posteriors: pip, top_component, alpha, rank, p_active, pi, effect, and effect_var into the cafeh.z.results or cafeh.beta.results table.

## Related tools

- **CAFEH** (Provides CAFEHSummary (cafeh/cafeh_summary.py) and the cafeh.py command-line driver for fitting the spike-and-slab variational model from summary statistics in z or beta mode) — https://github.com/karltayeb/cafeh
- **Python** (Runtime for the CAFEH package, CLI entry point (python cafeh.py), and Python API for downstream analysis of the pickled model) — https://github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml before pip-installing the package)
- **pip** (Installs the cafeh package into the conda environment (pip install .))

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Output directory contains exactly two files, cafeh.{mode}.model (pickled CAFEHSummary object) and cafeh.{mode}.results (summary table), when --save-model is used
- Results table has one row for every (SNP, study) pair and includes the columns pip, top_component, alpha, rank, p_active, pi, effect, and effect_var
- The model automatically removes components beyond those needed to explain the tested association signals, so the number of active components is <= the initialized K (20 per locus in the paper)
- PIPs, ranks, and alpha values match those from the reference cafeh package run on the same locus when used as a verification baseline
- Colocalization in component k is supported only when both traits are active in that component (st1k = st2k = 1); check that reported component activity is consistent with pip/p_active values

## Limitations

- CAFEH relies on a single LD matrix, so large differences in LD between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH includes a simple prior on variant causality, assuming equal prior probability across all evaluated variants for a locus (no informed priors)
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets
- It remains difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other

## Evidence

- [readme] summary-stats require LD matrix, z-scores, and sample sizes: "If you are running CAFEH with summary stats you will need to provide a reference LD matrix AND (effect sizes + standard errors OR z scores) AND sample sizes."
- [readme] z mode inputs: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [readme] z-scores derived from effect sizes and standard errors: "In lieu of providing z scores you can also use the effect sizes and standard errors as input via `-B` and `-S`. The script will use these to compute z scores."
- [methods] beta mode inputs per article methods: "To run CAFEH with effect sizes and standard errors, include `--mode beta` in the arguments. You must provide input `--ld (-R)`, `--betas (-B)`, `--standard-errors (-S)`"
- [discussion] single LD matrix limitation: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [discussion] missing data limitation: "in its current form, CAFEH does not handle missing data in the effect size matrix, which implicitly assumes that the causal variants are present in the tested datasets."
