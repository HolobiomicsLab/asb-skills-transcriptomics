---
name: cross-method-colocalization-comparison
description: Use when evaluating or comparing multi-trait colocalization/fine-mapping
  methods on eQTL summary statistics, especially to test whether allelic heterogeneity
  (multiple causal variants) changes method ranking via FDP-power curves.
when_to_use_negative:
- Studies where LD differs substantially between studies (e.g. ancestry differences),
  since CAFEH relies on a single LD matrix
- Datasets where causal variants may be missing from some tested studies, as CAFEH
  does not handle missing data in the effect size matrix
- Loca whose trait effects are mediated via coding variants without eQTL signal —
  these cannot be explored with CAFEH
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CAFEH
  role: Bayesian method under benchmarking; fit jointly over multiple phenotypes from
    summary statistics via cafeh.py in beta or z mode with --save-model
  repo: github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment from environment.yml before installing and running
    the benchmark
- name: pip
  role: Installs the CAFEH package inside the activated cafeh environment
- name: Python
  role: Scripting language for simulating loci, parsing per-(SNP, study) results tables,
    computing FDP/power, and plotting comparison curves
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:87ab5343046abbb7d870200f12038b16fc3489eff51bca13ffdd8691b7ed9e24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-method-colocalization-comparison

## Summary

Benchmark CAFEH against CAVIAR, eCAVIAR, FINEMAP, SuSiE and SuSiE-SS for joint colocalization and fine-mapping across multiple traits, quantifying false discovery proportion and power over a simulated grid of causal-variant counts (q) and LD/rho settings, as in the Arvanitis et al. 2022 allelic-heterogeneity study.

## When to use

Use when evaluating or comparing multi-trait colocalization/fine-mapping methods on eQTL summary statistics, especially to test whether allelic heterogeneity (multiple causal variants) changes method ranking via FDP-power curves.

## When NOT to use

- Studies where LD differs substantially between studies (e.g. ancestry differences), since CAFEH relies on a single LD matrix
- Datasets where causal variants may be missing from some tested studies, as CAFEH does not handle missing data in the effect size matrix
- Loca whose trait effects are mediated via coding variants without eQTL signal — these cannot be explored with CAFEH

## Inputs

- Per-locus tab-delimited eQTL summary statistics: LD matrix (R), effect sizes (B), standard errors (S), sample sizes (n) (or z-scores computed from B and S)
- GTEx genotypes (838 individuals) as simulation reference panel
- Outputs of comparator methods (CAVIAR, eCAVIAR, FINEMAP, SuSiE, SuSiE-SS) run at their paper-specified causal-variant caps

## Outputs

- Per-(SNP, study) results tables (e.g. output/cafeh.beta.results) with pip, alpha, rank, top_component, p_active, pi, effect/effect_var fields
- Credible sets and causal-variant calls per method per locus
- False discovery proportion and power at matched credible-set levels for every method and every (q, rho) grid point
- FDP-versus-power comparison curves reproducing Figures 3B/3C and S13-S15

## How to apply

Simulate G=1000 eQTL loci from the 838 GTEx genotypes over the q in {1,2,3} x rho in {0.01,0.05,0.1,0.2} grid with T=4 traits, emitting per-locus tab-delimited LD matrices (R), effect sizes (B), standard errors (S) and sample sizes (n). Run each comparator under its causal-variant cap (CAVIAR and eCAVIAR max 2 causal variants, FINEMAP max 5, SuSiE max 10, plus SuSiE-SS), and run CAFEH from summary statistics with a deliberately high component count (20 components per locus in the paper) so the algorithm prunes unneeded components. Parse each method's per-(SNP, study) output — for CAFEH the .results table with pip, alpha, rank, top_component, p_active, pi fields — to derive credible sets and causal-variant calls. Compute false discovery proportion and power at matched credible-set levels for every method at every (q, rho) grid point, then plot FDP-versus-power curves; interpret CAFEH's advantage as improved colocalization power when allelic heterogeneity exceeds comparators' restrictive causal-variant assumptions.

## Related tools

- **CAFEH** (Bayesian method under benchmarking; fit jointly over multiple phenotypes from summary statistics via cafeh.py in beta or z mode with --save-model) — github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment from environment.yml before installing and running the benchmark)
- **pip** (Installs the CAFEH package inside the activated cafeh environment)
- **Python** (Scripting language for simulating loci, parsing per-(SNP, study) results tables, computing FDP/power, and plotting comparison curves)

## Examples

```
python cafeh.py --mode beta -R example/LD_example.tsv -B example/beta_example.tsv -S example/stderr_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Reproduced FDP-versus-power curves match the qualitative ordering of methods reported in the paper (CAFEH achieving lower FDP at equivalent power), verified against Figures 3B/3C and S13-S15
- CAFEH output directory contains both cafeh.{mode}.results and, with --save-model, cafeh.{mode}.model files
- CAFEH results table contains one row per (SNP, study) pair with the documented pip, alpha, rank, top_component, p_active, pi columns populated
- FDP and power are computed at matched credible-set levels for every method at every (q, rho) grid point, so curves are comparable across methods
- CAFEH runs use an arbitrarily high number of causal components (20 per locus) and the algorithm prunes components beyond those needed to explain the signals

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g. ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets
- CAFEH includes a simple prior on variant causality (equal prior probability across all evaluated variants) and does not use informed priors
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; horizontal pleiotropy can produce colocalizing signals without a linking causal pathway

## Evidence

- [abstract] CAFEH outperforms previous approaches in colocalization and fine-mapping.: "CAFEH outperforms previous approaches in colocalization and fine-mapping."
- [methods] conda env create --file environment.yml  # create envrionment with dependencies: "conda env create --file environment.yml  # create envrionment with dependencies"
- [discussion] setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus): [evidence span withheld — anchor status: not_found]
- [readme] The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair.: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
- [readme] You can use CAFEH as a command line tool or a python package. Read below for a minimum example of how to use CAFEH from the command line.: "You can use CAFEH as a command line tool or a python package. Read below for a minimum example of how to use CAFEH from the command line."
