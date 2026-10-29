---
name: evidence-based-cell-type-scoring
description: Use when clusters from scRNA-seq data need automatic cell type labels
  without prior knowledge, and candidate cluster marker genes have been identified
  and matched against the CellMatch reference database.
when_to_use_negative:
- Marker genes have not yet been identified and matched against CellMatch — run findmarkergene
  and CellMatch matching first
- The tissue or cell types of interest are not represented in the CellMatch database,
  since performance is database-dependent and a limited reference database can lead
  to incorrect annotation
- Annotation must be at subtype or custom-taxonomy level not covered by CellMatch's
  353 cell types / 686 subtypes, unless a custom cellmatch panel is supplied
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: scCATCH
  role: Provides the findcelltype() function that scores cell types per cluster on
    validated marker genes with literature support and assigns the top-1 scored cell
    type, plus the cellmatch reference database
  repo: https://github.com/ZJUFanLab/scCATCH
- name: CellMatch
  role: Tissue-specific cellular taxonomy reference database against which cluster
    marker genes are matched to enumerate candidate cell types and their validated
    markers and supporting references
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
- name: Seurat
  role: Upstream clustering pipeline; cluster identities from a Seurat object can
    be passed to createscCATCH before scoring
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:6786250a4ab6ffd68bf2e8d2e22b89cacdbc0faa93a699260aa59d3a52f45f45
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
  - CellMatch
  - Seurat
  - scCATCH
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# evidence-based-cell-type-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score candidate cell types for each pre-computed scRNA-seq cluster by counting validated marker genes matched in the CellMatch database plus their supporting literature, then assign the top-1 scoring cell type as the cluster annotation (scCATCH findcelltype step).

## When to use

Use when clusters from scRNA-seq data need automatic cell type labels without prior knowledge, and candidate cluster marker genes have been identified and matched against the CellMatch reference database.

## When NOT to use

- Marker genes have not yet been identified and matched against CellMatch — run findmarkergene and CellMatch matching first
- The tissue or cell types of interest are not represented in the CellMatch database, since performance is database-dependent and a limited reference database can lead to incorrect annotation
- Annotation must be at subtype or custom-taxonomy level not covered by CellMatch's 353 cell types / 686 subtypes, unless a custom cellmatch panel is supplied

## Inputs

- scCATCH object (from createscCATCH) after findmarkergene, containing per-cluster marker genes
- cluster marker gene table (e.g., demo_marker from the scCATCH package)
- cellmatch reference data.frame packaged with scCATCH (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references)

## Outputs

- Per-cluster top-1 cell type annotations (e.g., sccatch_annotations.csv or cluster_celltype_annotations.csv with one annotated cell type per input cluster)
- Per-candidate-cell-type evidence scores per cluster supporting the top-1 assignment

## How to apply

Load the cluster marker gene results (e.g., the demo_marker table produced after findmarkergene) and the packaged cellmatch reference data.frame from scCATCH. Match each cluster's candidate marker genes against the CellMatch panel of 353 cell types and 686 subtypes across 184 tissue types, backed by 2,096 human and mouse references. For each cluster, compute an evidence score for every candidate cell type based on the matched validated marker genes and their supporting literature, mirroring the findcelltype(object) function. Rank candidate cell types by score and assign the highest-scoring (top 1) cell type to each cluster. Write the per-cluster top-1 assignments to an output table such as sccatch_annotations.csv for downstream validation.

## Related tools

- **scCATCH** (Provides the findcelltype() function that scores cell types per cluster on validated marker genes with literature support and assigns the top-1 scored cell type, plus the cellmatch reference database) — https://github.com/ZJUFanLab/scCATCH
- **CellMatch** (Tissue-specific cellular taxonomy reference database against which cluster marker genes are matched to enumerate candidate cell types and their validated markers and supporting references) — https://github.com/ZJUFanLab/scCATCH/tree/master/data
- **Seurat** (Upstream clustering pipeline; cluster identities from a Seurat object can be passed to createscCATCH before scoring)

## Examples

```
obj <- findcelltype(obj); write.csv(obj@celltype, 'cluster_celltype_annotations.csv')
```

## Evaluation signals

- Output file contains exactly one annotated cell type per input cluster
- Each assigned label is drawn from a valid CellMatch cell type
- The top-1 rule (highest evidence score) is reproducible from the stored per-type scores
- Qualitative concordance with the paper's reported 67%–100% (average 83%) annotation accuracy on six published scRNA-seq datasets

## Limitations

- Performance of scCATCH majorly depends on the reference database; a limited CellMatch database may lead to incorrect cell type annotation
- Inadequate clustering analysis (too many or too few clusters) can introduce errors into cell type annotation
- Pseudo marker genes upregulated in at least two clusters can propagate incorrect annotations if upstream marker identification is not stringent

## Evidence

- [methods] findcelltype function: [evidence span withheld — anchor status: not_found]
