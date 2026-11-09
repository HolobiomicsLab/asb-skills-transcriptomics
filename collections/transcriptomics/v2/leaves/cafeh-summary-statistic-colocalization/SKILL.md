---
name: cafeh-summary-statistic-colocalization
description: Use when you have GWAS/eQTL summary statistics (z scores or betas+SEs),
  a reference LD matrix, and per-study sample sizes, and need joint colocalization/fine-mapping
  across tens of traits or tissues to prioritize a causal target tissue.
when_to_use_negative:
- You only have individual-level genotype/phenotype data and no summary statistics
  — use CAFEH genotype mode (`--mode genotype`) with -X/-Y instead of --mode z
- The locus's effect is mediated by coding variants without eQTL signal, or there
  are large LD differences between studies (e.g. ancestry differences) — CAFEH relies
  on a single LD matrix and cannot explore such loci
- Causal variants are missing from some tested datasets — CAFEH does not handle missing
  data in the effect size matrix
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Bayesian method run via cafeh.py in --mode z to jointly fine-map and colocalize
    summary-statistic association signals across multiple studies/tissues
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml
  repo: https://github.com/karltayeb/cafeh
- name: pip
  role: Installs the CAFEH package inside the activated cafeh conda environment
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: 'Command line and package interface: cafeh.py script and fit_cafeh_genotype
    / CAFEHSummary classes for fitting and querying models'
  repo: https://github.com/karltayeb/cafeh
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:b1679f69acd99dca59b4ebf4c10892370aa1c5b86f2004dfcba858fec6c05814
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
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cafeh-summary-statistic-colocalization

## Summary

Run CAFEH (colocalization and fine-mapping in the presence of allelic heterogeneity) from summary statistics — a reference LD matrix plus z scores (or betas/standard errors) and per-study sample sizes — to jointly fine-map and colocalize genetic association signals across many traits/tissues, e.g. CAD GWAS against 49 GTEx tissue eQTL studies.

## When to use

Use when you have GWAS/eQTL summary statistics (z scores or betas+SEs), a reference LD matrix, and per-study sample sizes, and need joint colocalization/fine-mapping across tens of traits or tissues to prioritize a causal target tissue.

## When NOT to use

- You only have individual-level genotype/phenotype data and no summary statistics — use CAFEH genotype mode (`--mode genotype`) with -X/-Y instead of --mode z
- The locus's effect is mediated by coding variants without eQTL signal, or there are large LD differences between studies (e.g. ancestry differences) — CAFEH relies on a single LD matrix and cannot explore such loci
- Causal variants are missing from some tested datasets — CAFEH does not handle missing data in the effect size matrix

## Inputs

- Tab-delimited reference LD matrix (--ld / -R)
- Tab-delimited z-scores file (--zscores / -z), or betas (-B) plus standard errors (-S) from which z scores are computed
- Tab-delimited per-study sample-sizes file (--sample-sizes / -n)
- Number of causal components K (e.g. 20 per locus)

## Outputs

- cafeh.z.results table: one row per (SNP, study) pair with pip, top_component, alpha, rank, p_active, pi, effect, effect_var
- cafeh.z.model: pickled CAFEHSummary model (written when --save-model is set), loadable in Python for downstream queries

## How to apply

Install CAFEH by cloning github.com/karltayeb/cafeh, creating the conda environment from environment.yml (`conda env create --file environment.yml; conda activate cafeh`), and installing the package with `pip install .`. Format summary statistics as tab-delimited text: a reference LD matrix (-R/--ld), z scores (-z) or betas (-B) with standard errors (-S, from which the script computes z scores if z scores are absent), and sample sizes (-n/--sample-sizes). Run `python cafeh.py --mode z` with these inputs, setting an arbitrarily high number of causal components (20 per locus in this study) — CAFEH automatically removes unneeded components — and include `--save-model` to persist the fit and `--out` for the output directory. Read the resulting cafeh.z.results table, which has one row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, and effect/effect_var, to determine colocalization (active components shared with the trait) and fine-mapping (credible sets via alpha/rank); use `python cafeh.py -h` for the full option list.

## Related tools

- **CAFEH** (Bayesian method run via cafeh.py in --mode z to jointly fine-map and colocalize summary-statistic association signals across multiple studies/tissues) — https://github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml) — https://github.com/karltayeb/cafeh
- **pip** (Installs the CAFEH package inside the activated cafeh conda environment) — https://github.com/karltayeb/cafeh
- **Python** (Command line and package interface: cafeh.py script and fit_cafeh_genotype / CAFEHSummary classes for fitting and querying models) — https://github.com/karltayeb/cafeh

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Two output files exist in the --out directory: cafeh.z.results and (with --save-model) cafeh.z.model
- cafeh.z.results contains one row per (SNP, study) pair with the documented columns pip, top_component, alpha, rank, p_active, pi, effect, effect_var
- alpha values fall in [0,1] and encode the smallest credible-set level at which the SNP enters the top_component credible set; p_active near 1 marks components active in a given study
- Number of fitted components does not exceed the requested K (20 per locus here); CAFEH automatically removes additional components beyond those needed to explain the signals
- In a validated benchmark (CAD loci such as LDLR, APOE, PCSK9, ANGPTL4), CAFEH-S colocalizes with the known target tissue in ~80% of loci and outperforms single-signal pairwise COLOC when a genome-wide-significant eQTL is present

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH includes a simple prior on variant causality (equal prior probability across variants); no informed priors are supported
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in all tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD
- Colocalization does not necessarily imply mediation — horizontal pleiotropy can produce colocalizing signals without a causal pathway
- Loci whose effects are mediated via coding variants may not show eQTL effects and cannot be explored with CAFEH

## Evidence

- [readme] To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`.: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [readme] The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair.: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
- [discussion] setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus): [evidence span withheld — anchor status: not_found]
- [discussion] the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [methods] git clone https://github.com/karltayeb/cafeh.git ... pip install .  # install package in cafeh environment: [evidence span withheld — anchor status: not_found]
