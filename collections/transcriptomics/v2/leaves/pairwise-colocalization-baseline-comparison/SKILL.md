---
name: pairwise-colocalization-baseline-comparison
description: Use when benchmarking a colocalization or target-tissue prioritization
  method against a classical single-causal-variant pairwise baseline on loci with
  experimentally validated target genes/tissues.
when_to_use_negative:
- Loci where disease effects are mediated by coding variants rather than eQTLs — such
  loci may not demonstrate eQTL effects and cannot be explored with this framework
- Loci with large LD differences between studies (e.g., ancestry differences), where
  a single shared LD reference matrix compromises both CAFEH-S and the COLOC comparison
- Scenarios where the goal is to infer mediation rather than shared causal variants
  — colocalization does not necessarily suggest mediation due to possible horizontal
  pleiotropic effects
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Multi-trait colocalization/fine-mapping method being benchmarked; its CAFEH-S
    run (--mode z) is compared head-to-head against pairwise COLOC on the same locus-tissue
    pairs
  repo: https://github.com/karltayeb/cafeh
- name: COLOC
  role: Single-signal pairwise colocalization baseline; colocalization called from
    its posterior probability of sharing a single causal variant
- name: Python
  role: Used with matplotlib to compute stratified colocalization proportions and
    render the Figure 5B/5C grouped bar chart
- name: conda
  role: Creates the cafeh environment from environment.yml before running the comparison
- name: pip
  role: Installs the CAFEH package inside the activated cafeh conda environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:51bcdb327fcfd3b82aaba080894e82077df165df44d7ee6dd41a6128b279064c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - COLOC
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# pairwise-colocalization-baseline-comparison

## Summary

Establishes a single-signal pairwise colocalization (COLOC) baseline against which multi-signal methods like CAFEH-S are benchmarked for target-tissue prioritization at literature-curated CAD loci (e.g., LDLR, APOE, PCSK9, ANGPTL4).

## When to use

Use when benchmarking a colocalization or target-tissue prioritization method against a classical single-causal-variant pairwise baseline on loci with experimentally validated target genes/tissues.

## When NOT to use

- Loci where disease effects are mediated by coding variants rather than eQTLs — such loci may not demonstrate eQTL effects and cannot be explored with this framework
- Loci with large LD differences between studies (e.g., ancestry differences), where a single shared LD reference matrix compromises both CAFEH-S and the COLOC comparison
- Scenarios where the goal is to infer mediation rather than shared causal variants — colocalization does not necessarily suggest mediation due to possible horizontal pleiotropic effects

## Inputs

- Reference LD matrix per locus (tab-delimited, passed via -R)
- z-score matrix for CAD GWAS plus all 49 GTEx tissue eQTL studies (passed via -z)
- Per-study sample sizes (passed via -n)
- Literature-curated list of CAD loci with experimentally validated target genes/tissues
- eQTL significance calls (genome-wide-significant eQTL in the target tissue) for stratification

## Outputs

- Per locus-tissue pair colocalization calls from COLOC's posterior probability of sharing a single causal variant
- Per-method, per-stratum proportions of loci colocalizing in the known target tissue
- Grouped bar chart of stratified proportions (figure_5b_5c_coloc_proportions.png, reproducing Figure 5B/5C)

## How to apply

For each curated CAD locus with a validated target tissue (LDLR, APOE, APOB, PCSK9, ANGPTL4, LPL, ABCG8, CETP, TWIST1, etc.), run single-signal pairwise COLOC on the same locus-tissue pairs analyzed by CAFEH-S, calling colocalization from COLOC's posterior probability of sharing a single causal variant. Use the same reference LD matrix and summary statistics (CAD GWAS z scores plus the 49 GTEx tissue eQTL studies with per-study sample sizes) that feed the CAFEH-S run executed via `python cafeh.py --mode z` with -R, -z, -n and 20 causal components. Stratify loci by whether the validated target gene shows a genome-wide-significant eQTL in the target tissue, then compute, per method and stratum, the proportion of loci colocalizing in the known target tissue. The rationale is that CAFEH leverages allelic heterogeneity, so the comparison reveals that the advantage of CAFEH-S over COLOC concentrates in loci with a significant eQTL signal (80% concordance with validated target tissue in the CAD case study). Plot the stratified proportions as the Figure 5B/5C grouped bar chart with matplotlib.

## Related tools

- **CAFEH** (Multi-trait colocalization/fine-mapping method being benchmarked; its CAFEH-S run (--mode z) is compared head-to-head against pairwise COLOC on the same locus-tissue pairs) — https://github.com/karltayeb/cafeh
- **COLOC** (Single-signal pairwise colocalization baseline; colocalization called from its posterior probability of sharing a single causal variant)
- **Python** (Used with matplotlib to compute stratified colocalization proportions and render the Figure 5B/5C grouped bar chart)
- **conda** (Creates the cafeh environment from environment.yml before running the comparison)
- **pip** (Installs the CAFEH package inside the activated cafeh conda environment)

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- CAFEH-S colocalizes with the experimentally validated target tissue in ~80% of the curated CAD loci
- CAFEH-S outperforms COLOC specifically in the stratum of loci where the validated target gene shows a genome-wide-significant eQTL in the target tissue
- Each locus is evaluated on identical inputs (same LD matrix, z scores, sample sizes) for both CAFEH-S and COLOC, so per-locus method calls are directly comparable
- Per-method and per-stratum proportions must be computed only over loci with an experimentally validated target tissue
- CAFEH-S runs produce cafeh.z.results tables with one row per (SNP, study) pair containing pip, top_component, alpha, rank, p_active, and pi columns

## Limitations

- CAFEH relies on a single LD matrix, so large differences in LD between studies (e.g., ancestry differences) could impact colocalization inference — and thus the fairness of the comparison
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise from horizontal pleiotropic effects, so concordance with the validated target tissue is not proof of causality
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other, which can confound single-signal (COLOC) calls
- Bulk tissue eQTLs may not surrogate cell-type-specific signals, bounding what target-tissue prioritization can achieve in either method

## Evidence

- [readme] To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`.: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [other] Run single-signal pairwise COLOC on the same locus-tissue pairs and call colocalization from its posterior probability of sharing a single causal variant.: [evidence span withheld — anchor status: not_found]
- [abstract] CAFEH outperforms previous approaches in colocalization and fine-mapping.: "CAFEH outperforms previous approaches in colocalization and fine-mapping."
- [discussion] colocalization does not necessarily suggest mediation and it remains possible that colocalizing signals can be observed without a linking causal pathway due to horizontal pleiotropic effects.: "colocalization does not necessarily suggest mediation and it remains possible that colocalizing signals can be observed without a linking causal pathway due to horizontal pleiotropic effects."
- [discussion] the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
