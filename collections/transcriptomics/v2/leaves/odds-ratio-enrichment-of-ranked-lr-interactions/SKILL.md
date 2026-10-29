---
name: odds-ratio-enrichment-of-ranked-lr-interactions
description: 'Use when ranked ligand-receptor predictions from scRNA-seq must be validated
  against spatial transcriptomics: Visium spots with cell-type deconvolution, or seqFISH/merFISH
  cell-type maps, to test adjacency preference.'
when_to_use_negative:
- No spatial information is available (pure dissociated scRNA-seq) — adjacency cannot
  be defined and the odds ratio is not computable
- The goal is comparing method/resource combinations on expression grounds alone;
  this skill is a downstream spatial-agreement check, not a benchmarking oracle (no
  gold standard exists)
- The CCC analysis infers intracellular signal propagation (e.g. NicheNet, Cytotalk,
  SoptSC); these are not directly comparable to ligand-receptor ranking and cannot
  be validated this way
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: LIANA
  role: Ranks ligand-receptor interactions per cell-type pair with each of the 7 benchmarked
    methods and their consensus, providing the ranked interaction lists tested for
    adjacency enrichment
  repo: https://github.com/saezlab/liana
- name: SPOTlight
  role: Deconvolves Visium spot profiles against the scRNA-seq reference to estimate
    cell-type proportions used to compute colocalization z-scores
- name: Squidpy
  role: Runs neighbourhood enrichment with label permutation on seqFISH/merFISH data
    to define statistically adjacent cell-type pairs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:6ecb563a2abaf110859e3e77d8136e1ae38bf8198683da64a4b7dad111d7477f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - SPOTlight
  - Squidpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# odds-ratio enrichment of ranked LR interactions

## Summary

Quantifies whether top-ranked ligand-receptor interactions inferred by CCC methods (via LIANA) are preferentially found between spatially adjacent or colocalized cell-type pairs, expressed as odds ratios with a label-permutation null. It serves as an orthogonal spatial-validation layer for CCC predictions.

## When to use

Use when ranked ligand-receptor predictions from scRNA-seq must be validated against spatial transcriptomics: Visium spots with cell-type deconvolution, or seqFISH/merFISH cell-type maps, to test adjacency preference.

## When NOT to use

- No spatial information is available (pure dissociated scRNA-seq) — adjacency cannot be defined and the odds ratio is not computable
- The goal is comparing method/resource combinations on expression grounds alone; this skill is a downstream spatial-agreement check, not a benchmarking oracle (no gold standard exists)
- The CCC analysis infers intracellular signal propagation (e.g. NicheNet, Cytotalk, SoptSC); these are not directly comparable to ligand-receptor ranking and cannot be validated this way

## Inputs

- 10x Visium spatial slides (murine brain cortex and triple-negative breast cancer) with matching scRNA-seq reference for SPOTlight deconvolution
- Deconvolved cell-type proportions per Visium spot
- seqFISH and merFISH cell-type annotated spatial datasets
- Per cell-type pair ranked ligand-receptor interaction tables from LIANA (each method's score, e.g. LRscore, weight_norm, Probability, plus consensus)

## Outputs

- Pairwise cell-type colocalization z-scores per Visium slide (colocalized pairs at z >= 1.645)
- Squidpy neighbourhood-enrichment adjacency calls for seqFISH/merFISH
- Odds ratios of top-ranked LR interactions in adjacent vs non-adjacent pairs, per CCC method
- Label-permutation null distribution of odds ratios for merFISH
- Fig 6B (spatial colocalization agreement), Supplementary Fig S15 (adjacency odds ratios with null trend)
- spatial_colocalization_odds_ratios.csv

## How to apply

Deconvolve 10x Visium slides (murine brain cortex and triple-negative breast cancer) with SPOTlight to estimate per-spot cell-type proportions, then compute pairwise cell-type colocalization z-scores per slide, calling pairs colocalized at z >= 1.645 (one-sided 95% threshold). Independently, run Squidpy neighbourhood enrichment with label permutation on the seqFISH and merFISH datasets to identify statistically adjacent cell-type pairs. Rank ligand-receptor interactions per cell-type pair with LIANA (using the 7 benchmarked methods and their consensus, e.g. Natural Mean, Truncated Mean, LRscore, Crosstalk score), then compute the odds ratio of top-ranked interactions occurring between adjacent/colocalized pairs versus non-adjacent pairs. For merFISH, build a null distribution of odds ratios by label permutation to establish the expected trend under no adjacency preference. Report per-method odds ratios (Fig 6B, Supplementary Fig S15) and write spatial_colocalization_odds_ratios.csv.

## Related tools

- **LIANA** (Ranks ligand-receptor interactions per cell-type pair with each of the 7 benchmarked methods and their consensus, providing the ranked interaction lists tested for adjacency enrichment) — https://github.com/saezlab/liana
- **SPOTlight** (Deconvolves Visium spot profiles against the scRNA-seq reference to estimate cell-type proportions used to compute colocalization z-scores)
- **Squidpy** (Runs neighbourhood enrichment with label permutation on seqFISH/merFISH data to define statistically adjacent cell-type pairs)

## Examples

```
from scipy.stats import fisher_exact; OR, p = fisher_exact([[top_in_adjacent, top_in_nonadjacent], [rest_in_adjacent, rest_in_nonadjacent]])
```

## Evaluation signals

- Colocalized pairs are called only at one-sided 95% threshold: z-score >= 1.645
- Odds ratio > 1 for top-ranked interactions between adjacent vs non-adjacent pairs; verify direction per method
- merFISH odds ratios must be compared against the label-permutation null distribution, not raw values
- Output CSV spatial_colocalization_odds_ratios.csv must contain one odds ratio per CCC method per dataset
- Consistency check: in murine brain cortex, all methods except the Crosstalk score should show the positive association; in TNBC only the consensus and LogFC Mean should

## Limitations

- Agreement analyses are limited by restricted coverage of cytokine activity signatures and receptor proteins, and by technical shortcomings of current spatial transcriptomics technologies
- The association is a trend, not a definitive validation — no gold standard is available and the biological ground truth for CCC is largely unknown
- Strength of the spatial agreement is method-dependent (e.g. Crosstalk score fails in brain cortex; only consensus and LogFC Mean pass in TNBC)

## Evidence

- [results] spatial colocalization agreement finding: [evidence span withheld — anchor status: not_found]
- [abstract] method-reality agreement claim: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent"
- [readme] LIANA README scope: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus"
- [intro] workflow: evaluate spatial adjacency agreement: "Finally we evaluated the agreement of the different CCC methods with additional modalities, including spatial adjacency, cytokine activities, and protein abundance"
