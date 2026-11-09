---
name: master-regulator-analysis-with-rtn
description: Use when you need cell-type-specific master regulators (tTFs) from time-course
  scRNA-seq — e.g., to pair TF activity with inferred ligand-receptor signaling along
  an early-embryo developmental trajectory.
when_to_use_negative:
- 'Your goal is cell-cell communication between definitive cell types rather than
  temporal TF identification — TimeTalk''s downstream RunTimeTalk / CellPhoneDB /
  CellChat-style analysis is the wrong fit here and vice versa: MRA alone does not
  infer ligand-receptor interactions'
- 'You lack cell-type/cluster identities or a reconstructed trajectory: the workflow
  depends on per-cell-type TRNs and trajectory-variable genes (monocle2 differentialGeneTest
  qval = 0.01) as the MRA phenotype'
- Your expression matrix is raw counts without per-stage/pseudo-bulk processing compatible
  with the RTN 'tni' object built from log10(RPKM + 1)-normalized data
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RTN
  role: R/Bioconductor package providing the 'tni' object construction and the tna.mra
    / tna.get master regulator analysis functions
- name: TimeTalk
  role: R package whose myGetCellTypeSpecificTRN (GetCellTypeSpecificTRN) wraps RTN
    to build cell-type-specific TRNs and orchestrate the tTF pipeline
  repo: https://github.com/ChengLiLab/TimeTalk
- name: Seurat
  role: FindAllMarkers (only.pos = TRUE, p_val_adj < 0.05) supplies the cell-type
    marker genes used to filter retained regulators
- name: monocle2
  role: differentialGeneTest (qval = 0.01) along the DDRTree trajectory supplies the
    variable genes and variance used as the MRA phenotype
- name: preprocessCore
  role: normalize.quantiles performs quantile normalization of the merged expression
    matrix prior to TRN construction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:217d4aa69207c027d6483e7fe6f51d09ad3d6b62df2aa1b90b64fe3c147ab1b8
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# Master regulator analysis with RTN to identify temporal transcription factors (tTFs)

## Summary

Identifies temporal transcription factors (tTFs) from integrated pre-implantation scRNA-seq by running RTN master regulator analysis (tna.mra) on cell-type-specific transcriptional regulatory networks (TRNs), then hierarchically clustering regulators (ward.D2, six clusters C1–C6). In TimeTalk this is the by-product step that links eLR signaling to TF activity.

## When to use

Use when you need cell-type-specific master regulators (tTFs) from time-course scRNA-seq — e.g., to pair TF activity with inferred ligand-receptor signaling along an early-embryo developmental trajectory.

## When NOT to use

- Your goal is cell-cell communication between definitive cell types rather than temporal TF identification — TimeTalk's downstream RunTimeTalk / CellPhoneDB / CellChat-style analysis is the wrong fit here and vice versa: MRA alone does not infer ligand-receptor interactions
- You lack cell-type/cluster identities or a reconstructed trajectory: the workflow depends on per-cell-type TRNs and trajectory-variable genes (monocle2 differentialGeneTest qval = 0.01) as the MRA phenotype
- Your expression matrix is raw counts without per-stage/pseudo-bulk processing compatible with the RTN 'tni' object built from log10(RPKM + 1)-normalized data

## Inputs

- Merged pre-implantation scRNA-seq expression matrix (normalized to log10(RPKM + 1))
- AnimalTFDB 3.0 transcription factor list (gene.TF)
- Cell-type/cell-state labels from Seurat object (tmp.ident)
- Trajectory-variable genes and their variance from monocle2 differentialGeneTest (qval = 0.01) as phenotype
- Seurat FindAllMarkers cell-type marker genes (p_val_adj < 0.05)
- Essential gene set (DEG 10) and housekeeping gene set (HRT Atlas v1.0) for exclusion

## Outputs

- Per-cell-type RTN rtna objects (res/R/<celltype>RTN_rtna_object.rds)
- Merged master regulator analysis table (res/R/B_blastoid_RTN_mra_result.rds) with regulators filtered at Pvalue < 0.05
- Set of 229 temporal TFs (tTFs) with average expression profiles
- Six ward.D2 tTF clusters C1–C6 (Fig. 2f dendrogram/heatmap; Supplementary Data 1 cluster assignments)

## How to apply

Normalize the merged scRNA-seq matrix to log10(RPKM + 1) and compile the AnimalTFDB 3.0 TF list to build the RTN 'tni' object, excluding essential genes (DEG 10) and housekeeping genes (HRT Atlas v1.0) as candidate regulators. Build cell-type-specific TRNs per Seurat ident with TimeTalk's myGetCellTypeSpecificTRN and save each as res/R/<celltype>RTN_rtna_object.rds. Run master regulator analysis with tna.mra, supplying trajectory-variable genes (monocle2 differentialGeneTest, qval = 0.01) and their variance as the phenotype. Extract results with tna.get(rtna, what='mra', ntop = -1), then retain regulators with Pvalue < 0.05 whose Regulon targets are cell-type marker genes (Seurat FindAllMarkers, only.pos = TRUE, p_val_adj < 0.05), and merge per-cell-type results into res/R/B_blastoid_RTN_mra_result.rds. Collect the surviving tTFs (229 in the paper), compute average expression profiles, and hierarchically cluster with agglomeration method 'ward.D2', cutting the dendrogram into six clusters C1–C6 for Fig. 2f and Supplementary Data 1.

## Related tools

- **RTN** (R/Bioconductor package providing the 'tni' object construction and the tna.mra / tna.get master regulator analysis functions)
- **TimeTalk** (R package whose myGetCellTypeSpecificTRN (GetCellTypeSpecificTRN) wraps RTN to build cell-type-specific TRNs and orchestrate the tTF pipeline) — https://github.com/ChengLiLab/TimeTalk
- **Seurat** (FindAllMarkers (only.pos = TRUE, p_val_adj < 0.05) supplies the cell-type marker genes used to filter retained regulators)
- **monocle2** (differentialGeneTest (qval = 0.01) along the DDRTree trajectory supplies the variable genes and variance used as the MRA phenotype)
- **preprocessCore** (normalize.quantiles performs quantile normalization of the merged expression matrix prior to TRN construction)

## Examples

```
mra <- tna.get(rtna, what="mra", ntop = -1); mra.res <- mra %>% filter(Pvalue < 0.05) %>% filter(Regulon %in% tmp.gene.use)
```

## Evaluation signals

- Exactly 229 tTFs are recovered after the Pvalue < 0.05 and cell-type-marker (p_val_adj < 0.05) filters
- The 229 tTFs partition into six distinct ward.D2 clusters labeled C1–C6 whose membership matches Fig. 2f and Supplementary Data 1
- res/R/B_blastoid_RTN_mra_result.rds exists and contains only regulators passing Pvalue < 0.05 with Regulon targets in the Seurat marker set
- Per-cell-type objects res/R/<celltype>RTN_rtna_object.rds exist for every level(seu) ident
- Cluster GO enrichments are biologically coherent (e.g., C5 contains Gata6, Tead4, Cdx2, Nanog, Pou5f1 enriched in blastocyst development terms)

## Limitations

- Regulation relationships between ligand-receptor interactions and TFs inferred via this pipeline require further wet-lab experimental confirmation, which was not performed
- Downstream Granger causal conclusions are sensitive to the interpolation parameter winsz; users are advised to test multiple parameter combinations
- Master regulator analysis via RTN is computationally very time consuming — the README explicitly warns against running the TRN-building demo code on the demo data

## Evidence

- [discussion] the variable gene and their variance were input as a phenotype to perform master regulator analysis by function “tna.mra”.: "the variable gene and their variance were input as a phenotype to perform master regulator analysis by function “tna.mra”."
- [discussion] tTF identification via RTN: "we clustered the identified tTFs by hierarchical clustering (the agglomeration method was “ward.D2”)."
- [discussion] tni object construction: "the log10(RPKM + 1) normalized scRNA-seq data and TF list from AnimalTFDB (version 3.0)71 to build the “tni” object"
- [readme] MRA result extraction: "mra <- tna.get(rtna, what="mra", ntop = -1)"
- [readme] Cell-type-specific TRN construction: "rtna <- myGetCellTypeSpecificTRN(tmp.seu = seu, tmp.ident = ii, gene.TF = gene.TF, tmp.phenotype = tmp.phenotype)"
