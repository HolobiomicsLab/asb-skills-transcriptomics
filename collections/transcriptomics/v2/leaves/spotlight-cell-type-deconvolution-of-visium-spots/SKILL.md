---
name: spotlight-cell-type-deconvolution-of-visium-spots
description: Use when spot-level spatial data (e.g. 10x Visium slides) need cell-type
  proportion estimates from a matched scRNA-seq reference, to derive colocalization
  z-scores for validating CCC predictions against spatial adjacency.
when_to_use_negative:
- No matched, cell-type-annotated scRNA-seq reference is available for the tissue
  on the Visium slides, so SPOTlight cannot be trained
- The question concerns long-distance endocrine signalling or secreted/diffusing mediators,
  which spot-level colocalization in the sampled tissue cannot capture
- Input is already single-cell resolution spatial data (e.g. seqFISH/merFISH), where
  direct neighbourhood enrichment rather than spot deconvolution is appropriate
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPOTlight
  role: Deconvolves each 10x Visium spot into cell-type proportions using the matched
    scRNA-seq reference
- name: LIANA
  role: Ranks ligand-receptor interactions per cell-type pair so odds ratios of enrichment
    in colocalized pairs can be computed
  repo: https://github.com/saezlab/liana
- name: Squidpy
  role: Runs neighbourhood enrichment with label permutation on seqFISH/merFISH to
    identify statistically adjacent cell-type pairs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:428edbe75af9639cd11a3a2b35b27293d63bc74c4a3960930d1633199239e2d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - SPOTlight
  - Squidpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# spotlight-cell-type-deconvolution-of-visium-spots

## Summary

Deconvolves 10x Visium spatial transcriptomics spots into per-spot cell-type proportions with SPOTlight using a matched scRNA-seq reference, so that pairwise cell-type colocalization z-scores can be used to test whether prioritized cell-cell communication (CCC) predictions are enriched between spatially adjacent cell types.

## When to use

Use when spot-level spatial data (e.g. 10x Visium slides) need cell-type proportion estimates from a matched scRNA-seq reference, to derive colocalization z-scores for validating CCC predictions against spatial adjacency.

## When NOT to use

- No matched, cell-type-annotated scRNA-seq reference is available for the tissue on the Visium slides, so SPOTlight cannot be trained
- The question concerns long-distance endocrine signalling or secreted/diffusing mediators, which spot-level colocalization in the sampled tissue cannot capture
- Input is already single-cell resolution spatial data (e.g. seqFISH/merFISH), where direct neighbourhood enrichment rather than spot deconvolution is appropriate

## Inputs

- Eight 10x Visium slides (murine brain cortex and triple-negative breast cancer)
- Matching scRNA-seq reference dataset with annotated cell types
- LIANA-ranked ligand-receptor interactions per cell-type pair
- seqFISH and merFISH datasets (for orthogonal Squidpy neighbourhood enrichment validation)

## Outputs

- Per-spot cell-type proportion matrix from SPOTlight deconvolution
- Pairwise cell-type colocalization z-scores per slide
- Set of colocalized cell-type pairs (z-score >= 1.645)
- Odds-ratio table written to spatial_colocalization_odds_ratios.csv
- Fig 6B (spatial colocalization agreement) and Supplementary Fig S15 (adjacency odds ratios with merFISH null trend)

## How to apply

Load the 10x Visium slides (e.g. murine brain cortex and triple-negative breast cancer) together with the matching scRNA-seq reference and run SPOTlight deconvolution to estimate cell-type proportions per spatial spot. From the deconvolved proportions, compute pairwise cell-type colocalization z-scores per slide and call colocalized pairs as significant at z-score >= 1.645 (one-sided 95% threshold). Use these colocalized pairs alongside LIANA-ranked ligand-receptor interactions per cell-type pair to compute odds ratios of top-ranked interactions being preferentially found between colocalized versus non-colocalized pairs. Optionally cross-check adjacency with Squidpy neighbourhood enrichment (with label permutation) on seqFISH/merFISH data, and build a label-permutation null distribution of odds ratios for merFISH to establish the expected trend under no adjacency preference. Interpret per-method agreement: in this benchmark all methods except the Crosstalk scores showed the colocalization association in murine cortex, while in triple-negative breast cancer only the consensus and LogFC Mean did.

## Related tools

- **SPOTlight** (Deconvolves each 10x Visium spot into cell-type proportions using the matched scRNA-seq reference)
- **LIANA** (Ranks ligand-receptor interactions per cell-type pair so odds ratios of enrichment in colocalized pairs can be computed) — https://github.com/saezlab/liana
- **Squidpy** (Runs neighbourhood enrichment with label permutation on seqFISH/merFISH to identify statistically adjacent cell-type pairs)

## Evaluation signals

- Colocalized pairs are called only at z-score >= 1.645 (one-sided 95% threshold) per slide
- Odds ratios show a positive trend between cell type colocalisation and prioritised CCC interactions, as reported for the Visium spatial data
- Per-method pattern is reproduced: all methods except Crosstalk scores show the association in murine brain cortex; only consensus and LogFC Mean in triple-negative breast cancer
- merFISH odds ratios fall relative to the label-permutation null trend expected under no adjacency preference
- Odds-ratio table is produced in the expected schema as spatial_colocalization_odds_ratios.csv

## Limitations

- Agreement analyses are limited by restricted coverage of cytokine activity signatures and receptor proteins, and by technical shortcomings of current spatial transcriptomics technologies
- scRNA-seq gene expression does not capture long-distance endocrine signalling events or processes preceding transmitter-receiver interactions such as secretion and diffusion
- No gold standard is available for CCC, so a positive colocalization trend supports but does not definitively validate any method (only consensus and LogFC Mean showed the association in TNBC)

## Evidence

- [abstract] we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance"
- [methods] run SPOTlight deconvolution to estimate cell-type proportions per spatial spot: [evidence span withheld — anchor status: not_found]
- [methods] compute pairwise cell-type colocalization z-scores per slide and call colocalized pairs as significant at z-score >= 1.645 (one-sided 95% threshold): [evidence span withheld — anchor status: not_found]
- [results] In the Visium spatial data, a positive trend was observed between cell type colocalisation and prioritised CCC interactions: [evidence span withheld — anchor status: not_found]
- [discussion] The limitations include the restricted coverage of the cytokine activity signatures and receptor proteins, and the technical shortcomings of current spatial transcriptomics technologies.: "The limitations include the restricted coverage of the cytokine activity signatures and receptor proteins, and the technical shortcomings of current spatial transcriptomics technologies."
