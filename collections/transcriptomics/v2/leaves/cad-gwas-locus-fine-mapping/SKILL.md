---
name: cad-gwas-locus-fine-mapping
description: Use when a GWAS locus has a known/validated target gene or tissue and
  you need to determine which tissue's eQTL explains the trait association, especially
  under suspected allelic heterogeneity (multiple causal variants per locus).
when_to_use_negative:
- Loci whose trait effects are mediated via coding variants without eQTL effects —
  the article notes such loci 'cannot be explored with CAFEH'
- Studies with large LD differences between datasets (e.g., ancestry differences)
  — CAFEH relies on a single LD matrix shared across all studies
- Datasets with missing entries in the effect size/z-score matrix — CAFEH implicitly
  assumes causal variants are present in all tested datasets
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Bayesian joint fine-mapping/colocalization method run in z-score mode (CAFEH-S)
    with 20 causal components per locus to prioritize target tissue
  repo: github.com/karltayeb/cafeh
- name: COLOC
  role: Single-signal pairwise colocalization baseline run on the same locus-tissue
    pairs for comparison
- name: conda
  role: Creates the CAFEH environment with dependencies from environment.yml
  repo: github.com/karltayeb/cafeh
- name: pip
  role: Installs the CAFEH package inside the cafeh conda environment
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Runs the CAFEH command-line script and reproduces the Figure 5B/5C grouped
    bar chart with matplotlib
  repo: github.com/karltayeb/cafeh
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:44618a0d8b883ddb295ad13bd95e0136de54aef1d1c6c22e2fbd63ca9cd66625
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - COLOC
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cad-gwas-locus-fine-mapping

## Summary

Jointly fine-maps and colocalizes CAD GWAS signals with eQTLs across 49 GTEx tissue studies using CAFEH-S in z-score mode, to prioritize the target tissue at literature-validated CAD loci (LDLR, APOE, APOB, PCSK9, ANGPTL4, LPL, ABCG8, CETP, TWIST1, etc.) and benchmark against single-signal pairwise COLOC.

## When to use

Use when a GWAS locus has a known/validated target gene or tissue and you need to determine which tissue's eQTL explains the trait association, especially under suspected allelic heterogeneity (multiple causal variants per locus).

## When NOT to use

- Loci whose trait effects are mediated via coding variants without eQTL effects — the article notes such loci 'cannot be explored with CAFEH'
- Studies with large LD differences between datasets (e.g., ancestry differences) — CAFEH relies on a single LD matrix shared across all studies
- Datasets with missing entries in the effect size/z-score matrix — CAFEH implicitly assumes causal variants are present in all tested datasets

## Inputs

- Reference LD matrix for the locus (tab-delimited, -R)
- Z-score matrix: CAD GWAS plus 49 GTEx tissue eQTL summary stats (-z), or effect sizes and standard errors (-B/-S)
- Per-study sample sizes table (-n)
- Literature-curated CAD loci with experimentally validated target genes/tissues

## Outputs

- cafeh.z.results table with one row per (SNP, study) pair reporting pip, top_component, alpha, rank, p_active, pi, effect, effect_var
- Pickled CAFEH model (cafeh.z.model) when --save-model is used
- Per-method, per-stratum proportion of loci colocalizing in the known target tissue (Figure 5B/5C grouped bar chart, e.g. figure_5b_5c_coloc_proportions.png)

## How to apply

Set up the CAFEH environment from the repo (conda env create --file environment.yml; conda activate cafeh; pip install .). For each curated CAD locus, run CAFEH-S via `python cafeh.py --mode z`, supplying the reference LD matrix (-R), the z scores for the CAD GWAS plus all 49 GTEx tissue eQTL studies (-z), and per-study sample sizes (-n); effect sizes and standard errors (-B/-S) may be supplied instead, and the script will compute z scores. Set an arbitrarily high number of causal components (20 per locus in this study) — the algorithm automatically removes unneeded components — and include --save-model to persist the fit. Parse the saved cafeh.z.results table, which has one row per (SNP, study) pair with pip, top_component, alpha, rank, p_active, and pi, to decide whether the locus colocalizes in its validated target tissue under CAFEH-S. Run single-signal pairwise COLOC on the same locus-tissue pairs using its posterior probability of sharing a single causal variant, then stratify loci by whether the validated target gene shows a genome-wide-significant eQTL in the target tissue and compute the proportion of loci colocalizing in the known target tissue per method and stratum.

## Related tools

- **CAFEH** (Bayesian joint fine-mapping/colocalization method run in z-score mode (CAFEH-S) with 20 causal components per locus to prioritize target tissue) — github.com/karltayeb/cafeh
- **COLOC** (Single-signal pairwise colocalization baseline run on the same locus-tissue pairs for comparison)
- **conda** (Creates the CAFEH environment with dependencies from environment.yml) — github.com/karltayeb/cafeh
- **pip** (Installs the CAFEH package inside the cafeh conda environment) — github.com/karltayeb/cafeh
- **Python** (Runs the CAFEH command-line script and reproduces the Figure 5B/5C grouped bar chart with matplotlib) — github.com/karltayeb/cafeh

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Output directory contains both cafeh.z.results and cafeh.z.model (the latter only if --save-model was passed)
- cafeh.z.results has exactly one row per (SNP, study) pair and includes the columns pip, top_component, alpha, rank, p_active, and pi
- The validated target tissue appears among the top-ranked tissues by colocalization probability for each curated locus
- Benchmark reproduces the reported pattern: CAFEH-S colocalized with the known target tissue in 80% of validated CAD loci and outperformed COLOC when a genome-wide-significant eQTL was present
- Figure 5B/5C grouped bar chart stratifies loci by genome-wide-significant eQTL presence in the target tissue and shows per-method colocalization proportions

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- CAFEH uses a simple prior on variant causality with equal prior probability across all evaluated variants (no informed priors)
- CAFEH does not handle missing data in the effect size matrix and implicitly assumes causal variants are present in the tested datasets
- It remains difficult to distinguish multiple signals among variants in nearly perfect or very high LD
- Colocalization does not imply mediation; horizontal pleiotropy can produce colocalizing signals without a causal pathway

## Evidence

- [readme] To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`.: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [abstract] CAFEH can efficiently leverage the widespread allelic heterogeneity in genetic regulation of gene expression to prioritize the target tissue in genome-wide association complex trait loci: "CAFEH can efficiently leverage the widespread allelic heterogeneity in genetic regulation of gene expression to prioritize the target tissue"
- [discussion] CAFEH includes a simple prior on variant causality, assuming equal prior probability across all evaluated variants for a locus.: "CAFEH includes a simple prior on variant causality, assuming equal prior probability across all evaluated variants for a locus."
- [readme] It contains a row for each (SNP, study) pair.: "It contains a row for each (SNP, study) pair."
