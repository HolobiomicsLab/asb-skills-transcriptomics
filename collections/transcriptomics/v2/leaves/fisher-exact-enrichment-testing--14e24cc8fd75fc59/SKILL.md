---
name: fisher-exact-enrichment-testing
description: Use when comparing the composition of ligand-receptor sets from CCC resources
  against annotation terms (pathways, cancer genes, tissue-enriched proteins), or
  when testing whether top-ranked CCC interactions overlap significantly with active
  cytokines.
when_to_use_negative:
- The goal is to score cytokine activities or pathway activities from pseudobulk profiles
  themselves — use decoupleR's multivariate linear model (mlm) instead of an overlap
  enrichment test
- The goal is to compare the magnitude/ranking behaviour of CCC methods against spatial
  colocalisation or receptor protein abundance — this skill only tests set-overlap
  enrichment and cannot evaluate score calibration
- The aim is to infer actual communication events — the article notes such benchmarks
  'cannot distinguish simple co-expression from actual CCC events', so odds-ratio
  enrichment is a compositional, not mechanistic, signal
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LIANA
  role: Provides the 16 ligand-receptor resources (and pooled background) and ranks
    cytokine-receptor interactions per cell-type pair prior to overlap testing
  repo: github.com/saezlab/liana
- name: OmnipathR
  role: Retrieves cytokine-receptor interactions and cytokine target gene sets from
    OmniPath that define the annotation categories tested
  repo: saezlab/OmnipathR
- name: decoupleR
  role: Scores cytokine activities (mlm, estimate and p-value per cytokine per cell
    type) whose significant calls define the 'active' set in the Fig 6A overlap Fisher
    tests
  repo: saezlab/decoupleR
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:f1580047984a91f7c858a4a30807502018ef591161735dfcf14007bb9aaac11c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellChat
  - CellPhoneDB
  - LIANA
  - OmnipathR
  - decoupleR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fisher-exact-enrichment-testing

## Summary

Applies Fisher's exact tests on 2x2 contingency tables to quantify whether ligand-receptor interactions from individual CCC resources (or highly ranked interactions from CCC methods) are over- or under-represented in annotation sets, expressed as odds ratios with Benjamini-Hochberg FDR correction.

## When to use

Use when comparing the composition of ligand-receptor sets from CCC resources against annotation terms (pathways, cancer genes, tissue-enriched proteins), or when testing whether top-ranked CCC interactions overlap significantly with active cytokines.

## When NOT to use

- The goal is to score cytokine activities or pathway activities from pseudobulk profiles themselves — use decoupleR's multivariate linear model (mlm) instead of an overlap enrichment test
- The goal is to compare the magnitude/ranking behaviour of CCC methods against spatial colocalisation or receptor protein abundance — this skill only tests set-overlap enrichment and cannot evaluate score calibration
- The aim is to infer actual communication events — the article notes such benchmarks 'cannot distinguish simple co-expression from actual CCC events', so odds-ratio enrichment is a compositional, not mechanistic, signal

## Inputs

- Ligand-receptor interaction lists, receivers, and transmitters from each of the 16 CCC resources assembled in LIANA
- Annotation sets from SignaLink, NetPath, CancerSEA, Human Protein Atlas, and DisGeNet, plus the pooled background of all 16 resources
- Cytokine-receptor interactions and cytokine target gene sets retrieved from OmniPath via OmnipathR
- LIANA-ranked cytokine-receptor interactions per cell-type pair (after CellChat/CellPhoneDB post-filtering of insufficiently expressed receptor-complex subunits or transmitter/receiver genes)
- decoupleR mlm cytokine activity estimates and p-values per cytokine per cell type from HER2+ and triple-negative breast cancer pseudobulk profiles

## Outputs

- Per-resource-annotation Fisher's exact test p-values with Benjamini-Hochberg FDR-adjusted significance calls
- Odds-ratio matrix per resource-annotation pair, visualised as the Fig 4 odds-ratio heatmap (figure4_odds_ratio_heatmap.png)
- Fisher's exact test odds ratios per cumulative rank interval (100-10000) per dataset, assembled into the Fig 6A odds-ratio matrix and saved as fig6a_odds_ratios.csv

## How to apply

For resource-bias testing (Fig 4), pool the 16 ligand-receptor resources assembled in LIANA into a background collection, then for each resource-annotation pair (SignaLink, NetPath, CancerSEA, Human Protein Atlas, DisGeNet) build a 2x2 contingency table of in-annotation vs out-of-annotation counts for the resource vs the pooled background, and run fisher.test per resource in R. Correct the per-resource p-values across annotation terms with Benjamini-Hochberg FDR correction, compute odds ratios for each pair, assemble them into a matrix, and plot as an odds-ratio heatmap with significance calls (e.g. T cell receptor pathway underrepresentation in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, iTALK and overrepresentation in OmniPath and Cellinker). For the cytokine-activity agreement analysis (Fig 6A), rank cytokine-receptor interactions with LIANA per cell-type pair after CellChat/CellPhoneDB-style filtering, bin them into cumulative rank intervals spanning 100-10000, tabulate overlap with cytokines that have significant decoupleR mlm activity (p-value per cytokine per cell type on pseudobulk profiles), and compute Fisher's exact test odds ratios per rank interval per dataset. Decision point: significance is judged on FDR-adjusted p-values, and effect size is reported as the odds ratio rather than the p-value alone.

## Related tools

- **LIANA** (Provides the 16 ligand-receptor resources (and pooled background) and ranks cytokine-receptor interactions per cell-type pair prior to overlap testing) — github.com/saezlab/liana
- **OmnipathR** (Retrieves cytokine-receptor interactions and cytokine target gene sets from OmniPath that define the annotation categories tested) — saezlab/OmnipathR
- **decoupleR** (Scores cytokine activities (mlm, estimate and p-value per cytokine per cell type) whose significant calls define the 'active' set in the Fig 6A overlap Fisher tests) — saezlab/decoupleR

## Examples

```
tab <- table(in_annotation, in_resource); ft <- fisher.test(tab); or <- ft$estimate; p_adj <- p.adjust(ft$p.value, method='BH')
```

## Evaluation signals

- Each 2x2 contingency table's row/column margins must sum to the resource size and the pooled 16-resource background size (a+b+c+d consistent with N)
- Odds ratios recomputed as (a*d)/(b*c) should match the values assembled in the Fig 4 matrix; significant cells must be marked only when BH-FDR-adjusted p < 0.05
- fig6a_odds_ratios.csv must contain one odds ratio per dataset x method x cumulative rank interval (intervals spanning 100-10000), with odds ratios highest in the most prioritised rank bins and converging toward the random baseline as more interactions are considered
- Directionality sanity check: T cell receptor pathway should come out underrepresented (odds ratio < 1, FDR-significant) in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK, and overrepresented (odds ratio > 1) in OmniPath and Cellinker
- Fig 6A trends should be 'generally positive' between cytokine activities and the most prioritised CCC interactions across all methods, per the reported finding

## Limitations

- The benchmark cannot distinguish simple co-expression from actual CCC events and does not capture complex relationships between CCC events, so an enriched overlap does not prove communication
- Agreement analyses are limited by the restricted coverage of the cytokine activity signatures and receptor proteins
- In the cytokine-activity analysis, trends converge toward the random baseline as more interactions are considered, so only the top-ranked intervals are informative

## Evidence

- [abstract] Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins.: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [methods] For each resource-annotation pair, build a 2x2 contingency table (in-annotation vs out-of-annotation for the resource vs the pooled background) and run Fisher's exact test per resource in R.: [evidence span withheld — anchor status: not_found]
- [methods] Correct the per-resource p-values across annotation terms with Benjamini-Hochberg FDR correction.: [evidence span withheld — anchor status: not_found]
- [methods] Compute Fisher's exact test odds ratios per rank interval for each dataset and assemble the Fig 6A odds-ratio matrix.: [evidence span withheld — anchor status: not_found]
- [results] the T cell receptor pathway was significantly underrepresented in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK: [evidence span withheld — anchor status: not_found]
- [readme] LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus.: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
