---
name: hierarchical-clustering-of-transcription-factors
description: Use when a time-course scRNA-seq trajectory needs its developmental regulatory
  programs reduced to TF clusters — e.g., recovering Fig. 2f-style tTF groupings after
  eLR/communication inference.
when_to_use_negative:
- When the goal is inferring ligand-receptor-to-TF causal regulation rather than TF
  clustering — that requires downstream RunTimeTalk with Granger causality, not this
  skill alone
- 'When candidate regulators should not be restricted: this workflow deliberately
  excludes essential genes (DEG 10) and housekeeping genes (HRT Atlas), which would
  bias general TF-atlas analyses'
- When cell-type definitions do not yet exist in the Seurat object — GetCellTypeSpecificTRN
  requires per-cell-type identities (tmp.ident) and marker genes as inputs
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TimeTalk
  role: Provides GetCellTypeSpecificTRN to build cell-type-specific TRNs whose MRA
    output yields the tTFs
  repo: https://github.com/ChengLiLab/TimeTalk
- name: RTN
  role: Bioconductor package underlying the 'tni'/'tna' objects; tna.mra performs
    master regulator analysis and tna.get extracts MRA results
- name: Seurat
  role: FindAllMarkers supplies cell-type marker genes (p_val_adj < 0.05) used to
    filter MRA regulators
- name: monocle2
  role: differentialGeneTest (qval = 0.01) defines trajectory-variable genes and their
    variance used as the MRA phenotype
- name: preprocessCore
  role: normalize.quantiles performs quantile normalization of the merged expression
    matrix prior to TRN building
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:81c030489c192afe1935d7527e8a93b1d669006be3e791866f6c745df13850b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - RTN
  - Seurat
  - TimeTalk
  - monocle2
  - preprocessCore
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hierarchical-clustering-of-transcription-factors

## Summary

Identifies temporal transcription factors (tTFs) via RTN master regulator analysis on cell-type-specific TRNs built from log10(RPKM+1) normalized pre-implantation scRNA-seq, then partitions the resulting 229 tTFs into six clusters (C1–C6) by ward.D2 hierarchical clustering of their average expression profiles.

## When to use

Use when a time-course scRNA-seq trajectory needs its developmental regulatory programs reduced to TF clusters — e.g., recovering Fig. 2f-style tTF groupings after eLR/communication inference.

## When NOT to use

- When the goal is inferring ligand-receptor-to-TF causal regulation rather than TF clustering — that requires downstream RunTimeTalk with Granger causality, not this skill alone
- When candidate regulators should not be restricted: this workflow deliberately excludes essential genes (DEG 10) and housekeeping genes (HRT Atlas), which would bias general TF-atlas analyses
- When cell-type definitions do not yet exist in the Seurat object — GetCellTypeSpecificTRN requires per-cell-type identities (tmp.ident) and marker genes as inputs

## Inputs

- Merged pre-implantation scRNA-seq expression matrix (normalized to log10(RPKM + 1))
- AnimalTFDB 3.0 transcription factor list
- DEG 10 essential gene set and HRT Atlas v1.0 housekeeping gene set (excluded as candidate regulators)
- Trajectory-variable genes from monocle2 differentialGeneTest (qval = 0.01) with their variance, as the MRA phenotype
- Seurat FindAllMarkers cell-type marker genes (p_val_adj < 0.05)

## Outputs

- Per-cell-type RTN rtna objects saved as res/R/*RTN_rtna_object.rds
- Merged master regulator analysis table res/R/B_blastoid_RTN_mra_result.rds
- 229 identified tTFs with average expression profiles
- Six-cluster (C1–C6) ward.D2 dendrogram/heatmap (Fig. 2f) and tTF cluster assignment table (Supplementary Data 1)

## How to apply

Normalize the merged pre-implantation expression matrix to log10(RPKM + 1), compile the AnimalTFDB 3.0 TF list, and build an RTN 'tni' object; infer per-cell-type TRNs with TimeTalk's GetCellTypeSpecificTRN, excluding essential genes (DEG 10) and housekeeping genes (HRT Atlas) as candidate regulators. Run master regulator analysis with tna.mra using trajectory-variable genes (monocle2 differentialGeneTest, qval = 0.01) and their variance as the phenotype. Extract results via tna.get(rtna, what='mra', ntop = -1), retain regulators with Pvalue < 0.05 whose regulon targets are Seurat FindAllMarkers cell-type markers (p_val_adj < 0.05), and merge per-cell-type objects (res/R/*RTN_rtna_object.rds into res/R/B_blastoid_RTN_mra_result.rds). Collect the 229 tTFs, compute average expression profiles, and hierarchically cluster with hclust using the ward.D2 agglomeration method; cut into six clusters labeled C1–C6 (the dendrogram/heatmap is Fig. 2f, memberships go to Supplementary Data 1).

## Related tools

- **TimeTalk** (Provides GetCellTypeSpecificTRN to build cell-type-specific TRNs whose MRA output yields the tTFs) — https://github.com/ChengLiLab/TimeTalk
- **RTN** (Bioconductor package underlying the 'tni'/'tna' objects; tna.mra performs master regulator analysis and tna.get extracts MRA results)
- **Seurat** (FindAllMarkers supplies cell-type marker genes (p_val_adj < 0.05) used to filter MRA regulators)
- **monocle2** (differentialGeneTest (qval = 0.01) defines trajectory-variable genes and their variance used as the MRA phenotype)
- **preprocessCore** (normalize.quantiles performs quantile normalization of the merged expression matrix prior to TRN building)

## Examples

```
mra <- tna.get(rtna, what="mra", ntop = -1); mra.res <- mra %>% filter(Pvalue < 0.05) %>% filter(Regulon %in% tmp.gene.use)
```

## Evaluation signals

- Exactly 229 tTFs are recovered after the Pvalue < 0.05 and regulon-in-cell-type-marker filters
- Ward.D2 dendrogram of the 229 tTFs cuts cleanly into six clusters whose memberships match Fig. 2f and Supplementary Data 1
- Cluster-level GO enrichments are developmentally coherent, e.g., C5 contains Gata6, Tead4, Cdx2, Nanog, Pou5f1 enriched in blastocyst development terms
- Per-cell-type objects res/R/*RTN_rtna_object.rds and the merged res/R/B_blastoid_RTN_mra_result.rds exist and contain regulators passing the Pvalue < 0.05 filter

## Limitations

- The inferred ligand-receptor-to-TF regulatory relationships were not confirmed by wet-lab experiments; the authors note they 'should be confirmed by further wet experiments'
- Causal conclusions are sensitive to the interpolation parameter winsz (Supplementary Figs. 20 and 21), so users are advised to test multiple parameter combinations
- Regulon inference is time-consuming; the README explicitly warns that building cell-type-specific TRNs should not be run casually on demo data

## Evidence

- [discussion] we clustered the identified tTFs by hierarchical clustering (the agglomeration method was "ward.D2").: [evidence span withheld — anchor status: not_found]
- [discussion] the variable gene and their variance were input as a phenotype to perform master regulator analysis by function "tna.mra".: [evidence span withheld — anchor status: not_found]
- [readme] mra.res <- mra %>%
    filter(Pvalue < 0.05) %>%
    filter(Regulon %in% tmp.gene.use): "mra.res <- mra %>%
    filter(Pvalue < 0.05) %>%
    filter(Regulon %in% tmp.gene.use)"
- [discussion] the log10(RPKM + 1) normalized scRNA-seq data and TF list from AnimalTFDB (version 3.0)71 to build the "tni" object: [evidence span withheld — anchor status: not_found]
- [discussion] It should be admitted that the regulation relationship between ligand-receptor interaction and transcription factors inferred by the TimeTalk should be confirmed by further wet experiments.: "the regulation relationship between ligand-receptor interaction and transcription factors inferred by the TimeTalk should be confirmed by further wet experiments."
