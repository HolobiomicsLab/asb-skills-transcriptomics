---
name: eqtl-colocalization-analysis
description: Use when summarystatistic-level eQTL data (z-scores or betas+SEs) for
  many tissues, reference LD matrices, and a question of tissue sharing or GWAS target-tissue
  prioritization are available, especially when loci may harbor multiple causal variants.
when_to_use_negative:
- Loci whose trait effects are mediated by coding variants without eQTL effects —
  these cannot be explored with CAFEH
- Studies with large LD differences between datasets (e.g. ancestry differences),
  since CAFEH relies on a single LD matrix
- Effect-size matrices with missing data, as CAFEH implicitly assumes causal variants
  are present in all tested datasets
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Bayesian joint fine-mapping and colocalization across tissues in genotype,
    beta, or z mode; produces pip, p_active, top_component, rank, and pi per (SNP,
    tissue) pair
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml
  repo: https://github.com/karltayeb/cafeh
- name: pip
  role: Installs the CAFEH package into the activated cafeh environment
  repo: https://github.com/karltayeb/cafeh
- name: coloc
  role: Approximate Bayes pairwise colocalization across 49 GTEx v8 tissues (PPH4
    >= 0.5) used to validate tissue-sharing patterns
- name: eCAVIAR
  role: Pairwise colocalization on a random subset of ~1,000 genes allowing up to
    2 causal variants per locus
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:7c09de795020c6c3093a7e83bee8320e102399a1810641b5e76f767152cc86f0
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
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# eqtl-colocalization-analysis

## Summary

Jointly fine-map and colocalize cis-eQTL signals across many tissues using CAFEH, a Bayesian method that accommodates allelic heterogeneity (multiple causal variants per locus) and LD structure, to quantify whether eQTL effects are tissue specific or shared and to prioritize target tissues for GWAS loci.

## When to use

Use when summarystatistic-level eQTL data (z-scores or betas+SEs) for many tissues, reference LD matrices, and a question of tissue sharing or GWAS target-tissue prioritization are available, especially when loci may harbor multiple causal variants.

## When NOT to use

- Loci whose trait effects are mediated by coding variants without eQTL effects — these cannot be explored with CAFEH
- Studies with large LD differences between datasets (e.g. ancestry differences), since CAFEH relies on a single LD matrix
- Effect-size matrices with missing data, as CAFEH implicitly assumes causal variants are present in all tested datasets

## Inputs

- Tab-delimited cis-eQTL summary statistics per tissue (z-scores, or effect sizes and standard errors) for GTEx v8 tissues
- Reference LD matrix (-R) for each gene locus
- Sample sizes per study/tissue (-n)
- Optionally individual-level genotypes (-X), phenotypes (-Y), and covariates (-c) for genotype mode

## Outputs

- output/cafeh.z.results: per-(SNP, study) table with pip, top_component, alpha, rank, p_active, pi, effect, effect_var
- output/cafeh.z.model: pickled CAFEH model (when --save-model is used) for downstream Python queries
- Tissue-specificity classifications (e.g. coloc_ecaviar_tissue_specificity.csv, tissue-sharing quartiles, p_active vs tissues-shared statistics)

## How to apply

Create the cafeh conda environment from environment.yml and pip install the karltayeb/cafeh package. For each locus, run CAFEH from the command line in z-score mode (--mode z) supplying a reference LD matrix (-R), per-tissue cis-eQTL z-scores (-z, or betas -B and standard errors -S from which z-scores are computed), and sample sizes (-n), with --save-model and an arbitrarily high number of causal components (K = 20 per locus) so the algorithm automatically prunes unneeded components. Parse the output/cafeh.z.results table, which reports per-(SNP, tissue) pip, top_component, alpha, rank, p_active, and pi, and identify active components via high p_active. Count the GTEx tissues in which each component is active to classify genes/components as tissue specific or shared, e.g. assigning components to tissue-sharing quartiles or fitting a linear mixed model of mean p_active versus number of tissues shared (reported p = 3.3e-16 for the inverse relationship). Optionally complement with pairwise LD-aware colocalization (COLOC with priors p1=1e-4, p2=1e-4, p12=1e-6, colocalized when PPH4 >= 0.5, and eCAVIAR allowing up to 2 causal variants) for validating tissue-sharing patterns.

## Related tools

- **CAFEH** (Bayesian joint fine-mapping and colocalization across tissues in genotype, beta, or z mode; produces pip, p_active, top_component, rank, and pi per (SNP, tissue) pair) — https://github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml) — https://github.com/karltayeb/cafeh
- **pip** (Installs the CAFEH package into the activated cafeh environment) — https://github.com/karltayeb/cafeh
- **coloc** (Approximate Bayes pairwise colocalization across 49 GTEx v8 tissues (PPH4 >= 0.5) used to validate tissue-sharing patterns)
- **eCAVIAR** (Pairwise colocalization on a random subset of ~1,000 genes allowing up to 2 causal variants per locus)

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- The .results table contains a row with pip, top_component, alpha, rank, p_active, and pi for every analyzed (SNP, tissue) pair
- Setting K=20 causal components per locus yields models in which the algorithm automatically prunes components beyond those needed to explain the signals
- A linear mixed model of mean p_active versus number of tissues shared reproduces the significant inverse association (reported p = 3.3e-16)
- Tissue-specific vs shared gene distribution (pie chart/histograms, Figure 1C/1D) qualitatively matches the paper's reported results
- The 2x2 CAD artery-versus-liver colocalization comparison qualitatively matches Figure 4D/4E

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH does not handle missing data in the effect size matrix and implicitly assumes causal variants are present in all tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise from horizontal pleiotropic effects
- CAFEH uses a simple prior on variant causality with equal prior probability across all evaluated variants (no informed priors)
- Bulk tissue eQTLs are unlikely to be a good surrogate for cell-type-specific signals, especially for non-dominant cell types

## Evidence

- [abstract] We propose CAFEH (colocalization and fine-mapping in the presence of allelic heterogeneity), a Bayesian method that integrates genetic association data across multiple traits: "We propose CAFEH (colocalization and fine-mapping in the presence of allelic heterogeneity), a Bayesian method that integrates genetic association data across multiple traits"
- [readme] z-mode invocation from README: "python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model"
- [readme] results table fields: "p_active` gives the probability that `top_component` is an active component in this study"
- [discussion] high component count rationale: [evidence span withheld — anchor status: not_found]
- [discussion] single-LD limitation: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [methods] install steps: "conda env create --file environment.yml  # create envrionment with dependencies"
