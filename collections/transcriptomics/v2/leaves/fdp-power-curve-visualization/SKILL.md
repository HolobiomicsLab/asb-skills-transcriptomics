---
name: fdp-power-curve-visualization
description: Use when evaluating fine-mapping or colocalization methods on simulated
  loci with known causal variants (ground truth), to quantify the FDP-power trade-off
  across varying numbers of causal variants (q) and signal strengths (rho).
when_to_use_negative:
- Real-data analyses without simulated ground truth — FDP and power require known
  causal variants, so this skill only applies to benchmark simulations
- Single-method analyses where no cross-method FDP/power comparison is needed (the
  curves only add value when comparing methods)
- Evaluating methods under conditions where their restrictive assumptions are already
  satisfied and allelic heterogeneity is absent — differences like CAFEH's power advantage
  will not be visible
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CAFEH
  role: Method under benchmark; run on summary statistics via cafeh.py in beta or
    z mode, with pip/alpha/rank/top_component parsed from results tables to derive
    credible sets for FDP/power computation
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Language used to parse per-(SNP, study) result tables, compute FDP and power
    at matched credible-set levels, and plot the comparison curves (cafeh/plotting.py
    provides plotting helpers)
  repo: github.com/karltayeb/cafeh
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:14ef02cdc10d5199ffa17388e2d13137284f7ec3c414fcf213858dd0f6c2ebc1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fdp-power-curve-visualization

## Summary

Benchmarking visualization that computes false discovery proportion (FDP) and power at matched credible-set levels for multiple fine-mapping/colocalization methods (CAFEH vs CAVIAR, eCAVIAR, FINEMAP, SuSiE, SuSiE-SS) and plots FDP-versus-power curves, reproducing the paper's Figure 3B/3C and S13–S15 comparisons.

## When to use

Use when evaluating fine-mapping or colocalization methods on simulated loci with known causal variants (ground truth), to quantify the FDP-power trade-off across varying numbers of causal variants (q) and signal strengths (rho).

## When NOT to use

- Real-data analyses without simulated ground truth — FDP and power require known causal variants, so this skill only applies to benchmark simulations
- Single-method analyses where no cross-method FDP/power comparison is needed (the curves only add value when comparing methods)
- Evaluating methods under conditions where their restrictive assumptions are already satisfied and allelic heterogeneity is absent — differences like CAFEH's power advantage will not be visible

## Inputs

- Simulated eQTL summary statistics per locus: tab-delimited LD matrix (R), effect sizes (B), standard errors (S), sample sizes (n)
- Ground-truth causal variant calls from simulation (per locus, per grid point q x rho)
- Per-(SNP, study) results tables from each method (e.g., output/cafeh.{mode}.results with pip, alpha, rank, top_component; analogous outputs from CAVIAR, eCAVIAR, FINEMAP, SuSiE)
- Credible-set level(s) alpha at which FDP and power are matched across methods

## Outputs

- FDP-versus-power curves comparing all methods across the (q, rho) simulation grid
- Per-method, per-grid-point tables of false discovery proportion and power at matched credible-set levels
- Comparison figures reproducing the paper's Figures 3B/3C and S13–S15

## How to apply

Run each method over a grid of simulated loci (here G=1000 loci simulated from 838 GTEx genotypes over q in {1,2,3} x rho in {0.01,0.05,0.1,0.2}, T=4 traits), respecting each method's causal-variant cap (CAVIAR/eCAVIAR max 2, FINEMAP max 5, SuSiE max 10) and running CAFEH on summary statistics with a high component count (e.g., K=20, letting the algorithm prune inactive components). Parse each method's per-(SNP, study) output (for CAFEH: pip, alpha, rank, top_component from cafeh.{mode}.results) to derive credible sets and causal-variant calls, then compute false discovery proportion and power at matched credible-set levels for every method and every (q, rho) grid point. Finally, plot FDP-versus-power curves comparing all methods; the rationale is that curves at matched credible-set levels directly expose whether a method like CAFEH achieves lower FDP at equivalent power under allelic heterogeneity.

## Related tools

- **CAFEH** (Method under benchmark; run on summary statistics via cafeh.py in beta or z mode, with pip/alpha/rank/top_component parsed from results tables to derive credible sets for FDP/power computation) — github.com/karltayeb/cafeh
- **Python** (Language used to parse per-(SNP, study) result tables, compute FDP and power at matched credible-set levels, and plot the comparison curves (cafeh/plotting.py provides plotting helpers)) — github.com/karltayeb/cafeh

## Examples

```
python cafeh.py --mode beta -R LD_example.tsv -B beta_example.tsv -S stderr_example.tsv -n n_example.tsv --out output --save-model
```

## Evaluation signals

- Reproduced FDP-versus-power curves match the qualitative ordering of methods reported in the paper (verified against Figures 3B/3C and S13–S15)
- CAFEH achieves lower FDP at equivalent power, especially at grid points with more extensive allelic heterogeneity (larger q, rho)
- At grid points where each method's causal-variant assumptions are satisfied (e.g., q=1–2 for CAVIAR/eCAVIAR), CAFEH performs similarly to existing methods
- FDP and power are computed at matched credible-set levels (alpha) across all methods, derived from pip/alpha/rank fields, not raw PIP thresholds applied inconsistently
- Every (method, q, rho) grid point yields a finite FDP/power pair; missing pairs indicate a parsing or run failure in the upstream method

## Limitations

- Credible-set and FDP/power conclusions depend on each method's causal-variant cap (CAVIAR/eCAVIAR max 2, FINEMAP max 5, SuSiE max 10), so apparent differences partly reflect those restrictive assumptions
- It remains difficult to distinguish multiple signals among variants in nearly perfect or very high LD, which can distort causal-variant calls feeding the FDP/power computation
- CAFEH's inference relies on a single LD matrix; large LD differences between studies could impact results, and its prior assumes equal prior probability across all variants at a locus
- Colocalization does not necessarily suggest mediation — horizontal pleiotropy can produce colocalizing signals, a caveat to keep in mind when interpreting colocalization-based power

## Evidence

- [abstract] CAFEH outperforms previous approaches in colocalization and fine-mapping.: "CAFEH outperforms previous approaches in colocalization and fine-mapping."
- [readme] pip posterior inclusion probability: "`pip` The posterior inclusion probability for that SNP in that study"
- [readme] alpha credible set level: "`alpha` the smallest credible set level at which the SNP would be included in the credible set for `top_component`"
- [discussion] CAFEH automatically removes extra components: "The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals."
- [discussion] single LD matrix caveat: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
