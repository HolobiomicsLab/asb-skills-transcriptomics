---
name: forest-plot-visualization-across-traits
description: Use when per-trait fgwas annotation effect sizes and 95% confidence intervals
  (e.g. fgwas_sQTL_effect_sizes.csv) must be compared visually to show sQTL enrichment
  exceeding expression-only variants.
when_to_use_negative:
- When the goal is annotating individual sQTLs functionally (e.g. with LOFTEE or gkmSVM)
  rather than summarizing per-trait enrichment estimates
- When no fitted fgwas annotation model exists yet — the plot requires maximum-likelihood
  effect sizes and CIs from a fitted model
- For visualizing individual GWAS loci or regional association signals (a LocusZoom-style
  regional plot is the appropriate choice)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: fgwas
  role: Fits genomic annotation enrichment models by maximum likelihood to produce
    the per-trait effect-size estimates and 95% CIs that the forest plot summarizes
  repo: https://github.com/joepickrell/fgwas
- name: sQTLseekeR2
  role: Supplies the sQTL mapping results used to classify variants into the sQTL
    vs expression-only binary annotation tracks
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Reproducible Nextflow pipeline that generated the sQTL catalog from GTEx data
    underlying the annotation classification
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:9017dad24c1b90a99930f6d2d863c842fd57831937f478a59980507c97638056
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joepickrell/fgwas
  tools:
  - fgwas
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Forest-plot visualization of fgwas sQTL effect sizes across GWAS traits

## Summary

Renders a Fig. 5b-style forest plot comparing maximum-likelihood fgwas effect-size estimates (with 95% CIs) for the sQTL annotation versus expression-only variants across eight GWAS traits (asthma, breast cancer, coronary artery disease, heart rate, height, LDL, rheumatoid arthritis, schizophrenia).

## When to use

Use when per-trait fgwas annotation effect sizes and 95% confidence intervals (e.g. fgwas_sQTL_effect_sizes.csv) must be compared visually to show sQTL enrichment exceeding expression-only variants.

## When NOT to use

- When the goal is annotating individual sQTLs functionally (e.g. with LOFTEE or gkmSVM) rather than summarizing per-trait enrichment estimates
- When no fitted fgwas annotation model exists yet — the plot requires maximum-likelihood effect sizes and CIs from a fitted model
- For visualizing individual GWAS loci or regional association signals (a LocusZoom-style regional plot is the appropriate choice)

## Inputs

- fgwas_sQTL_effect_sizes.csv with per-trait maximum-likelihood effect-size estimates and 95% confidence intervals for the sQTL annotation
- fgwas fitted annotation model output for the eight GWAS traits
- Binary functional annotation tracks classifying variants as sQTLs vs expression-only variants

## Outputs

- Fig. 5b-style forest plot of sQTL effect sizes with 95% CIs across the eight traits
- fgwas_sQTL_effect_sizes.csv (per-trait effect sizes and CIs)

## How to apply

First fit fgwas genomic annotation enrichment models per trait by maximum likelihood, contrasting the binary sQTL annotation (derived from sQTLseekeR2/sqtlseeker2-nf mapping of GTEx data) against the expression-only annotation. Extract each trait's maximum-likelihood effect-size estimate and its 95% confidence interval for the sQTL annotation, and save them to fgwas_sQTL_effect_sizes.csv. Then plot each trait as a forest-plot row with the point estimate and CI whiskers, ordering rows to highlight traits where the CI excludes zero (asthma, breast cancer, heart rate, height). The rationale is the paper's conclusion that genetic variants affecting splicing can have a stronger phenotypic impact than those affecting gene expression, visualized as sQTL effect sizes above those of expression-only variants.

## Related tools

- **fgwas** (Fits genomic annotation enrichment models by maximum likelihood to produce the per-trait effect-size estimates and 95% CIs that the forest plot summarizes) — https://github.com/joepickrell/fgwas
- **sQTLseekeR2** (Supplies the sQTL mapping results used to classify variants into the sQTL vs expression-only binary annotation tracks) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Reproducible Nextflow pipeline that generated the sQTL catalog from GTEx data underlying the annotation classification) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
Rscript plot_forest.R fgwas_sQTL_effect_sizes.csv  # renders Fig. 5b-style forest plot from per-trait fgwas effect sizes and 95% CIs
```

## Evaluation signals

- For asthma, breast cancer, heart rate and height, the plotted sQTL effect sizes must be positive with 95% CIs excluding zero, matching the published maximum-likelihood estimates
- fgwas_sQTL_effect_sizes.csv must contain one estimate and CI pair per trait for all eight GWAS traits
- Point estimates and CI bounds in the plot must reproduce the fgwas maximum-likelihood model output without manual adjustment
- The forest plot should visually show sQTL effect sizes above (or at least not below) the expression-only annotation estimates for the reported traits

## Limitations

- fgwas enrichment tests work best when there are at least ~20 independent loci with p-values less than 5e-8 per trait, so traits with fewer genome-wide significant loci yield poorly powered effect-size estimates
- Cannot distinguish whether overlapping sQTL and eQTL effects are caused by the same variant or two different variants in LD, so the sQTL vs expression-only contrast is partially confounded
- A difference in sQTL vs eQTL counts could be confounded by different preprocessing steps and statistical methodologies applied in each analysis

## Evidence

- [results] sQTLs show stronger GWAS associations than expression-only variants for traits including asthma, breast cancer, heart rate and height: [evidence span withheld — anchor status: not_found]
- [abstract] Genetic variants affecting splicing can have a stronger phenotypic impact than gene expression variants: "Genetic variants affecting splicing can have a stronger phenotypic impact than those affecting gene expression."
- [readme] fgwas integrates functional genomic information into GWAS: [evidence span withheld — anchor status: external_doc]
- [readme] fgwas README: enrichment of associations in particular types of genome regions: [evidence span withheld — anchor status: external_doc]
