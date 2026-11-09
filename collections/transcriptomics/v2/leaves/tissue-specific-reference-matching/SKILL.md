---
name: tissue-specific-reference-matching
description: Use when annotating scRNA-seq clusters by matching identified marker
  genes (e.g. from a paired one-against-all comparison) against a curated tissue-specific
  reference of literature-reported markers to filter pseudo markers and rank candidate
  cell types.
when_to_use_negative:
- The tissue or cell types of interest are absent from CellMatch — the article notes
  a limited reference database potentially led to incorrect cell type annotation
- Input is single-cell data without cluster definitions and derived marker genes (this
  skill annotates clusters, not individual cells)
- A cell-level annotation method is required where multiple distinct cell types are
  mixed within one cluster and cluster-level annotation is inappropriate
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scCATCH
  role: Provides findmarkergene() and findcelltype() to identify cluster markers and
    match/score them against the CellMatch reference for automatic cluster annotation
  repo: https://github.com/ZJUFanLab/scCATCH
- name: cellmatch
  role: The tissue-specific cell taxonomy reference data.frame (353 cell types, 686
    subtypes, 184 tissues, 2,096 human/mouse references) that marker genes are matched
    against; custom versions are supported
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_008
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:7667767aff81b62604ddde43a37604aefa493b17d49f5629306647c6468fb28f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scCATCH
  repo_url: https://github.com/ZJUFanLab/scCATCH
  tools:
  - SingleR
  - cellmatch
  - scCATCH
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tissue-specific-reference-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Match cluster-derived candidate marker genes against a tissue-specific cell taxonomy reference database (CellMatch: 353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references) to obtain validated marker-supported candidate cell types for each cluster, as implemented in scCATCH.

## When to use

Use when annotating scRNA-seq clusters by matching identified marker genes (e.g. from a paired one-against-all comparison) against a curated tissue-specific reference of literature-reported markers to filter pseudo markers and rank candidate cell types.

## When NOT to use

- The tissue or cell types of interest are absent from CellMatch — the article notes a limited reference database potentially led to incorrect cell type annotation
- Input is single-cell data without cluster definitions and derived marker genes (this skill annotates clusters, not individual cells)
- A cell-level annotation method is required where multiple distinct cell types are mixed within one cluster and cluster-level annotation is inappropriate

## Inputs

- scCATCH object from createscCATCH(data = sc_data, cluster = sc_cluster) holding the scRNA-seq count matrix and cluster labels
- Cluster marker gene results (e.g. demo_marker table after findmarkergene)
- cellmatch reference data.frame bundled with scCATCH (353 cell types / 686 subtypes / 184 tissues / 2,096 human and mouse references)
- geneinfo annotation data (scCATCH 'geneinfo' / 'demo_geneinfo') for gene name handling

## Outputs

- Per-cluster candidate cell types supported by matched validated marker genes and their supporting literature references
- Evidence scores per candidate cell type and the top-1 assigned cell type for each cluster (via findcelltype)
- Per-cluster cell type annotation table (e.g. cluster_celltype_annotations.csv)

## How to apply

First identify cluster marker genes (scCATCH's findmarkergene with species, marker, tissue, cancer arguments) using the paired one-against-all criteria: log10 fold change >= 0.25, Wilcoxon p < 0.05, expressed in >= 25% of cells in the cluster. Then match each cluster's candidate marker genes against the packaged cellmatch reference data.frame, which covers 353 cell types, 686 subtypes, 184 tissue types, and 2,096 human and mouse literature references, restricting to the relevant tissue or cancer to reduce false matches. Matching yields, per cluster, candidate cell types supported by validated marker genes plus their supporting literature; scCATCH's findcelltype() then scores candidates by evidence (matched validated markers and references) and assigns the top-1 scoring type. The rationale is that one-against-all marker identification can produce pseudo marker genes (upregulated in multiple clusters), so tissue-specific reference matching filters these before annotation. Users may supply a custom cellmatch data.frame or add markers for other species if the built-in database lacks the tissue of interest.

## Related tools

- **scCATCH** (Provides findmarkergene() and findcelltype() to identify cluster markers and match/score them against the CellMatch reference for automatic cluster annotation) — https://github.com/ZJUFanLab/scCATCH
- **cellmatch** (The tissue-specific cell taxonomy reference data.frame (353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references) that marker genes are matched against; custom versions are supported) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmarker', tissue = 'Kidney', cancer = 'all'); obj <- findcelltype(obj)
```

## Evaluation signals

- Every input cluster receives exactly one annotated cell type, and each assignment is a valid CellMatch cell type
- The top-1 rule is reproducible: re-ranking the stored per-type evidence scores must reproduce the assigned annotation
- On literature-validated benchmarks (Chen kidney, Xin pancreatic islet, Gierahn peripheral blood), 100% of clusters receive the literature-reported cell type (consistent rate of 100%, Table 1 / Figure 2)
- Marker genes entering matching satisfy the stated filters: log10 fold change >= 0.25, Wilcoxon p < 0.05, expressed in >= 25% of the cluster's cells
- Substituting CellMatch into SingleR lifts annotation consistency from ~0%-10% (default references) to 80%-90%, confirming reference quality drives accuracy

## Limitations

- Performance of scCATCH majorly depends on the reference database; a limited reference database can lead to incorrect cell type annotation
- Pseudo marker genes from one-against-all comparisons (significantly upregulated in at least two clusters) can cause incorrect annotation if not filtered
- Inadequate clustering (too many or too few cells per cluster) introduces errors into downstream annotation
- Built-in references cover human and mouse only; other species require supplying custom markers

## Evidence
