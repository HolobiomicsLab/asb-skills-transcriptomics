---
name: gene-set-consistency-filtering-across-clusters
description: Use when several related (e.g., subtype-level) cell clusters each yield
  a ranked COVID-vs-healthy biomarker list and you need the gene subset whose expression
  direction is consistent across all of them.
when_to_use_negative:
- Only a single cell cluster is under analysis — the cross-cluster intersection that
  defines consistency is undefined
- The goal is pathway or GO enrichment of DE genes rather than identifying genes consistently
  differentially expressed across related clusters
- Cell clusters were not detected as related novel subtypes (e.g., they are unrelated
  reference cell types), so requiring identical expression direction may be biologically
  inappropriate
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCANPY
  role: Performs the per-cluster differential expression analysis (rank_genes_groups)
    whose biomarker lists are intersected across the three Macrophage clusters
- name: scNovel
  role: Upstream tool that detects the three candidate novel Macrophage subtype clusters
    whose DE gene sets are filtered for consistency
  repo: github.com/chuanyang-Zheng/scNovel
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:e20820be2aedcdf20cb4887a2f91dde0306e4fd9b0e5d3cdc075fd3f4b4c5162
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gene-set-consistency-filtering-across-clusters

## Summary

A downstream differential-expression interpretation step that intersects per-cluster COVID-vs-healthy biomarker lists from scNovel-discovered novel Macrophage clusters to retain only genes with consistent differential expression across all dependent clusters, reproducing the Figure 6H gene sets.

## When to use

Use when several related (e.g., subtype-level) cell clusters each yield a ranked COVID-vs-healthy biomarker list and you need the gene subset whose expression direction is consistent across all of them.

## When NOT to use

- Only a single cell cluster is under analysis — the cross-cluster intersection that defines consistency is undefined
- The goal is pathway or GO enrichment of DE genes rather than identifying genes consistently differentially expressed across related clusters
- Cell clusters were not detected as related novel subtypes (e.g., they are unrelated reference cell types), so requiring identical expression direction may be biologically inappropriate

## Inputs

- AnnData object of the GSE171524 bronchoalveolar immune cell atlas (116,314 cells) with patient and COVID-status annotations
- Cluster labels for the three scNovel-discovered Macrophage subtype clusters (clusters 3, 5, and 12)
- Per-cluster ranked differential expression tables from Scanpy rank_genes_groups (top 8 biomarkers per cluster)

## Outputs

- Table of 7 up-regulated genes (ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1) with direction labels
- Table of 9 down-regulated genes (ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC) with direction labels
- Figure 6H-style consistent gene set across the three Macrophage clusters

## How to apply

Load the scNovel-detected novel clusters from the GSE171524 bronchoalveolar immune cell atlas (116,314 cells from 7 healthy controls and 20 COVID-19 samples) in Scanpy, keeping patient/COVID-status annotations, and select the three candidate Macrophage subtype clusters (3, 5, and 12) identified by high CD163 expression. Run differential expression with Scanpy's rank_genes_groups between COVID and healthy samples within each cluster, taking the top 8 biomarkers per cluster as in Figure 6G. Filter to the intersection: genes appearing as biomarkers in all three clusters (7 up-regulated in COVID: ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1; 9 down-regulated: ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC). Rationale: requiring consistent expression patterns across the three novel Macrophage subtypes distinguishes robust disease-associated genes from cluster-specific noise.

## Related tools

- **SCANPY** (Performs the per-cluster differential expression analysis (rank_genes_groups) whose biomarker lists are intersected across the three Macrophage clusters)
- **scNovel** (Upstream tool that detects the three candidate novel Macrophage subtype clusters whose DE gene sets are filtered for consistency) — github.com/chuanyang-Zheng/scNovel

## Examples

```
import scanpy as sc; sc.tl.rank_genes_groups(mac_adata, groupby='covid_status', groups=['COVID'], reference='healthy')
```

## Evaluation signals

- Output table contains exactly 7 genes labeled up-regulated and 9 genes labeled down-regulated
- Gene symbols match the reported Figure 6H sets: up-regulated = {ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1}; down-regulated includes {ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC}
- Every retained gene appears as a biomarker (top 8) in all three clusters (3, 5, and 12), not just one or two
- Each output row carries an explicit expression-direction label (up-regulated / down-regulated)

## Limitations

- The consistency criterion depends on the 'top 8 biomarkers per cluster' cutoff from Figure 6G; a different cutoff changes the intersected gene set
- scNovel's own score function could be improved, per the authors, so the upstream cluster definitions feeding this filter may shift
- Validated only on the GSE171524 COVID-19 bronchoalveolar dataset (7 healthy, 20 COVID samples); consistency in other tissues or diseases is untested

## Evidence

- [abstract] COVID-related differential genes show consistent patterns: "three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis"
- [results] Task finding on seven up-regulated genes: [evidence span withheld — anchor status: not_found]
- [discussion] Pipeline combining novel cell identification and DE analysis: "We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression."
- [readme] Scanpy is the analysis dependency: "- Scanpy (Compatible with all versions)"
