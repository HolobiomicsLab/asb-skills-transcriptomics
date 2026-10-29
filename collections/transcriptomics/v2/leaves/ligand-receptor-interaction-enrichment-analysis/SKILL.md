---
name: ligand-receptor-interaction-enrichment-analysis
description: Use when you have multiple ligand-receptor interaction resources and
  want to detect over-/underrepresentation of pathway, cancer, or tissue-enriched
  protein annotations in each resource relative to a pooled background.
when_to_use_negative:
- When the goal is benchmarking CCC inference methods against expression data (spatial
  colocalisation, cytokine activities, receptor protein abundance) rather than characterizing
  annotation bias of the resources themselves
- When quantifying raw resource overlap (number of unique or shared interactions)
  rather than testing enrichment against a pooled background
- When no pooled multi-resource background is available, since Fisher's exact test
  requires the 2x2 contingency of resource vs pooled background membership
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: LIANA
  role: Assembles and loads the 16 CCC ligand-receptor resources (interactions, receivers,
    transmitters) used as the test sets and the pooled enrichment background
  repo: https://github.com/saezlab/liana
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:cd391766cdfe9c420ea71018805a942da1d9ede4768dcf36a4e229c6721fce9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ligand-receptor-interaction-enrichment-analysis

## Summary

Tests whether individual cell-cell communication (CCC) ligand-receptor resources are biased toward specific biological annotations (SignaLink/NetPath pathways, CancerSEA cancer gene sets, Human Protein Atlas tissue-enriched proteins, DisGeNet) using Fisher's exact tests against the pooled collection of all 16 resources, with odds ratios summarized in a heatmap.

## When to use

Use when you have multiple ligand-receptor interaction resources and want to detect over-/underrepresentation of pathway, cancer, or tissue-enriched protein annotations in each resource relative to a pooled background.

## When NOT to use

- When the goal is benchmarking CCC inference methods against expression data (spatial colocalisation, cytokine activities, receptor protein abundance) rather than characterizing annotation bias of the resources themselves
- When quantifying raw resource overlap (number of unique or shared interactions) rather than testing enrichment against a pooled background
- When no pooled multi-resource background is available, since Fisher's exact test requires the 2x2 contingency of resource vs pooled background membership

## Inputs

- Ligand-receptor interaction lists (with transmitters and receivers) from the 16 CCC resources assembled in LIANA (Ramilowski, ConnectomeDB, CellChatDB, CellPhoneDB, LRdb, OmniPath, Cellinker, iTALK, etc.)
- Pathway annotation sets from SignaLink and NetPath
- Cancer gene sets from CancerSEA
- Tissue-enriched protein annotations from Human Protein Atlas
- Disease-associated gene annotations from DisGeNet
- Pooled collection of all 16 resources used as the enrichment background

## Outputs

- Per-resource-annotation Fisher's exact test p-values with Benjamini-Hochberg FDR correction
- Odds ratio matrix (resources x annotation terms)
- Significance-annotated odds-ratio heatmap (figure4_odds_ratio_heatmap.png) with under/overrepresentation calls per annotation term

## How to apply

Load the ligand-receptor interactions, receivers, and transmitters from each of the 16 CCC resources assembled in LIANA (e.g., Ramilowski, ConnectomeDB, CellTalkDB, LRdb, OmniPath, Cellinker), then load annotation sets from SignaLink, NetPath, CancerSEA, Human Protein Atlas, and DisGeNet and pool all 16 resources into a single background collection. For each resource-annotation pair, build a 2x2 contingency table (in-annotation vs out-of-annotation for the resource vs the pooled background) and run Fisher's exact test per resource in R. Correct per-resource p-values across annotation terms with Benjamini-Hochberg FDR correction. Compute odds ratios for each resource-annotation pair, assemble them into a matrix, and plot the odds-ratio heatmap as in Fig 4 of the paper, marking significance after FDR correction (e.g., the T-cell receptor pathway underrepresentation in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK vs overrepresentation in OmniPath and Cellinker).

## Related tools

- **LIANA** (Assembles and loads the 16 CCC ligand-receptor resources (interactions, receivers, transmitters) used as the test sets and the pooled enrichment background) — https://github.com/saezlab/liana

## Examples

```
pvals <- sapply(terms, function(t) fisher.test(table(resource_in_ann, in_annotation))$p.value); padj <- p.adjust(pvals, method='BH')
```

## Evaluation signals

- Every resource-annotation pair yields a 2x2 contingency table whose row totals match the resource size and pooled background size
- Benjamini-Hochberg FDR correction is applied across annotation terms within each resource (per-resource p-value vectors)
- Reproduced directional calls match the paper: T cell receptor pathway significantly underrepresented in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK, and overrepresented in OmniPath and Cellinker
- Odds ratio heatmap (Fig 4 equivalent) is saved and marks significance only after FDR correction
- Odds ratios for pairs with FDR-adjusted p-values above threshold are plotted as non-significant rather than interpreted

## Limitations

- Enrichment reflects curation/annotation bias of resources, not biological ground truth; since a gold standard is not available, uneven coverage cannot be judged right or wrong
- Reported curation disagreements and uneven functional distributions among resources call for further large-scale curation efforts before enrichment patterns can be fully interpreted
- Resources are human-focused (single-species), so annotations tested are limited to human pathway/cancer/tissue/disease sets

## Evidence

- [abstract] Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins.: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [intro] First, we explored the degree of overlap among resources and whether certain resources are biased toward specific biological terms, such as pathways and tissue-enriched proteins.: "First, we explored the degree of overlap among resources and whether certain resources are biased toward specific biological terms, such as pathways and tissue-enriched proteins."
- [discussion] the variable overlap between the resources, their uneven functional distributions, and the reported curation disagreements are a call for further large-scale curation efforts.: "the variable overlap between the resources, their uneven functional distributions, and the reported curation disagreements are a call for further large-scale curation efforts."
