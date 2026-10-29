---
name: fine-mapping-method-benchmarking
description: Use when quantifying whether a multi-trait fine-mapping/colocalization
  method (e.g. CAFEH) improves power or FDP over CAVIAR, eCAVIAR, FINEMAP, or SuSiE
  under allelic heterogeneity in simulated loci.
when_to_use_negative:
- The loci under study contain variants in nearly perfect or very high LD with each
  other where distinguishing multiple signals remains difficult even for the best
  method
- Studies differ substantially in LD structure (e.g. ancestry differences), since
  CAFEH relies on a single LD matrix
- The goal is inference from individual-level data rather than the summary-statistic
  simulation benchmark (use CAFEH genotype mode instead of the beta/z benchmark path)
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3308
tools:
- name: CAFEH
  role: Method under test; fit jointly across traits with summary stats (beta or z
    mode), K set to 20 components per locus
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Driver for simulation, output parsing, FDP/power computation, and plotting
    of comparison curves
- name: conda
  role: Create the cafeh environment with dependencies from environment.yml
- name: pip
  role: Install the CAFEH package inside the activated cafeh conda environment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:7f18df041f1b3188673c312d7e222739bb290a3574eace9111dbe425b5a1a865
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
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# fine-mapping-method-benchmarking

## Summary

A simulation-based benchmarking workflow that compares CAFEH's colocalization and fine-mapping performance against CAVIAR, eCAVIAR, FINEMAP, and SuSiE across varying numbers of causal variants (q) and signal strengths (rho), evaluated by false discovery proportion versus power.

## When to use

Use when quantifying whether a multi-trait fine-mapping/colocalization method (e.g. CAFEH) improves power or FDP over CAVIAR, eCAVIAR, FINEMAP, or SuSiE under allelic heterogeneity in simulated loci.

## When NOT to use

- The loci under study contain variants in nearly perfect or very high LD with each other where distinguishing multiple signals remains difficult even for the best method
- Studies differ substantially in LD structure (e.g. ancestry differences), since CAFEH relies on a single LD matrix
- The goal is inference from individual-level data rather than the summary-statistic simulation benchmark (use CAFEH genotype mode instead of the beta/z benchmark path)

## Inputs

- Simulated eQTL summary statistics for ~1000 loci: per-locus LD matrix (R), effect sizes (B), standard errors (S), and sample sizes (n) as tab-delimited text files
- 838 GTEx genotypes used as the simulation reference panel
- Comparator method outputs (CAVIAR, eCAVIAR, FINEMAP, SuSiE, SuSiE-SS) with credible sets and causal-variant calls

## Outputs

- Per-(SNP, study) results tables (e.g. output/cafeh.beta.results) with pip, alpha, rank, top_component, p_active, pi fields
- Saved CAFEH model pickles (output/cafeh.{mode}.model) via --save-model
- FDP-versus-power curves per (q, rho) grid point comparing all methods, reproducing Figures 3B/3C and S13-S15

## How to apply

Simulate eQTL summary statistics (LD matrices R, effect sizes B, standard errors S, sample sizes n in tab-delimited text) for G=1000 loci from 838 GTEx genotypes over a q in {1,2,3} x rho in {0.01,0.05,0.1,0.2} grid with T=4 traits. Install CAFEH by cloning github.com/karltayeb/cafeh, creating the conda environment from environment.yml, and running 'pip install .' in the activated cafeh environment. Run each comparator with its causal-variant cap (CAVIAR/eCAVIAR max 2, FINEMAP max 5, SuSiE max 10, plus SuSiE-SS) and run CAFEH on the summary statistics in --mode beta (or --mode z with z-scores computed from effect sizes and standard errors), setting an arbitrarily high number of causal components (20 per locus in the study) since CAFEH automatically removes unnecessary components. Parse the per-(SNP, study) results tables extracting pip, alpha, rank, and top_component to derive credible sets and causal-variant calls, then compute FDP and power at matched credible-set levels for every method and every (q, rho) grid point. Plot FDP-versus-power curves per condition and check that the reproduced method ordering matches the published figures.

## Related tools

- **CAFEH** (Method under test; fit jointly across traits with summary stats (beta or z mode), K set to 20 components per locus) — github.com/karltayeb/cafeh
- **Python** (Driver for simulation, output parsing, FDP/power computation, and plotting of comparison curves)
- **conda** (Create the cafeh environment with dependencies from environment.yml)
- **pip** (Install the CAFEH package inside the activated cafeh conda environment)

## Examples

```
python cafeh.py --mode beta -R LD -B betas -S stderr -n n --out output --save-model
```

## Evaluation signals

- Reproduced FDP-versus-power curves show the qualitative method ordering reported in the paper: CAFEH achieves lower FDP at equivalent power (Figures 3B/3C, S13-S15)
- CAFEH matches existing methods when their restrictive assumptions (number of causal variants) are satisfied, and shows improved colocalization power with more extensive allelic heterogeneity
- Each simulated locus yields a valid results table with one row per (SNP, study) pair containing pip, top_component, alpha, and rank fields
- CAFEH's fitted component count per locus does not exceed the number needed to explain the tested association signals (excess components auto-removed)

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies (e.g. ancestry differences) could impact inference
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in all tested datasets
- CAFEH uses a simple prior on variant causality with equal prior probability across all variants; no informed priors
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other

## Evidence

- [abstract] CAFEH outperforms previous approaches in colocalization and fine-mapping.: "CAFEH outperforms previous approaches in colocalization and fine-mapping."
- [discussion] 20 components per locus: [evidence span withheld — anchor status: not_found]
- [discussion] LD matrix limitation: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [readme] install steps: "git clone https://github.com/karltayeb/cafeh.git
cd cafeh
conda env create --file environment.yml"
- [readme] beta mode invocation: "To run CAFEH with effect sizes and standard errors, include `--mode beta` in the arguments."
- [readme] output fields: "- `pip` The posterior inclusion probability for that SNP in that study
- `top_component` the CAFEH component with the largest probability for that SNP"
