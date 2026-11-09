---
name: spatial-adjacency-permutation-null-testing
description: Use when ligand-receptor interactions inferred from scRNA-seq must be
  validated against spatial transcriptomics — e.g. checking whether top-ranked CCC
  pairs occur more often between adjacent/colocalized cell types.
when_to_use_negative:
- The data lack any spatial coordinates or matched spatial transcriptomics modality,
  so adjacency/colocalization cannot be assessed
- The goal is to benchmark CCC methods against cytokine activities or receptor protein
  abundance rather than spatial adjacency (these are separate agreement analyses in
  the paper)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: LIANA
  role: Ranks ligand-receptor interactions per cell-type pair using 7 methods plus
    consensus, supplying the interaction rankings tested for adjacency enrichment
  repo: https://github.com/saezlab/liana
- name: SPOTlight
  role: Deconvolves Visium spot data to estimate cell-type proportions per spatial
    spot for colocalization z-scores
- name: Squidpy
  role: Runs neighbourhood enrichment with label permutation to identify statistically
    adjacent cell-type pairs in seqFISH/merFISH data
- name: NATMI
  role: One of the CCC methods (Mean-expression edge weight from ConnectomeDB) whose
    ranked predictions are evaluated against spatial adjacency
  repo: https://github.com/asrhou/NATMI.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:5d184f94da0af504986b70a09ac06d916af22882d9eaf0375baa740a84c7970e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - NATMI
  - SPOTlight
  - Squidpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# spatial-adjacency-permutation-null-testing

## Summary

A validation strategy that tests whether top-ranked cell-cell communication (CCC) ligand-receptor predictions are preferentially enriched between spatially adjacent or colocalized cell types, using label-permutation null distributions and odds ratios to benchmark CCC methods.

## When to use

Use when ligand-receptor interactions inferred from scRNA-seq must be validated against spatial transcriptomics — e.g. checking whether top-ranked CCC pairs occur more often between adjacent/colocalized cell types.

## When NOT to use

- The data lack any spatial coordinates or matched spatial transcriptomics modality, so adjacency/colocalization cannot be assessed
- The goal is to benchmark CCC methods against cytokine activities or receptor protein abundance rather than spatial adjacency (these are separate agreement analyses in the paper)

## Inputs

- 10x Visium spatial slides with matched scRNA-seq reference (for SPOTlight deconvolution)
- seqFISH and merFISH spatial datasets with cell-type labels
- scRNA-seq expression matrix with cell-type annotations for LIANA ligand-receptor ranking

## Outputs

- Pairwise cell-type colocalization z-scores per Visium slide
- Statistically adjacent cell-type pairs from Squidpy neighbourhood enrichment
- Odds ratios of top-ranked interactions between adjacent/colocalized vs non-adjacent pairs
- Label-permutation null distribution of odds ratios (merFISH)
- spatial_colocalization_odds_ratios.csv odds-ratio table, Fig 6B and Supplementary Fig S15 plots

## How to apply

Deconvolve spatial data (e.g. eight 10x Visium slides of murine brain cortex and triple-negative breast cancer) with SPOTlight to obtain per-spot cell-type proportions, then compute pairwise cell-type colocalization z-scores per slide, calling pairs colocalized at z >= 1.645 (one-sided 95% threshold). Independently, run Squidpy neighbourhood enrichment with label permutation on seqFISH/merFISH data to identify statistically adjacent cell-type pairs. Rank ligand-receptor interactions per cell-type pair with LIANA (7 methods plus consensus), then compute odds ratios of top-ranked interactions falling between adjacent or colocalized pairs versus non-adjacent pairs. For merFISH, build a label-permutation null distribution of odds ratios to establish the expected trend under no adjacency preference. Consistent positive trends (all methods except Crosstalk scores in murine cortex; only consensus and LogFC Mean in TNBC) indicate spatial coherence of a method's predictions.

## Related tools

- **LIANA** (Ranks ligand-receptor interactions per cell-type pair using 7 methods plus consensus, supplying the interaction rankings tested for adjacency enrichment) — https://github.com/saezlab/liana
- **SPOTlight** (Deconvolves Visium spot data to estimate cell-type proportions per spatial spot for colocalization z-scores)
- **Squidpy** (Runs neighbourhood enrichment with label permutation to identify statistically adjacent cell-type pairs in seqFISH/merFISH data)
- **NATMI** (One of the CCC methods (Mean-expression edge weight from ConnectomeDB) whose ranked predictions are evaluated against spatial adjacency) — https://github.com/asrhou/NATMI.git

## Examples

```
import squidpy as sq; sq.gr.nhood_enrichment(adata, cluster_key="cell_type", n_perms=1000)
```

## Evaluation signals

- Colocalized pairs are called at z-score >= 1.645 (one-sided 95% threshold) from SPOTlight deconvolution proportions
- Odds ratios > 1 for top-ranked interactions between adjacent/colocalized vs non-adjacent pairs indicate the expected positive trend
- Observed merFISH odds ratios are compared against the label-permutation null distribution of odds ratios under no adjacency preference
- Expected method-level pattern: in murine brain cortex all methods except Crosstalk scores show the association; in TNBC only consensus and LogFC Mean do
- The odds-ratio table is written to spatial_colocalization_odds_ratios.csv and plots match Fig 6B / Supplementary Fig S15

## Limitations

- No gold standard is available and the biological ground truth for CCC is largely unknown, so agreement with spatial colocalisation cannot identify a definitively best method
- Benchmarks of this kind cannot distinguish simple co-expression from actual CCC events, and do not capture complex relationships between CCC events
- Agreement analyses are limited by restricted coverage of cytokine activity signatures and receptor proteins, and by technical shortcomings of current spatial transcriptomics technologies
- Gene expression is a proxy of protein levels alone and does not capture long-distance endocrine signalling or processes preceding transmitter-receiver interactions

## Evidence

- [results] Positive trend between colocalization and ranked interactions: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those"
- [intro] Systematic method-resource comparison scope: [evidence span withheld — anchor status: not_found]
- [intro] Spatial adjacency agreement evaluation step: "Finally we evaluated the agreement of the different CCC methods with additional modalities, including spatial adjacency, cytokine activities, and protein abundance."
- [discussion] No gold standard limitation: "Since a gold standard is currently not available and the biological ground truth is largely unknown2,25, our analyses cannot give a definitive answer of what method is best."
- [discussion] Benchmark confounding limitation: "such benchmarks cannot distinguish simple co-expression from actual CCC events, and do not capture complex relationships between CCC events."
- [abstract] LIANA framework supports method-resource combinations: "we provide LIANA, a LIgand-receptor ANalysis frAmework as an open-source interface to all the resources and methods"
