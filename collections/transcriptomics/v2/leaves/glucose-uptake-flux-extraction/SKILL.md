---
name: glucose-uptake-flux-extraction
description: Use when METAFlux flux outputs (bulk RNA-seq samples run under human_blood
  medium) are available and a per-sample glucose uptake quantity is needed for correlation
  with FDG-PET SUV or subgroup survival analysis.
when_to_use_negative:
- Input fluxes were computed under cell_medium for cell-line models rather than human_blood
  for human-derived samples — the HMR_9034 extraction was validated under the blood
  nutrient profile
- The goal is genome-wide pathway activity rather than a single metabolite (glucose)
  exchange flux
- Only metabolic gene expression data are available without METAFlux flux computation
  — the article notes the LUAD subgroup result is 'not obtainable from metabolic gene
  expression data alone'
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
tools:
- name: METAFlux
  role: Computes MRAS from expression via GPR rules and solves FBA over the 13082-reaction
    Human-GEM model under human_blood medium to yield per-sample fluxes including
    HMR_9034
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Execution environment for METAFlux (library(METAFlux)), reaction extraction,
    Spearman correlation, and survival analysis
- name: Seurat
  role: Used to cluster TCGA LUAD flux profiles and embed with UMAP to identify the
    LUSC-like LUAD group whose glucose uptake differs
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:347cfb59d607d86480c917c09e78c545662b1b98e3d803c6360f59d0db30fc38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# glucose-uptake-flux-extraction

## Summary

Extracts per-sample glucose uptake fluxes from METAFlux flux-balance outputs by pulling the HMR_9034 exchange reaction, enabling comparison of predicted glucose uptake with measured phenotypes such as FDG-PET SUV or survival-defined lung cancer subgroups.

## When to use

Use when METAFlux flux outputs (bulk RNA-seq samples run under human_blood medium) are available and a per-sample glucose uptake quantity is needed for correlation with FDG-PET SUV or subgroup survival analysis.

## When NOT to use

- Input fluxes were computed under cell_medium for cell-line models rather than human_blood for human-derived samples — the HMR_9034 extraction was validated under the blood nutrient profile
- The goal is genome-wide pathway activity rather than a single metabolite (glucose) exchange flux
- Only metabolic gene expression data are available without METAFlux flux computation — the article notes the LUAD subgroup result is 'not obtainable from metabolic gene expression data alone'

## Inputs

- METAFlux flux matrix (samples x 13082 Human-GEM reactions) from compute_flux under human_blood medium
- Nutrient lookup files listing 1648 metabolite exchange Reaction IDs (to identify HMR_9034)
- Matched measured FDG-PET SUV values per sample (for correlation validation)
- Normalized, non-negative bulk RNA-seq gene x sample matrix with human gene symbols as row names

## Outputs

- Per-sample glucose uptake flux table (HMR_9034 values, CSV)
- Spearman correlation coefficient and P-value between predicted glucose uptake and measured SUV
- Group-level glucose uptake comparison statistics (e.g., LUSC vs LUAD) and survival-curve PNGs when paired with clustering

## How to apply

Run METAFlux on a gene x sample expression matrix (human gene symbols as row names, normalized, non-negative) by first computing single-sample normalized MRAS with calculate_reaction_score, then solving fluxes with compute_flux(mras=scores, medium=human_blood), optionally applying cubic-root normalization cbrt(x) = sign(x)*abs(x)^(1/3). Locate the glucose exchange reaction ID HMR_9034 by searching the nutrient lookup files (1648 exchange reactions), then extract that reaction's flux per sample, remembering that negative values denote uptake for exchange reactions. In TNBC validation, the extracted fluxes should correlate with measured SUV at Spearman rho=0.31 (P=0.003); in TCGA LUAD/LUSC, glucose uptake is expected to be higher in LUSC and in the LUSC-like LUAD cluster (which shows worse survival, log-rank P=0.0084). Export the per-sample flux table (CSV) alongside correlation or survival statistics.

## Related tools

- **METAFlux** (Computes MRAS from expression via GPR rules and solves FBA over the 13082-reaction Human-GEM model under human_blood medium to yield per-sample fluxes including HMR_9034) — https://github.com/KChen-lab/METAFlux
- **R** (Execution environment for METAFlux (library(METAFlux)), reaction extraction, Spearman correlation, and survival analysis)
- **Seurat** (Used to cluster TCGA LUAD flux profiles and embed with UMAP to identify the LUSC-like LUAD group whose glucose uptake differs)

## Examples

```
library(METAFlux); flux <- compute_flux(mras=scores, medium=human_blood); guf <- flux['HMR_9034', ]; cor.test(guf, suv, method='spearman')
```

## Evaluation signals

- TNBC validation: Spearman rho between extracted glucose uptake flux (HMR_9034) and measured SUV equals 0.31 with P=0.003 across 84 samples
- TCGA validation: LUSC glucose uptake flux significantly exceeds LUAD, and the LUSC-like LUAD cluster shows higher glucose uptake with worse survival (log-rank P=0.0084)
- Sign convention check: extracted HMR_9034 values are negative, since negative values denote uptake for exchange reactions
- Reaction ID check: HMR_9034 is confirmed as the glucose exchange reaction via the nutrient lookup files before extraction
- Flux matrix completeness: output contains 13082 reaction flux scores per bulk sample

## Limitations

- Glucose uptake does not correlate as consistently as other metabolites in the NCI-60 panel, so accuracy is dataset-dependent
- Requires the human_blood medium choice (64 blood metabolites) for human-derived samples; using cell_medium changes the nutrient constraints
- Exchange-reaction sign convention (negative = uptake) must be handled correctly, and cubic-root normalization is optional and changes the scale of flux values

## Evidence

- [results] Extract glucose uptake exchange reaction HMR_9034: [evidence span withheld — anchor status: not_found]
- [results] Negative value denotes uptake: [evidence span withheld — anchor status: not_found]
- [results] Spearman rho 0.31 with SUV: [evidence span withheld — anchor status: not_found]
- [methods] Nutrient lookup files exchange reactions: "'Nutrient look up files' contains 1648 exchange reactions"
- [methods] Compute flux with human_blood medium: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [readme] README FBA capabilities: [evidence span withheld — anchor status: external_doc]
