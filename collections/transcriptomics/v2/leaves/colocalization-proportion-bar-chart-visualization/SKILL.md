---
name: colocalization-proportion-bar-chart-visualization
description: Use when comparing target-tissue prioritization accuracy across colocalization
  methods on loci with experimentally validated target genes/tissues, and the proportions
  per method and stratum must be presented as a grouped bar chart.
when_to_use_negative:
- The loci lack experimentally validated target genes/tissues, so there is no ground
  truth to compute colocalization proportions against.
- The goal is fine-mapping or effect-size visualization (e.g., PIP plots from cafeh
  plotting.py) rather than summarizing colocalization accuracy as proportions.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: CAFEH
  role: Provides the CAFEH-S colocalization results (pip, p_active, pi per SNP-study
    pair) whose target-tissue colocalization calls are summarized as proportions
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Used with matplotlib to compute stratified colocalization proportions and
    render the grouped bar chart
- name: COLOC
  role: Baseline single-signal pairwise colocalization method whose accuracy is compared
    against CAFEH-S in the bar chart
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:b3c8f619abc5547307f04a89f340e6525f0e7067722eca5963854e93c3d8f1f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - COLOC
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# colocalization-proportion-bar-chart-visualization

## Summary

Visualize the proportion of GWAS loci that colocalize with their known target tissue under each colocalization method (e.g., CAFEH-S vs COLOC), stratified by eQTL signal strength, as a grouped bar chart like Figure 5B/5C of the CAD case study.

## When to use

Use when comparing target-tissue prioritization accuracy across colocalization methods on loci with experimentally validated target genes/tissues, and the proportions per method and stratum must be presented as a grouped bar chart.

## When NOT to use

- The loci lack experimentally validated target genes/tissues, so there is no ground truth to compute colocalization proportions against.
- The goal is fine-mapping or effect-size visualization (e.g., PIP plots from cafeh plotting.py) rather than summarizing colocalization accuracy as proportions.

## Inputs

- cafeh.z.results tables (one row per (SNP, study) pair with pip, top_component, alpha, rank, p_active, pi)
- COLOC posterior probabilities of sharing a single causal variant for each locus-tissue pair
- Curated CAD locus list with experimentally validated target genes/tissues (LDLR, APOE, APOB, PCSK9, ANGPTL4, LPL, ABCG8, CETP, TWIST1, etc.)
- Genome-wide-significant eQTL status of the validated target gene in the target tissue (stratification variable)

## Outputs

- Per-method, per-stratum proportions of loci colocalizing in the known target tissue
- Grouped bar chart figure saved as figure_5b_5c_coloc_proportions.png (reproducing Figure 5B/5C)

## How to apply

First determine, per locus, whether each method (CAFEH-S via `python cafeh.py --mode z` with -R, -z, -n, and 20 causal components; COLOC via its posterior probability of sharing a single causal variant) colocalizes the CAD GWAS signal in the experimentally validated target tissue using the cafeh.z.results table columns pip/top_component/alpha/rank/p_active/pi. Then stratify the curated CAD loci (LDLR, APOE, APOB, PCSK9, ANGPTL4, LPL, ABCG8, CETP, TWIST1, etc.) by whether the validated target gene shows a genome-wide-significant eQTL in the target tissue. Compute, per method and stratum, the proportion of loci colocalizing in the known target tissue (CAFEH-S reached 80% overall and outperformed COLOC when a significant eQTL signal was present), and render the grouped bar chart with Python matplotlib, saving as figure_5b_5c_coloc_proportions.png.

## Related tools

- **CAFEH** (Provides the CAFEH-S colocalization results (pip, p_active, pi per SNP-study pair) whose target-tissue colocalization calls are summarized as proportions) — github.com/karltayeb/cafeh
- **Python** (Used with matplotlib to compute stratified colocalization proportions and render the grouped bar chart)
- **COLOC** (Baseline single-signal pairwise colocalization method whose accuracy is compared against CAFEH-S in the bar chart)

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Per-locus colocalization calls are binary (colocalizes in validated target tissue or not) and every curated locus is assigned to exactly one eQTL-significance stratum.
- Proportions per method/stratum lie in [0, 1] and each is computed from the same set of loci across methods for a fair comparison.
- CAFEH-S proportion in the validated loci reproduces the reported ~80% target-tissue colocalization rate.
- The rendered figure contains grouped bars (one group per stratum, one bar per method: CAFEH-S and COLOC) and is saved as figure_5b_5c_coloc_proportions.png.
- COLOC bars are lower than CAFEH-S bars in the stratum where the validated target gene has a genome-wide-significant eQTL.

## Limitations

- Colocalization does not necessarily suggest mediation; colocalizing signals may arise without a linking causal pathway due to horizontal pleiotropic effects.
- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization inference and hence the proportions plotted.
- Loci whose effects are mediated via coding variants may not show eQTL effects and cannot be explored with CAFEH, so they are excluded from the visualization.

## Evidence

- [other] In the CAD case study, CAFEH-S colocalized with the known target tissue in 80% of the validated loci and outperformed COLOC when a significant eQTL signal was present.: [evidence span withheld — anchor status: not_found]
- [other] Stratify by significant eQTL and compute proportions: [evidence span withheld — anchor status: not_found]
- [readme] CAFEH-S run command: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [abstract] CAFEH outperforms previous approaches: "CAFEH outperforms previous approaches in colocalization and fine-mapping."
- [discussion] Colocalization is not mediation: "colocalization does not necessarily suggest mediation and it remains possible that colocalizing signals can be observed without a linking causal pathway due to horizontal pleiotropic effects."
