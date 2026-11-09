---
name: scrna-seq-cluster-annotation-criteria
description: Use when annotating cell types for scRNA-seq clusters and marker genes
  must be identified by one-against-one paired cluster comparison with explicit fold-change,
  significance, and percent-expression thresholds.
when_to_use_negative:
- Input is not cluster-level scRNA-seq data (e.g. bulk RNA-seq, a feature table without
  per-cell cluster assignments, or pre-computed marker lists)
- A one-against-all differential expression strategy is explicitly desired, since
  scCATCH's rationale is to avoid pseudo marker genes from one-against-all comparisons
- The species or tissue is not covered by CellMatch (human or mouse references); the
  article notes that a limited reference database can lead to incorrect annotation
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: 'Primary tool: createscCATCH builds the object, findmarkergene performs the
    paired-cluster marker gene identification with the filter thresholds, findcelltype
    performs evidence-based annotation'
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Comparison baseline using one-against-all marker identification and source
    of scCATCH objects via createscCATCH(data = Seurat_obj[['RNA']]@data, cluster
    = as.character(Idents(Seurat_obj)))
- name: CellMatch
  role: Tissue-specific cellular taxonomy reference database used downstream to match
    validated marker genes to cell types
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:485868ddfd12d2d22fb06bb8a29a9a1c26fc439b3b4908ac2bbb0a1f1b41be5b
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scrna-seq-cluster-annotation-criteria

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for identifying potential marker genes per cluster in scRNA-seq data using scCATCH's one-against-one paired-cluster comparison, retaining genes significantly upregulated in only one cluster (log10 fold change >= 0.25, Wilcoxon p < 0.05, expressed in >= 25% of cells) to avoid pseudo marker genes and support accurate automatic cell type annotation.

## When to use

Use when annotating cell types for scRNA-seq clusters and marker genes must be identified by one-against-one paired cluster comparison with explicit fold-change, significance, and percent-expression thresholds.

## When NOT to use

- Input is not cluster-level scRNA-seq data (e.g. bulk RNA-seq, a feature table without per-cell cluster assignments, or pre-computed marker lists)
- A one-against-all differential expression strategy is explicitly desired, since scCATCH's rationale is to avoid pseudo marker genes from one-against-all comparisons
- The species or tissue is not covered by CellMatch (human or mouse references); the article notes that a limited reference database can lead to incorrect annotation

## Inputs

- scRNA-seq expression data matrix (e.g. demo_data or a Seurat object's RNA @data slot)
- Character vector of cluster labels for each cell (e.g. demo_cluster or as.character(Idents(Seurat_obj)))
- Species and tissue specification (e.g. 'Human', with tissue used to restrict CellMatch annotation)
- Optional custom marker database (if_use_custom_marker) instead of the built-in paired comparison

## Outputs

- scCATCH object with available marker genes per cluster
- Per-cluster marker gene table (e.g. marker_genes.csv) with genes passing log10 FC >= 0.25, Wilcoxon p < 0.05, and >= 25% cell expression
- Downstream cell type annotations via findcelltype using evidence-based scoring against CellMatch

## How to apply

Build a scCATCH object from the scRNA-seq matrix and cluster labels with createscCATCH(data = sc_data, cluster = sc_cluster), then run findmarkergene with species and tissue set (and if_use_custom_marker = FALSE to use the built-in paired comparison). Unlike Seurat's one-against-all approach, scCATCH compares each cluster against every other cluster and retains only genes significantly upregulated in exactly one cluster, which avoids pseudo marker genes upregulated in multiple clusters. Apply the Figure 1A filter criteria: logfc = 0.25 (log10 fold change), pvalue = 0.05 (Wilcoxon rank-sum test), and cell_min_pct = 0.25 (expressed in >= 25% of that cluster's cells). Extract the per-cluster marker gene table from the returned object and validate that every retained gene satisfies all three thresholds before feeding it into findcelltype, which scores cell types against the tissue-specific CellMatch database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references).

## Related tools

- **scCATCH** (Primary tool: createscCATCH builds the object, findmarkergene performs the paired-cluster marker gene identification with the filter thresholds, findcelltype performs evidence-based annotation) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Comparison baseline using one-against-all marker identification and source of scCATCH objects via createscCATCH(data = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj))))
- **CellMatch** (Tissue-specific cellular taxonomy reference database used downstream to match validated marker genes to cell types) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = demo_data, cluster = demo_cluster); obj <- findmarkergene(obj, species = 'Human', tissue = 'Bone marrow'); obj <- findcelltype(obj)
```

## Evaluation signals

- marker_genes.csv exists and is non-empty
- Every retained gene row satisfies log10 fold change >= 0.25, Wilcoxon p < 0.05, and expression in >= 25% of cells in its assigned cluster
- Each retained marker gene is significantly upregulated in exactly one cluster (not at least two clusters), consistent with the one-against-one paired comparison
- The scCATCH object returned by findmarkergene contains a non-empty available marker gene table for each input cluster
- Downstream findcelltype annotations score cell types using only validated marker genes and their supporting literature from CellMatch

## Limitations

- scCATCH's performance majorly depends on the reference database; a limited CellMatch reference can lead to incorrect cell type annotation
- Inadequate clustering analysis (too many or too few clusters) would introduce errors into the annotation process
- The method was evaluated on human and mouse references; markers from other species require the custom-marker workflow
- Effects of multiple clustering algorithms on cell type annotations had not yet been evaluated

## Evidence

- [intro] Pseudo marker genes rationale: [evidence span withheld — anchor status: not_found]
