---
name: eqtl-summary-statistics-simulation
description: Use when benchmarking fine-mapping/colocalization methods on summary
  statistics and you need simulated loci with a controlled number of causal variants
  and signal strengths.
when_to_use_negative:
- You have individual-level genotypes/phenotypes rather than summary statistics —
  CAFEH genotype mode takes -X/-Y directly instead
- Your loci have large LD differences between studies (e.g. ancestry differences),
  since CAFEH relies on a single LD matrix
- The causal variants may be missing from some tested datasets — CAFEH does not handle
  missing data in the effect size matrix
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Method under benchmark; run in beta or z mode on simulated summary statistics
    with --save-model
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: Runs the CAFEH command-line script, parses per-(SNP, study) results tables,
    and plots FDP-versus-power curves
- name: conda
  role: Creates the cafeh environment from environment.yml before installation
- name: pip
  role: Installs the CAFEH package inside the activated cafeh environment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:7d4eb433d37cf073da60e3409ad61fb0318c3a17e361e726d50ca5a3572dbf10
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# eqtl-summary-statistics-simulation

## Summary

Simulate multi-trait eQTL summary statistics (LD matrices, effect sizes, standard errors, sample sizes) across allelic-heterogeneity grids to benchmark CAFEH against CAVIAR, FINEMAP, SuSiE, and eCAVIAR on fine-mapping and colocalization power/FDP.

## When to use

Use when benchmarking fine-mapping/colocalization methods on summary statistics and you need simulated loci with a controlled number of causal variants and signal strengths.

## When NOT to use

- You have individual-level genotypes/phenotypes rather than summary statistics — CAFEH genotype mode takes -X/-Y directly instead
- Your loci have large LD differences between studies (e.g. ancestry differences), since CAFEH relies on a single LD matrix
- The causal variants may be missing from some tested datasets — CAFEH does not handle missing data in the effect size matrix

## Inputs

- 838 GTEx genotypes as reference panel for haplotype/LD simulation
- Simulation grid: q (number of causal variants) in {1,2,3} x rho in {0.01,0.05,0.1,0.2}, T=4 traits, 1000 loci
- Summary-statistics files in tab-delimited text: LD matrix (R), effect sizes (B), standard errors (S), sample sizes (n)

## Outputs

- Per-locus simulated summary-statistics files (tab-delimited R, B, S, n)
- Per-(SNP, study) results tables with pip, alpha, rank, top_component fields (e.g. output/cafeh.beta.results)
- FDP-versus-power curves per method per (q, rho) grid point (comparing to Figures 3B/3C and S13-S15)

## How to apply

Simulate G=1000 loci of eQTL summary statistics from 838 GTEx genotypes over a grid of q causal variants in {1,2,3} and rho in {0.01,0.05,0.1,0.2}, with T=4 traits per locus, emitting per-locus LD matrices (R), effect sizes (B), standard errors (S), and sample sizes (n) as tab-delimited text. Install CAFEH from github.com/karltayeb/cafeh via `conda env create --file environment.yml` then `pip install .`, and run it on the simulated summary statistics in beta (or z) mode with `--save-model`. Run each comparator under its per-method causal-variant cap (CAVIAR and eCAVIAR max 2 causal variants, FINEMAP max 5, SuSiE max 10, plus SuSiE-SS). Parse per-(SNP, study) result tables, extract pip/alpha/rank/top_component to build credible sets and causal-variant calls, then compute false discovery proportion and power at matched credible-set levels for every method at every (q, rho) grid point. Give CAFEH an arbitrarily high number of causal components (20 per locus in the study) since it prunes inactive components automatically, which is what lets it handle loci with more causal variants than competitors' caps allow.

## Related tools

- **CAFEH** (Method under benchmark; run in beta or z mode on simulated summary statistics with --save-model) — https://github.com/karltayeb/cafeh
- **Python** (Runs the CAFEH command-line script, parses per-(SNP, study) results tables, and plots FDP-versus-power curves)
- **conda** (Creates the cafeh environment from environment.yml before installation)
- **pip** (Installs the CAFEH package inside the activated cafeh environment)

## Examples

```
python cafeh.py --mode beta -R example/LD_example.tsv -B example/beta_example.tsv -S example/stderr_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Reproduced FDP-versus-power curves qualitatively match the ordering of methods in Figures 3B/3C and S13-S15
- CAFEH achieves lower FDP at equivalent power, especially at grid points with q > 2 causal variants where CAVIAR/eCAVIAR caps are violated
- All methods show comparable performance when their restrictive causal-variant-cap assumptions (e.g. max 2 for CAVIAR) are satisfied
- Results tables contain one row per (SNP, study) pair with non-empty pip, alpha, rank, and top_component fields
- CAFEH keeps at most ~20 components per locus and automatically prunes components not needed to explain the signals

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g. ancestry differences) could impact inference
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- The simulation uses T=4 traits and a single reference panel; results may not generalize to tens of phenotypes or ancestry-mixed panels

## Evidence

- [readme] CAFEH is a method that performs finemapping and colocalization jointly over multiple phenotypes.: "CAFEH is a method that performs finemapping and colocalization jointly over multiple phenotypes."
- [readme] If you are running CAFEH with summary stats you will need to provide a reference LD matrix AND (effect sizes + standard errors OR z scores) AND sample sizes.: "If you are running CAFEH with summary stats you will need to provide a reference LD matrix AND (effect sizes + standard errors OR z scores) AND sample sizes."
- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [abstract] CAFEH outperforms previous approaches in colocalization and fine-mapping.: "CAFEH outperforms previous approaches in colocalization and fine-mapping."
- [discussion] The current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference.: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
