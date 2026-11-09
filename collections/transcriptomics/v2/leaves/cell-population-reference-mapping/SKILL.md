---
name: cell-population-reference-mapping
description: Use when a single-cell AnnData carries multiple conflicting annotation
  columns (e.g., cellHarmony label transfer to Hay et al. CD34+ reference populations,
  Leiden at several resolutions, or RNA vs. clonal-genotype GoT subclusters) and a
  stable per-cell population assignment is needed.
when_to_use_negative:
- When data-level integration is required (joint latent space or geometric graph,
  e.g., Seurat-WNN or TotalVI) rather than decision-level reconciliation of already-computed
  annotation labels.
- When only a single annotation set exists — scTriangulate reconciles multiple conflicting
  cluster-label assignments, so it needs at least two query columns.
- When over 15 annotation sets are supplied and strict Shapley-based selection is
  required — the program auto-substitutes a combined rank-based selection to reduce
  runtime.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0203
tools:
- name: scTriangulate
  role: 'Core framework: mix-and-matches conflicting cluster annotations via Shapley
    value and stability metrics (reassign, TFIDF5/10, SCCAF scores) with lazy_run
    and preprocessing helpers (concat_rna_and_other, scanpy_recipe, add_annotation,
    add_umap, plot_two_column_sankey)'
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Underlying framework in which scTriangulate is seamlessly integrated; scanpy_recipe
    handles QC, filtering, scaling, and UMAP generation
- name: cellHarmony
  role: Supervised label transfer of cells to reference populations (Hay et al. CD34+
    cell populations) providing one of the conflicting annotation sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:9b6c9c462bff493d8ab854be13b203a9ee3c1d12f3e1c2bdca4939ce4dc9f03c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - cellHarmony
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# cell-population-reference-mapping

## Summary

Decision-level integration of conflicting single-cell annotations (reference label transfer, clustering algorithms, modalities, resolutions) into one stable per-cell population label using scTriangulate's Shapley-value-based mix-and-match of cluster sets plus stability metrics, rather than a joint embedding.

## When to use

Use when a single-cell AnnData carries multiple conflicting annotation columns (e.g., cellHarmony label transfer to Hay et al. CD34+ reference populations, Leiden at several resolutions, or RNA vs. clonal-genotype GoT subclusters) and a stable per-cell population assignment is needed.

## When NOT to use

- When data-level integration is required (joint latent space or geometric graph, e.g., Seurat-WNN or TotalVI) rather than decision-level reconciliation of already-computed annotation labels.
- When only a single annotation set exists — scTriangulate reconciles multiple conflicting cluster-label assignments, so it needs at least two query columns.
- When over 15 annotation sets are supplied and strict Shapley-based selection is required — the program auto-substitutes a combined rank-based selection to reduce runtime.

## Inputs

- AnnData with normalized adata.X (log CPT RNA; CLR-normalized ADT/other modality), at least two conflicting annotation columns in adata.obs, and adata.obsm['X_umap'] (no adata.raw attribute)
- Pre-processed count matrices for gene expression and genotyping of transcriptomes (e.g., GEO GSE117825 MPN GoT dataset)
- Reference cell-population labels (e.g., Hay et al. CD34+ populations via cellHarmony) and/or Leiden clusterings at multiple resolutions

## Outputs

- Cell-level final annotation table with one row per cell giving the reconciled population/clone label (e.g., WT, SF3B1 mutant, SF3B1-CALR, SF3B1-CALR-NFE2)
- Results saved to the user-specified output directory (reconciled clustering solution, unstable-cluster flags, nearest-neighbor reassignments)
- Two-column Sankey diagram (plot_two_column_sankey) mapping original to final labels

## How to apply

Prepare an AnnData whose adata.X is properly normalized (log CPT for RNA, CLR for other modalities via scTriangulate.preprocessing.Normalization.CLR_normalization) with adata.obsm['X_umap'] set and no adata.raw. Concatenate the normalized RNA matrix and the conflicting annotation sets (e.g., cellHarmony clusters and clonal-genotype subclusters) with scTriangulate.preprocessing.concat_rna_and_other, adding annotations and UMAP coordinates via add_annotation/add_umap. Run ScTriangulate.lazy_run (version 0.9.0) passing the annotation columns as the query; the Shapley value from coalitional game theory picks the stable annotation per cell, with default pruning (win_fraction_cutoff < 0.25 flagged unstable, abs_thresh = 10 minimum cluster size) and reclassification of unstable cells by nearest-neighbor strategy. If more than 15 annotation sets are supplied, the program automatically substitutes Shapley selection with combined rank-based selection to reduce runtime.

## Related tools

- **scTriangulate** (Core framework: mix-and-matches conflicting cluster annotations via Shapley value and stability metrics (reassign, TFIDF5/10, SCCAF scores) with lazy_run and preprocessing helpers (concat_rna_and_other, scanpy_recipe, add_annotation, add_umap, plot_two_column_sankey)) — https://github.com/frankligy/scTriangulate
- **scanpy** (Underlying framework in which scTriangulate is seamlessly integrated; scanpy_recipe handles QC, filtering, scaling, and UMAP generation)
- **cellHarmony** (Supervised label transfer of cells to reference populations (Hay et al. CD34+ cell populations) providing one of the conflicting annotation sets)

## Examples

```
from sctriangulate import *; sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- Output directory contains a cell-level table with one row per cell and columns for the final population/clone assignment; every cell appears exactly once.
- Clusters with winning fraction wfc < 0.25 are flagged unstable and their cells re-classified by nearest-neighbor strategy to a stable cluster.
- Minimum final cluster size respects abs_thresh = 10 (default), which produced consistent reliable results across evaluated datasets (Supplementary Figure 7a).
- For the MPN GoT application (Fig. 6c,d), SF3B1 single-mutant cells are reassigned to transcriptionally more similar clonal populations than their original genotype-based assignment.
- Sankey diagram renders a coherent two-column mapping from original to final labels with no dangling cell categories.

## Limitations

- Programmatic default analytical settings (e.g., wfc = 0.25, abs_thresh = 10) may not be optimal for all study designs.
- Integration of cluster predictions from dozens of sources significantly increases runtime, forcing the automatic rank-based substitute for Shapley selection above 15 annotation sets.
- Naming of resulting populations remains largely non-standardized, an unmet challenge for downstream interpretation.
- Current cancer analysis is limited to select somatic mutations within a single tumor; support for integrating multiple samples of the same modality and across technologies is planned but not yet available.

## Evidence

- [intro] scTriangulate uses the Shapley Value from coalitional game theory combined with cluster stability metrics to mix-and-match cellular clusters: [evidence span withheld — anchor status: not_found]
- [methods] We apply the scTriangulate lazy_run function in version 0.9.0 on these four conflicting annotation solutions to derive the final cluster assignment (default settings).: "We apply the scTriangulate lazy_run function in version 0.9.0 on these four conflicting annotation solutions to derive the final cluster assignment (default settings)."
- [methods] the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined) and re-classified using a nearest-neighbor strategy.: "the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined) and re-classified using a nearest-neighbor strategy."
- [methods] we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function"
- [readme] sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']): "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])"
- [methods] Pre-processed count matrices for gene expression and genotyping of transcriptomes were downloaded from GEO (GSE117825).: "Pre-processed count matrices for gene expression and genotyping of transcriptomes were downloaded from GEO (GSE117825)."
