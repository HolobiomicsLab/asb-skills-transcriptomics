---
name: linear-gene-expression-prediction
description: Use when you have cell-type-specific gene-TF affinity matrices (e.g.
  STARE ABC-mode *_TF_Gene_Affinities.txt.gz derived from gABC, Hi-C, regular ABC
  or Cicero mappings) and need to quantify which enhancer-gene assignment approach
  predicts observed cell-type expression.
when_to_use_negative:
- Input is already a raw single-cell count/accessibility matrix without a gene-TF
  affinity mapping — first derive affinities and gene-level summaries (STARE/TEPIC),
  then apply this skill
- The goal is TF discovery or differential-regulator calling rather than benchmarking
  enhancer-gene mappings — DYNAMITE or network-inference tools like SCENIC are the
  appropriate downstream analyses, not linear expression-prediction comparison
- Predicting expression in cell types with no defined specific-gene set or no corresponding
  activity column in the affinity matrix — the models are trained per cell type on
  that cell type's specific genes
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TEPIC 2 (INVOKE)
  role: Provides the INVOKE linear regression pipeline that learns expression-prediction
    models from TF-gene scores and judges feature importance via F-test
  repo: https://github.com/SchulzLab/TEPIC
- name: STARE
  role: Produces the gene-level TF affinity matrices (one *_TF_Gene_Affinities.txt.gz
    per activity column) that serve as features for the linear models
  repo: https://github.com/schulzlab/STARE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:fded0aa4f33d388dc7043c5a949be7e10e05cc6007df0520b33acd88d39c8131
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchulzLab/TEPIC
  tools:
  - Cicero
  - STARE
  - TEPIC 2 (INVOKE)
  - bedtools
  - stare-abc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# linear-gene-expression-prediction

## Summary

Train INVOKE linear regression models that predict cell-type-specific gene expression from gene-level TF affinity features, and evaluate each regulatory-interaction mapping by the Pearson correlation between predicted and observed expression. In the heart single-cell ATAC-seq case study this distinguishes the gABC mapping (μ ≈ 0.563) from regular ABC (μ ≈ 0.538) and Cicero co-accessibility (μ ≈ 0.505).

## When to use

Use when you have cell-type-specific gene-TF affinity matrices (e.g. STARE ABC-mode *_TF_Gene_Affinities.txt.gz derived from gABC, Hi-C, regular ABC or Cicero mappings) and need to quantify which enhancer-gene assignment approach best predicts observed cell-type expression.

## When NOT to use

- Input is already a raw single-cell count/accessibility matrix without a gene-TF affinity mapping — first derive affinities and gene-level summaries (STARE/TEPIC), then apply this skill
- The goal is TF discovery or differential-regulator calling rather than benchmarking enhancer-gene mappings — DYNAMITE or network-inference tools like SCENIC are the appropriate downstream analyses, not linear expression-prediction comparison
- Predicting expression in cell types with no defined specific-gene set or no corresponding activity column in the affinity matrix — the models are trained per cell type on that cell type's specific genes

## Inputs

- Gene-TF affinity matrix per cell type and per interaction-derivation approach (STARE output *_TF_Gene_Affinities.txt.gz, affinities summed over linked regions weighted by intergenicScore/adapted activity, ABC cutoff -t 0.02)
- Observed cell-type-specific gene expression values for the heart cell types
- Definition of cell-type-specific genes per heart cell type

## Outputs

- Fitted INVOKE linear regression model per cell type per approach
- Predicted expression vector for cell-type-specific genes
- Pearson correlation between predicted and observed expression per cell type and approach, summarised as μ across cell types (e.g. gABC μ ≈ 0.563, regular ABC μ ≈ 0.538, Cicero μ ≈ 0.505)

## How to apply

For each interaction-derivation approach (gABC with adapted activity (-q True) and all_tss TSS-mode (-i all_tss), the Fulco et al. 2019 average hg19 Hi-C matrix, a 500-kb window variant, regular non-adapted ABC (-q False), and Cicero co-accessibility supplied via --existing_abc), take the per-cell-type TF-gene affinity matrix produced by STARE's ABC-mode scaling, in which TF affinities from regions assigned to a gene are summed weighted by the intergenicScore/adapted activity with ABC cutoff -t default 0.02. Using the INVOKE pipeline of TEPIC, train a linear expression-prediction model for the cell-type-specific genes of each heart cell type on each matrix. Predict expression with the fitted model and compute the Pearson correlation between predicted and observed expression per cell type; average across cell types to obtain the μ per approach. The rationale is that a mapping that captures true enhancer-gene links yields affinity features that better explain expression, so higher μ indicates a more accurate interaction assignment (expected ordering: gABC ~0.563 ≥ 500-kb ~0.562 ≥ average Hi-C ~0.556 > regular ABC ~0.538 > Cicero ~0.505).

## Related tools

- **TEPIC 2 (INVOKE)** (Provides the INVOKE linear regression pipeline that learns expression-prediction models from TF-gene scores and judges feature importance via F-test) — https://github.com/SchulzLab/TEPIC
- **STARE** (Produces the gene-level TF affinity matrices (one *_TF_Gene_Affinities.txt.gz per activity column) that serve as features for the linear models) — https://github.com/schulzlab/STARE

## Evaluation signals

- Pearson correlation between predicted and observed expression falls in the expected per-approach ranges: gABC μ ≈ 0.563, 500-kb μ ≈ 0.562, average Hi-C μ ≈ 0.556, regular ABC μ ≈ 0.538, Cicero μ ≈ 0.505
- Every cell type has a fitted model and predicted/observed vector for each of the five mappings; no cell type is missing from the comparison
- The affinity matrix consumed by INVOKE matches the STARE ABC-mode scaling (affinities summed over linked regions weighted by intergenicScore/adapted activity with -t 0.02)
- Relative ordering of approaches is consistent across heart cell types, with gABC and the 500-kb/average Hi-C variants outperforming regular ABC and Cicero

## Limitations

- The approach benchmarks mappings only through linear expression prediction; it does not directly validate enhancer-gene links against perturbation data
- Performance depends on upstream choices (5MB vs 500-kb window, adapted vs non-adapted activity, TSS mode all_tss, ABC cutoff 0.02); changing these alters the affinity features and thus the correlations
- Models are trained per cell type on that cell type's specific genes, so results are specific to the heart single-cell dataset and cell-type gene definitions

## Evidence

- [abstract] We combined our generalized ABC model with TF binding information and illustrated an analysis of a single-cell ATAC-seq dataset of the human heart, where we were able to characterize cell: "We combined our generalized ABC model with TF binding information and illustrated an analysis of a single-cell ATAC-seq dataset of the human heart"
- [intro] Those two parts are combined to summarise TF affinities on gene level, which allows for a variety of downstream analyses: "Those two parts are combined to summarise TF affinities on gene level, which allows for a variety of downstream analyses"
- [readme] Using a linear regression analysis to highlight potential key TFs by predicting gene expression within a sample of interest [MachineLearningPipelines/INVOKE]: [evidence span withheld — anchor status: not_found]
- [methods] STARE summarises the TF-affinities from regions that were assigned to that gene: "STARE summarises the TF-affinities from regions that were assigned to that gene"
