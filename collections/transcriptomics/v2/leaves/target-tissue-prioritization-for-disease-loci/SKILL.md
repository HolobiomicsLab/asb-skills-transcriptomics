---
name: target-tissue-prioritization-for-disease-loci
description: Use when you have GWAS summary statistics for a complex trait plus eQTL
  summary statistics for many tissues, a reference LD matrix, and you need to identify
  which tissue's regulatory effects explain a disease locus.
when_to_use_negative:
- Loci whose effects are mediated via coding variants with no eQTL signal — these
  cannot be explored with CAFEH
- Studies with large LD differences (e.g., different ancestries), since CAFEH relies
  on a single shared LD matrix
- Datasets with missing effect-size entries, because CAFEH does not handle missing
  data and implicitly assumes causal variants are present in all tested studies
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: CAFEH
  role: Jointly fine-maps and colocalizes GWAS and multi-tissue eQTL summary statistics
    to prioritize the target tissue
  repo: https://github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml
- name: pip
  role: Installs the CAFEH package inside the activated cafeh environment
- name: Python
  role: Runs the cafeh.py CLI, reads the .results tables, and produces the matplotlib
    comparison figure
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:6abcca3c6fdfcd9da42e5e996e1e191b786d572c9390687716f281936304581a
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# target-tissue-prioritization-for-disease-loci

## Summary

Prioritize the causal target tissue(s) of GWAS disease loci by jointly fine-mapping and colocalizing the trait's summary statistics with eQTL summary statistics across many tissues (e.g., 49 GTEx studies) using CAFEH-S, which leverages allelic heterogeneity instead of single-signal pairwise colocalization.

## When to use

Use when you have GWAS summary statistics for a complex trait plus eQTL summary statistics for many tissues, a reference LD matrix, and you need to identify which tissue's regulatory effects explain a disease locus.

## When NOT to use

- Loci whose effects are mediated via coding variants with no eQTL signal — these cannot be explored with CAFEH
- Studies with large LD differences (e.g., different ancestries), since CAFEH relies on a single shared LD matrix
- Datasets with missing effect-size entries, because CAFEH does not handle missing data and implicitly assumes causal variants are present in all tested studies

## Inputs

- Tab-delimited reference LD matrix for the locus (-R)
- Tab-delimited z scores matrix: GWAS trait plus tissue eQTL studies (e.g., 49 GTEx tissues) (-z)
- Per-study sample sizes file (-n)
- Curated list of disease loci with experimentally validated target genes/tissues (e.g., CAD loci: LDLR, APOE, APOB, PCSK9, ANGPTL4, LPL, ABCG8, CETP, TWIST1)

## Outputs

- cafeh.z.results table with a row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, effect, and effect_var
- Pickled CAFEH model (cafeh.z.model) when --save-model is used
- Per-locus tissue colocalization calls and comparison table vs COLOC, stratified by presence of genome-wide-significant target-tissue eQTL
- Grouped bar chart of colocalization proportions (e.g., figure_5b_5c_coloc_proportions.png via matplotlib)

## How to apply

Create the cafeh conda environment from environment.yml and install the package with pip from github.com/karltayeb/cafeh. For each locus, run CAFEH-S in z-score mode (`python cafeh.py --mode z`) supplying the reference LD matrix (-R), z scores for the GWAS plus all tissue eQTL studies (-z), and per-study sample sizes (-n); set the number of causal components to an arbitrarily high value of 20 per locus, since the algorithm automatically removes unneeded components, and include --save-model to persist the fit. Read the saved cafeh.z.results tables, which contain one row per (SNP, study) pair with pip, top_component, alpha, rank, p_active, and pi, and call a locus as colocalizing in a tissue when the target tissue's study shows an active component (high p_active/pi for the credible-set SNPs). Benchmark against single-signal pairwise COLOC on the same locus-tissue pairs, stratifying loci by whether the validated target gene has a genome-wide-significant eQTL in the target tissue, and compare the proportion of loci colocalizing in the known target tissue per method and stratum. In the CAD case study this yielded 80% correct tissue prioritization, with CAFEH-S outperforming COLOC when significant eQTL signal was present.

## Related tools

- **CAFEH** (Jointly fine-maps and colocalizes GWAS and multi-tissue eQTL summary statistics to prioritize the target tissue) — https://github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml)
- **pip** (Installs the CAFEH package inside the activated cafeh environment)
- **Python** (Runs the cafeh.py CLI, reads the .results tables, and produces the matplotlib comparison figure)

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- The cafeh.z.results file exists and contains the expected columns (pip, top_component, alpha, rank, p_active, pi, effect, effect_var) for every (SNP, study) pair
- The saved cafeh.z.model pickle loads in Python and the fitted number of components is <= the 20 configured (extra components are automatically removed)
- Known-target-tissue prioritization rate is around 80% for validated CAD loci, and CAFEH-S beats pairwise COLOC in the stratum with significant target-tissue eQTLs
- All loci run without missing-data errors, confirming every causal variant is present in all 50 (GWAS + 49 GTEx) studies

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH does not handle missing data in the effect size matrix and implicitly assumes causal variants are present in all tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; horizontal pleiotropy can produce colocalizing signals without a causal pathway
- CAFEH uses a simple equal prior on variant causality with no informed priors, and loci mediated via coding variants may not show eQTL effects

## Evidence

- [abstract] CAFEH can efficiently leverage the widespread allelic heterogeneity in genetic regulation of gene expression to prioritize the target tissue in genome-wide association complex trait loci: "CAFEH can efficiently leverage the widespread allelic heterogeneity in genetic regulation of gene expression to prioritize the target tissue"
- [methods] z mode invocation: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [discussion] component count rationale: [evidence span withheld — anchor status: not_found]
- [readme] results table schema: "It contains a row for each (SNP, study) pair. It reports - `pip` The posterior inclusion probability for that SNP in that study"
- [readme] install procedure: "conda env create --file environment.yml  # create envrionment with dependencies"
- [discussion] single LD matrix caveat: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
