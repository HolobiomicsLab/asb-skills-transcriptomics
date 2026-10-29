---
name: tissue-sharing-stratification-of-causal-components
description: Use when you have CAFEH z-mode results for genome-wide significant GWAS
  loci across multiple tissues (GTEx) and need to quantify whether tissue-specific
  vs tissue-shared eQTL components differ in colocalization with disease loci.
when_to_use_negative:
- Loci whose trait effects are mediated via coding variants that may not demonstrate
  eQTL effects and therefore cannot be explored with CAFEH
- Studies with large LD differences between GWAS and eQTL data (e.g., ancestry differences),
  since the current CAFEH version relies on a single LD matrix
- Datasets with missing entries in the effect-size/z-score matrix — CAFEH implicitly
  assumes the causal variants are present in all tested datasets
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: CAFEH
  role: Bayesian colocalization/fine-mapping method run in z mode to estimate per-component
    p_active across tissues
  repo: github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml
- name: pip
  role: Installs the CAFEH package inside the activated cafeh conda environment
- name: Python
  role: Used to parse the .results tables, count tissues per active component, and
    fit the linear mixed model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:971dd16796ec564bf55d1c922607e4c39ddadaf3b008ecd7128b9ddffe049338
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
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tissue-sharing-stratification-of-causal-components

## Summary

Stratify CAFEH-identified causal eQTL components by how many GTEx v8 tissues they are active in, and test whether tissue-specific components (shared by few tissues) are more likely to be active in GWAS loci than tissue-shared components, e.g. across 19 UK Biobank traits.

## When to use

Use when you have CAFEH z-mode results for genome-wide significant GWAS loci across multiple tissues (GTEx) and need to quantify whether tissue-specific vs tissue-shared eQTL components differ in colocalization with disease loci.

## When NOT to use

- Loci whose trait effects are mediated via coding variants that may not demonstrate eQTL effects and therefore cannot be explored with CAFEH
- Studies with large LD differences between GWAS and eQTL data (e.g., ancestry differences), since the current CAFEH version relies on a single LD matrix
- Datasets with missing entries in the effect-size/z-score matrix — CAFEH implicitly assumes the causal variants are present in all tested datasets

## Inputs

- Reference LD matrix per locus (tab-delimited, -R)
- Cis-eQTL z-scores for 49 GTEx v8 tissues per locus (tab-delimited, -z)
- Per-study sample sizes (tab-delimited, -n)
- Genome-wide significant UK Biobank GWAS loci coordinates

## Outputs

- output/cafeh.z.results table with one row per (SNP, study/tissue) pair containing pip, top_component, rank, p_active, pi, alpha, effect, effect_var
- output/cafeh.z.model (pickled CAFEH model)
- Per-component tissue-sharing counts and quartile assignments
- Linear mixed model of mean p_active vs number of tissues shared (expected inverse association, p = 3.3e-16)
- CAD artery-vs-liver 2x2 contingency table (Figure 4D/4E comparison)

## How to apply

Run CAFEH from the command line in z-score mode (--mode z) for each genome-wide significant UK Biobank locus, providing the reference LD matrix (-R), cis-eQTL z-scores for the 49 GTEx v8 tissues (-z), and sample sizes (-n), with --save-model and K=20 causal components per locus so the algorithm can prune unneeded components. Parse each output/cafeh.z.results table, extracting per-(SNP, tissue) pip, top_component, rank, pi, and p_active columns to identify active components (high p_active) in each tissue. Count the number of GTEx tissues in which each active component is active and assign components to tissue-sharing quartiles. Fit a linear mixed model of mean p_active versus number of tissues shared, reproducing the reported inverse association (p = 3.3e-16), and assemble the CAD artery-versus-liver 2x2 contingency comparison corresponding to Figure 4D/4E.

## Related tools

- **CAFEH** (Bayesian colocalization/fine-mapping method run in z mode to estimate per-component p_active across tissues) — github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml)
- **pip** (Installs the CAFEH package inside the activated cafeh conda environment)
- **Python** (Used to parse the .results tables, count tissues per active component, and fit the linear mixed model)

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- The .results table contains pip, p_active, top_component, rank, and pi fields for every (SNP, tissue) pair
- Linear mixed model reproduces the significant negative p_active-vs-tissues-shared association (p = 3.3e-16)
- The 2x2 CAD artery/liver contingency comparison matches Figure 4D/4E qualitatively
- Components shared by fewer tissues show higher probability of being active in GWAS (inverse relationship between p_active and number of colocalizing tissues)
- Credible-set annotation (alpha) is interpretable, e.g. alpha = 0.3 means the SNP is included in every credible set with a >= 0.3

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH includes a simple prior on variant causality, assuming equal prior probability across all evaluated variants at a locus (no informed priors)
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD
- Colocalization does not necessarily suggest mediation; horizontal pleiotropy could produce colocalizing signals without a causal pathway

## Evidence

- [methods] To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`.: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`. In lieu of providing z scores you can also use the effect sizes"
- [discussion] Set an arbitrarily high number of causal components (20 per locus): [evidence span withheld — anchor status: not_found]
- [readme] p_active definition in the results table: "`p_active` gives the probability that `top_component` is an active component in this study"
- [readme] Results table schema: one row per (SNP, study) pair: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
- [discussion] Single-LD-matrix limitation: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [discussion] Colocalization does not imply mediation: "colocalization does not necessarily suggest mediation and it remains possible that colocalizing signals can be observed without a linking causal pathway due to horizontal pleiotropic effects."
