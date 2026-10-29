---
name: annotation-resource-gene-mapping
description: Use when you need to test whether CCC resources (Ramilowski, ConnectomeDB,
  OmniPath, Cellinker, etc.) are biased toward specific pathways or tissue-enriched
  proteins by enriching resource genes against annotation sets.
when_to_use_negative:
- Input is raw scRNA-seq counts or expression matrices — this skill operates on curated
  ligand-receptor interaction lists and annotation gene sets, not expression quantification
- The goal is inferring pathway/cytokine activities from expression (e.g. PROGENy
  + decoupleR) rather than testing whether resource curation is enriched for specific
  biological terms
- The goal is benchmarking CCC methods against spatial adjacency, cytokine activities,
  or receptor protein abundance — that is the methods-agreement analysis, not resource
  annotation enrichment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3366
tools:
- name: LIANA
  role: Assembles and provides the 16 ligand-receptor interaction resources (interactions,
    receivers, transmitters) used as the resource side of the contingency tables and
    as the pooled background
  repo: https://github.com/saezlab/liana
- name: OmnipathR
  role: R client to retrieve OmniPath (and other) resource data used in the resource/annotation
    mapping
  repo: https://github.com/saezlab/OmnipathR
- name: NATMI
  role: Source of the connectomeDB2020 curated ligand-receptor interaction list, one
    of the resources mapped against annotation sets
  repo: https://github.com/asrhou/NATMI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:43ba7552301eae5b86027478b3ed075feb3d608a0847ab87cb81a51ef3d3073f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - NATMI
  - OmnipathR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# annotation-resource-gene-mapping

## Summary

Maps ligand-receptor genes from each of 16 cell-cell communication (CCC) resources (assembled in LIANA) onto external annotation sets (SignaLink, NetPath, CancerSEA, Human Protein Atlas, DisGeNet) and quantifies over-/under-representation of biological terms with per-resource 2x2 Fisher's exact tests and Benjamini-Hochberg FDR correction.

## When to use

Use when you need to test whether CCC resources (Ramilowski, ConnectomeDB, OmniPath, Cellinker, etc.) are biased toward specific pathways or tissue-enriched proteins by enriching resource genes against annotation sets.

## When NOT to use

- Input is raw scRNA-seq counts or expression matrices — this skill operates on curated ligand-receptor interaction lists and annotation gene sets, not expression quantification
- The goal is inferring pathway/cytokine activities from expression (e.g. PROGENy + decoupleR) rather than testing whether resource curation is enriched for specific biological terms
- The goal is benchmarking CCC methods against spatial adjacency, cytokine activities, or receptor protein abundance — that is the methods-agreement analysis, not resource annotation enrichment

## Inputs

- Ligand-receptor interaction lists (source/target genesymbol, receivers, transmitters) from 16 CCC resources available via LIANA (e.g. Ramilowski, ConnectomeDB, CellChatDB, CellPhoneDB, LRdb, OmniPath, Cellinker, CellTalkDB, iTALK)
- Annotation gene sets from SignaLink, NetPath, CancerSEA, Human Protein Atlas, and DisGeNet
- Pooled background collection of all 16 resources' genes

## Outputs

- Per resource-annotation pair: 2x2 contingency table, Fisher's exact p-value, BH-adjusted q-value, and odds ratio
- Resource x annotation odds-ratio matrix with significance markers
- Odds-ratio heatmap (figure4_odds_ratio_heatmap.png) with T-cell receptor pathway under/overrepresentation calls

## How to apply

Load the ligand-receptor interactions, receivers, and transmitters from each of the 16 CCC resources assembled in LIANA, and load annotation gene sets from SignaLink, NetPath, CancerSEA, Human Protein Atlas, and DisGeNet; also pool all 16 resources into a background collection. For each resource-annotation pair, build a 2x2 contingency table (in-annotation vs out-of-annotation, resource vs pooled background) and run Fisher's exact test per resource in R. Correct per-resource p-values across annotation terms with Benjamini-Hochberg FDR, then compute odds ratios per resource-annotation pair. Assemble odds ratios into a matrix and plot the heatmap (Fig 4), marking significance after FDR correction — e.g. the T cell receptor pathway is significantly underrepresented in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK but overrepresented in OmniPath and Cellinker.

## Related tools

- **LIANA** (Assembles and provides the 16 ligand-receptor interaction resources (interactions, receivers, transmitters) used as the resource side of the contingency tables and as the pooled background) — https://github.com/saezlab/liana
- **OmnipathR** (R client to retrieve OmniPath (and other) resource data used in the resource/annotation mapping) — https://github.com/saezlab/OmnipathR
- **NATMI** (Source of the connectomeDB2020 curated ligand-receptor interaction list, one of the resources mapped against annotation sets) — https://github.com/asrhou/NATMI

## Examples

```
Rscript -e 'ct <- matrix(c(n_rr, n_ro, n_or, n_oo), nrow=2); fisher.test(ct)'  # 2x2 table: resource-vs-pooled-background x in-annotation-vs-out
```

## Evaluation signals

- Each contingency table's cell counts sum to the correct resource size (row margin) and pooled-background size (column margin) for every resource-annotation pair
- Every resource-annotation pair yields a finite odds ratio and BH-corrected q-value in [0,1]
- Directionality check: T cell receptor pathway is underrepresented (significantly, after FDR) in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK, and overrepresented in OmniPath and Cellinker
- Only few interactions are unique to individual resources, consistent with a high background overlap across the 16 resources
- Output heatmap reproduces the Fig 4 layout with FDR-significance markers and is saved as figure4_odds_ratio_heatmap.png

## Limitations

- No gold standard exists for CCC, so enrichment of curated resources reflects curation coverage, not biological ground truth about best resources
- Annotation-set coverage is restricted (e.g. cytokine activity signatures and receptor proteins), which limits which terms can be tested
- Current resources and mappings are effectively single-species (human gene symbols); interspecies communication cannot be directly tested

## Evidence

- [abstract] Uneven coverage finding: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [intro] Analysis goal of resource bias testing: "First, we explored the degree of overlap among resources and whether certain resources are biased toward specific biological terms, such as pathways and tissue-enriched proteins."
- [readme] OmnipathR as access client: [evidence span withheld — anchor status: external_doc]
- [readme] connectomeDB2020 resource content: [evidence span withheld — anchor status: external_doc]
