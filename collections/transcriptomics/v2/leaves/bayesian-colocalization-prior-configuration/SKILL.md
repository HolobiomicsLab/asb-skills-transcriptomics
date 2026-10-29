---
name: bayesian-colocalization-prior-configuration
description: Use when testing whether cis-eQTL signals are shared across tissues (e.g.,
  GTEx v8) and LD inflation from distinct causal variants must be accounted for via
  Bayesian colocalization.
when_to_use_negative:
- Loci whose disease effects are mediated via coding variants that show no eQTL signal
  — these cannot be explored with CAFEH
- Studies with large LD differences (e.g., ancestry differences) between cohorts,
  since CAFEH relies on a single LD matrix
- Situations where the causal variant may be missing from tested datasets, as CAFEH
  does not handle missing data in the effect size matrix
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Bayesian method integrating genetic association data and LD structure across
    multiple traits to jointly fine-map and colocalize while accommodating allelic
    heterogeneity
  repo: https://github.com/karltayeb/cafeh
- name: coloc
  role: Approximate Bayes colocalization run pairwise across 49 GTEx v8 tissues with
    priors p1=1e-4, p2=1e-4, p12=1e-6; colocalization called at PPH4 >= 0.5
- name: eCAVIAR
  role: Run on a random subset of 1,000 genes allowing a maximum of 2 causal variants
    per locus to complement COLOC evidence
- name: Python
  role: Used to run CAFEH as a Python package (e.g., fit_cafeh_genotype) and to render
    pie charts and histograms
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment from environment.yml
  repo: https://github.com/karltayeb/cafeh
- name: pip
  role: Installs the CAFEH package inside the cafeh conda environment
  repo: https://github.com/karltayeb/cafeh
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:5e929bc3f29cafa6056a711192f1d1a982f06200cc1003243989289ed48566d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - coloc
  - conda
  - eCAVIAR
  - pip
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# bayesian-colocalization-prior-configuration

## Summary

Configure and run Bayesian colocalization methods (COLOC's approximate Bayes approach and CAFEH) on cis-eQTL summary statistics to assess whether causal variants are shared across GTEx tissues, using explicit priors and posterior thresholds rather than LD-blind meta-analysis.

## When to use

Use when testing whether cis-eQTL signals are shared across tissues (e.g., GTEx v8) and LD inflation from distinct causal variants must be accounted for via Bayesian colocalization.

## When NOT to use

- Loci whose disease effects are mediated via coding variants that show no eQTL signal — these cannot be explored with CAFEH
- Studies with large LD differences (e.g., ancestry differences) between cohorts, since CAFEH relies on a single LD matrix
- Situations where the causal variant may be missing from tested datasets, as CAFEH does not handle missing data in the effect size matrix

## Inputs

- GTEx v8 cis-eQTL summary statistics (effect sizes, standard errors or z scores, sample sizes) as tab-delimited files
- Reference LD matrices for each gene locus (e.g., LD_example.tsv format)
- Sample size file per study (n_example.tsv)
- Optional individual-level genotypes (X), phenotypes (y), and covariates for CAFEH genotype mode

## Outputs

- Per-gene tissue-sharing classification table (coloc_ecaviar_tissue_specificity.csv)
- CAFEH results table (cafeh.{mode}.results) with per-(SNP, study) pip, top_component, alpha, rank, p_active, pi, effect and effect_var
- Pie chart and histograms reproducing Figure 1C/1D distribution of tissue-specific vs shared genes

## How to apply

Load GTEx v8 cis-eQTL summary statistics (effect sizes, standard errors or z scores, sample sizes) together with reference LD matrices per gene locus. Run COLOC's approximate Bayes colocalization pairwise across all 49 GTEx v8 tissues with priors p1=1e-4, p2=1e-4, p12=1e-6, and call a tissue pair colocalized when PPH4 >= 0.5. Optionally complement this with CAFEH, which jointly fine-maps and colocalizes while accommodating allelic heterogeneity by setting an arbitrarily high number of causal components (20 per locus in the study); CAFEH accepts individual-level genotype data (--mode genotype), effect sizes plus standard errors (--mode beta), or z scores with a reference LD matrix (--mode z). Combine COLOC and eCAVIAR evidence (eCAVIAR run on ~1,000 randomly sampled genes allowing up to 2 causal variants per locus) to classify each gene as tissue specific or shared. Rationale: LD within loci plus multiple causal alleles can mislead meta-analytic sharing estimates, so LD-aware colocalization with explicit priors yields far less inflated sharing estimates.

## Related tools

- **CAFEH** (Bayesian method integrating genetic association data and LD structure across multiple traits to jointly fine-map and colocalize while accommodating allelic heterogeneity) — https://github.com/karltayeb/cafeh
- **coloc** (Approximate Bayes colocalization run pairwise across 49 GTEx v8 tissues with priors p1=1e-4, p2=1e-4, p12=1e-6; colocalization called at PPH4 >= 0.5)
- **eCAVIAR** (Run on a random subset of 1,000 genes allowing a maximum of 2 causal variants per locus to complement COLOC evidence)
- **Python** (Used to run CAFEH as a Python package (e.g., fit_cafeh_genotype) and to render pie charts and histograms) — https://github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment from environment.yml) — https://github.com/karltayeb/cafeh
- **pip** (Installs the CAFEH package inside the cafeh conda environment) — https://github.com/karltayeb/cafeh

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Every analyzed gene has a classification entry (tissue specific or shared) in coloc_ecaviar_tissue_specificity.csv — no missing rows
- The reproduced pie chart and histograms qualitatively match the paper's Figure 1C/1D distribution of tissue-specific vs shared genes
- CAFEH results table contains one row per (SNP, study) pair with defined pip, top_component, alpha, rank, p_active, pi, effect, and effect_var columns
- Tissue pairs are called colocalized only when COLOC PPH4 >= 0.5 using the specified priors (p1=1e-4, p2=1e-4, p12=1e-6)
- CAFEH with K=20 components per locus automatically prunes components beyond those needed to explain the tested association signals

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH does not handle missing data in the effect size matrix and implicitly assumes causal variants are present in the tested datasets
- It remains difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise without a linking causal pathway due to horizontal pleiotropy
- CAFEH includes a simple prior on variant causality with equal prior probability across all evaluated variants (no informed priors)

## Evidence

- [abstract] We find evidence of pervasive tissue specificity of eQTLs, often masked by linkage disequilibrium that misleads traditional meta-analytic approaches.: "We find evidence of pervasive tissue specificity of eQTLs, often masked by linkage disequilibrium that misleads traditional meta-analytic approaches."
- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [readme] To run CAFEH with effect sizes and standard errors, include `--mode beta` in the arguments.: "To run CAFEH with effect sizes and standard errors, include `--mode beta` in the arguments. You must provide input `--ld (-R)`, `--betas (-B)`, `--standard-errors (-S)`"
- [intro] linkage disequilibrium (LD) within each locus along with the presence of multiple causal alleles within or between cell types may skew the quantification of sharing of genetic effects between tissues: "linkage disequilibrium (LD) within each locus along with the presence of multiple causal alleles within or between cell types may skew the quantification of sharing of genetic effects between tissues"
- [readme] The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair.: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
