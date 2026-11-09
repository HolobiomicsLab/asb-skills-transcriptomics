---
name: spatial-colocalization-zscore-estimation
description: Use when spot-level spatial transcriptomics data (e.g., 10x Visium slides
  of murine brain cortex or triple-negative breast cancer) must be converted into
  colocalized vs non-colocalized cell-type pairs to validate ligand-receptor predictions.
when_to_use_negative:
- Input is single-cell (non-spatial) data only, with no spot-level deconvolution available
  — spatial colocalization z-scores cannot be computed and Squidpy adjacency on seqFISH/merFISH
  is the alternative.
- The goal is intracellular signalling inference (NicheNet, Cytotalk, SoptSC), which
  the paper notes is not directly comparable to ligand-receptor methods benchmarked
  against spatial colocalization.
- The study concerns long-distance endocrine signalling, which scRNA-seq gene expression
  does not capture.
tools:
- name: SPOTlight
  role: Deconvolves 10x Visium spot expression using the scRNA-seq reference to estimate
    cell-type proportions per spot, the basis for colocalization z-scores.
- name: LIANA
  role: Ranks ligand-receptor interactions per cell-type pair across methods (including
    the consensus) so odds ratios against colocalized vs non-colocalized pairs can
    be computed.
  repo: github.com/saezlab/liana
- name: Squidpy
  role: Runs neighbourhood enrichment with label permutation on seqFISH/merFISH data
    to identify statistically adjacent cell-type pairs and to build the permutation
    null for odds ratios.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:61e0b73b2775ec0eadab18dc8e7c71950237c270138c7d323002ef7c4c8d4540
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LIANA
  - SPOTlight
  - Squidpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spatial-colocalization-zscore-estimation

## Summary

Quantifies whether cell types spatially co-occur by computing pairwise cell-type colocalization z-scores from SPOTlight deconvolved 10x Visium spot proportions, flagging colocalized pairs at z-score >= 1.645 (one-sided 95% threshold), to test agreement of prioritized CCC interactions with spatial adjacency.

## When to use

Use when spot-level spatial transcriptomics data (e.g., 10x Visium slides of murine brain cortex or triple-negative breast cancer) must be converted into colocalized vs non-colocalized cell-type pairs to validate ligand-receptor predictions.

## When NOT to use

- Input is single-cell (non-spatial) data only, with no spot-level deconvolution available — spatial colocalization z-scores cannot be computed and Squidpy adjacency on seqFISH/merFISH is the alternative.
- The goal is intracellular signalling inference (NicheNet, Cytotalk, SoptSC), which the paper notes is not directly comparable to ligand-receptor methods benchmarked against spatial colocalization.
- The study concerns long-distance endocrine signalling, which scRNA-seq gene expression does not capture.

## Inputs

- 10x Visium spatial slides with spot-level gene expression (murine brain cortex and triple-negative breast cancer)
- Matched scRNA-seq reference dataset with cell-type annotations
- seqFISH and merFISH datasets for adjacency cross-validation
- LIANA-ranked ligand-receptor interaction table per cell-type pair

## Outputs

- Per-spot cell-type proportion matrix from SPOTlight deconvolution
- Pairwise cell-type colocalization z-scores per slide with colocalized/non-colocalized labels (z >= 1.645)
- Odds ratios of top-ranked CCC interactions between colocalized vs non-colocalized pairs (spatial_colocalization_odds_ratios.csv)
- Plots of colocalization agreement (Fig 6B, Supplementary Fig S15 with merFISH permutation-null trend)

## How to apply

Load the 10x Visium slides together with the matching scRNA-seq reference and run SPOTlight deconvolution to estimate cell-type proportions per spatial spot. From the deconvolved proportions, compute pairwise cell-type colocalization z-scores per slide; pairs with z-score >= 1.645 (one-sided 95% threshold) are called colocalized, the remainder non-colocalized. Rank ligand-receptor interactions per cell-type pair with LIANA (across the 7 compared methods plus the consensus, including Crosstalk scores and LogFC Mean), then compute odds ratios of top-ranked interactions being preferentially found between colocalized versus non-colocalized pairs. Cross-validate the colocalization-based calls against Squidpy neighbourhood enrichment (label permutation) on seqFISH/merFISH data, building a permutation null distribution of odds ratios to establish the expected trend under no adjacency preference. The rationale is that CCC predictions that reflect real communication should be coherent with physical cell-type proximity.

## Related tools

- **SPOTlight** (Deconvolves 10x Visium spot expression using the scRNA-seq reference to estimate cell-type proportions per spot, the basis for colocalization z-scores.)
- **LIANA** (Ranks ligand-receptor interactions per cell-type pair across methods (including the consensus) so odds ratios against colocalized vs non-colocalized pairs can be computed.) — github.com/saezlab/liana
- **Squidpy** (Runs neighbourhood enrichment with label permutation on seqFISH/merFISH data to identify statistically adjacent cell-type pairs and to build the permutation null for odds ratios.)

## Evaluation signals

- Colocalized pairs are exactly those with pairwise z-score >= 1.645 per slide (one-sided 95% threshold).
- In the murine brain cortex, all methods except the Crosstalk scores should show the positive colocalization–CCC association; in triple-negative breast cancer only the consensus and LogFC Mean should.
- merFISH odds ratios fall relative to the label-permutation null distribution of odds ratios (expected trend under no adjacency preference).
- Output table spatial_colocalization_odds_ratios.csv contains odds ratios for each method per dataset/slide.
- Figure 6B reproduces the positive trend between cell type colocalisation and prioritised CCC interactions in the Visium spatial data.

## Limitations

- The association is method-dependent: Crosstalk scores fail to show it in the murine brain cortex, and in triple-negative breast cancer only the consensus and LogFC Mean do.
- Spatial transcriptomics technical shortcomings and restricted receptor-protein coverage limit the agreement analyses.
- scRNA-seq gene expression does not capture long-distance endocrine signalling events, so spatial proximity validation cannot cover such mechanisms.
- No gold standard exists for CCC, so colocalization coherence is an indirect, not definitive, validation.

## Evidence

- [other] From the deconvolved proportions, compute pairwise cell-type colocalization z-scores per slide and call colocalized pairs as significant at z-score >= 1.645 (one-sided 95% threshold).: [evidence span withheld — anchor status: not_found]
- [other] Positive trend between colocalisation and prioritised CCC interactions.: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract: predictions are generally coherent with spatial colocalisation.: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those"
- [discussion] Discussion: spatial technologies limit agreement analyses.: "The limitations include the restricted coverage of the cytokine activity signatures and receptor proteins, and the technical shortcomings of current spatial transcriptomics technologies."
- [other] Cortex/TNBC method-specific agreement pattern.: [evidence span withheld — anchor status: not_found]
